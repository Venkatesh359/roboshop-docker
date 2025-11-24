# 📦 Docker Volumes – Complete Guide 

Docker volumes are used to **persist data** even when containers are stopped or removed.  

This README provides **clear explanations, real-world examples, and all volume types**.

---

# 🧱 1. Types of Docker Volumes

Docker supports **three types of volumes**, each used for different workloads.

---

## 🔹 **A. Named Volumes (Most Common)**

### ✔️ Description  
Volumes created and fully managed by Docker.  
Reliable, safe, and best for production.

### ✔️ Create a named volume
```bash
docker volume create mydata
```

### ✔️ Use in a container
```bash
docker run -d -v mydata:/var/lib/mysql mysql
```

### ✔️ Host location
```
/var/lib/docker/volumes/mydata/_data
```

### ⭐ Best for:
- Databases (MySQL, MongoDB, Redis)
- Long‑term persistent data  

---

## 🔹 **B. Anonymous Volumes**

### ✔️ Description  
Docker automatically creates a volume with a random name if no name is specified.

### Example
```bash
docker run -v /data/db mongo
```

Anonymous volume path example:
```
/var/lib/docker/volumes/asdj23hdsjk3/_data
```

### ⚠️ Issues:
- Difficult to track  
- Hard to clean up  
- Often left behind  

### ⭐ Best for:
- Temporary containers  
- Fast testing  

---

## 🔹 **C. Bind Mounts**

### ✔️ Description  
Mount a **specific host directory** into the container.

### Example
```bash
docker run -v /home/venkatesh/app:/usr/src/app node
```

### ⭐ Best for:
- Local development  
- Real-time code sync  
- Log sharing  
- Debugging  

---

# 📊 2. Quick Comparison of Volume Types

| Type | Managed By | Host Path | Best Use-Case |
|------|------------|-----------|---------------|
| **Named Volume** | Docker | `/var/lib/docker/volumes/...` | Databases, Prod data |
| **Anonymous Volume** | Docker | Random ID | Temporary testing |
| **Bind Mount** | User | Any path | Development, Logs, Config |

---

# 🔍 3. List All Docker Volumes

```bash
docker volume ls
```

### Example Output
```
DRIVER    VOLUME NAME
local     mongodb_data
local     mysql_data
local     mydata
```

---

# 🔎 4. Inspect a Volume

```bash
docker volume inspect mongodb_data
```

### Important fields:
```
"Mountpoint": "/var/lib/docker/volumes/mongodb_data/_data",
"Driver": "local",
"Name": "mongodb_data"
```

---

# 📁 5. View Volume Data on the Host (Linux / EC2)

### Volume root directory
```
/var/lib/docker/volumes/
```

### List volumes
```bash
sudo ls /var/lib/docker/volumes/
```

### View data inside a volume
```bash
sudo ls /var/lib/docker/volumes/mongodb_data/_data
```

---

# 🛠️ 6. Check Volumes Used by a Container

```bash
docker inspect <container-name>
```

Look under **Mounts**:

```json
"Mounts": [
  {
    "Type": "volume",
    "Source": "mongodb_data",
    "Destination": "/data/db"
  }
]
```

---

# 🧪 7. Practical Example: MongoDB With Persistent Volume

```bash
docker run -d   --name mongodb   -p 27017:27017   -v mongodb_data:/data/db   mongo:latest
```

### What happens?
- Volume `mongodb_data` is created  
- Data stored in `/data/db` inside the container  
- Data persists even after deleting the container  

---

# 🧪 8. Delete a Docker Volume  
⚠️ Warning: This permanently deletes all stored data.

```bash
docker volume rm mongodb_data
```

---

# 🧹 9. Clean Up Unused Volumes

```bash
docker volume prune
```

---

# 🎯 Summary Table

| Task | Command |
|------|---------|
| List volumes | `docker volume ls` |
| Inspect volume | `docker volume inspect <name>` |
| View data | `/var/lib/docker/volumes/<name>/_data` |
| Check container mounts | `docker inspect <container>` |
| Create volume | `docker volume create vol1` |
| Remove volume | `docker volume rm vol1` |

---



