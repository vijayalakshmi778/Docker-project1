# Docker Website Deployment on AWS EC2

## Project Overview

This project shows how to create a simple HTML website, build it as a Docker image, run it as a Docker container, attach a Docker volume, and deploy it on an AWS EC2 Ubuntu server.

## Final Flow

AWS EC2 Server
    |
    v
Install Docker
    |
    v
Create Website
    |
    v
Create Dockerfile
    |
    v
Build Docker Image
    |
    v
Create Docker Volume
    |
    v
Run Docker Container
    |
    v
Port Mapping
    |
    v
Open EC2 Public IP
    |
    v
Website

---

# STEP 1 - Create AWS EC2 Server

Go to:

AWS Console -> EC2 -> Instances -> Launch Instance

Select:

- Name: docker-server
- AMI: Ubuntu Server 24.04 LTS
- Instance Type: t3.micro
- Create or select a Key Pair

## Security Group

Add these inbound rules:

| Type | Port | Source |
|---|---:|---|
| SSH | 22 | My IP |
| Custom TCP | 5000 | 0.0.0.0/0 |

Launch the EC2 instance.

---

# STEP 2 - Connect to EC2

From your local terminal:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Example:

```bash
ssh -i myserver.pem ubuntu@13.234.56.78
```

Check the current user:

```bash
whoami
```

Expected:

```text
ubuntu
```

---

# STEP 3 - Update Ubuntu

```bash
sudo apt update
```

```bash
sudo apt upgrade -y
```

---

# STEP 4 - Install Docker

```bash
sudo apt install docker.io -y
```

Check Docker:

```bash
docker --version
```

Start Docker:

```bash
sudo systemctl start docker
```

Enable Docker:

```bash
sudo systemctl enable docker
```

Check Docker:

```bash
sudo systemctl status docker
```

Expected:

```text
active (running)
```

Press `q` to exit.

---

# STEP 5 - Allow Ubuntu User to Run Docker

Run:

```bash
sudo usermod -aG docker $USER
```

Exit the EC2 server:

```bash
exit
```

Connect again:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Test:

```bash
docker ps
```

If there is no permission error, Docker is ready.

---

# STEP 6 - Test Docker

```bash
docker run hello-world
```

Expected output contains:

```text
Hello from Docker!
```

---

# STEP 7 - Create Project Folder

```bash
mkdir docker-website
```

```bash
cd docker-website
```

Check:

```bash
pwd
```

Expected:

```text
/home/ubuntu/docker-website
```

---

# STEP 8 - Create Website

Create the HTML file:

```bash
nano index.html
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Website</title>
</head>
<body>

    <h1>Welcome to My Docker Website</h1>

    <h2>Running on AWS EC2</h2>

    <p>This website is running inside a Docker container.</p>

    <p>Docker Volume is used for persistent website data.</p>

</body>
</html>
```

Save:

```text
CTRL + O
ENTER
CTRL + X
```

Check:

```bash
cat index.html
```

---

# STEP 9 - Create Dockerfile

Create:

```bash
nano Dockerfile
```

Add:

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Save:

```text
CTRL + O
ENTER
CTRL + X
```

Check:

```bash
cat Dockerfile
```

---

# STEP 10 - Build Docker Image

Run:

```bash
docker build -t my-website:v1 .
```

The `.` means the current directory.

Check:

```bash
docker images
```

Expected:

```text
REPOSITORY    TAG    IMAGE ID
my-website    v1     xxxxxxxxx
```

---

# STEP 11 - Create Docker Volume

Create a named volume:

```bash
docker volume create website-data
```

Check:

```bash
docker volume ls
```

Expected:

```text
local    website-data
```

---

# STEP 12 - Run Docker Container with Volume

Run:

```bash
docker run -d \
--name my-website-container \
-p 5000:80 \
-v website-data:/usr/share/nginx/html \
my-website:v1
```

## Command Meaning

`-d`

Runs the container in the background.

`--name my-website-container`

Gives the container a name.

`-p 5000:80`

Maps:

```text
EC2 Port 5000
      |
      v
Container Port 80
```

`-v website-data:/usr/share/nginx/html`

Maps:

```text
Docker Volume
      |
      v
website-data
      |
      v
/usr/share/nginx/html
```

`my-website:v1`

This is the Docker image used to create the container.

---

# STEP 13 - Check Container

```bash
docker ps
```

Check port mapping:

```bash
docker port my-website-container
```

Expected:

```text
80/tcp -> 0.0.0.0:5000
```

---

# STEP 14 - Test Website Inside EC2

```bash
curl http://localhost:5000
```

You should see the HTML content.

If the HTML appears, the website is running successfully inside EC2.

---

# STEP 15 - Open Website in Browser

Find the EC2 Public IPv4 address.

Example:

```text
13.234.56.78
```

Open:

```text
http://13.234.56.78:5000
```

The website should open in your browser.

---

# STEP 16 - Check Docker Volume

List volumes:

```bash
docker volume ls
```

Inspect the volume:

```bash
docker volume inspect website-data
```

---

# STEP 17 - Verify Volume Persistence

Stop the container:

```bash
docker stop my-website-container
```

Remove the container:

```bash
docker rm my-website-container
```

Check containers:

```bash
docker ps -a
```

Check the volume:

```bash
docker volume ls
```

The volume should still exist:

```text
website-data
```

---

# STEP 18 - Create a New Container Using the Same Volume

Run:

```bash
docker run -d \
--name my-new-website-container \
-p 5000:80 \
-v website-data:/usr/share/nginx/html \
my-website:v1
```

Check:

```bash
docker ps
```

Test:

```bash
curl http://localhost:5000
```

Open:

```text
http://YOUR_EC2_PUBLIC_IP:5000
```

The website should be available again.

---

# Complete Command Flow

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

Reconnect to EC2.

```bash
docker run hello-world
```

```bash
mkdir docker-website
cd docker-website
```

Create:

```text
index.html
Dockerfile
```

Build image:

```bash
docker build -t my-website:v1 .
```

Create volume:

```bash
docker volume create website-data
```

Run container:

```bash
docker run -d \
--name my-website-container \
-p 5000:80 \
-v website-data:/usr/share/nginx/html \
my-website:v1
```

Check:

```bash
docker ps
```

Test:

```bash
curl http://localhost:5000
```

Open:

```text
http://YOUR_EC2_PUBLIC_IP:5000
```

---

# Useful Docker Commands

## Images

```bash
docker images
```

```bash
docker image inspect my-website:v1
```

## Containers

```bash
docker ps
```

```bash
docker ps -a
```

```bash
docker stop my-website-container
```

```bash
docker start my-website-container
```

```bash
docker restart my-website-container
```

```bash
docker rm my-website-container
```

## Logs

```bash
docker logs my-website-container
```

## Volumes

```bash
docker volume ls
```

```bash
docker volume inspect website-data
```

## Container Shell

```bash
docker exec -it my-website-container bash
```

Exit:

```bash
exit
```

---

# Troubleshooting

## Website Not Opening

Check:

```bash
docker ps
```

Check logs:

```bash
docker logs my-website-container
```

Check port:

```bash
docker port my-website-container
```

Test inside EC2:

```bash
curl http://localhost:5000
```

If `curl` works but the browser does not, check the AWS Security Group and make sure TCP port `5000` is allowed.

## Port Already in Use

Check:

```bash
docker ps
```

Stop the container using port 5000:

```bash
docker stop CONTAINER_NAME
```

Or use another host port:

```bash
docker run -d \
--name my-website-container \
-p 8080:80 \
-v website-data:/usr/share/nginx/html \
my-website:v1
```

Add TCP port `8080` to the EC2 Security Group and open:

```text
http://YOUR_EC2_PUBLIC_IP:8080
```

## Container Not Running

```bash
docker ps -a
```

```bash
docker logs my-website-container
```

## Docker Service Check

```bash
sudo systemctl status docker
```

If Docker is stopped:

```bash
sudo systemctl start docker
```

---

# Final Architecture

```text
                    INTERNET
                       |
                       v
                +-------------+
                |   Browser   |
                +------+------+
                       |
                       | :5000
                       v
                +-------------+
                | AWS EC2     |
                | Ubuntu      |
                +------+------+
                       |
                       v
                +-------------+
                |   Docker    |
                +------+------+
                       |
                       v
              +------------------+
              | Docker Container |
              |      Nginx       |
              |      :80         |
              +--------+---------+
                       |
                       v
              +------------------+
              | Docker Volume    |
              |  website-data    |
              +--------+---------+
                       |
                       v
              /usr/share/nginx/html
                       |
                       v
                  index.html
```

# Final Result

```text
AWS EC2
   |
   v
Docker
   |
   v
Docker Image
   |
   v
Docker Container
   |
   v
Docker Volume
   |
   v
Nginx Website
   |
   v
EC2 Public IP
   |
   v
Browser
```
