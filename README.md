# 🚀 ROBOSHOP MICROSERVICES – DOCKER

---

# 📘 TABLE OF CONTENTS

1. Architecture Overview  
2. High-Level Architecture Diagram  
3. Network Diagram  
4. Volume Architecture  
5. Microservice Communication Flow  
6. Building Docker Images  
7. Running Containers Manually  
8. Docker Compose Explanation  
9. Stateful vs Stateless  
10. Multi-Stage Builds  
11. Docker Optimization Best Practices  
12. Testing Guide  
13. Troubleshooting Guide  
14. Full docker-compose.yml (Explained)  

---

# 1️⃣ ARCHITECTURE OVERVIEW

Roboshop is a multi-container microservices application that uses:

- **Frontend** (Node.js / Nginx)
- **Backend Services**:
  - Catalogue
  - User
  - Cart
  - Shipping
  - Payment
- **Databases**:
  - MongoDB
  - Redis
  - MySQL
  - RabbitMQ

All containers run in a **single Docker network** and communicate internally.

---

# 2️⃣ HIGH-LEVEL ARCHITECTURE DIAGRAM

```
HOST MACHINE (EC2 / VM / Laptop)
│
├── Docker Engine
│
└── Network: roboshop (bridge)
      ├── frontend (port 80)
      ├── catalogue
      ├── user
      ├── cart
      ├── shipping
      ├── payment
      ├── mongodb (volume)
      ├── redis   (volume)
      ├── mysql   (volume)
      └── rabbitmq (volume)
```

---

# 3️⃣ NETWORK DIAGRAM

```
frontend → catalogue
frontend → user
frontend → cart
frontend → shipping
frontend → payment

catalogue → mongodb
user → mongodb
user → redis
cart → redis
cart → catalogue
shipping → mysql
shipping → cart
payment → rabbitmq
payment → cart
payment → user
```

---

# 4️⃣ VOLUME ARCHITECTURE (HOST ↔ CONTAINER)

| Service  | Host Path | Container Path |
|----------|-----------|----------------|
| MongoDB  | /var/lib/docker/volumes/mongodb/_data | /data/db |
| Redis    | /var/lib/docker/volumes/redis/_data   | /data |
| MySQL    | /var/lib/docker/volumes/mysql/_data   | /var/lib/mysql |
| RabbitMQ | /var/lib/docker/volumes/rabbitmq/_data | /var/lib/rabbitmq |

---

# 5️⃣ MICROSERVICE COMMUNICATION FLOW

```
frontend ───▶ catalogue ───▶ mongodb
frontend ───▶ user ─────────▶ mongodb
frontend ───▶ user ─────────▶ redis
frontend ───▶ cart ─────────▶ redis
frontend ───▶ cart ─────────▶ catalogue
frontend ───▶ shipping ─────▶ mysql
frontend ───▶ shipping ─────▶ cart
frontend ───▶ payment ──────▶ rabbitmq
frontend ───▶ payment ──────▶ user
frontend ───▶ payment ──────▶ cart
```

---

# 6️⃣ BUILDING ALL DOCKER IMAGES

Run this automation loop:

```bash
for i in mongodb mysql catalogue user cart shipping payment redis rabbitmq frontend; do
  cd $i
  docker build -t $i:v1 .
  cd ..
done
```

---

# 7️⃣ RUNNING CONTAINERS MANUALLY (WITHOUT COMPOSE)

### Step 1 — Create network
```
docker network create roboshop
```

### Step 2 — Run frontend container example
```
docker run -d -p 80:80 --name frontend --network roboshop frontend:v1
```

---

# 8️⃣ DOCKER COMPOSE — FULL EXPLANATION

Docker Compose allows:

- Bringing all containers up/down at once  
- Define networks  
- Define volumes  
- Manage service dependencies  
- Maintain startup order  


build all services:
```
docker compose build
```

Build specific services (optional):
```
docker compose build <service_name>
```

Force rebuilding:
```
docker compose build --no-cache
```

Start all services:
```
docker compose up -d
```

Stop services:
```
docker compose down
```

---

# 9️⃣ STATEFUL VS STATELESS CONTAINERS

| Type | Services |
|------|----------|
| **Stateful (need volumes)** | MongoDB, Redis, MySQL, RabbitMQ |
| **Stateless (only code)** | Catalogue, User, Cart, Shipping, Payment, Frontend |

---

# 🔟 MULTI-STAGE BUILDS

Used to:

- Reduce image size  
- Remove build dependencies  
- Improve security  
- Speed up deployments  

Example:
- Stage 1: Install dependencies & build app  
- Stage 2: Copy only required files into final image  

---

# 1️⃣1️⃣ DOCKER IMAGE OPTIMIZATION BEST PRACTICES

✔ Use small base images (Alpine)  

✔ Use multi-stage builds

✔ use volumes and custom networks

✔ Use labels and expose 

✔ Optimize layers 

✔ Use `.dockerignore`  

✔ Don’t store secrets inside images 

✔ use entrypoint and cmd 

✔ Avoid root user 

✔ Use volumes for persistent storage  

✔ Limit the resources and perform health checks

✔  Optimise layering:

	✔  Reduce number of layers, frequently changing instruction should be at last


	✔ Combine multiple instructions into single instruction, that speeds up the build prcess
---



# 1️⃣2️⃣ TESTING GUIDE

### Test frontend:
```
curl http://<EC2-IP>
```

### Test backend API:
```
curl http://<EC2-IP>/catalogue
curl http://<EC2-IP>/user
curl http://<EC2-IP>/cart
```

### Check logs:
```
docker logs <container> -f
```

---

# 1️⃣3️⃣ TROUBLESHOOTING

### Container restarting?
```
docker logs <container>
```

### Port already in use?
```
sudo lsof -i :80
```

### Volume not persisting?
```
docker volume inspect mongodb
```

### Network issues?
```
docker network inspect roboshop
```

### Image build failed?
```
docker build --no-cache .
```

---

# 🚀 How to Deploy

```
docker compose up -d
docker compose ps
docker network inspect roboshop
docker volume ls
```

---

# 🧪 How to Test

```
curl http://<EC2-IP>
curl http://<EC2-IP>/catalogue
curl http://<EC2-IP>/user
```

---

# 🛠 Logs

```
docker logs frontend -f
docker logs catalogue -f
docker logs user -f
```

---

# 🚑 Troubleshooting

### ❌ Container restarts?
```
docker logs <container>
```

### ❌ Port already in use?
```
sudo lsof -i :80
```

### ❌ Volume not persisting?
```
docker volume inspect mongodb
```
---------------------------------------------------------------------
# 🔥  DOCKER COMPOSE EXPLANATION 
---------------------------------------------------------------------
# 🧾 docker-compose.yml — Fully Explained

```yaml
services:                                # Root section for all microservices (containers)

  # ------------------- CATALOGUE SERVICE -------------------
  catalogue:
    image: catalogue:v1                  # Docker image name & tag to run for catalogue
    container_name: catalogue            # Name assigned to the container
    depends_on:                          # Ensures dependency startup order
    - mongodb                            # catalogue must start AFTER MongoDB

  # ------------------- MONGODB SERVICE ---------------------
  mongodb:
    image: mongodb:v1                    # MongoDB image provided by you
    container_name: mongodb              # Custom container name
    volumes:                             # Attach persistent storage
    - mongodb:/data/db                   # HOST volume (mongodb) → container /data/db

  # ------------------- REDIS SERVICE -----------------------
  redis:
    image: redis:7.0                     # Official Redis 7.0 image
    container_name: redis                # Container name for Redis
    volumes:
    - redis:/data                        # Persisted Redis data storage

  # ------------------- MYSQL SERVICE -----------------------
  mysql:
    image: mysql:v1                      # MySQL image
    container_name: mysql                # Container name
    volumes:
    - mysql:/var/lib/mysql               # HOST volume → MySQL database directory

  # ------------------- RABBITMQ SERVICE --------------------
  rabbitmq:
    image: rabbitmq:3                    # RabbitMQ broker
    container_name: rabbitmq             # Container name
    volumes:
    - rabbitmq:/var/lib/rabbitmq         # Persistent broker queues storage
    environment:                         # RabbitMQ default credentials
      RABBITMQ_DEFAULT_USER: roboshop    # Username injected into container
      RABBITMQ_DEFAULT_PASS: roboshop123 # Password injected into container

  # ------------------- USER SERVICE ------------------------
  user:
    image: user:v1                       # User microservice image
    container_name: user                 # Container name
    depends_on:                          # Required services
    - mongodb                            # User stores data in MongoDB
    - redis                              # User interacts with Redis cache

  # ------------------- CART SERVICE ------------------------
  cart:
    image: cart:v1                       # Cart microservice image
    container_name: cart                 # Container name
    depends_on:
    - redis                              # Cart uses Redis cache
    - catalogue                          # Cart queries catalogue data

  # ------------------- SHIPPING SERVICE --------------------
  shipping:
    image: shipping:v1                   # Shipping microservice image
    container_name: shipping             # Container name
    depends_on:
    - mysql                              # Shipping data stored in MySQL
    - cart                               # Shipping needs cart info

  # ------------------- PAYMENT SERVICE ---------------------
  payment:
    image: payment:v1                    # Payment microservice image
    container_name: payment              # Container name
    depends_on:
    - rabbitmq                           # Payment uses message queue
    - cart                               # Payment validates cart
    - user                               # Payment validates user

  # ------------------- FRONTEND SERVICE --------------------
  frontend:
    image: frontend:v1                   # UI microservice image
    container_name: frontend             # Container name
    ports:
    - "80:80"                            # HOST:80 → CONTAINER:80 (public access)
    depends_on:
    - catalogue                          # Frontend calls catalogue APIs
    - user                               # Calls user APIs
    - cart                               # Calls cart APIs
    - shipping                           # Calls shipping APIs
    - payment                            # Calls payment APIs

# -------------------- NETWORK CONFIGURATION --------------------
networks:
  default:                               # Default network block
    driver: bridge                       # Network type = bridge
    name: roboshop                       # Network name
    external: false                      # Docker Compose auto-creates the network

# -------------------- VOLUME DECLARATIONS ---------------------

volumes:
  mongodb:                               # Host volume 1
  redis:                                 # Host volume 2
  mysql:                                 # Host volume 3
  rabbitmq:                              # Host volume 4
```

---
