# Install haproxy and configure it
at first add repository of haproxy on linux
In this document , we install haproxy on RHEL

get haproxy packages
```
dnf install haproxy
```
```
systemctl enable haproxy.service --now
```

## configure haproxy
there is haproxy config file in /etc/haproxy directory.

ls -l /etc/haproxy
```
drwxr-xr-x. 2 root root    6 Mar  6  2024 conf.d
-rw-r--r--. 1 root root 3284 Mar  6  2024 haproxy.cfg
```
