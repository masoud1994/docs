# Install gitlab on k8s
#################
Installing GitLab on Kubernetes with your own SSL certificate involves several steps. I'll guide you through the process step by step.

Prerequisites:
Kubernetes Cluster: You need access to a Kubernetes cluster (minikube, GKE, AKS, EKS, or any other Kubernetes service).
kubectl: Install and configure kubectl to interact with your Kubernetes cluster.
Helm: Helm is a Kubernetes package manager that simplifies the installation of applications like GitLab. If you don't have it, you can install it using the following instructions: Helm installation guide.
GitLab Certificate: Your SSL certificate (including private key and any intermediate certificates) should be ready for use.

## Step 1: Create a Namespace for GitLab
First, create a dedicated namespace in your Kubernetes cluster for GitLab:

```
kubectl create namespace gitlab
```
# Step 2: Install Helm (if not installed)
If you don’t have Helm installed yet, you can install it as follows:

Linux/macOS: Follow the official Helm installation instructions.
Windows: You can use the choco install kubernetes-helm command or download it directly from Helm releases.
Once Helm is installed, initialize Helm on your Kubernetes cluster:
Once you have all of your configuration options collected, we can get any dependencies and run Helm. In this example, we’ve named our Helm release gitlab.
```
helm repo add gitlab https://charts.gitlab.io/
helm repo update
```
## Step 3 Download the Gitlab Helm chart:
```
helm pull gitlab/gitlab --version <chart-version> --untar
```
Replace <chart-version> with the version of the gitlab chart you want to download, or leave it out to get the latest version.

This will create a folder called cilium containing the chart files.

### Download dependencies:
If there are dependencies for the Cilium chart, you can download them using:

```
helm dependency update ./gitlab
```
After downloading the chart and its dependencies, you can package it into a .tgz file for offline installation.
```
helm package ./gitlab
```


## Step 4: Create Kubernetes Secrets for SSL Certificate
Before installing GitLab, you need to create Kubernetes secrets for your SSL certificate. If you have your SSL certificate and private key, you can use the following commands to create a Kubernetes secret.

Assuming you have the following files:

tls.crt - Your SSL certificate.
tls.key - Your private key.
You can create a Kubernetes secret like this:
```
kubectl create secret tls gitlab-tls --cert=tls.crt --key=tls.key --namespace gitlab
```
Make sure you replace the tls.crt and tls.key file names with the correct paths to your certificate files.

## Step 5: Configure GitLab with Helm Values

Now edit the values.yaml file to include SSL configuration. Open gitlab-values.yaml in a text editor, and find the global section. Update it to reference the secret you created earlier:

```
global:
  hosts:
    domain: <your-domain.com>
    https: true
  ingress:
    configureCertmanager: false
    tls:
      - secretName: gitlab-tls  # The secret we created in Step 3
```

Here’s what each part of this configuration does:

domain: The domain where your GitLab instance will be hosted (e.g., gitlab.example.com).
https: Set to true to enable SSL.
tls.secretName: The name of the Kubernetes secret containing your SSL certificate.
If you're using a different ingress controller or have custom configurations, you'll need to adjust this YAML further.

## Step 6: Install GitLab Using Helm
With the configuration in place, you can now install GitLab using Helm. Run the following command to install GitLab in the gitlab namespace:
```
helm upgrade --install gitlab . -n gitlab 

```
Helm will install GitLab and configure it according to the settings in the gitlab-values.yaml file. This may take a few minutes.

## Step 7: Verify the Installation
Once GitLab is installed, you can check the status of the deployment:

```
kubectl get pods --namespace gitlab
```
You should see several GitLab pods starting up (such as gitlab-webservice, gitlab-sidekiq, and others).

To check the GitLab service, run:

```
kubectl get svc --namespace gitlab
```
You should see an Ingress service or a LoadBalancer depending on your setup.

## Step 8: Access GitLab
Once GitLab is installed, you can access it via the domain you set earlier (gitlab.example.com or your configured domain). If you are using an Ingress controller, make sure your DNS points to the external IP of the Kubernetes cluster.

To check the external IP of your ingress:

```
kubectl get ingress --namespace gitlab
```
The ADDRESS field should show the external IP or hostname where GitLab is accessible.

Step 8: Configure GitLab
Access GitLab via your domain (e.g., https://gitlab.example.com).
The first time you access GitLab, it will ask you to set an admin password.
Once the admin password is set, you can log in with the username root and the password you just created.
## Step 10: Optional – Set Up GitLab Backup and Monitoring
You might want to set up regular backups for your GitLab instance, along with monitoring and logging. GitLab provides built-in functionality for backups and can be integrated with monitoring solutions like Prometheus and Grafana.

To enable backups, you would typically configure GitLab’s backup settings in the gitlab-values.yaml file and ensure persistent volumes are set up correctly.

Step 10: (Optional) Enable Automatic SSL with Cert-Manager
If you don't have an SSL certificate and would like to automatically get one using Let’s Encrypt, you can integrate Cert-Manager with GitLab and use automatic certificate management.

However, since you already have a custom SSL certificate, this step is not necessary for you.

That’s it! You now have GitLab running on Kubernetes with your custom SSL certificate. If you encounter any issues or need further customization, feel free to ask!



