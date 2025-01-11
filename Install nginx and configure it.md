# nginx concept
* https://nginx.org/en/docs/?_ga=2.166036984.758380344.1734785468-1250368289.1727119417
* https://nginx.org/en/docs/beginners_guide.html


# Install nginx and configure it
at first add repository of haproxy on linux
In this document , we install nginx on RHEL

get nginx packages
```
dnf install nginx
```
```
systemctl enable nginx.service --now
```

## configure haproxy
there is nginx config file in /etc/nginx/nginx.conf .
If you have a firewall running, you need to allow traffic on the new port.

For example, if you changed the port to 8080, run the following commands:

Allow HTTP traffic on port 8080:
```
sudo firewall-cmd --permanent --add-port=8080/tcp
```
Reload the firewall to apply the changes:
```
sudo firewall-cmd --reload
```
Check that the new port is open:
```
sudo firewall-cmd --list-all
```
when you want to change default port on nginx ,please check selinux
```
sestatus 
```


## config nginx
make /data/images and cp some images it
```
mkdir /data/images
```
```
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
    server {
        listen 8080;
        root /data/up1;

        location / {
        }
    }
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        #root         /usr/share/nginx/html;
        location / {
        proxy_pass http://127.0.0.1:8080;
        }
        location ~ \.(gif|jpg|png)$ {
        root /data;
        }
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }


```