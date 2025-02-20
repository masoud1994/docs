# buid base image

## build an image with curl docker helm jq commands

vim Dockerfile
```

FROM debian:buster
RUN apt-get update && \
apt-get install -y \
curl \
ca-certificates \
docker.io \
openssh-server \
dos2unix \
jq \
sshpass \
wget \
sshfs \
git && \
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash && \
rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
apt-get clean
CMD ["bash"]
```

build image:
```
docker build . -t debian-buster:curl-docker-helm-jq
```