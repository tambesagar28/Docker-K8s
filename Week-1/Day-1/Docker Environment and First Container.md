# Day 1: Docker Environment and First Container

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain what Docker is and why it is used.
- Understand the difference between a Docker image and a container.
- Explain the basic roles of Docker CLI, Docker Engine, `containerd`, and `runc`.
- Verify a Docker installation.
- Run and inspect Docker containers.
- Understand basic container lifecycle operations.
- Run an NGINX web server inside a container and access it through a browser.

---

## 1. What Is Docker?

Docker is a platform that allows us to build, package, distribute, and run applications in containers.

An application usually depends on several components, such as:

- Application source code
- Programming language runtime
- Libraries and dependencies
- Configuration files
- System-level packages

Without a consistent environment, an application may work on one machine but fail on another because of differences in dependencies or configuration.

Docker helps package the application and its required filesystem and configuration into an image. This image can then be used to create containers in different supported environments.

### Problems Docker Helps Solve

- Environment inconsistencies between development and production.
- Dependency conflicts between applications.
- Difficulties distributing applications with their required dependencies.
- Slow and inconsistent application deployment processes.
- Resource overhead associated with running a separate virtual machine for every application.

### Containers vs. Virtual Machines

| Containers | Virtual Machines |
|---|---|
| Share the host operating system kernel. | Run a guest operating system kernel through virtualization. |
| Usually start quickly. | Usually require more time to boot. |
| Typically have lower overhead. | Typically have higher resource overhead. |
| Isolate application processes and associated resources. | Provide isolation through a virtualized machine boundary. |

**Important:** Containers provide isolation, but they are not equivalent to complete virtual machines.

---

## 2. Docker Image vs. Docker Container

### Docker Image

A Docker image is an immutable, layered template used to create containers.

It contains the filesystem and configuration required to run an application, including the application files and any dependencies packaged into the image.

Examples:

- `hello-world`
- `nginx:alpine`
- `ubuntu:24.04`

### Docker Container

A container is an instance created from an image.

It has its own runtime configuration, writable container layer, and application processes. A container can be running, stopped, or in another lifecycle state.

### Key Differences

| Docker Image | Docker Container |
|---|---|
| Acts as a template. | Acts as an instance created from an image. |
| Immutable by design. | Has a writable layer by default, unless configured otherwise. |
| Stored locally or in a container registry. | Exists as a managed runtime object on the Docker host. |
| Used to create containers. | Runs or has run application processes. |
| Multiple containers can use the same image. | Each container has its own runtime state. |

### Example

```bash
docker pull nginx:alpine
docker run -d --name web1 nginx:alpine
docker run -d --name web2 nginx:alpine
```

Both containers use the same image but have separate container identities and runtime state.

---

## 3. Basic Docker Architecture

A common Docker Engine setup includes the following components.

```text
User
 |
 v
Docker CLI
 |
 | Docker API
 v
Docker Daemon (dockerd)
 |
 v
containerd
 |
 v
runc
 |
 v
Container Process
```

### Docker CLI

The Docker CLI is the command-line client used to interact with Docker.

Examples:

```bash
docker run hello-world
docker ps
docker images
```

The CLI sends requests to the Docker daemon through the Docker API.

### Docker Daemon (`dockerd`)

The Docker daemon is a background service that manages Docker resources and processes API requests.

Its responsibilities include:

- Managing containers and images.
- Managing Docker networks and volumes.
- Coordinating container creation and removal.
- Communicating with runtime components.

### containerd

`containerd` is a container runtime component responsible for container lifecycle operations, image transfer and storage, and coordinating container execution.

### runc

`runc` is a low-level OCI-compliant runtime commonly used to create and start container processes according to the OCI Runtime Specification.

**Note:** This is a simplified view of a common Docker Engine architecture. The exact implementation can vary by environment.

---

## 4. Verify the Docker Installation

Before running containers, verify that Docker is installed and operational.

### Check the Docker CLI Version

```bash
docker --version
```

This displays the installed Docker CLI version.

### Check Client and Server Information

```bash
docker version
```

This displays information about the Docker client and server.

A working Docker Engine should normally show both Client and Server sections.

### Check Docker Engine Information

```bash
docker info
```

This displays information about the Docker daemon, including configuration, storage, and runtime details.

**Important:** A successful `docker --version` command alone does not confirm that the Docker daemon is running.

If the server is unavailable, verify that Docker Desktop or the Docker Engine service is running.

---

## 5. Run the First Container: Hello World

Execute:

```bash
docker run hello-world
```

### What Happens?

1. Docker checks whether the image is available locally.
2. If the image is missing, Docker attempts to pull it from the configured registry.
3. Docker creates a new container from the image.
4. The program inside the container executes.
5. The program prints a success message and exits.

Because the `hello-world` program finishes quickly, the container does not remain running.

### Inspect Local Images

```bash
docker images
```

This lists locally available images.

### List Running Containers

```bash
docker ps
```

This lists only running containers by default.

### List All Containers

```bash
docker ps -a
```

This lists all containers, including stopped and exited containers.

### Understanding the Exit Status

The `hello-world` container may display a status similar to:

```text
Exited (0)
```

Exit code `0` indicates that the program completed successfully.

A stopped container is not necessarily a failed container.

---

## 6. Run an NGINX Container

NGINX is a web server that can serve web pages and handle HTTP requests.

In this exercise, we will run NGINX inside a Docker container.

### Step 1: Pull the NGINX Image

```bash
docker pull nginx:alpine
```

This downloads the `nginx:alpine` image to the local machine.

The `alpine` tag identifies an Alpine Linux-based image variant.

### Step 2: Run the Container

```bash
docker run -d --name day1-nginx -p 8080:80 nginx:alpine
```

### Command Breakdown

| Component | Description |
|---|---|
| `docker run` | Creates and starts a container. |
| `-d` | Runs the container in detached mode. |
| `--name day1-nginx` | Assigns a name to the container. |
| `-p 8080:80` | Maps host port 8080 to container port 80. |
| `nginx:alpine` | Specifies the image and tag. |

### Understanding Port Mapping

```text
Host Machine                         NGINX Container
+---------------------+              +---------------------+
|                     |              |                     |
| localhost:8080 ---->|------------->| Port 80             |
|                     |              | NGINX Web Server    |
+---------------------+              +---------------------+
```

The port mapping allows requests received on host port `8080` to reach port `80` inside the container.

### Step 3: Verify the Container

```bash
docker ps
```

Confirms whether the container is running.

```bash
docker images
```

Lists the locally available images.

```bash
docker port day1-nginx
```

Displays the container's port mappings.

```bash
docker logs day1-nginx
```

Displays the container's standard output and error logs.

### Step 4: Access NGINX

Open the following URL in your browser:

http://localhost:8080

Expected result: The NGINX welcome page should appear.

If the page does not load, check the container status, port mapping, and Docker daemon.

---

## 7. Inspect a Container

Docker provides the `docker inspect` command to retrieve detailed information about a container.

### Inspect the Complete Container Configuration

```bash
docker inspect day1-nginx
```

The output is JSON and contains details about the container's configuration, state, networking, mounts, and other metadata.

### Check the Container Status

```bash
docker inspect --format '{{.State.Status}}' day1-nginx
```

Expected output:

```text
running
```

### Check the Image Used by the Container

```bash
docker inspect --format '{{.Config.Image}}' day1-nginx
```

Expected output:

```text
nginx:alpine
```

### Check Port Mapping Information

```bash
docker inspect --format '{{json .NetworkSettings.Ports}}' day1-nginx
```

This displays the container's configured ports and host bindings.

---

## 8. Understand the Container Lifecycle

This exercise demonstrates that a container and its image are separate resources.

### Step 1: Check the Container and Image

```bash
docker ps
docker images
```

Confirm that the `day1-nginx` container is running and that the NGINX image is available locally.

### Step 2: Stop the Container

```bash
docker stop day1-nginx
```

Now execute:

```bash
docker ps
docker ps -a
```

Observe:

- The container no longer appears in `docker ps`.
- The container remains visible in `docker ps -a`.
- The NGINX image remains available locally.

Stopping a container does not remove it or delete its image.

### Step 3: Start the Same Container Again

```bash
docker start day1-nginx
docker ps
```

Open the browser again:

http://localhost:8080

The NGINX welcome page should be accessible again.

The same container has been restarted. Docker has not created a new container.

### Step 4: Remove the Container

After completing the experiment:

```bash
docker rm -f day1-nginx
```

The `-f` flag forces removal if the container is running.

Verify the result:

```bash
docker ps -a
docker images
```

The container should no longer appear, while the `nginx:alpine` image should remain locally available.

**Key takeaway:** Containers can be started, stopped, and removed independently of their images.

---

## 9. Important Docker Commands — Day 1

| Command | Purpose |
|---|---|
| `docker --version` | Displays the Docker CLI version. |
| `docker version` | Displays client and server version information. |
| `docker info` | Displays Docker Engine information. |
| `docker pull IMAGE` | Downloads an image. |
| `docker images` | Lists local images. |
| `docker run IMAGE` | Creates and starts a container. |
| `docker ps` | Lists running containers. |
| `docker ps -a` | Lists all containers. |
| `docker start CONTAINER` | Starts an existing stopped container. |
| `docker stop CONTAINER` | Stops a running container. |
| `docker logs CONTAINER` | Displays container logs. |
| `docker port CONTAINER` | Displays published port mappings. |
| `docker inspect CONTAINER` | Displays detailed container information. |
| `docker rm CONTAINER` | Removes a stopped container. |
| `docker rm -f CONTAINER` | Forces removal of a container. |

---

## 10. Day 1 Practical Checklist

- [ ] Verified Docker CLI and Docker Engine.
- [ ] Successfully ran the `hello-world` container.
- [ ] Inspected local images and containers.
- [ ] Pulled the `nginx:alpine` image.
- [ ] Started the NGINX container in detached mode.
- [ ] Accessed the NGINX welcome page through `localhost:8080`.
- [ ] Inspected the container using `docker inspect`.
- [ ] Stopped and restarted the same container.
- [ ] Removed the test container.
- [ ] Documented the difference between images and containers.
- [ ] Reviewed the basic Docker architecture.

---

## 11. Knowledge Check

Answer these questions without referring to the notes.

1. What problem does Docker solve?
2. What is the difference between a Docker image and a container?
3. What is the difference between `docker run` and `docker start`?
4. Why might a `hello-world` container not appear in `docker ps`?
5. In `-p 8080:80`, which port belongs to the host and which belongs to the container?
6. Does stopping a container delete its image?
7. What information does `docker inspect` provide?
8. What is the purpose of detached mode?
9. What are the responsibilities of Docker CLI, `dockerd`, `containerd`, and `runc`?
10. What happens when you remove a container but keep its image?

---

## 12. Official Documentation

Read the following official Docker documentation:

- [Docker Overview](https://docs.docker.com/get-started/docker-overview/)
- [Docker Images](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)
- [Docker Containers](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/)
- [Docker Engine Architecture](https://docs.docker.com/get-started/docker-overview/#docker-architecture)

---

## Day 1 Deliverable

By completing this lesson, you should have:

- A working Docker environment.
- Successfully executed `hello-world` and NGINX containers.
- Verified NGINX access through the browser.
- Practised basic container lifecycle operations.
- Understood the fundamental differences between images and containers.
- Documented the basic Docker architecture and essential commands.
