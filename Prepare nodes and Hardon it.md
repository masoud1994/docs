# Prepare nodes and Hardon it

Step 1:  set proper time zone

```
timedatectl set-timezone "Asia/Tehran"
```

set static ip address on node
```
vim /etc/netplan/00-installer-config.yaml
netplan apply
```

edit hostname of the vm
```
hostnamectl set-hostname master1
```

set hostname of the node in /etc/hosts

vim /etc/hosts

set password for roo and enable ssh root login
```
sudo -i
passwd

vim /etc/ssh/sshd_config
PermitRootLogin yes
```
```
systemctl restart sshd
```


## Step 2: disable and remove swap file 
```
cat /proc/swaps
sudo swapoff -a
sudo rm /swap.img
vim /etc/fstab
init 6
swapon
free -h
```


## Step 3: Configure Date and Time 
set gateway ip address for the chronyd server
```
vim /etc/chrony.conf
```
set gateway ip address for the chronyd server

```
systemctl restart chronyd.service
chronyc tracking #wait untill it gets syncing
```

```

chmod +x /home/tehran-nodst-1403/asia-tehran.sh

cd /home/tehran-nodst-1403

./asia-tehran.sh

date
```


# Step 4: edit /etc/hosts with proper
```
vim /etc/hosts

10.60.98.109 dockerrepo-qmb-banco.noc-tosan.com
10.60.98.109 novinrepo
```


# Step 5: configure YAST / Disable IPV6 / Enable Ip Forwarding 
```
yast lan
```
* Disable ipv6

* Enable IPV4 Forwarding

* Set proper IP Address

* Set DNS Server to Managment Gateway

* Set Domain Name


## Step 6: Configure Daemon.json
```
vim /etc/docker/daemon.json

{
  "log-level": "warn",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "5"
  } ,
  "insecure-registries": [
     "novinrepo:8082"
                    ] ,
     "live-restore": true
}
```
```
systemctl restart docker
```


## Step 7: Copy Certificates and test connectivity to Harbor 

```
 scp 

 docker login novinrepo:8082 -u docker -p Admin123
 docker login novinrepo:8082 -u docker -p Admin123

```


## conncet to arvan cloud
```
sudo bash -c 'cat > /etc/docker/daemon.json <<EOF
{
  "insecure-registries" : ["https://docker.arvancloud.ir"],
  "registry-mirrors": ["https://docker.arvancloud.ir"]
}
EOF'
```













