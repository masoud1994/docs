# How To Install and Use Docker Compose on Ubuntu 20.04
download docker-compose and make an example with docker-compose
## URLs
[https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04#step-1-installing-docker-compose](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04#step-1-installing-docker-compose)

## Step 1 — Installing Docker Compose
First, confirm the latest version available in their [releases](https://github.com/docker/compose/releases) page. At the time of this writing, the most current stable version is 1.29.2.
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```
## Step 2 — Setting Up a docker-compose.yml File
To demonstrate how to set up a docker-compose.yml file and work with Docker Compose, you’ll create a web server environment using the official Nginx image from Docker Hub, the public Docker registry. This containerized environment will serve a single static HTML file.

Start off by creating a new directory in your home folder, and then moving into it:
```
mkdir ~/compose-demo
cd ~/compose-demo
```
In this directory, set up an application folder to serve as the document root for your Nginx environment:
```
mkdir app
```
Using your preferred text editor, create a new index.html file within the app folder:
```
nano app/index.html
```
Place the following content into this file:
```~/compose-demo/app/index.html

<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Docker Compose Demo</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/kognise/water.css@latest/dist/dark.min.css">
</head>
<body>

    <h1>This is a Docker Compose Demo Page.</h1>
    <p>This content is being served by an Nginx container.</p>

</body>
</html>
```
Save and close the file when you’re done. If you are using nano, you can do that by typing CTRL+X, then Y and ENTER to confirm.

Next, create the docker-compose.yml file:
```
nano docker-compose.yml
```

Insert the following content in your docker-compose.yml file:
```docker-compose.yml
version: '3.7'
services:
  web:
    image: nginx:alpine
    ports:
      - "8000:80"
    volumes:
      - ./app:/usr/share/nginx/html
      
```
## Step 3 — Running Docker Compose
With the docker-compose.yml file in place, you can now execute Docker Compose to bring your environment up. The following command will download the necessary Docker images, create a container for the web service, and run the containerized environment in background mode:

```
docker-compose up -d
```