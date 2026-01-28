# Deployment Guide for Railway (Pipeshub AI)

This document outlines the services, images, and environment variables required to deploy Pipeshub AI on Railway or similar PaaS providers.

## Architecture & Services

The application consists of the following components:

| Service Name       | Docker Image / Context            | Description                       |
| :----------------- | :-------------------------------- | :-------------------------------- |
| **Frontend**       | `./frontend`                      | React/Vite application            |
| **Backend Node**   | `./backend/nodejs`                | Main API Gateway & Orchestrator   |
| **Backend Python** | `./backend/python`                | AI Services, Indexing, & Querying |
| **MongoDB**        | `mongo:8.0.17`                    | Primary document store            |
| **Redis**          | `redis:bookworm`                  | Caching & Message Broker          |
| **ArangoDB**       | `arangodb:3.12.7.1`               | Graph Database                    |
| **Qdrant**         | `qdrant/qdrant:v1.15`             | Vector Database                   |
| **Kafka**          | `confluentinc/cp-kafka:7.9.0`     | Event Streaming                   |
| **Zookeeper**      | `confluentinc/cp-zookeeper:7.9.0` | Coordinator for Kafka             |
| **Etcd**           | `quay.io/coreos/etcd:v3.5.17`     | Distributed Key-Value Store       |

---

## Environment Variables Reference

### 1. Frontend (`frontend/`)

These variables are required at **build time**.

| Variable           | Description                           | Example / Default                                     |
| :----------------- | :------------------------------------ | :---------------------------------------------------- |
| `VITE_SERVER_URL`  | Public URL of the Node.js Backend     | `https://backend-node-production-c467.up.railway.app` |
| `VITE_AUTH_URL`    | Public URL for Auth (Node.js Backend) | `https://backend-node-production-c467.up.railway.app` |
| `VITE_BACKEND_URL` | Base API URL                          | `https://backend-node-production-c467.up.railway.app` |
| `VITE_IAM_URL`     | IAM Service URL                       | `https://backend-node-production-c467.up.railway.app` |

> **Note**: For production, ensure these point to your deployed backend domain (https).

### 2. Backend Node (`backend/nodejs/apps/`)

| Variable                   | Description                               | Example / Default                                                    |
| :------------------------- | :---------------------------------------- | :------------------------------------------------------------------- |
| **Core**                   |                                           |                                                                      |
| `PORT`                     | Service Port                              | `3000`                                                               |
| `NODE_ENV`                 | Environment                               | `production`                                                         |
| `LOG_LEVEL`                | Logging verbosity                         | `info`                                                               |
| `ALLOWED_ORIGINS`          | CORS Allowlist (Comma separated)          | `https://your-frontend.com`                                          |
| `SECRET_KEY`               | **Critical** Security Key                 | `your-secure-random-secret`                                          |
| **Service URLs**           |                                           |                                                                      |
| `FRONTEND_PUBLIC_URL`      | Public URL of Frontend                    | `https://your-frontend.com`                                          |
| `QUERY_BACKEND`            | Internal URL for Python Query Service     | `http://backend-python.railway.internal:8000`                        |
| `CONNECTOR_BACKEND`        | Internal URL for Python Connector Service | `http://backend-python.railway.internal:8088`                        |
| `INDEXING_BACKEND`         | Internal URL for Python Indexing Service  | `http://backend-python.railway.internal:8091`                        |
| `CONNECTOR_PUBLIC_BACKEND` | Public URL for Connector (if different)   | _Optional_                                                           |
| **Databases**              |                                           |                                                                      |
| `MONGO_URI`                | MongoDB Connection String                 | `mongodb://user:pass@mongo.railway.internal:27017/enterprise-search` |
| `MONGO_DB_NAME`            | MongoDB Database Name                     | `enterprise-search`                                                  |
| `REDIS_HOST`               | Redis Host                                | `redis.railway.internal`                                             |
| `REDIS_PORT`               | Redis Port                                | `6379`                                                               |
| `REDIS_PASSWORD`           | Redis Password                            | `your-redis-password`                                                |
| `REDIS_URL`                | Full Redis Connection URL                 | `redis://:pass@redis.railway.internal:6379`                          |
| `ARANGO_URL`               | ArangoDB URL                              | `http://arangodb.railway.internal:8529`                              |
| `ARANGO_DB_NAME`           | ArangoDB Database Name                    | `es`                                                                 |
| `ARANGO_USERNAME`          | ArangoDB User                             | `root`                                                               |
| `ARANGO_PASSWORD`          | ArangoDB Password                         | `your-arango-password`                                               |
| `ETCD_URL`                 | Etcd URL                                  | `http://etcd.railway.internal:2379`                                  |
| `ETCD_HOST`                | Etcd Hostname                             | `etcd.railway.internal`                                              |
| `QDRANT_API_KEY`           | Qdrant API Key                            | `your-qdrant-key`                                                    |
| `QDRANT_HOST`              | Qdrant Host                               | `qdrant.railway.internal`                                            |
| `QDRANT_GRPC_PORT`         | Qdrant GRPC Port                          | `6334`                                                               |
| `QDRANT_PORT`              | Qdrant HTTP Port                          | `6333`                                                               |
| `KAFKA_BROKERS`            | Kafka Brokers List                        | `cp-kafka.railway.internal:9092`                                     |
| `OLLAMA_API_URL`           | Ollama URL (if using self-hosted LLM)     | `http://host.docker.internal:11434`                                  |
| **Integrations**           |                                           |                                                                      |
| `SLACK_SIGNING_SECRET`     | Slack App Signing Secret                  | _Required for Slack Bot_                                             |
| `BOT_TOKEN`                | Slack Bot Token                           | _Required for Slack Bot_                                             |

### 3. Backend Python (`backend/python/`)

The Python backend shares many variables with the Node.js backend (Databases, Kafka, etc.). Ensure they match.

| Variable                 | Description               | Example / Default                                    |
| :----------------------- | :------------------------ | :--------------------------------------------------- |
| **Specific Configs**     |                           |                                                      |
| `MAX_TABLE_ROWS_FOR_LLM` | Limit rows sent to LLM    | `1000`                                               |
| `OPIK_API_KEY`           | Opik AI Key (Optional)    |                                                      |
| `OPIK_WORKSPACE`         | Opik Workspace (Optional) |                                                      |
| **Integrations**         |                           |                                                      |
| `TRELLO_API_KEY`         | Trello Power-Up API Key   | _If using Trello_                                    |
| `TRELLO_API_TOKEN`       | Trello Token              | _If using Trello_                                    |
| **Databases**            | (Same as Node.js)         | Match Node.js values for Redis, Arango, Mongo, Kafka |

### 4. Database Service Configurations

When deploying the database services, use these variables to initialize them securely.

#### MongoDB (`mongo:8.0.17`)

- `MONGO_INITDB_ROOT_USERNAME`: `admin`
- `MONGO_INITDB_ROOT_PASSWORD`: `your-mongo-password`

#### Redis (`redis:bookworm`)

- `REDIS_PASSWORD`: `your-redis-password`
- **Command Override**: `redis-server --requirepass your-redis-password`

#### ArangoDB (`arangodb:3.12.7.1`)

- `ARANGO_ROOT_PASSWORD`: `your-arango-password`

#### Qdrant (`qdrant/qdrant:v1.15`)

- `QDRANT__SERVICE__API_KEY`: `your-qdrant-key`
- `QDRANT__SERVICE__MAX_REQUEST_SIZE_MB`: `64`

#### Etcd (`quay.io/coreos/etcd:v3.5.17`)

**Railway Custom Start Command:**

```bash
etcd --name etcd-node --data-dir /etcd-data --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://etcd.railway.internal:2379 --listen-peer-urls http://0.0.0.0:2380 --initial-advertise-peer-urls http://etcd.railway.internal:2380 --initial-cluster etcd-node=http://etcd.railway.internal:2380
```

_Note: Make sure your service name is exactly `etcd` so the hostname matches._

#### Kafka & Zookeeper (`confluentinc`)

- **Zookeeper**:
  - `ZOOKEEPER_CLIENT_PORT`: `2181`
  - `ZOOKEEPER_TICK_TIME`: `2000`
- **Kakfa Listeners (Crucial for Railway)**:
  - `KAFKA_BROKER_ID`: `1`
  - `KAFKA_ZOOKEEPER_CONNECT`: `cp-zookeeper.railway.internal:2181`
    - _Ensure your Zookeeper service is named `cp-zookeeper`, or update this to `<your-zookeeper-service-name>.railway.internal:2181`._
  - `KAFKA_ADVERTISED_LISTENERS`: `ACCESS://cp-kafka.railway.internal:9092`
    - _Confirmed value for your setup._
  - `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP`: `ACCESS:PLAINTEXT`
  - `KAFKA_LISTENERS`: `ACCESS://0.0.0.0:9092`
  - `KAFKA_INTER_BROKER_LISTENER_NAME`: `ACCESS`

### How to find your Service Name

1. Go to your Railway Project Canvas.
2. Click on the Kafka service.
3. Look at the top left for the name (e.g., "Kafka", "kafka-1", "My Kafka").
4. Convert it to a "slug" (lowercase, spaces become dashes).
   - "Kafka" -> `kafka`
   - "Kafka Service" -> `kafka-service`
5. Append `.railway.internal` to get the hostname.

## Deployment Notes

1.  **CORS**: Ensure `ALLOWED_ORIGINS` in Backend Node matches your Frontend's public domain.
2.  **Internal Networking**: In Railway, services can often communicate via their service names as hostnames within the private network. Use these private addresses for `MONGO_URI`, `REDIS_HOST`, etc.
3.  **Public Access**: Explicitly expose `PORT` (Node) and `8000/8088/8091` (Python) only if necessary. Usually, only the Node.js backend needs to be public.
