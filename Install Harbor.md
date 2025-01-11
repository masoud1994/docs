# Install Harbor

URLs for download harbor:
* [https://goharbor.io/docs/2.0.0/install-config](https://goharbor.io/docs/2.0.0/install-config)
* [https://goharbor.io/docs/2.0.0/install-config/harbor-compatibility-list/](https://goharbor.io/docs/2.0.0/install-config/harbor-compatibility-list/)

Installation Process

The standard Harbor installation process involves the following stages:

* Make sure that your target host meets the [Harbor Installation Prerequisites](https://goharbor.io/docs/2.0.0/install-config/installation-prereqs/).
* [Download the Harbor Installer](https://goharbor.io/docs/2.0.0/install-config/download-installer/)
* [Configure HTTPS Access to Harbor](https://goharbor.io/docs/2.0.0/install-config/configure-https/)
* [Configure the Harbor YML File](https://goharbor.io/docs/2.0.0/install-config/configure-yml-file/)
* [Configure Enabling Internal TLS](https://goharbor.io/docs/2.0.0/install-config/configure-internal-tls/)
* [Run the Installer Script](https://goharbor.io/docs/2.0.0/install-config/run-installer-script/)

## Download the Harbor Installer
You download the Harbor installers from the [official releases](https://github.com/goharbor/harbor/releases) page. Download either the online installer or the offline installer.

Online installer: The online installer downloads the Harbor images from Docker hub. For this reason, the installer is very small in size.

Offline installer: Use the offline installer if the host to which are are deploying Harbor does not have a connection to the Internet. The offline installer contains pre-built images, so it is larger than the online installer.

The installation processes are almost the same for both the online and offline installers.

## Configure HTTPS Access to Harbor
1.Generate a Server Certificate

```
cat > domains.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage=digitalSignature,nonRepudiation,keyEncipherment,dataEncipherment
subjectAltName=@alt_names
[alt_names]
DNS.1 = localhost
DNS.2 = dockerrepo.example.com
DNS.3 = www.dockerrepo.example.com
IP.1 = 127.0.0.1
IP.1 = 192.168.0.12  #harbor ip
EOF
```
```
openssl req -x509 -nodes -new -sha256 -days 3650 -newkey rsa:2048 -keyout rootca.key -out rootca.pem -subj "/C=IR/CN=htst=root-ca"

openssl x509 -outform pem -in rootca.pem -out rootca.crt

openssl req -new -nodes -newkey rsa:2048 -keyout dockerrepo.example.com.key -out dockerrepo.example.com.csr -subj "/C=IR/ST=Asia/L=Tehran/O=Modern/CN=dockerrepo-htst.noc-tosan.com"

openssl x509 -req -sha256 -days 1825 -in dockerrepo.example.com.csr -CA rootca.crt -CAkey rootca.key -CAcreateserial -extfile domains.ext -out dockerrepo.example.com.crt

```

## Step 2: Configure the Harbor YML File
Create harbor.yml file
```
cp harbor.yml.tmpl harbor.yml
```
vim harbor.yml and set hostname , http and https port and config ssl for 
```
hostname: dockerrepo.example.com

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 8082

# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /var/platform/harbor/ssl/dockerrepo.example.com.crt
  private_key: /var/platform/harbor/ssl/dockerrepo.example.com.key

```

you can set a volume for harbor in harbor.yml
```
# The default data volume
data_volume: ./data

```
Log setting in harbor.yml
```
log:
  # options are debug, info, warning, error, fatal
  level: info
  # configs for logs in local storage
  local:
    # Log files are rotated log_rotate_count times before being removed. If count is 0, old versions are removed rather than rotated.
    rotate_count: 50
    # Log files are rotated only if they grow bigger than log_rotate_size bytes. If size is followed by k, the size is assumed to be in kilobytes.
    # If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G
    # are all valid.
    rotate_size: 200M
    # The directory on your host that store log
    location: ./data/log/harbor

```
thare are 2 scripts for config and install harbor
at first asigne executable to them 
```
 chmod a+x install.sh prepare
```
Set domain name or hostname of harbor in /etc/hosts
```
192.168.0.12 dockerrepo.example.com
```
execute install.sh
```
./install.sh
```

after that please cp rootca to trust certificates in linux
```
cp rootca /usr/local/share/ca-certificates/  #ubuntu
cp rootca.crt /etc/pki/trust/anchors/ca-certificate.crt #SLES

```
## test harbor
you must create project and user in harvor UI
you canuse harbor with http and https protocl
```
docker login dockerrepo.example.com 
```
