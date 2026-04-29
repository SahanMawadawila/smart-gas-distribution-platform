# LPG Distribution & Queue Management System

A microservices-based smart LPG distribution and queue management system built to solve the real-world problem of LPG shortages and long queues in Sri Lanka. Citizens can check stock availability and claim virtual tokens remotely, eliminating the need to physically wait in queues without knowing whether gas is available.

---

## System Architecture

Event-Driven Microservices Architecture using the Netflix OSS stack.

```
                        ┌─────────────────┐
                        │   API Gateway   │
                        │   (Port 8080)   │
                        └────────┬────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
     ┌────────▼───────┐ ┌───────▼────────┐ ┌───────▼────────┐
     │  User Service  │ │Inventory Service│ │ Queue Service  │
     │  (Port 8081)   │ │  (Port 8082)   │ │  (Port 8083)   │
     └────────┬───────┘ └───────┬────────┘ └───────┬────────┘
              │                  │                  │
     ┌────────▼───────┐ ┌───────▼────────┐ ┌───────▼────────┐
     │  PostgreSQL    │ │  PostgreSQL    │ │    MongoDB     │
     │  (userdb)      │ │  (inventorydb) │ │  (queuedb)     │
     └────────────────┘ └────────────────┘ └────────────────┘

                        ┌─────────────────┐
                        │  Eureka Server  │
                        │   (Port 8761)   │
                        └─────────────────┘
```

---

## Services Overview

| Service | Port | Database | Description |
|---|---|---|---|
| Discovery Server (Eureka) | 8761 | — | Service registry and health monitoring |
| API Gateway | 8080 | — | Single entry point, routing, load balancing |
| User Service | 8081 | PostgreSQL (5432) | Auth, profiles, role management |
| Inventory Service | 8082 | PostgreSQL (5433) | Stock management, dealer inventory |
| Queue Service | 8083 | MongoDB (27017) | Token management, queue processing |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.2.x |
| Service Discovery | Netflix Eureka |
| API Gateway | Spring Cloud Gateway |
| Inter-service Communication | OpenFeign |
| ORM | Spring Data JPA / Hibernate |
| Relational Database | PostgreSQL 15 |
| Document Database | MongoDB 7 |
| Security | Spring Security + JWT |
| Containerization | Docker + Docker Compose |
| Build Tool | Maven |

---

## Prerequisites

Make sure the following are installed on your machine:

- [Java 17+](https://adoptium.net/)
- [Maven 3.8+](https://maven.apache.org/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Git](https://git-scm.com/)
- [IntelliJ IDEA](https://www.jetbrains.com/idea/) (recommended) or VS Code


---

## Project Structure

```
lpg-system/
├── discovery-server/           # Eureka Service Registry
│   ├── src/
│   ├── Dockerfile
│   └── pom.xml
├── api-gateway/                # Spring Cloud Gateway
│   ├── src/
│   ├── Dockerfile
│   └── pom.xml
├── user-service/               # User & Auth Service
│   ├── src/
│   ├── Dockerfile
│   └── pom.xml
├── inventory-service/          # Inventory & Stock Service
│   ├── src/
│   ├── Dockerfile
│   └── pom.xml
├── queue-service/              # Queue & Token Service
│   ├── src/
│   ├── Dockerfile
│   └── pom.xml
├── frontend/                   # Web UI (Next.js) or Postman collection
├── docker-compose.yml          # Full system orchestration
└── README.md
```

---

## Getting Started

### Option 1 — Run Everything with Docker Compose (Recommended)

This starts all services, databases, and the network with one command.

**Step 1 — Clone the repository**

```bash
git clone https://github.com/your-org/lpg-system.git
cd lpg-system
```

**Step 2 — Build all services**

```bash
cd discovery-server && ./mvnw clean package -DskipTests && cd ..
cd user-service && ./mvnw clean package -DskipTests && cd ..
cd inventory-service && ./mvnw clean package -DskipTests && cd ..
cd queue-service && ./mvnw clean package -DskipTests && cd ..
cd api-gateway && ./mvnw clean package -DskipTests && cd ..
```

**Step 3 — Start everything**

```bash
docker compose up --build
```

**Step 4 — Verify**

| URL | Description |
|---|---|
| http://localhost:8761 | Eureka Dashboard — all services should appear |
| http://localhost:8080 | API Gateway entry point |
| http://localhost:8081 | User Service (direct) |
| http://localhost:8082 | Inventory Service (direct) |
| http://localhost:8083 | Queue Service (direct) |

---

### Option 2 — Run Services Locally (Development Mode)

Use this during development — databases in Docker, services running locally for hot reload.

**Step 1 — Start databases and Eureka only**

```bash
docker compose up postgres-user postgres-inventory mongodb eureka-server
```

**Step 2 — Run each service in a separate terminal**

```bash
# Terminal 1
cd user-service
./mvnw spring-boot:run

# Terminal 2
cd inventory-service
./mvnw spring-boot:run

# Terminal 3
cd queue-service
./mvnw spring-boot:run

# Terminal 4
cd api-gateway
./mvnw spring-boot:run
```

---

## API Endpoints

All requests go through the API Gateway at `http://localhost:8080`.

### User Service — `/api/users`

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/api/users/register` | Register new user | No |
| POST | `/api/users/login` | Login, returns JWT | No |
| GET | `/api/users/{id}` | Get user by ID | Yes |
| PUT | `/api/users/{id}` | Update user profile | Yes |
| GET | `/api/users/role/{role}` | Get users by role | ADMIN |
| DELETE | `/api/users/{id}` | Delete user | ADMIN |

### Inventory Service — `/api/inventory`

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/api/inventory` | Add dealer inventory | DEALER |
| GET | `/api/inventory/{id}` | Get inventory by ID | Yes |
| GET | `/api/inventory/dealer/{dealerId}` | Get dealer stock | Yes |
| GET | `/api/inventory/available` | Get all dealers with stock | Yes |
| PUT | `/api/inventory/{id}/stock` | Update stock level | DEALER |
| GET | `/api/inventory/location/{location}` | Get stock by location | Yes |

### Queue Service — `/api/queue`

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/api/queue/token` | Claim a token | CITIZEN |
| GET | `/api/queue/{dealerId}/status` | Get queue status | Yes |
| GET | `/api/queue/token/{tokenId}` | Get token details | Yes |
| PUT | `/api/queue/{dealerId}/advance` | Advance queue | DEALER |
| DELETE | `/api/queue/token/{tokenId}` | Cancel token | CITIZEN |
| GET | `/api/queue/citizen/{citizenId}` | Get citizen active token | Yes |

---

## User Roles

| Role | Description | Key Permissions |
|---|---|---|
| `CITIZEN` | Registered public user | Claim tokens, check stock |
| `DEALER` | LPG dealer / distributor | Update stock, advance queue |
| `ADMIN` | Government / system admin | Full access, reporting |

---

## Inter-Service Communication

Services communicate via OpenFeign through the Eureka service registry:

```
Queue Service ──► Inventory Service   (verify stock before issuing token)
Queue Service ──► User Service        (verify citizen identity)
Inventory Service ──► User Service    (verify dealer role before stock update)
```

No hardcoded URLs — Eureka resolves service locations dynamically.

---

## Docker Network

All containers run on a shared bridge network `lpg-network`:

```bash
# Inspect the network
docker network inspect lpg-network

# Check running containers
docker ps
```

---

## Useful Commands

```bash
# Start all services
docker compose up

# Start in background
docker compose up -d

# Stop all services (data preserved)
docker compose down

# Stop and remove all data (fresh start)
docker compose down -v

# View logs for a specific service
docker compose logs user-service

# Restart a single service
docker compose restart inventory-service

# Rebuild a single service after code change
docker compose up --build user-service
```

---

