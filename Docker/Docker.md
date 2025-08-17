
https://docs.docker.com/engine/install/ubuntu/

- Docker is an **open-source platform** that makes it easier to build, run, and manage applications inside **containers**
- A **container** is a lightweight, portable, and isolated environment that runs an application along with everything it needs (code, libraries, dependencies, configuration)

![[Pasted image 20250817215639.png]]
![[Pasted image 20250817220029.png]]

![[Pasted image 20250817215758.png]]


![[Pasted image 20250817221503.png]]







# Command
![[Pasted image 20250817215931.png]]






## docker ps
The command **docker ps** is used to **list running containers** in Docker:
```bash
docker ps
```

This shows a table with details of currently running containers, including:
- **CONTAINER ID** → A unique identifier for the container
- **IMAGE** → The Docker image the container is based on
- **COMMAND** → The command the container is running
- **CREATED** → When the container was started
- **STATUS** → Current status (e.g., `Up 5 minutes`)
- **PORTS** → Port mappings between the host and container
- **NAMES** → Human-readable name of the container

```nginx
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
3f8c9f6a2d1e   nginx:latest  "nginx -g 'daemon of…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp     web_server

```

- `docker ps -a` → Shows **all containers** (running, stopped, exited).
- `docker ps -q` → Shows only container **IDs** (useful in scripts).
- `docker ps --filter "status=exited"` → Shows only stopped containers.

## docker run
- It is used to **create and start a new container** from a specified image.

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
- **IMAGE** → The Docker image to use (e.g., `nginx`, `ubuntu`, `mysql`)
- **COMMAND [ARG...]** → (Optional) Command to run inside the container
- **OPTIONS** → Flags that modify behavior (network, ports, volumes, etc.)
![[Pasted image 20250817215320.png]]

- `-i` → Interactive (keeps STDIN open)
- `-t` → Allocate a pseudo-TTY (gives you a terminal)
- `-d` → Detached mode (runs in background)
- `-p HOST:CONTAINER` → Maps port 8080 on your computer to port 80 inside the container.
```bash
docker run -d -p 8080:80 nginx
```
![[Pasted image 20250817220210.png]]


- `-v host_path:container_path` → Mounts a folder from your machine into the container.

```bash
docker run -d -v /my/data:/usr/share/nginx/html nginx
```

- `--name` → Easier to reference containers instead of using container IDs.
```
docker run -d --name webserver nginx
```


## docker images
- The command **`docker images`** is used to **list all Docker images** stored locally on your system.

```bash
docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
nginx         latest    7b8b6451c85f   2 days ago      133MB
ubuntu        22.04     1d622ef86b13   3 weeks ago     77MB
mysql         8.0       6bdf28a4c987   1 month ago     546MB


```
 - Meaning of columns:
	- **REPOSITORY** → Name of the image (e.g., `nginx`, `ubuntu`).
	- **TAG** → Version or variant (e.g., `latest`, `22.04`, `8.0`).
	- **IMAGE ID** → Unique identifier for the image.
	- **CREATED** → When the image was built.
	- **SIZE** → How much disk space the image takes.


- List only image IDs
```bash
docker images -q
```
- Filter images

```bash
docker images --filter "dangling=true"
```

(Shows "dangling" images (unused, `<none>` repository or tag).)

- **Show all intermediate images** (created during builds)
```bash
docker images -a
```

- Remove an image
```bash
docker rmi IMAGE_ID
```


## docker log

- The command **`docker logs`** is used to **view the logs (output) of a running or stopped container**

![[Pasted image 20250817220827.png]]

- **Follow logs in real time** (like `tail -f`)
- Show timestamps ( -t )
- 3. **Show only the last N lines**
```bash
docker logs --tail 50 webserver
```

## docker exec

The command **`docker exec`** is used to **run a command inside a running container**.  
It’s like opening a door into the container and telling it to execute something.
```
docker exec [OPTIONS] <container_id_or_name> <command>
```

- Open a shell inside a container

```bash
docker exec -it mycontainer bash
```
	- `-i` → Interactive (keep STDIN open)
	- `-t` → Allocate a terminal (TTY)
	- Opens a Bash shell inside `mycontainer`.  
	    (If the container doesn’t have `bash`, try `sh` instead.)

- Run a single command inside a container

```bash
docker exec mycontainer ls /app
```
	- Runs `ls /app` inside the container named `mycontainer`

- Check environment variables
```bash
docker exec mycontainer env
```
- View running processes
```bash
docker exec mycontainer ps aux
```

## Difference from similar commands
- **`docker run`** → Creates **a new container** and runs a command there.
- **`docker exec`** → Runs a command inside an **already running container**.
- **`docker logs`** → Shows past output of the container, not execute commands.


## docker network
The **`docker network`** command is used to **manage networking in Docker**.
It lets containers talk to each other, the host machine, or the outside world

```bash
docker network [COMMAND]
```

Where `[COMMAND]` can be:
- `ls` → List networks
- `inspect` → Show details of a network
- `create` → Create a new network
- `connect` → Connect a container to a network
- `disconnect` → Disconnect a container from a network
- `rm` → Remove a network


## docker build
The **`docker build`** command is used to **create a Docker image** from a set of instructions written in a **Dockerfile**.
That image can then be run as a container.
```bash
docker build [OPTIONS] PATH | URL | -
```
- **PATH** → Directory containing your `Dockerfile` and any app files.
- **URL** → Can build directly from a Git repo.
- **-** → Read build context from `stdin`.




# Developing with Containers

## Using docker compose
(Running multiple services)
- Using .yaml file to setup iur configurations
![[Pasted image 20250817223218.png]]

- End then run the command
```bash
docker-compose -f file.yaml up
```
- Top stop 
```
docker-compose -f file.yaml down
```
- common command:
```bash
docker-compose up        # Start all services
docker-compose up -d     # Run in background
docker-compose down      # Stop and remove containers/networks
docker-compose ps        # List running services
docker-compose logs      # Show logs from all services
docker-compose exec web bash   # Run a shell inside the "web" container
```
`docker-compose` = **multi-container management made simple**.  
It’s like saying:  
👉 “Here’s my whole app stack in one file. Spin it up for me.”


## Building our own Docker Image
![[Pasted image 20250817223704.png]]

![[Pasted image 20250817223825.png]]

![[Pasted image 20250817223903.png]]
![[Pasted image 20250817223922.png]]

![[Pasted image 20250817223946.png]]
![[Pasted image 20250817224010.png]]
![[Pasted image 20250817224046.png]]

![[Pasted image 20250817224145.png]]

![[Pasted image 20250817224204.png]]

Now build the image:


```bash
docker build -t mypythonapp .
```
- `-t mypythonapp` → Tags the image with a name (`mypythonapp`).
- `.` → Build context = current directory.
Then run it:

```bash
docker run -d --name myapp mypythonapp
```
-  Useful options
- **`-t name:tag`** → Tag the image (default tag = `latest`).
```bash
docker build -t myapp:1.0 .
```
- **`-f Dockerfile.dev`** → Use a custom Dockerfile.
```bash
docker build -f Dockerfile.dev -t myapp:dev .
```
- **`--no-cache`** → Ignore cache, rebuild everything.
- **`--build-arg`** → Pass build-time variables.
```bash
docker build --build-arg APP_ENV=prod -t myapp .
```
- What happens during `docker build`
1. Docker reads your **Dockerfile** instructions **line by line**.  
2. Each instruction creates a **layer** (cached if not changed).
3. At the end, you get a new **image** ready to run.

## Pusing our build Docker
![[Pasted image 20250817224847.png]]


![[Pasted image 20250817225007.png]]

![[Pasted image 20250817225125.png]]
![[Pasted image 20250817225151.png]]

![[Pasted image 20250817225208.png]]

![[Pasted image 20250817225237.png]]



![[Pasted image 20250817225348.png]]


## Docker volume
By default, everything inside a container is **ephemeral**:
- If you **delete or recreate** a container → all data inside is lost.
- Containers are meant to be **stateless**.
But many apps need data to **persist** (databases, logs, uploads).  
That’s where **volumes** come in.
A **Docker volume** is storage managed by Docker that lives **outside the container’s filesystem**, so it isn’t deleted when the container goes away.

![[Pasted image 20250817225806.png]]

 ![[Pasted image 20250817225827.png]]

![[Pasted image 20250817225914.png]]

![[Pasted image 20250817225959.png]]

![[Pasted image 20250817230043.png]]


![[Pasted image 20250817230135.png]]

![[Pasted image 20250817230243.png]]

![[Pasted image 20250817230312.png]]




















 












