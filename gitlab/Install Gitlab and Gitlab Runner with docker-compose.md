# Install Gitlab and Gitlab Runner with docker-compose

we check following steps
* download gitlab and runner images
* create docker-compose.yml file and config it
* config gitlab.rb
* docker-compose up
* git command for git commit
* Register runner

URLS:
* [https://docs.gitlab.com/runner/install/](https://docs.gitlab.com/runner/install/)


## step 1: install docker-compose



## Step 2: Load Gitlab & gitlab runner images:
```
docker pull docker.arvancloud.ir/gitlab/gitlab-runner:latest
docker pull docker.arvancloud.ir/gitlab/gitlab-runner:latest

```

## step 3: Edit docker-compose file & up gitlab services
```
mkdir gitlab;cd gitlab

vim docker-compose.yaml
```
```docker-compose.yaml
version: '3.5'

services:
    gitlab:
      image: novinrepo:8082/docker/gitlab/gitlab-ce:15.11.0-ce.0
      restart: always
      ports:
        - '80:80'
        - '10022:22'
      volumes:
        - './data/config:/etc/gitlab'
        - './data/logs:/var/log/gitlab'
        - './data/data:/var/opt/gitlab'
      privileged: true
      networks:
        internal_network:
          ipv4_address: 172.20.1.10
      logging:
        driver: "json-file"
        options:
          max-size: "50m"

    gitlab-runner:
      image: novinrepo:8082/docker/gitlab/gitlab-runner:v15.10.0
      container_name: 'gitlab-runner'
      volumes:
        - './gitlab-runner/config/:/etc/gitlab-runner'
        - '/var/run/docker.sock:/var/run/docker.sock'
      restart: always
      networks:
        internal_network:
            ipv4_address: 172.20.1.11
      logging:
        driver: "json-file"
        options:
          max-size: "50m"

networks:
   internal_network:
      ipam:
         driver: default
         config:
            - subnet: "172.20.1.0/24"

```
```
docker-compose up -d 
```

## step 4: edit gitlab.rb file with proper options


```
docker-copmose down
vim ./gitlab/config/gitlab.rb
```
```
external_url 'http://192.168.31.54'
prometheus_monitoring['enable'] = false
gitlab_rails['initial_root_password'] = "zaighoojee2eey3meiGh"
gitlab_rails['display_initial_root_password'] = true
gitlab_rails['store_initial_root_password'] = true


gitlab_rails['ldap_enabled'] = true
gitlab_rails['prevent_ldap_sign_in'] = false
gitlab_rails['ldap_servers'] = {
'main' => {
  'label' => 'GitLab AD',
  'host' =>  'dcn03.tosanltd.com',
  'port' => 389,
  'uid' => 'sAMAccountName',
  'encryption' => 'plain',
  'verify_certificates' => true,
  'bind_dn' => 'CN=git_acc,OU=ServiceAccounts,OU=Computers_Users,DC=tosanltd,DC=com',
  'password' => '************',
  'timeout' => 10,
  'active_directory' => true,
  'lowercase_usernames' => false,
  'allow_username_or_email_login' => false,
  'base' => 'DC=tosanltd,DC=com',
  'user_filter' => '',
  'group_base' => '',
  'admin_group' => '',
 }
}



gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "mail.tosanltd.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_domain'] = "tosanltd.com"
gitlab_rails['smtp_authentication'] = false
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_ssl'] = false
gitlab_rails['smtp_openssl_verify_mode'] = 'none'
gitlab_rails['gitlab_email_from'] = 'gitlab@tosan.com'
```




```
docker-compose up -d
```


wait for gitlab containers to be healthy and ready 

----- Important Note: it will take some time to get unhealthy then after sometime
will be healthy!




## step 5: configure gitlab ui

connect to gitlab ui with proper username and password




## step 6: connect gitlab ui to gitlab

```
ssh-keygen -t rsa
cat /root/.ssh/id_rsa.pub
```
now go to:

Gitlab ui ----> preferences ----> ssh keys ----> paste your public key in key section

like this :

ssh-rsa 
AAAAB3NzaC1yc2EAAAADAQABAAABgQDRDEOzxd2Yv6/DiSwKTZVg+89ZVIxlZZgr+Vgx/Padjh5NykSKMzPDZ9Rixjof+wAhVyoHNlKGTPf8iJHjwMqZjJtVAsiq5ttkQEwsvQIrmVCSxJxrNFGEtOJ5I3LVcKUQKeRMFVgwdguVvj45DCCOInnYwUi7HyLVEETV4bO8b7uZJth9KR8MWxWmfUVbov3UaP6E5CpO0+l74bsl/sY+zAvoXPdyOjqvIZxt+g3UZZ6C0w7Q+UYa4geWfJp5CLz5IUyY1TzKOy8F01h3Z93r8JIDdUfE4b8ag5fX/kJspPgyB/IgLX1/u9ZjJj9SYQ6xckoz1vaVOXH0PpLLF5ZqikrrtxRsEkbaXuElI+wLXEkMxbNpvHPSMRs/ThNrzeZl/sBdttFOINsCXy1RBFNug0talo8VX2onatNKfR+ADdxImz5ZXjyUXMRcyqXIx80nvnfaenK8eEEhINSJdJ7pnfKnixu99HXmCuSRT9Pa0SgYIagzZ+nsZ1yMyoCnmfM= root@gitlab-54



## step 7: import template projects
```
mkdir /var/gitlab/pipeline
cp templates-main.zip /var/gitlab/pipeline
unzip /var/gitlab/pipeline/templates-main.zip
cd /var/gitlab/pipeline/templates-main
git init
git config --global user.name "devops"
git config --global user.email "devops@tosan.com"
git remote add origin ssh://git@192.168.108.105:10022/cd/templates.git
git add .
git commit -m "Initial commit"
git checkout -b main
git push -u origin main
```

git remote add origin ssh://git@192.168.31.54:10022/cd/tools/tools.git


Note: you should push 3 projects:

first create CD group then push the following templates.
you should create other_templates & tools Groups too.


cd/templates.git
cd/other_templates/tosan-kubernetes-cluster.git
cd/tools/cd-assets.git



## step 8: register runner for project

go to cd project ----> Runners ----> register a group runner ----> copy registration token


now on the gitlab server:

8.1. test connectivity to private registry

Add following in /etc/docker/daemon.json

  "insecure-registries": ["novinrepo:8082"] ,
  "live-restore": true

systemctl restart docker


then test login into registry

docker login novinrepo:8082 -u docker -p Admin123
docker pull novinrepo:8082/docker/alpine:latest



8.2. export proper variables for connecting gitlab runner to gitlab server:

```
export GITLAB_TOKEN="GR1348941iN1VBP5zFQ9asGa2fU76"
export GITLAB_SERVER_IP="192.168.31.54"
```

8.3. connect gitlab runner to gitlab server

``` 
docker exec -it gitlab-runner gitlab-runner register \
    --non-interactive \
    --url http://$GITLAB_SERVER_IP/ \
    --registration-token $GITLAB_TOKEN \
    --executor docker \
    --docker-image novinrepo:8082/docker/alpine:latest \
    --docker-volumes "/var/run/docker.sock:/var/run/docker.sock:ro"
```
now you should see your runner in cd project runners


8.4. configure runner
```
cd /var/gitlab/gitlab-runner/config
vim config.toml
```
ADD this line:

```
concurrent = 10
ADD allowed_pull_policies = ["always","if-not-present"]
```




step 8: configure gitlab connection to active directory


8.1. add active directory ip and url to gitlab /etc/hosts

# vim /etc/hosts
192.168.31.60   dc.tosanltd.com


8.2. create ou structure you defined in gitlab.rb to your active directory:

'CN=git_acc,OU=ServiceAccounts,OU=Computers_Users,DC=tosanltd,DC=com'

create OU=Computers_Users
create OU=ServiceAccounts
create CN=git_acc

git_acc is a user in gitlab server that have password


8.3. 


gitlab-rake gitlab:ldap:check

echo "192.168.31.60 dc.tosanltd.com" | tee -a /etc/hosts



