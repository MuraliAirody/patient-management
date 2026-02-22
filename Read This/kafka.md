# Kafka Environment Variables Explained

## Overview

These environment variables configure Apache Kafka running in **KRaft mode** (no Zookeeper required). KRaft is Kafka's built-in consensus protocol introduced to replace Zookeeper.

---

## Full Configuration

```
KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094
KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER
KAFKA_CONTROLLER_QUORUM_VOTERS=0@localhost:9093
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
KAFKA_NODE_ID=0
KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
KAFKA_PROCESS_ROLES=broker,controller
KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1
KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1
```

---

## Variable-by-Variable Breakdown

### 1. `KAFKA_LISTENERS`
```
KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
```
Defines **where Kafka listens** for incoming connections inside the container.

| Listener | Port | Purpose |
|---|---|---|
| `PLAINTEXT` | 9092 | Internal communication between containers |
| `CONTROLLER` | 9093 | KRaft controller (leader election, metadata) |
| `EXTERNAL` | 9094 | External access (terminal, apps outside Docker) |

> `::9092` means listen on **all interfaces** on port 9092 inside the container.

---

### 2. `KAFKA_ADVERTISED_LISTENERS`
```
KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094
```
Defines **what address Kafka tells clients to connect to** after initial handshake.

| Listener | Address | Who Uses It |
|---|---|---|
| `PLAINTEXT` | `kafka:9092` | Other Docker containers (using Docker network hostname `kafka`) |
| `EXTERNAL` | `localhost:9094` | Your terminal, Spring Boot app running on host machine |

> **Why two different addresses?**
> Docker containers talk to each other using service names (e.g., `kafka`), but your terminal/laptop uses `localhost`.

---

### 3. `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP`
```
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
```
Maps each **listener name → security protocol**.

| Listener Name | Protocol | Meaning |
|---|---|---|
| `CONTROLLER` | `PLAINTEXT` | No encryption (plain text) |
| `EXTERNAL` | `PLAINTEXT` | No encryption (plain text) |
| `PLAINTEXT` | `PLAINTEXT` | No encryption (plain text) |

> For production, you would use `SSL` or `SASL_SSL` instead of `PLAINTEXT`.

---

### 4. `KAFKA_CONTROLLER_LISTENER_NAMES`
```
KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER
```
Tells Kafka which listener is dedicated to **KRaft controller communication**.

- The `CONTROLLER` listener (port 9093) handles leader election and cluster metadata.
- This port should **never** be used by producers or consumers.

---

### 5. `KAFKA_CONTROLLER_QUORUM_VOTERS`
```
KAFKA_CONTROLLER_QUORUM_VOTERS=0@localhost:9093
```
Defines the list of **KRaft controller nodes** that participate in voting (like a cluster members list).

Format: `nodeId@host:port`

| Part | Value | Meaning |
|---|---|---|
| `0` | Node ID | This controller's ID (matches `KAFKA_NODE_ID`) |
| `localhost` | Host | Where the controller runs |
| `9093` | Port | Controller listener port |

> In a multi-broker setup, this would list all controllers:
> `0@broker1:9093,1@broker2:9093,2@broker3:9093`

---

### 6. `KAFKA_NODE_ID`
```
KAFKA_NODE_ID=0
```
A **unique numeric ID** for this Kafka broker/controller node.

- Must be unique across all nodes in the cluster.
- Here it is `0` because we have only one node.
- Must match the ID used in `KAFKA_CONTROLLER_QUORUM_VOTERS`.

---

### 7. `KAFKA_PROCESS_ROLES`
```
KAFKA_PROCESS_ROLES=broker,controller
```
Defines what **roles** this Kafka node plays in KRaft mode.

| Role | Responsibility |
|---|---|
| `broker` | Handles producers and consumers (stores/serves messages) |
| `controller` | Manages cluster metadata, leader election |

> In KRaft mode, a single node can be **both** broker and controller (good for local dev).
> In production, these roles are typically separated across different nodes.

---

### 8. `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR` ⭐
```
KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
```
Sets the replication factor for the **internal `__consumer_offsets` topic**.

This topic tracks where each consumer group has read up to (like a bookmark).

| Value | Meaning |
|---|---|
| `1` | Only 1 copy of data (fine for single broker / local dev) |
| `3` | 3 copies across 3 brokers (recommended for production) |

> **This was the root cause of our issue!**
> Default is `3`, but with only 1 broker, Kafka couldn't create this topic and kept failing.

---

### 9. `KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR` ⭐
```
KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1
```
Sets the replication factor for the **internal `__transaction_state` topic**.

This topic tracks the state of Kafka transactions (used when producers send messages atomically).

| Value | Meaning |
|---|---|
| `1` | Single copy — fine for local dev |
| `3` | Three copies — required for production reliability |

> Same fix as above — default `3` breaks with single broker.

---

### 10. `KAFKA_TRANSACTION_STATE_LOG_MIN_ISR` ⭐
```
KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1
```
**ISR = In-Sync Replicas** — the minimum number of replicas that must acknowledge a write before it's considered successful.

| Value | Meaning |
|---|---|
| `1` | Only 1 replica needs to confirm (matches single broker) |
| `2` | At least 2 replicas must confirm (safer for production) |

> Must be ≤ replication factor. With replication factor `1`, this must also be `1`.

---

## Port Summary

```
┌─────────────────────────────────────────────────────┐
│                 Docker Container                     │
│                                                     │
│  Port 9092 (PLAINTEXT)                              │
│  → Used by: other Docker containers                 │
│  → Address: kafka:9092                              │
│                                                     │
│  Port 9093 (CONTROLLER)                             │
│  → Used by: KRaft internal only                     │
│  → Never use this for producers/consumers           │
│                                                     │
│  Port 9094 (EXTERNAL)                               │
│  → Used by: your terminal, Spring Boot on host      │
│  → Address: localhost:9094                          │
└─────────────────────────────────────────────────────┘
```

---

## Dev vs Production Settings

| Variable | Dev (Single Broker) | Production (Multi Broker) |
|---|---|---|
| `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR` | `1` | `3` |
| `KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR` | `1` | `3` |
| `KAFKA_TRANSACTION_STATE_LOG_MIN_ISR` | `1` | `2` |
| `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP` | `PLAINTEXT` | `SSL` or `SASL_SSL` |
| `KAFKA_PROCESS_ROLES` | `broker,controller` | Separate nodes |

---

## Quick Reference — Which Port to Use?

| Scenario | Bootstrap Server |
|---|---|
| Terminal inside container | `localhost:9092` or `localhost:9094` |
| Spring Boot app on host machine | `localhost:9094` |
| Another Docker container | `kafka:9092` |
| Controller operations (internal only) | `localhost:9093` — never use directly |