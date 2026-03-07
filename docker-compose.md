# Docker Compose Command Reference

A concise list of commonly used **Docker Compose commands** for managing multi-container applications.

---

## 1. Basic Commands

```bash
docker compose up
```

Start containers defined in `docker-compose.yml`.

```bash
docker compose up -d
```

Start containers in **detached mode** (background).

```bash
docker compose down
```

Stop and remove containers, networks, and default volumes.

```bash
docker compose stop
```

Stop running containers without removing them.

```bash
docker compose start
```

Start previously stopped containers.

```bash
docker compose restart
```

Restart services.

---

## 2. Build & Rebuild

```bash
docker compose build
```

Build images for services.

```bash
docker compose build --no-cache
```

Rebuild images without using cache.

```bash
docker compose up --build
```

Build images and start containers.

---

## 3. Service Management

```bash
docker compose ps
```

List running services.

```bash
docker compose logs
```

View logs from all services.

```bash
docker compose logs -f
```

Follow logs in real time.

```bash
docker compose logs <service-name>
```

View logs for a specific service.

---

## 4. Execute Commands Inside Containers

```bash
docker compose exec <service-name> bash
```

Open bash shell inside container.

```bash
docker compose exec <service-name> sh
```

Open shell if bash is unavailable.

```bash
docker compose run <service-name> <command>
```

Run a one-off command in a service container.

Example:

```bash
docker compose run api-gateway curl http://patient-service:4000
```

---

## 5. Scaling Services

```bash
docker compose up --scale <service>=<number>
```

Example:

```bash
docker compose up --scale patient-service=3
```

Runs **3 instances** of patient-service.

---

## 6. Remove Containers & Volumes

```bash
docker compose down -v
```

Remove containers **and volumes**.

```bash
docker compose rm
```

Remove stopped containers.

---

## 7. Inspect Configuration

```bash
docker compose config
```

Validate and view the resolved compose configuration.

---

## 8. View Container Status

```bash
docker compose top
```

Display running processes in containers.

```bash
docker compose events
```

Stream real-time events from containers.

---

## 9. Pull & Push Images

```bash
docker compose pull
```

Pull images defined in compose file.

```bash
docker compose push
```

Push images to registry.

---

## 10. Useful Development Commands

```bash
docker compose down
docker compose up --build
```

Common workflow to rebuild and restart services.

```bash
docker compose up -d --build
```

Start services in background with rebuilt images.

---

## 11. Check Version

```bash
docker compose version
```

Displays installed Docker Compose version.

---

# Typical Development Workflow

```bash
docker compose down
docker compose build
docker compose up -d
docker compose logs -f
```

---

# Helpful Tip

Use `.env` file with Docker Compose for environment variables:

```env
DB_HOST=postgres
DB_PORT=5432
```

Then reference in `docker-compose.yml`:

```yaml
environment:
  DB_HOST: ${DB_HOST}
```

---

This command set covers most daily operations when working with Docker Compose for microservices and containerized applications.
