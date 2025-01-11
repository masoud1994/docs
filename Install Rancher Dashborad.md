# Install Rancher Dashborad
At first you need deploy metalLB and Nginx Ingress Controller.
In this [url](https://www.suse.com/suse-rancher/support-matrix/) check Supported Kubernetes Platforms for Rancher Manager.
## Install Rancher with Helm [#](https://ranchermanager.docs.rancher.com/v2.10/getting-started/quick-start-guides/deploy-rancher-manager/helm-cli#install-rancher-with-helm)
Then from your local workstation, run the following commands. You will need to have kubectl and helm. installed.

Before Install Rancher , install cert-manager 
You can use cert-manager for manage Rancher Certificate 
To see options on how to customize the cert-manager install (including for cases where your cluster uses PodSecurityPolicies), see the [cert-manager docs](https://artifacthub.io/packages/helm/cert-manager/cert-manager#configuration).


```
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

kubectl create namespace cattle-system
```
Deploy cert-manager on k8s.

```
mkdir cert-manager; cd cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/<VERSION>/cert-manager.crds.yaml

helm repo add jetstack https://charts.jetstack.io

helm repo update

helm fetch jetstack/cert-manager --version <version>

helm install cert-manager ./cert-manager --namespace cert-manager --create-namespace

```

Note: use this [url](https://github.com/cert-manager/cert-manager/releases) for see version of cert-manager. This document uses 1.16.2 version for cert-manager.

Deploy Rancher
```
helm fetch rancher-latest/rancher
tar -xzvf rancher-2.10.1.tgz;
```

Set domain rancher.platform.lab for rancher dashbaord .
Edit rancher/values.yaml
```
hostname: rancher.platform.lab
```
```
helm install rancher ./rancher --namespace cattle-system
```

Check Rancher service
```
kubectl get ingress -n cattle-system
NAME      CLASS   HOSTS                  ADDRESS        PORTS     AGE
rancher   nginx   rancher.platform.lab   192.168.0.80   80, 443   4h41m

```
Note: 192.168.0.80 is Nginx Ingress Controller IP Address.
you can use below url for watching Rancher Dashboard in browser
```
https://rancher.platform.lab
```
