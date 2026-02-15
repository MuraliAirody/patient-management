## Docker Commands and Instructions


```shell
docker info
```
Provides detailed information about the Docker environment—both client and server (daemon).

- Whether the Docker daemon is running

- Docker version and storage driver

- Number of containers and images

- CPU, memory, and OS details used by Docker

```shell
docker login
```
Authenticates your local Docker CLI with a container registry (e.g., Docker Hub, AWS ECR, Azure ACR).

- To pull private images

- To push images you own

- To avoid rate limits on Docker Hub

```shell
docker ps
```
Lists containers on your system.

- Only running containers:

- Container ID

- Image name

- Command

- Status

- Ports

- Container name

| Command                            | Meaning                 |
| ---------------------------------- | ----------------------- |
| `docker ps -a`                     | All containers          |
| `docker ps -q`                     | Only container IDs      |
| `docker ps --filter status=exited` | Only stopped containers |
