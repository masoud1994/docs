# install haproxy
* https://www.haproxy.com/documentation/haproxy-configuration-tutorials/core-concepts/backends/

## Generate ssl
* https://gist.github.com/yuezhu/47b15b4b8e944221861ccf7d7f5868f5
```
frontend main
    bind *:80
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js
    redirect location https://devops.masoud.ir
  #  use_backend static          if url_static
  #  default_backend             app

frontend https
        bind *:443 ssl crt /etc/ssl/private/devops.masoud.ir.pem
        default_backend app
#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance     roundrobin
    server  app1 127.0.0.1:8081 check
    server  app2 127.0.0.1:8082 check

```