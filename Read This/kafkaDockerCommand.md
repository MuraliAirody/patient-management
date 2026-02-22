# Docker & Kafka Commands — Complete Reference

---

## Docker Commands

### Container Management

| Action | Command |
|---|---|
| List running containers | `docker ps` |
| List all containers (including stopped) | `docker ps -a` |
| Start a container | `docker start <container_name>` |
| Stop a container | `docker stop <container_name>` |
| Restart a container | `docker restart <container_name>` |
| Remove a container | `docker rm <container_name>` |
| Force remove running container | `docker rm -f <container_name>` |
| Enter a container | `docker exec -it <container_name> bash` |
| Exit a container | `exit` |

---

### Image Management

| Action | Command |
|---|---|
| List all images | `docker images` |
| Pull an image | `docker pull <image_name>` |
| Remove an image | `docker rmi <image_name>` |
| Force remove an image | `docker rmi -f <image_name>` |
| Build an image | `docker build -t <image_name> .` |
| Inspect an image | `docker inspect <image_name>` |

---

### Volume Management

| Action | Command |
|---|---|
| List all volumes | `docker volume ls` |
| Inspect a volume | `docker volume inspect <volume_name>` |
| Remove a volume | `docker volume rm <volume_name>` |
| Remove all unused volumes | `docker volume prune` |
| List dangling volumes | `docker volume ls -f dangling=true` |

---

### Logs & Monitoring

| Action | Command |
|---|---|
| View container logs | `docker logs <container_name>` |
| Follow live logs | `docker logs -f <container_name>` |
| Last 50 lines of logs | `docker logs <container_name> --tail 50` |
| Filter logs by keyword | `docker logs <container_name> \| grep "ERROR"` |
| Check started/error logs | `docker logs <container_name> \| grep -E "started\|ERROR\|WARN"` |
| Container stats (live) | `docker stats <container_name>` |
| Inspect container details | `docker inspect <container_name>` |
| Check container ports | `docker inspect <container_name> \| grep -i port` |

---

### Network Commands

| Action | Command |
|---|---|
| List networks | `docker network ls` |
| Inspect a network | `docker network inspect <network_name>` |
| Create a network | `docker network create <network_name>` |
| Remove a network | `docker network rm <network_name>` |

---

### Docker Compose Commands

| Action | Command |
|---|---|
| Start all services | `docker-compose up` |
| Start in background | `docker-compose up -d` |
| Stop all services | `docker-compose down` |
| Stop and remove volumes | `docker-compose down -v` |
| View logs | `docker-compose logs -f` |
| View specific service logs | `docker-compose logs -f kafka` |
| Restart a service | `docker-compose restart kafka` |
| Rebuild and start | `docker-compose up --build` |

---

### Full Cleanup (Nuclear Option ⚠️)

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Remove all volumes
docker volume prune

# Remove everything unused at once
docker system prune -a
```

> ⚠️ `docker system prune -a` removes ALL unused containers, images, volumes, and networks. Use with caution!

---

---

## Kafka Commands (Inside Container)

First, enter the container and navigate to bin:
```bash
docker exec -it <container_name> bash
cd /opt/kafka/bin
```

---

### Topic Commands

#### List all topics
```bash
./kafka-topics.sh --list \
  --bootstrap-server localhost:9092
```

#### Create a topic
```bash
./kafka-topics.sh --create \
  --topic <topic_name> \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1
```

#### Describe a topic
```bash
./kafka-topics.sh --describe \
  --topic <topic_name> \
  --bootstrap-server localhost:9092
```

#### Delete a topic
```bash
./kafka-topics.sh --delete \
  --topic <topic_name> \
  --bootstrap-server localhost:9092
```

#### Alter topic partitions
```bash
./kafka-topics.sh --alter \
  --topic <topic_name> \
  --partitions 3 \
  --bootstrap-server localhost:9092
```

---

### Producer Commands

#### Basic producer
```bash
./kafka-console-producer.sh \
  --topic <topic_name> \
  --bootstrap-server localhost:9092
```

#### Producer with key
```bash
./kafka-console-producer.sh \
  --topic <topic_name> \
  --bootstrap-server localhost:9092 \
  --property "key.separator=:" \
  --property "parse.key=true"
```
Then type: `mykey:myvalue`

#### Producer with acknowledgement
```bash
./kafka-console-producer.sh \
  --topic <topic_name> \
  --bootstrap-server localhost:9092 \
  --producer-property acks=all
```

---

### Consumer Commands

#### Basic consumer (new messages only)
```bash
./kafka-console-consumer.sh \
  --topic <topic_name> \
  --bootstrap-server localhost:9092
```

#### Consumer from beginning
```bash
./kafka-console-consumer.sh \
  --topic <topic_name> \
  --bootstrap-server localhost:9092 \
  --from-beginning
```

#### Consumer with key displayed
```bash
./kafka-console-consumer.sh \
  --topic <topic_name> \
  --bootstrap-server localhost:9092 \
  --from-beginning \
  --property print.key=true \
  --property key.separator=" -> "
```

#### Consumer with consumer group
```bash
./kafka-console-consumer.sh \
  --topic <topic_name> \
  --bootstrap-server localhost:9092 \
  --from-beginning \
  --group my-consumer-group
```

#### Consumer with timestamp
```bash
./kafka-console-consumer.sh \
  --topic <topic_name> \
  --bootstrap-server localhost:9092 \
  --from-beginning \
  --property print.timestamp=true
```

#### Consumer — limit number of messages
```bash
./kafka-console-consumer.sh \
  --topic <topic_name> \
  --bootstrap-server localhost:9092 \
  --from-beginning \
  --max-messages 10
```

---

### Consumer Group Commands

#### List all consumer groups
```bash
./kafka-consumer-groups.sh --list \
  --bootstrap-server localhost:9092
```

#### Describe a consumer group
```bash
./kafka-consumer-groups.sh --describe \
  --group <group_name> \
  --bootstrap-server localhost:9092
```

#### Reset consumer group offset (to beginning)
```bash
./kafka-consumer-groups.sh \
  --group <group_name> \
  --topic <topic_name> \
  --reset-offsets \
  --to-earliest \
  --execute \
  --bootstrap-server localhost:9092
```

#### Delete a consumer group
```bash
./kafka-consumer-groups.sh --delete \
  --group <group_name> \
  --bootstrap-server localhost:9092
```

---

### Useful Kafka Checks

#### Check if Kafka broker is reachable
```bash
./kafka-broker-api-versions.sh \
  --bootstrap-server localhost:9092
```

#### Check listening ports (inside container)
```bash
netstat -tlnp | grep 909
```

#### Check Kafka logs location
```bash
ls /opt/kafka/logs/
```

---

## Port Quick Reference

| Port | Listener | Use For |
|---|---|---|
| `9092` | PLAINTEXT | Other Docker containers / internal |
| `9093` | CONTROLLER | KRaft internal only — never use directly |
| `9094` | EXTERNAL | Terminal, Spring Boot on host machine |

---

## Bootstrap Server Quick Reference

| Scenario | Bootstrap Server |
|---|---|
| Inside container terminal | `localhost:9092` |
| Spring Boot app on host | `localhost:9094` |
| Another Docker container | `kafka:9092` |

---

## Complete Test Flow

```bash
# Step 1 — Enter container
docker exec -it <container_name> bash
cd /opt/kafka/bin

# Step 2 — Create topic
./kafka-topics.sh --create --topic test-topic \
  --bootstrap-server localhost:9092 \
  --partitions 1 --replication-factor 1

# Step 3 — Verify topic
./kafka-topics.sh --list --bootstrap-server localhost:9092

# Step 4 — Open Consumer (Terminal 1)
./kafka-console-consumer.sh --topic test-topic \
  --bootstrap-server localhost:9092 --from-beginning

# Step 5 — Open Producer (Terminal 2)
./kafka-console-producer.sh --topic test-topic \
  --bootstrap-server localhost:9092

# Step 6 — Type messages in Terminal 2, see them in Terminal 1 ✅
```

---

## Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `command not found` | Not in bin directory | Use `./kafka-topics.sh` with `./` prefix |
| `INVALID_REPLICATION_FACTOR` | Replication factor > broker count | Set `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1` |
| `Connection refused` | Kafka not running | Check `docker ps` and `docker logs` |
| `Topic already exists` | Topic was not deleted properly | Restart container with `docker restart` |
| `__consumer_offsets` keeps failing | Missing env variables | Add the 3 replication factor env vars |