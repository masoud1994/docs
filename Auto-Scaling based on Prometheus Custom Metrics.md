
# Auto-Scaling based on Prometheus Custom Metrics
We want to create HPA for nginx deployment and scale down and scale up nginx base on size of the queue in rabbitmq.

* https://github.com/kubernetes-sigs/prometheus-adapter/
* https://medium.com/engineering-housing/auto-scaling-based-on-prometheus-custom-metrics-688a92e0a796

* Install Prometheus for monitoring
* Install RabbitMQ on Kubernetes
* Create Queue in Rabbitmq
* Check Rabbitmq metrics in prometheus
* Enable ServiceMonitor for RabbitMQ
* Install Prometheus Adapter
* Create NGINX Deployment
* Create Horizontal Pod Autoscaler (HPA) for NGINX based on RabbitMQ queue size



## Step 1: Install Prometheus in Kubernetes
Install Prometheus with Helm

Prometheus can be installed with Helm to scrape metrics and monitor RabbitMQ.

Add the Prometheus community Helm repository:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
Install Prometheus:

```
helm fetch  prometheus-community/kube-prometheus-stack --untar

```

Now edit values.yaml and config NodePort for prometheus and servicemonitore for scrape all namespaces.
```
## Deploy a Prometheus instance
##
prometheus:
  enabled: true

    ## Port for Prometheus Service to listen on
    ##
    port: 9090

    ## To be used with a proxy extraContainer port
    targetPort: 9090

    ## Port for Prometheus Reloader to listen on
    ##
    reloaderWebPort: 8080

    ## List of IP addresses at which the Prometheus server service is available
    ## Ref: https://kubernetes.io/docs/user-guide/services/#external-ips
    ##
    externalIPs: []

    ## Port to expose on each node
    ## Only used if service.type is 'NodePort'
    ##
    nodePort: 30090
    
 ######
     serviceMonitorSelector:
    ## Example which selects ServiceMonitors with label "prometheus" set to "somelabel"
    # serviceMonitorSelector:
      matchLabels:
        app.kubernetes.io/name: rabbitmq

```
Install it with Helm
```
helm upgrade --install prometheus . -n prometheus --create-namespaces
```
Verify Prometheus Installation

```
kubectl get pods -n prometheus
```



## Step 2: Install RabbitMQ in Kubernetes
Create a RabbitMQ Deployment and Service

First, let's deploy RabbitMQ using Helm, a package manager for Kubernetes.

Add the RabbitMQ Helm chart repository:
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
Set username and password in values.yaml
```
auth:
  ## @param auth.username RabbitMQ application username
  ## ref: https://github.com/bitnami/containers/tree/main/bitnami/rabbitmq#environment-variables
  ##
  username: quest
  ## @param auth.password RabbitMQ application password
  ## ref: https://github.com/bitnami/containers/tree/main/bitnami/rabbitmq#environment-variables
  ##
  password: "quest"
```

### Enable Metrics for RabbitMQ to expose metrics for prometheus 
prometheus get metrics of rabbitmq with service monitor , so we need enable prometheus exporter in rabbitmq.

Point: you now prometheus works with  servicemonitor
Edit values.yaml
```
metrics:
  ## @param metrics.enabled Enable exposing RabbitMQ metrics to be gathered by Prometheus
  ##
  enabled: true
  ## @param metrics.plugins Plugins to enable Prometheus metrics in RabbitMQ
  ##
  plugins: "rabbitmq_prometheus"
```
And enable default , perObject , detailed servicemonitors
For more information please see below this [URL](https://github.com/bitnami/charts/tree/main/bitnami/rabbitmq#integration-with-prometheus-operator)
```
  serviceMonitor:

    default:
      ## @param metrics.serviceMonitor.default.enabled Enable default metrics endpoint (`GET /metrics`) to be scraped by the ServiceMonitor
      ##
      enabled: true
      
    perObject:
      ## @param metrics.serviceMonitor.perObject.enabled Enable per-object metrics endpoint (`GET /metrics/per-object`) to be scraped by the ServiceMonitor
      ##
      enabled: true

    detailed:
      ## @param metrics.serviceMonitor.detailed.enabled Enable detailed metrics endpoint (`GET /metrics/detailed`) to be scraped by the ServiceMonitor
      ##
      enabled: true
      
```
Install RabbitMQ:

```
helm upgrade --install  rabbitmq -n rabbitmq --create-namespaces
```
This will install RabbitMQ with the username and password set to guest. You can adjust authentication credentials as needed.

Verify RabbitMQ Deployment

```
kubectl get pods -n rabbitmq
```

## Step3: Create a Queue in RabbitMQ (nginx)

You can create the queue manually or via an application. To create the queue via an application, ensure you connect to RabbitMQ using the provided guest credentials and create a queue called nginx.

You can port-fprward 15672 with velow command
```
kubectl port-forward --namespace rabbitmq svc/rabbitmq 15672:15672 &
```
Now Create a queue with name nginx 
So with below command add record to the queue.

```
curl -s -u guest:guest -H "Accept: application/json" -H "Content-Type:application/json" -X POST --data '{"properties":{"delivery_mode":2},"routing_key":"nginx","payload":"{\"sample\":\"load\"}","payload_encoding":"string"}' http://127.0.0.1:15672/api/exchanges/%2f/amq.default/publish
```

## Step 4: Check metrics in prometheus
Now you can open the url of prometheus on browser and see targets
```
https://node-ip:30090/targets
```
In query menu you can see metrics of nginx queue with the command
```
rabbitmq_queue_messages{namespace="rabbitmq",queue="nginx"}
```
So you can see the number of nginx queue records in this case.
## Step 5: Install Prometheus Adapter for HPA
The Prometheus Adapter enables Kubernetes Horizontal Pod Autoscaler (HPA) to scale based on custom metrics like queue length in RabbitMQ.

Install Prometheus Adapter using Helm:

```
helm fetch p prometheus-community/prometheus-adapter --untar 
```

Now Create a custom metric in values.yaml for getting metric from prometheus.
Edit rules in values.yaml
```
rules:
  default: true

  external:
    - seriesQuery: 'rabbitmq_queue_messages{namespace="rabbitmq",queue="nginx"}'
      resources:
        #template: <<.Resource>>
        overrides:
          namespace: {resource: "namespace"}
      name:
         matches: "rabbitmq_queue_messages"
         as: "rabbitmq_queue_messages"
      metricsQuery: 'rabbitmq_queue_messages{namespace="rabbitmq",queue="nginx"}'


```
Set prometheus url in values.yaml
```
prometheus:
  # Value is templated
  url: http://192.168.101.152
  port: 30090
  path: ""
```
Install prometheus adaptor in prometheus namespace.
```
helm upgrade --install prometheus-adaptor . -n prometheus
```
Verify Adapter Deployment:

```
kubectl get pods prometheus-adapter -n prometheus
```

Now test fetching metric from prometheus with prometheus adaptor

```
kubectl get --raw /apis/external.metrics.k8s.io/v1beta1/namespaces/prometheus/rabbitmq_queue_messages
```
## Step 5: Create NGINX Deployment
Create an NGINX Deployment and Service:

Create an NGINX deployment YAML file, for example, nginx-deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: hpa  # Change this to your desired namespace
spec:
  replicas: 1  # Start with one replica
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest  # The Nginx image to use (you can replace with any custom image)
          ports:
            - containerPort: 80
```
Apply the deployment:

```
kubectl apply -f nginx-deployment.yaml
```
Expose NGINX via a Service:


Step 6: Create HPA (Horizontal Pod Autoscaler)
Now, create an HPA for the NGINX deployment that scales based on the queue_messages metric from RabbitMQ.

Create an HPA resource that scales based on RabbitMQ metrics:

Here's an example of the nginx-hpa.yaml:

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
  namespace: hpa  # Replace with your Nginx namespace
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx  # Replace with your Nginx deployment name
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: External
      external:
        metric:
          name: rabbitmq_queue_messages
          selector:
            matchLabels:
              queue: "nginx"  # Ensure this matches your custom metric
        target:
          type: Value
          value: "1"  # Set the target value for scaling

```
Apply the HPA configuration:

```
kubectl apply -f nginx-hpa.yaml
```

```
kubectl get hpa nginx-hpa
```









