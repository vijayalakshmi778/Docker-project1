# Docker Static Website + Volumes + Bind Mount + Network --- Complete Lab

## 1. Lab Objective

In this lab, you will build a beginner-friendly Docker environment
covering:

-   Static website with Nginx
-   Dockerfile and image creation
-   Container creation
-   Port mapping
-   Multiple static website projects
-   Named volumes
-   Bind mounts
-   Docker networks
-   Container-to-container communication
-   Cleanup and troubleshooting

Final browser endpoints:

``` text
http://localhost:8080
http://localhost:8081
```

------------------------------------------------------------------------

## 2. Prerequisites

Install/start:

-   Docker Desktop
-   VS Code
-   Git

Verify Docker:

``` powershell
docker --version
docker info
```

> For Windows + VS Code, Docker commands can be run from the PowerShell
> terminal. The containers themselves run Linux-based Nginx images.

------------------------------------------------------------------------

# Part 1 --- Static Website with Nginx

## 3. Create the Project

Create and open:

``` text
docker-static-website-lab
```

Initial structure:

``` text
docker-static-website-lab/
├── site1/
│   ├── index.html
│   └── Dockerfile
├── site2/
│   ├── index.html
│   └── Dockerfile
├── bind-site/
│   └── index.html
└── README.md
```

------------------------------------------------------------------------

## 4. Create `site1/index.html`

``` html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Static Website</title>
</head>
<body>

    <h1>Welcome to Docker Static Website</h1>
    <h2>Project 1</h2>
    <p>This website is running inside an Nginx Docker container.</p>
    <p>Host Port: 8080</p>

</body>
</html>
```

------------------------------------------------------------------------

## 5. Create `site1/Dockerfile`

``` dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

### Explanation

``` dockerfile
FROM nginx:alpine
```

Uses Nginx as the base image.

``` dockerfile
COPY index.html /usr/share/nginx/html/index.html
```

Copies the website into the directory served by Nginx.

------------------------------------------------------------------------

## 6. Build the Image

From the VS Code terminal:

``` powershell
cd site1
```

Build:

``` powershell
docker build -t static-site-1 .
```

Check:

``` powershell
docker images
```

You should see `static-site-1`.

------------------------------------------------------------------------

## 7. Run the Container

``` powershell
docker run -d --name site1 -p 8080:80 static-site-1
```

Understand:

``` text
-p 8080:80

Host Port       Container Port
    8080   →          80
```

Nginx listens on port `80` inside the container.

Open:

``` text
http://localhost:8080
```

------------------------------------------------------------------------

## 8. Check the Container

``` powershell
docker ps
```

View logs:

``` powershell
docker logs site1
```

Inspect:

``` powershell
docker inspect site1
```

Stop:

``` powershell
docker stop site1
```

Start:

``` powershell
docker start site1
```

Remove:

``` powershell
docker stop site1
docker rm site1
```

------------------------------------------------------------------------

# Part 2 --- Multiple Static Website Projects

## 9. Create Project 2

Create `site2/index.html`:

``` html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Project 2</title>
</head>
<body>

    <h1>Docker Multi Project Demo</h1>
    <h2>Project 2</h2>
    <p>This is a second static website running in another Nginx container.</p>
    <p>Host Port: 8081</p>

</body>
</html>
```

Create `site2/Dockerfile`:

``` dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

------------------------------------------------------------------------

## 10. Build Project 2

``` powershell
cd ..\site2
docker build -t static-site-2 .
```

------------------------------------------------------------------------

## 11. Run Project 2

``` powershell
docker run -d --name site2 -p 8081:80 static-site-2
```

Open:

``` text
http://localhost:8081
```

Now:

``` text
Project 1
localhost:8080 → site1 → Nginx:80

Project 2
localhost:8081 → site2 → Nginx:80
```

### Important

Both containers can use the same internal port `80`.

Only the host ports need to be different:

``` text
8080 → 80
8081 → 80
```

------------------------------------------------------------------------

# Part 3 --- Named Volumes

## 12. What Is a Named Volume?

A named volume is storage managed by Docker.

``` text
Docker Volume
      ↓
Container
      ↓
Application Data
```

It is useful when data should survive container removal.

------------------------------------------------------------------------

## 13. Create a Volume

``` powershell
docker volume create website-data
```

List volumes:

``` powershell
docker volume ls
```

Inspect:

``` powershell
docker volume inspect website-data
```

------------------------------------------------------------------------

## 14. Mount the Volume into Nginx

``` powershell
docker run -d --name volume-site -p 8082:80 -v website-data:/usr/share/nginx/html nginx:alpine
```

Understand:

``` text
-v SOURCE:DESTINATION

website-data
     ↓
/usr/share/nginx/html
```

Open:

``` text
http://localhost:8082
```

------------------------------------------------------------------------

## 15. Demonstrate Volume Persistence

Stop and remove the container:

``` powershell
docker stop volume-site
docker rm volume-site
```

Check:

``` powershell
docker volume ls
```

The volume still exists.

Now reuse it:

``` powershell
docker run -d --name volume-site-2 -p 8083:80 -v website-data:/usr/share/nginx/html nginx:alpine
```

Open:

``` text
http://localhost:8083
```

### Key point

``` text
Container removed
       ↓
Named volume remains
       ↓
Volume can be reused
```

------------------------------------------------------------------------

# Part 4 --- Bind Mount

## 16. What Is a Bind Mount?

A bind mount connects a folder on the host directly to a folder inside
the container.

``` text
Windows Folder
      ↕
Bind Mount
      ↕
Container Folder
      ↓
Nginx
```

This is especially useful during development.

------------------------------------------------------------------------

## 17. Create Bind-Mount Website

Create:

``` text
bind-site/
└── index.html
```

`bind-site/index.html`:

``` html
<!DOCTYPE html>
<html>
<head>
    <title>Bind Mount Demo</title>
</head>
<body>

    <h1>Bind Mount Demo</h1>
    <p>This website is connected directly to a Windows folder.</p>

</body>
</html>
```

------------------------------------------------------------------------

## 18. Run Nginx with a Bind Mount

From the project root:

``` powershell
docker run -d --name bind-site -p 8084:80 -v "${PWD}/bind-site:/usr/share/nginx/html" nginx:alpine
```

Open:

``` text
http://localhost:8084
```

------------------------------------------------------------------------

## 19. Test the Bind Mount

Change the HTML in VS Code:

``` html
<h1>Website Updated from VS Code</h1>
```

Save it and refresh:

``` text
http://localhost:8084
```

The updated content should appear without rebuilding the image.

Why?

``` text
VS Code
   ↓
Windows Folder
   ↕
Bind Mount
   ↕
Container
   ↓
Nginx
```

------------------------------------------------------------------------

# Part 5 --- Named Volume vs Bind Mount

  Feature               Named Volume             Bind Mount
  --------------------- ------------------------ -------------------------------------
  Managed by            Docker                   User/host OS
  Source                Docker-managed storage   Specific host folder
  Development editing   Less direct              Very convenient
  Persistent data       Yes                      Yes
  Example               `website-data:/data`     `./bind-site:/usr/share/nginx/html`

Simple explanation:

> Named volume = Docker manages the storage.

> Bind mount = You choose and manage the host folder.

------------------------------------------------------------------------

# Part 6 --- Docker Network

## 20. What Is a Docker Network?

A Docker network allows containers to communicate with each other.

Create:

``` powershell
docker network create web-network
```

List:

``` powershell
docker network ls
```

Inspect:

``` powershell
docker network inspect web-network
```

------------------------------------------------------------------------

## 21. Run Two Containers on the Network

``` powershell
docker run -d --name network-site-1 --network web-network nginx:alpine
```

``` powershell
docker run -d --name network-site-2 --network web-network nginx:alpine
```

Architecture:

``` text
              web-network
             /                       ↓             ↓
   network-site-1   network-site-2
        Nginx             Nginx
```

------------------------------------------------------------------------

## 22. Test Container-to-Container Communication

Run a temporary container on the same network:

``` powershell
docker run --rm --network web-network busybox wget -qO- http://network-site-1
```

You should receive the Nginx HTML response.

The important concept is that containers on the same user-defined
network can communicate using the container name.

``` text
http://network-site-1
```

You do not need to manually find the container IP for this basic
demonstration.

------------------------------------------------------------------------

# Part 7 --- Multi-Project Environment

## 23. Create a Realistic Multi-Project Structure

``` text
docker-multi-project/
├── website/
│   └── index.html
├── admin/
│   └── index.html
└── README.md
```

Create a network:

``` powershell
docker network create company-network
```

------------------------------------------------------------------------

## 24. Run Website Project

From the project root:

``` powershell
docker run -d --name company-website --network company-network -p 8085:80 -v "${PWD}/website:/usr/share/nginx/html" nginx:alpine
```

Open:

``` text
http://localhost:8085
```

------------------------------------------------------------------------

## 25. Run Admin Project

``` powershell
docker run -d --name company-admin --network company-network -p 8086:80 -v "${PWD}/admin:/usr/share/nginx/html" nginx:alpine
```

Open:

``` text
http://localhost:8086
```

Architecture:

``` text
                 company-network
                /                               ↓                  ↓
       company-website     company-admin
             Nginx               Nginx
               |                   |
          Host :8085          Host :8086
               |                   |
               ↓                   ↓
      localhost:8085       localhost:8086
```

This demonstrates:

-   Multiple projects
-   Multiple containers
-   Different host ports
-   Same container port
-   Bind mounts
-   Shared Docker network

------------------------------------------------------------------------

# Part 8 --- Useful Docker Commands

## Containers

``` powershell
docker ps
docker ps -a
docker stop CONTAINER_NAME
docker start CONTAINER_NAME
docker restart CONTAINER_NAME
docker rm CONTAINER_NAME
docker logs CONTAINER_NAME
docker inspect CONTAINER_NAME
```

## Images

``` powershell
docker images
docker build -t IMAGE_NAME .
docker rmi IMAGE_NAME
```

## Volumes

``` powershell
docker volume ls
docker volume create VOLUME_NAME
docker volume inspect VOLUME_NAME
docker volume rm VOLUME_NAME
```

## Networks

``` powershell
docker network ls
docker network create NETWORK_NAME
docker network inspect NETWORK_NAME
docker network rm NETWORK_NAME
```

------------------------------------------------------------------------

# Part 9 --- Cleanup

Stop the containers created in this lab:

``` powershell
docker stop site1 site2 volume-site-2 bind-site network-site-1 network-site-2 company-website company-admin
```

Remove them:

``` powershell
docker rm site1 site2 volume-site-2 bind-site network-site-1 network-site-2 company-website company-admin
```

Remove custom networks:

``` powershell
docker network rm web-network company-network
```

Remove the named volume when you no longer need it:

``` powershell
docker volume rm website-data
```

Verify:

``` powershell
docker ps -a
docker images
docker volume ls
docker network ls
```

> Do not remove an image, volume, or network that is still being used by
> another project.

------------------------------------------------------------------------

# Part 10 --- Student Assignments

## Assignment 1 --- Static Website

Create:

``` text
student-site/
├── index.html
└── Dockerfile
```

Requirements:

-   Use Nginx
-   Build a Docker image
-   Run a container
-   Expose it on host port `8090`

Expected:

``` text
http://localhost:8090
```

------------------------------------------------------------------------

## Assignment 2 --- Two Websites

Create:

``` text
website-a
website-b
```

Run:

``` text
website-a → localhost:8091
website-b → localhost:8092
```

Both must use Nginx.

------------------------------------------------------------------------

## Assignment 3 --- Named Volume

Create:

``` text
student-data
```

Then:

1.  Create the volume.
2.  Run a container with the volume.
3.  Stop the container.
4.  Remove the container.
5.  Verify the volume still exists.
6.  Create another container using the same volume.

------------------------------------------------------------------------

## Assignment 4 --- Bind Mount

Create:

``` text
my-website/
└── index.html
```

Mount it into Nginx.

Change the HTML from VS Code and refresh the browser.

The student must demonstrate that the content changes without rebuilding
the image.

------------------------------------------------------------------------

## Assignment 5 --- Docker Network

Create:

``` text
student-network
```

Run two containers on the same network.

Test communication using a container name.

------------------------------------------------------------------------

# Part 11 --- Interview Questions

### 1. What is a Docker image?

A read-only template used to create containers.

### 2. What is a container?

A running instance of an image.

### 3. What does `-p 8080:80` mean?

It maps host port `8080` to container port `80`.

### 4. Can two containers use port 80 internally?

Yes. They can both listen on container port 80 while using different
host ports.

### 5. What is a Docker volume?

Docker-managed persistent storage.

### 6. What is a bind mount?

A host directory mounted directly into a container.

### 7. Why use Docker networks?

To allow containers to communicate with each other.

### 8. Can containers communicate using names?

Yes. Containers on the same user-defined Docker network can use
container/service names through Docker's internal DNS.

### 9. What happens to a named volume when its container is removed?

The volume normally remains until explicitly removed.

### 10. Why are bind mounts useful during development?

Changes made to the host files can immediately be available inside the
container.

------------------------------------------------------------------------

# Part 12 --- Final Learning Flow

``` text
Docker Installation
        ↓
Docker Image
        ↓
Dockerfile
        ↓
Docker Build
        ↓
Docker Container
        ↓
Port Mapping
        ↓
Nginx Static Website
        ↓
Multiple Containers
        ↓
Named Volumes
        ↓
Bind Mounts
        ↓
Docker Networks
        ↓
Container-to-Container Communication
        ↓
Multi-Project Environment
        ↓
Docker Compose
        ↓
Docker Hub
        ↓
GitHub Actions + Docker
        ↓
AWS Deployment
```

------------------------------------------------------------------------

# Key Commands to Remember

Build an image:

``` powershell
docker build -t myimage .
```

Run a container:

``` powershell
docker run -d --name mycontainer -p 8080:80 myimage
```

Create a volume:

``` powershell
docker volume create myvolume
```

Run with named volume:

``` powershell
docker run -d --name mycontainer -v myvolume:/data nginx:alpine
```

Create a network:

``` powershell
docker network create mynetwork
```

Run on a network:

``` powershell
docker run -d --name mycontainer --network mynetwork nginx:alpine
```

Run with bind mount:

``` powershell
docker run -d --name mycontainer -p 8080:80 -v "${PWD}/website:/usr/share/nginx/html" nginx:alpine
```

------------------------------------------------------------------------

# Final Student Goal

By the end of this lab, the student should be able to explain:

> I can create a Docker image using a Dockerfile, run it as a container,
> map container ports to host ports, run multiple Nginx static websites,
> use named volumes for persistent storage, use bind mounts during
> development, create Docker networks, and allow multiple containers to
> communicate with each other.

This lab provides the foundation for the next practical stage: **Docker
Compose and a multi-container application**.
