# Docker Setup: Patient Service & PostgreSQL

This document describes how the PostgreSQL database container (patient-service-db) and the Spring Boot application container (patient-service) were created, configured, and connected using Docker.

---
### 1. Creating the Docker Network

An internal Docker bridge network was created to allow containers to communicate securely using container names as hostnames.

```shell
docker network create internal
```
**Purpose**
- Enables container-to-container communication

- Provides automatic DNS resolution using container names

- Isolates internal services from the host and other containers

---
### 2. Creating the PostgreSQL Container (patient-service-db)

The PostgreSQL database was started as a Docker container with required credentials and attached to the internal network.

```shell
docker run \
--name patient-service-db \
--network internal \
-e POSTGRES_USER=admin_user \
-e POSTGRES_PASSWORD=password \
-e POSTGRES_DB=db \
-p 6000:5432 \
-d postgres
```

**Explanation**

| Option                      | Description                            |
| --------------------------- | -------------------------------------- |
| `--name patient-service-db` | Assigns a fixed container name         |
| `--network internal`        | Attaches container to internal network |
| `POSTGRES_USER`             | Database username                      |
| `POSTGRES_PASSWORD`         | Database password                      |
| `POSTGRES_DB`               | Initial database name                  |
| `-p 6000:5432`              | Exposes DB to host (optional)          |
| `-d`                        | Runs container in detached mode        |


**Result**

- PostgreSQL runs on port 5432 inside the container

- Optionally accessible on port 6000 from the host

- Reachable by other containers using hostname patient-service-db

---
### 3. Creating the Spring Boot Image (patient-service)

The Spring Boot application was packaged into a Docker image using a Dockerfile.

**Build the image**

From the directory containing the Dockerfile:

```shell
docker build -t patient-service:1.0 .
```

**Explanation**

| Part                  | Meaning                                 |
| --------------------- | --------------------------------------- |
| `patient-service:1.0` | Image name and version                  |
| `.`                   | Current directory containing Dockerfile |

---
### **Dockerfile Explanation: patient-service**

This section explains the multi-stage Dockerfile used to build and run the Spring Boot application.

Dockerfile Used
```dockerfile
FROM maven:3.9.9-eclipse-temurin-21 AS builder

WORKDIR /app

COPY pom.xml .

RUN mvn dependency:go-offline -B

COPY src ./src

RUN mvn clean package

FROM eclipse-temurin:21-jdk AS runner

WORKDIR /app

COPY --from=builder ./app/target/patient-service-0.0.1-SNAPSHOT.jar ./app.jar

EXPOSE 4000

ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Stage 1: Build Stage (builder)**
```dockerfile
FROM maven:3.9.9-eclipse-temurin-21 AS builder
```

- Uses Maven with Java 21

- Contains Maven + JDK → required only for building

- Named builder for reference in later stages

```dockerfile
WORKDIR /app
```

- Sets working directory inside the container

- All subsequent commands run from /app

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline -B
```


- Copies only pom.xml first

- Downloads dependencies in advance

- Enables Docker layer caching

- Prevents re-downloading dependencies on every build
- 
```dockerfile
COPY src ./src
RUN mvn clean package
```


- Copies application source code

- Builds the Spring Boot JAR

Output:
```shell
target/patient-service-0.0.1-SNAPSHOT.jar
```
**Stage 2: Runtime Stage (runner)**
```dockerfile
FROM eclipse-temurin:21-jdk AS runner
```

- Lightweight Java 21 runtime image

- Does not include Maven

- Results in a smaller, cleaner image

```dockerfile
WORKDIR /app
```

- Runtime working directory

```dockerfile
COPY --from=builder ./app/target/patient-service-0.0.1-SNAPSHOT.jar ./app.jar
```

- Copies only the compiled JAR from the build stage

- No source code or Maven dependencies included

- Improves security and reduces image size

```dockerfile
EXPOSE 4000
```

- Documents the application’s listening port

- Does not publish the port

- Actual publishing is done using -p in docker run

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

- Starts the Spring Boot application

- Runs automatically when the container starts

**Why Multi-Stage Build is Used**

| Benefit          | Description                   |
| ---------------- | ----------------------------- |
| Smaller image    | Maven not included in runtime |
| Faster builds    | Dependency caching            |
| Better security  | No build tools in final image |
| Production ready | Clean separation of concerns  |

---

### 4. Running the Spring Boot Container (patient-service)

The application container was started and connected to the same internal network.

```shell
docker run \
--name patient-service \
--network internal \
-p 8080:8080 \
-d patient-service:1.0
```

**Explanation**

| Option                   | Description                 |
| ------------------------ | --------------------------- |
| `--name patient-service` | Container name              |
| `--network internal`     | Same network as DB          |
| `-p 8080:8080`           | Exposes application to host |
| `-d`                     | Detached mode               |
| `patient-service:1.0`    | Image name                  |

---
### 5. How patient-service Connects to patient-service-db
   **Key Concept: Docker Network DNS**

When two containers are attached to the same Docker network:

- Docker provides automatic DNS resolution

- A container can be reached using its container name as hostname

**Database Connection Configuration**

Inside the Spring Boot application:

```text
spring.datasource.url=jdbc:postgresql://patient-service-db:5432/db
spring.datasource.username=admin_user
spring.datasource.password=password
```
**Why this works**

| Component | Value                                 |
| --------- | ------------------------------------- |
| Hostname  | `patient-service-db` (container name) |
| Port      | `5432` (Postgres internal port)       |
| Network   | `internal` (shared bridge network)    |


Note: Containers communicate using container ports, not host-mapped ports.

---
### 6. Final Architecture

```text

   Host Machine
   │
   ├── http://localhost:8080  → patient-service
   │
   └── Docker internal network
   ├── patient-service (Spring Boot)
   │     └── connects to patient-service-db:5432
   │
   └── patient-service-db (PostgreSQL)
   └── listens on 5432
```

---
### 7. Verification Commands
```shell
docker ps
   
docker network inspect internal

docker logs patient-service
```
