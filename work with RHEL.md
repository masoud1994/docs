# work with RHEL

if you dont license , you can use below repositories
* https://mirror.cs.princeton.edu/pub/mirrors/rocky/9.5/AppStream/x86_64/os/
* https://mirror.cs.princeton.edu/pub/mirrors/rocky/9.5/BaseOS/x86_64/os/

It's important there is repodata directory in repository url

config  /etc/yum.repos.d/redhat.repo and add below command it 
```
[rhelsecond]
#name=CentOS $releasever - $basearch
baseurl=https://mirror.cs.princeton.edu/pub/mirrors/rocky/9.5/BaseOS/x86_64/os
enabled=1
gpgcheck=0

#####
[rhel]
#name=CentOS $releasever - $basearch
baseurl=https://mirror.cs.princeton.edu/pub/mirrors/rocky/9.5/AppStream/x86_64/os/
enabled=1
gpgcheck=0

```
after that you use dnf update
```
dnf update
```
for example: install nginx 
```
dnf se nginx     # search nginx in repositories
dnf info nginx   # show verion of nginx
```

## Register and automatically subscribe in one step
Use the following command to register the system, then automatically associate any available subscription matching that system:
* https://access.redhat.com/solutions/253273
```
subscription-manager register --username <username> --password <password> --auto-attach
```

## Install Docker Engine on RHEL
* https://docs.docker.com/engine/install/rhel/
Uninstall Old Docker Versions (If Any)
If you have any older versions of Docker installed, first remove them:
```
sudo yum remove docker docker-common docker-snapshot docker-engine
```
Set Up Docker Repository
Docker provides an official repository for RHEL. You need to configure this repository to install the appropriate Docker version.
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

```
Set Up the Docker Repository:
```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
```
 sudo dnf install docker-ce-V20.10.24 docker-ce-cli-20.10.24 containerd.io docker-buildx-plugin docker-compose-plugin

```

Start Docker Engine.
```
sudo systemctl enable --now docker
```
```
 sudo docker run hello-world
 ```