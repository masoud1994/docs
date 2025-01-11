# Nginx Ingress Controller and metalLb concepts

In section , explain below concepts
* MetalLB Concepts
* Deploy MetalLb
* Testing MetalLB
* Deploy Ingress-Nginx Controller


## [MetalLB Concepts](https://metallb.io/concepts/)

MetalLB hooks into your Kubernetes cluster, and provides a network load-balancer implementation. In short, it allows you to create Kubernetes services of type LoadBalancer in clusters that don’t run on a cloud provider, and thus cannot simply hook into paid products to provide load balancers.

It has two features that work together to provide this service: address allocation, and external announcement.

### External announcement
After MetalLB has assigned an external IP address to a service, it needs to make the network beyond the cluster aware that the IP “lives” in the cluster. MetalLB uses standard networking or routing protocols to achieve this, depending on which mode is used: ARP, NDP, or BGP.

### Layer 2 mode (ARP/NDP)
In layer 2 mode, one machine in the cluster takes ownership of the service, and uses standard address discovery protocols (ARP for IPv4, NDP for IPv6) to make those IPs reachable on the local network. From the LAN’s point of view, the announcing machine simply has multiple IP addresses.

The layer 2 mode sub-page has more details on the behavior and limitations of layer 2 mode.

### BGP
In BGP mode, all machines in the cluster establish BGP peering sessions with nearby routers that you control, and tell those routers how to forward traffic to the service IPs. Using BGP allows for true load balancing across multiple nodes, and fine-grained traffic control thanks to BGP’s policy mechanisms.

The BGP mode sub-page has more details on BGP mode’s operation and limitations.


## Deploy MetalLb
### [Installation by manifest](https://metallb.io/installation/#installation-by-manifest)

``` 
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```
when run command kubectl get pod -n metallb-system
```
kubectl get pod -n metallb-system
```
you can see some pods
```
NAME                         READY   STATUS    RESTARTS   AGE
controller-7dcb87658-n285s   1/1     Running   0          3h30m
speaker-7wvqg                1/1     Running   0          3h30m

```
The metallb-system/controller deployment. This is the cluster-wide controller that handles IP address assignments.
The metallb-system/speaker daemonset. This is the component that speaks the protocol(s) of your choice to make the services reachable.

### Create L2Advertisement , IPAddressPool
* https://metallb.io/configuration/_advanced_l2_configuration/
* https://metallb.io/configuration/_advanced_ipaddresspool_configuration/

Create ip-address.yaml file 
In this file you can set ip address pool for asinge to Loadbalancer services.

```
vim ip-address.yaml
```
put below values in ip-address.yaml
```
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: pool1
  namespace: metallb-system
spec:
  ipAddressPools:
  - pool1

---

apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: pool1
  namespace: metallb-system
spec:
  addresses:
  - 192.168.0.80-192.168.0.100

```

Apply manifest to cluster
```
kubectl apply -f ip-address.yaml
```

## Testing MetalLB
for testing MetalLB , create nginx-deploymnet.yaml and LoadBalancer service
a=At first create troubleshoot

```
kubectl create namespace troubleshoot
```
```
vim nginx-deployment.yml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: troubleshoot  # specify the namespace here
spec:
  replicas: 1  # number of Nginx pods
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
        image: nginx:1.26  # specify the Nginx image
        ports:
        - containerPort: 80
```
```
kubectl apply -f nginx-deployment.yml
```
Then create service for Nginx
```
vim nginx-loadbalancer.yml
```

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: troubleshoot  # specify the namespace here
spec:
  selector:
    app: nginx  # matches the label of the Nginx deployment
  ports:
    - protocol: TCP
      port: 80        # the port the service will expose
      targetPort: 80   # the port Nginx container is listening on
  type: LoadBalancer  # exposes the service externally with a cloud load balancer

```

```
kubectl apply -f nginx-loadbalancer.yml
```

So see the pod and service
```
kubectl get pod,svc -n troubleshoot

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6587db7797-znwhb   1/1     Running   0          3h10m

NAME                    TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)        AGE
service/nginx-service   LoadBalancer   10.43.2.186   192.168.0.80   80:31056/TCP   3h10m

```
EXTERNAL-IP field shows th ip address that metalLb asinges to the Loadbalancer service
You can use below url in browser
```
http://192.168.0.80
```


## Deploy Ingress-Nginx Controller
At first you need install metalLB or some tools else for asignning External Ip Address to LoadBalancer service type. 
Ingress-Nginx Controller uses laodbalancer service type.

There are multiple ways to install the Ingress-Nginx Controller:

* with Helm, using the project repository chart ([quick-start](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)) ;
* with kubectl apply, using YAML manifests;

## Quick start
If you have Helm, you can deploy the ingress controller with the following command:
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx;
helm repo update;
helm fetch ingress-nginx/ingress-nginx;
```
## Set ingressClassResource as default in values.yaml

```
 ingressClassResource:
    # -- Name of the IngressClass
    name: nginx
    # -- Create the IngressClass or not
    enabled: true
    # -- If true, Ingresses without `ingressClassName` get assigned to this IngressClass on creation.
    # Ingress creation gets rejected if there are multiple default IngressClasses.
    # Ref: https://kubernetes.io/docs/concepts/services-networking/ingress/#default-ingress-class
    default: true

```

## Set imagePullSecret:
If use loacl registry you need set imagePullSecret for pulling Ingres Nginx images

## Deploy ingress-nginx
```
helm upgrade --install ingress-nginx ingress-nginx  --namespace ingress-nginx --create-namespace
```

## Check and Test Ingress-Nginx Controller
```
kubectl get pod,svc -n ingress-nginx;

NAME                                           READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-d49697d5f-87dm6   1/1     Running   0          8h

NAME                                         TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.43.142.122   192.168.0.81   80:32320/TCP,443:32684/TCP   8h
service/ingress-nginx-controller-admission   ClusterIP      10.43.232.139   <none>         443/TCP                      8h

```
In this case Ingress-Nginx Controller got 192.168.0.81 for EXTERNAL-IP (from metalLB)
for testing Ingress-Nginx Controller , create ingres manifest for nginx-deployment
```
vim nginx-ingress.yml
```
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: troubleshoot  # specify the namespace here
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /  # optional: rewrite the path if needed
spec:
  rules:
  - host: nginx.example.com  # the domain name to use for the service
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service  # the name of the service
            port:
              number: 80  # the service port
```
Nginx-depolyment uses clusterip service type (nginx-service).
In this case nginx-ingress uses Nginx ingress controller because it's defalut ingress controller.
Now you can use below url in browser.
```
http://gitlab.example.com
```

```

kubectl get ingresses.networking.k8s.io -n troubleshoot
NAME            CLASS   HOSTS                 ADDRESS        PORTS   AGE
nginx-ingress   nginx   nginx.example.com   192.168.0.81   80      5h2m

```