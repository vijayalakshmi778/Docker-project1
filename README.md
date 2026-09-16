# DOCKER PRACTICAL LAB — REAL-WORLD PROJECT
## Project: Containerized Python Task Manager with Persistent Data, Network, Compose, Healthcheck and Docker Hub

Audience: Beginner DevOps students
Prerequisites: Docker installed and working
Style: Practical lab notes — execute, verify, troubleshoot

---

# 1. WHAT YOU WILL BUILD

You will build a small Python Flask Task Manager and run it using Docker.

Final architecture:

    Browser
       |
       v
    Flask Container
       |
       +------> Docker Named Volume
       |              |
       |              v
       |         SQLite database
       |
       +------> Docker Network
       |
       v
    Docker Compose

Concepts covered:

    Dockerfile
    Image
    Container
    Port Mapping
    Logs
    docker exec
    Environment Variables
    Volumes
    Bind Mounts
    Networks
    Container-to-container communication
    Docker Compose
    Healthcheck
    Restart Policy
    Image Tagging
    Docker Hub
    Troubleshooting
    Cleanup

IMPORTANT:
Do every command yourself. After each major step, verify the result.

---

# 2. PROJECT FOLDER

Create:

    docker-task-manager/

Final structure:

    docker-task-manager/
    |
    +-- app.py
    +-- requirements.txt
    +-- Dockerfile
    +-- .dockerignore
    +-- docker-compose.yml
    +-- README.md
    +-- data/

---

# 3. CHECK DOCKER

Run:

    docker --version

Then:

    docker info

Test:

    docker run hello-world

Check containers:

    docker ps

    docker ps -a

If hello-world works, continue.

---

# 4. CREATE PROJECT

Linux / macOS:

    mkdir docker-task-manager
    cd docker-task-manager
    mkdir data
    touch app.py requirements.txt Dockerfile .dockerignore docker-compose.yml README.md

Windows PowerShell:

    mkdir docker-task-manager
    cd docker-task-manager
    mkdir data
    New-Item app.py
    New-Item requirements.txt
    New-Item Dockerfile
    New-Item .dockerignore
    New-Item docker-compose.yml
    New-Item README.md

---

# 5. CREATE THE PYTHON APPLICATION

Open app.py and add:

    from flask import Flask, request, jsonify
    import sqlite3
    import os

    app = Flask(__name__)

    DB_DIR = os.getenv("DB_DIR", "/app/data")
    DB_PATH = os.path.join(DB_DIR, "tasks.db")

    def get_db():
        os.makedirs(DB_DIR, exist_ok=True)
        conn = sqlite3.connect(DB_PATH)
        conn.row_factory = sqlite3.Row
        return conn

    def init_db():
        conn = get_db()
        conn.execute("""
            CREATE TABLE IF NOT EXISTS tasks (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT NOT NULL,
                status TEXT NOT NULL DEFAULT 'pending'
            )
        """)
        conn.commit()
        conn.close()

    @app.route("/")
    def home():
        return """
        <h1>Docker Task Manager</h1>
        <p>Application is running inside Docker.</p>
        <p>Use /tasks to view tasks.</p>
        """

    @app.route("/health")
    def health():
        return jsonify({"status": "healthy"})

    @app.route("/tasks", methods=["GET"])
    def get_tasks():
        conn = get_db()
        rows = conn.execute("SELECT * FROM tasks").fetchall()
        conn.close()
        return jsonify([dict(row) for row in rows])

    @app.route("/tasks", methods=["POST"])
    def add_task():
        data = request.get_json()

        if not data or "title" not in data:
            return jsonify({"error": "title is required"}), 400

        conn = get_db()
        cursor = conn.execute(
            "INSERT INTO tasks (title) VALUES (?)",
            (data["title"],)
        )
        conn.commit()
        task_id = cursor.lastrowid
        conn.close()

        return jsonify({
            "id": task_id,
            "title": data["title"],
            "status": "pending"
        }), 201

    @app.route("/tasks/<int:task_id>", methods=["DELETE"])
    def delete_task(task_id):
        conn = get_db()
        cursor = conn.execute(
            "DELETE FROM tasks WHERE id = ?",
            (task_id,)
        )
        conn.commit()
        conn.close()

        if cursor.rowcount == 0:
            return jsonify({"error": "task not found"}), 404

        return jsonify({"message": "task deleted"})

    if __name__ == "__main__":
        init_db()
        app.run(host="0.0.0.0", port=5000)

IMPORTANT:
Do not use:

    app.run(host="127.0.0.1", port=5000)

Inside Docker, Flask must listen on:

    0.0.0.0

---

# 6. CREATE requirements.txt

Add:

    Flask==3.1.2

Verify:

Linux:

    cat requirements.txt

PowerShell:

    Get-Content requirements.txt

---

# 7. TEST THE APP WITHOUT DOCKER

Optional but recommended.

Install:

    pip install -r requirements.txt

Run:

    python app.py

Open:

    http://localhost:5000

Test:

    http://localhost:5000/health

    http://localhost:5000/tasks

Stop:

    CTRL + C

If port 5000 is already occupied, stop the process using it or use another host port later.

---

# 8. CREATE Dockerfile

Add:

    FROM python:3.12-slim

    WORKDIR /app

    COPY requirements.txt .

    RUN pip install --no-cache-dir -r requirements.txt

    COPY app.py .

    RUN mkdir -p /app/data

    EXPOSE 5000

    ENV DB_DIR=/app/data

    CMD ["python", "app.py"]

---

# 9. CREATE .dockerignore

Add:

    __pycache__
    *.pyc
    .git
    .gitignore
    venv
    .venv
    data/*.db
    README.md

Purpose:

Docker should not copy unnecessary files into the image.

---

# 10. BUILD THE IMAGE

Run:

    docker build -t docker-task-manager:v1 .

Check:

    docker images

Inspect:

    docker image inspect docker-task-manager:v1

---

# 11. RUN THE FIRST CONTAINER

Run:

    docker run -d --name task-manager-v1 -p 5000:5000 docker-task-manager:v1

Check:

    docker ps

Open:

    http://localhost:5000

Health:

    http://localhost:5000/health

Tasks:

    http://localhost:5000/tasks

---

# 12. CONTAINER LOGS

Run:

    docker logs task-manager-v1

Follow live logs:

    docker logs -f task-manager-v1

Stop following:

    CTRL + C

Last 50 lines:

    docker logs --tail 50 task-manager-v1

With timestamps:

    docker logs -t task-manager-v1

---

# 13. ENTER THE CONTAINER

Run:

    docker exec -it task-manager-v1 sh

Inside:

    pwd

Expected:

    /app

Check:

    ls

Check data:

    ls data

Check environment variable:

    echo $DB_DIR

Expected:

    /app/data

Exit:

    exit

TEACHING POINT:

Files inside a normal container filesystem are tied to that container's lifecycle.

This is why persistent storage requires a volume.

---

# 14. VOLUME — FIRST PRACTICAL

Stop and remove the old container:

    docker stop task-manager-v1
    docker rm task-manager-v1

Create a named volume:

    docker volume create task-manager-data

Check:

    docker volume ls

Inspect:

    docker volume inspect task-manager-data

---

# 15. RUN CONTAINER WITH NAMED VOLUME

Run:

    docker run -d --name task-manager-volume -p 5000:5000 -v task-manager-data:/app/data docker-task-manager:v1

Check:

    docker ps

---

# 16. CREATE DATABASE DATA

Create a task.

Linux / macOS:

    curl -X POST http://localhost:5000/tasks \
      -H "Content-Type: application/json" \
      -d '{"title":"Learn Docker Volumes"}'

Windows PowerShell:

    Invoke-RestMethod -Method Post `
      -Uri http://localhost:5000/tasks `
      -ContentType "application/json" `
      -Body '{"title":"Learn Docker Volumes"}'

Check:

    curl http://localhost:5000/tasks

Or open:

    http://localhost:5000/tasks

Expected:

    [
      {
        "id": 1,
        "title": "Learn Docker Volumes",
        "status": "pending"
      }
    ]

---

# 17. PROVE VOLUME PERSISTENCE

Stop:

    docker stop task-manager-volume

Remove:

    docker rm task-manager-volume

Check volume:

    docker volume ls

The volume must still exist.

Create a NEW container using the SAME volume:

    docker run -d --name task-manager-volume-new -p 5000:5000 -v task-manager-data:/app/data docker-task-manager:v1

Check:

    curl http://localhost:5000/tasks

The previous task should still exist.

PROOF:

    Container deleted
           |
           v
    Volume remains
           |
           v
    Database remains
           |
           v
    New container connects
           |
           v
    Old data is available

---

# 18. BIND MOUNT PRACTICAL

Stop/remove current container:

    docker stop task-manager-volume-new
    docker rm task-manager-volume-new

Linux / macOS:

    docker run -d --name task-manager-bind -p 5000:5000 -v "$(pwd)/data:/app/data" docker-task-manager:v1

Windows PowerShell:

    docker run -d --name task-manager-bind -p 5000:5000 -v "${PWD}/data:/app/data" docker-task-manager:v1

Create a task again.

Then check local data directory.

Linux:

    ls data

PowerShell:

    Get-ChildItem data

You should see:

    tasks.db

TEACHING POINT:

Named volume:

    -v task-manager-data:/app/data

Bind mount:

    -v ./data:/app/data

Named volume:
Docker manages the storage.

Bind mount:
Host controls the directory.

---

# 19. DOCKER NETWORK PRACTICAL

Stop/remove bind container:

    docker stop task-manager-bind
    docker rm task-manager-bind

Create network:

    docker network create task-network

Check:

    docker network ls

Inspect:

    docker network inspect task-network

Run app on network:

    docker run -d --name task-manager-network --network task-network -p 5000:5000 -v task-manager-data:/app/data docker-task-manager:v1

---

# 20. TEST CONTAINER-TO-CONTAINER COMMUNICATION

Run a temporary curl container on the same network:

    docker run --rm --network task-network curlimages/curl:latest http://task-manager-network:5000/health

Expected:

    {"status":"healthy"}

IMPORTANT:

From another container:

    http://task-manager-network:5000

Do NOT use:

    http://localhost:5000

Why?

Inside a container, localhost means that same container.

Docker's network DNS lets containers reach each other using container/service names.

---

# 21. DOCKER COMPOSE

Stop/remove manually created app container:

    docker stop task-manager-network
    docker rm task-manager-network

Create docker-compose.yml:

    services:

      task-manager:
        build: .
        container_name: task-manager-compose
        ports:
          - "5000:5000"
        environment:
          DB_DIR: /app/data
        volumes:
          - task-manager-data:/app/data
        restart: unless-stopped
        healthcheck:
          test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://127.0.0.1:5000/health')"]
          interval: 30s
          timeout: 5s
          retries: 3
          start_period: 10s

    volumes:
      task-manager-data:

---

# 22. VALIDATE COMPOSE

Run:

    docker compose config

If no error is displayed, continue.

If your installation uses the older command:

    docker-compose config

---

# 23. BUILD AND START COMPOSE

Build:

    docker compose build

Start:

    docker compose up -d

Check:

    docker compose ps

Open:

    http://localhost:5000

Health:

    http://localhost:5000/health

---

# 24. COMPOSE LOGS

Run:

    docker compose logs

Follow:

    docker compose logs -f

Only application:

    docker compose logs task-manager

Last 50:

    docker compose logs --tail 50 task-manager

---

# 25. HEALTHCHECK

Check:

    docker ps

Eventually the status should contain:

    (healthy)

Inspect:

    docker inspect task-manager-compose

Find:

    State
    Health

Manual health test:

    docker exec task-manager-compose python -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:5000/health').read())"

---

# 26. RESTART POLICY

The Compose file contains:

    restart: unless-stopped

Test:

    docker kill task-manager-compose

Wait a few seconds.

Then:

    docker ps

Docker Compose should recreate/restart the container according to the configured restart policy.

Check logs:

    docker compose logs --tail 30

---

# 27. DATA PERSISTENCE WITH COMPOSE

Create task:

Linux / macOS:

    curl -X POST http://localhost:5000/tasks \
      -H "Content-Type: application/json" \
      -d '{"title":"Learn Docker Compose"}'

Check:

    curl http://localhost:5000/tasks

Stop stack:

    docker compose down

IMPORTANT:

Do not use -v for this test.

Start again:

    docker compose up -d

Check:

    curl http://localhost:5000/tasks

The task should still exist.

Reason:

    docker compose down
            |
            v
       containers removed
            |
            v
       named volume remains
            |
            v
       database remains

---

# 28. IMPORTANT: docker compose down -v

This command:

    docker compose down -v

also removes Compose-managed volumes.

Use it only when you intentionally want to delete the stored database data.

For normal shutdown:

    docker compose down

For shutdown + volume deletion:

    docker compose down -v

---

# 29. CONTAINER LIFECYCLE PRACTICE

Run and understand:

    docker start task-manager-compose

    docker stop task-manager-compose

    docker restart task-manager-compose

    docker ps

    docker ps -a

For Compose:

    docker compose up -d
    docker compose stop
    docker compose start
    docker compose restart
    docker compose down

Students must understand:

    stop  = stop process/container, keep container
    start = start stopped container
    restart = stop + start
    rm = remove container
    down = remove Compose-created containers/network
    down -v = also remove volumes

---

# 30. EXEC PRACTICE

Run:

    docker exec -it task-manager-compose sh

Inside:

    pwd
    ls
    ls data
    env
    python --version

Check database:

    ls -lh /app/data

Exit:

    exit

---

# 31. ENVIRONMENT VARIABLES

The application uses:

    DB_DIR

Check:

    docker exec task-manager-compose sh -c 'echo $DB_DIR'

Expected:

    /app/data

Test overriding an environment variable:

    docker run --rm -e DB_DIR=/tmp/testdata docker-task-manager:v1 sh -c 'echo $DB_DIR'

Expected:

    /tmp/testdata

TEACHING POINT:

Environment variables allow configuration without rebuilding the image.

---

# 32. IMAGE TAGGING

Check:

    docker images

Create another tag:

    docker tag docker-task-manager:v1 docker-task-manager:latest

Check:

    docker images docker-task-manager

You should see:

    v1
    latest

---

# 33. DOCKER HUB

Login:

    docker login

Assume Docker Hub username:

    YOUR_USERNAME

Tag:

    docker tag docker-task-manager:v1 YOUR_USERNAME/docker-task-manager:v1

Check:

    docker images

Push:

    docker push YOUR_USERNAME/docker-task-manager:v1

---

# 34. PULL AND RUN FROM DOCKER HUB

Pull:

    docker pull YOUR_USERNAME/docker-task-manager:v1

Run:

    docker run -d --name dockerhub-task-manager -p 5000:5000 -v task-manager-data:/app/data YOUR_USERNAME/docker-task-manager:v1

Test:

    curl http://localhost:5000/health

---

# 35. IMAGE INSPECTION

Run:

    docker image inspect docker-task-manager:v1

Look for:

    Cmd
    Env
    WorkingDir
    ExposedPorts

Image history:

    docker history docker-task-manager:v1

Teaching point:

Docker images are built from layers.

Dockerfile instructions such as RUN and COPY contribute to the image build process and layers.

---

# 36. CONTAINER INSPECTION

Run:

    docker inspect task-manager-compose

Students should locate:

    Config
    Env
    Cmd
    Mounts
    NetworkSettings
    RestartPolicy

---

# 37. VOLUME INSPECTION

Run:

    docker volume ls

Then:

    docker volume inspect task-manager-data

Look for:

    Mountpoint

Do not manually modify Docker-managed storage unless you understand the environment.

---

# 38. TROUBLESHOOTING — CONTAINER EXITS

Run:

    docker ps -a

Then:

    docker logs <container-name>

Example:

    docker logs task-manager-compose

Inspect:

    docker inspect task-manager-compose

Look at:

    State
    ExitCode
    Error

---

# 39. TROUBLESHOOTING — PORT ALREADY ALLOCATED

Typical error:

    port is already allocated

Check:

    docker ps

Stop the container using the port:

    docker stop <container-name>

OR change Compose:

    ports:
      - "5001:5000"

Then open:

    http://localhost:5001

Meaning:

    5001:5000
       |
       +-- host port
          container port

---

# 40. TROUBLESHOOTING — WEBSITE NOT OPENING

Check:

    docker ps

Check:

    docker logs task-manager-compose

Check mapping:

    docker port task-manager-compose

Test from inside:

    docker exec task-manager-compose python -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:5000/health').read())"

If internal test works but host access fails, check:

    port mapping
    Docker Desktop
    firewall
    cloud security group
    host port

---

# 41. TROUBLESHOOTING — FLASK LOCALHOST ISSUE

Wrong:

    app.run(host="127.0.0.1", port=5000)

Correct:

    app.run(host="0.0.0.0", port=5000)

Rebuild:

    docker compose build --no-cache

Start:

    docker compose up -d

---

# 42. TROUBLESHOOTING — DATA DISAPPEARS

Check:

    docker inspect task-manager-compose

Find:

    Mounts

Expected destination:

    /app/data

Check volume:

    docker volume ls

Avoid:

    docker compose down -v

when the database must be preserved.

---

# 43. TROUBLESHOOTING — DOCKERFILE CHANGES NOT APPEARING

Run:

    docker compose build

Then:

    docker compose up -d --force-recreate

If necessary:

    docker compose build --no-cache

Then:

    docker compose up -d

---

# 44. TROUBLESHOOTING — CONTAINER NAME ALREADY EXISTS

Check:

    docker ps -a

Remove old container:

    docker rm -f <container-name>

Then run again.

---

# 45. TROUBLESHOOTING — IMAGE NOT FOUND

Check:

    docker images

Pull:

    docker pull YOUR_USERNAME/docker-task-manager:v1

If private:

    docker login

---

# 46. TROUBLESHOOTING — COMPOSE COMMAND

Check:

    docker compose version

If unavailable, check:

    docker-compose version

Modern Docker uses:

    docker compose

---

# 47. TROUBLESHOOTING — WINDOWS BIND MOUNT

PowerShell:

    -v "${PWD}/data:/app/data"

If Docker Desktop reports a path/mount problem:

    Check Docker Desktop is running.
    Check the data folder exists.
    Check the path is correct.
    Check Docker Desktop file-sharing/security permissions if prompted.

---

# 48. TROUBLESHOOTING — BUILD FAILURE

Check:

    docker build -t docker-task-manager:v1 .

If dependency installation fails:

    cat requirements.txt

Then:

    docker build --no-cache -t docker-task-manager:v1 .

On PowerShell:

    Get-Content requirements.txt

Do not change multiple things randomly.

Read the first meaningful error in the build output.

---

# 49. TROUBLESHOOTING — HEALTHCHECK UNHEALTHY

Run:

    docker ps

Then:

    docker inspect task-manager-compose

Read:

    State.Health

Test manually:

    docker exec task-manager-compose python -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:5000/health').read())"

Then:

    docker compose logs --tail 50 task-manager

---

# 50. TROUBLESHOOTING METHOD

Always use:

    OBSERVE
       |
       v
    IDENTIFY
       |
       v
      FIX
       |
       v
    VERIFY

Recommended order:

    1. docker ps
    2. docker ps -a
    3. docker logs <container>
    4. docker inspect <container>
    5. docker images
    6. docker volume ls
    7. docker network ls
    8. Test application inside container
    9. Check port mapping
    10. Rebuild only when required

---

# 51. FINAL COMMAND CHEAT SHEET

Images:

    docker images
    docker pull IMAGE
    docker build -t IMAGE:TAG .
    docker image inspect IMAGE
    docker history IMAGE
    docker rmi IMAGE
    docker tag SOURCE TARGET
    docker push IMAGE

Containers:

    docker ps
    docker ps -a
    docker run
    docker start
    docker stop
    docker restart
    docker rm
    docker rm -f
    docker logs
    docker exec
    docker inspect

Volumes:

    docker volume ls
    docker volume create NAME
    docker volume inspect NAME
    docker volume rm NAME
    docker volume prune

Networks:

    docker network ls
    docker network create NAME
    docker network inspect NAME
    docker network rm NAME

Compose:

    docker compose config
    docker compose build
    docker compose up -d
    docker compose ps
    docker compose logs
    docker compose restart
    docker compose stop
    docker compose start
    docker compose down

---

# 52. FINAL PROJECT CHECKLIST

Run:

    docker compose ps

Application:

    curl http://localhost:5000/health

Tasks:

    curl http://localhost:5000/tasks

Logs:

    docker compose logs --tail 50

Volume:

    docker volume ls

Network:

    docker network ls

Image:

    docker images

Container:

    docker inspect task-manager-compose

Students must demonstrate:

    [ ] Build an image
    [ ] Run a container
    [ ] Map a port
    [ ] Read logs
    [ ] Enter a container
    [ ] Use environment variables
    [ ] Create a named volume
    [ ] Prove persistent data
    [ ] Use a bind mount
    [ ] Create a network
    [ ] Communicate between containers
    [ ] Use Docker Compose
    [ ] Configure healthcheck
    [ ] Configure restart policy
    [ ] Inspect image/container/volume
    [ ] Tag an image
    [ ] Push to Docker Hub
    [ ] Pull image from Docker Hub
    [ ] Troubleshoot common Docker problems

---

# 53. STUDENT ASSIGNMENT — DO NOT GIVE THE SOLUTION

TASK 1:
Add:

    GET /tasks/<id>

TASK 2:
Add:

    PUT /tasks/<id>

Request example:

    {
      "status": "completed"
    }

TASK 3:
Add environment variable:

    APP_PORT

Use it to configure the application port.

TASK 4:
Create a second Compose service:

    admin-client

TASK 5:
Put both services on the same Compose network.

TASK 6:
Make admin-client call:

    http://task-manager:5000/health

TASK 7:
Add a healthcheck.

TASK 8:
Add a restart policy.

TASK 9:
Push the final image to Docker Hub.

TASK 10:
Delete the original application container.

TASK 11:
Create a new container from the Docker Hub image.

TASK 12:
Reconnect the same named volume.

TASK 13:
Prove the database data remains.

TASK 14:
Create a GitHub repository and push:

    app.py
    requirements.txt
    Dockerfile
    docker-compose.yml
    .dockerignore
    README.md

Do NOT push:

    tasks.db
    .env
    passwords
    API keys
    secrets

---

# 54. INTERVIEW QUESTIONS FROM THIS LAB

1. Difference between Docker image and container?

2. What happens to container filesystem after container deletion?

3. Why do we need Docker volumes?

4. Named volume vs bind mount?

5. What does -p 5000:5000 mean?

6. Why use 0.0.0.0 inside a container?

7. What is Docker Compose?

8. Why use environment variables?

9. What is a Docker network?

10. How do containers communicate?

11. What does docker logs do?

12. What does docker exec do?

13. What does Dockerfile do?

14. Why use .dockerignore?

15. What is a healthcheck?

16. What does restart: unless-stopped do?

17. Difference between docker stop and docker rm?

18. Difference between docker compose down and docker compose down -v?

19. What does docker build --no-cache do?

20. How do you troubleshoot a container that exits immediately?

---

# 55. DOCKER LEARNING ROADMAP AFTER THIS LAB

Continue with practical labs in this order:

    LAB 1
    Dockerfile + Image + Container
          |
          v
    LAB 2
    Volumes + Bind Mounts
          |
          v
    LAB 3
    Docker Networks
          |
          v
    LAB 4
    Environment Variables + .env
          |
          v
    LAB 5
    Docker Compose
          |
          v
    LAB 6
    Multi-container Application
          |
          v
    LAB 7
    Flask + MySQL + Docker
          |
          v
    LAB 8
    Flask + MySQL + Docker Compose
          |
          v
    LAB 9
    Healthcheck + Restart + Logs
          |
          v
    LAB 10
    Docker Image Optimization
          |
          v
    LAB 11
    Multi-stage Builds
          |
          v
    LAB 12
    Docker Hub / Registry
          |
          v
    LAB 13
    Docker + Jenkins CI/CD
          |
          v
    FINAL DEVOPS PROJECT

    GitHub
       |
       v
    Jenkins
       |
       v
    Docker Build
       |
       v
    Docker Image
       |
       v
    Docker Registry
       |
       v
    AWS EC2
       |
       v
    Docker Container
       |
       v
    Web Application

---

# 56. CLEANUP

Keep database:

    docker compose down

Remove database volume too:

    docker compose down -v

Remove stopped containers:

    docker container prune

Remove unused images:

    docker image prune

Remove unused volumes:

    docker volume prune

WARNING:
Prune commands can remove resources you intended to keep.
Inspect resources before confirming.

---

# END OF PRACTICAL LAB

Student progression:

    Dockerfile
        |
        v
    Image
        |
        v
    Container
        |
        v
    Volume
        |
        v
    Bind Mount
        |
        v
    Network
        |
        v
    Environment Variables
        |
        v
    Compose
        |
        v
    Healthcheck
        |
        v
    Restart Policy
        |
        v
    Docker Hub
        |
        v
    Troubleshooting
        |
        v
    Jenkins CI/CD
        |
        v
    AWS Deployment
