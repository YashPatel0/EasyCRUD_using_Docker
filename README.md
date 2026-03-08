# 🚀 EasyCRUD – Three Tier Application Deployment using Docker on AWS EC2

This project demonstrates how to deploy a **Three-Tier Web Application** using **Docker and Docker Compose** on an **AWS EC2 instance**.

The application consists of:

* **Frontend** – Built with Node/Vite and served using Apache HTTPD
* **Backend** – Spring Boot REST API
* **Database** – MariaDB

All services are containerized and orchestrated using **Docker Compose**.

---

# 🏗️ Architecture

User → EC2 Instance → Docker Containers

```
Frontend (Apache HTTPD - Port 80)
        ↓
Backend (Spring Boot - Port 8080)
        ↓
Database (MariaDB - Port 3306)
```

---

# ⚙️ Prerequisites

Before deployment ensure you have:

* AWS Account
* EC2 Instance (Ubuntu)
* Security group allowing ports:

| Port | Purpose     |
| ---- | ----------- |
| 22   | SSH         |
| 80   | Frontend    |
| 8080 | Backend API |
| 3306 | Database    |

---

# ☁️ Step 1: Launch EC2 Instance

1. Launch an **Ubuntu EC2 instance**
2. Connect using SSH

```
ssh -i key.pem ubuntu@<EC2_PUBLIC_IP>
```

---

# 🐳 Step 2: Install Docker

Update packages

```
sudo apt update
```

Install dependencies

```
sudo apt install ca-certificates curl
```

Add Docker’s official GPG key

```
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
-o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add Docker repository

```
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Update packages again

```
sudo apt update
```

Install Docker Engine

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verify installation

```
docker --version
docker compose version
```

---

# 📥 Step 3: Clone Project

Clone the repository from GitHub.

```
git clone https://github.com/<name>
cd EasyCRUD
```

---

# 🧩 Step 4: Backend Dockerfile

Create a **Dockerfile inside the backend directory**

```
backend/Dockerfile
```

```
FROM maven:3.8.3-openjdk-17 AS build

WORKDIR /opt

COPY . .

RUN mvn clean package -DskipTests

FROM openjdk:17.0.2-jdk

COPY --from=build /opt/target/student-registration-backend-0.0.1-SNAPSHOT.jar /opt/studentapp.jar

EXPOSE 8080

CMD ["java", "-jar", "/opt/studentapp.jar"]
```

---

# ⚙️ Backend Configuration

Update `application.properties`

```
server.port=8080

spring.datasource.url=jdbc:mariadb://${DB_HOST}:3306/${DB_NAME}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

spring.jpa.database-platform=org.hibernate.dialect.MariaDBDialect
```

---

# 🎨 Step 5: Frontend Dockerfile

Create a **Dockerfile inside the frontend directory**

```
frontend/Dockerfile
```

```
FROM node:22-alpine AS build

WORKDIR /opt

COPY . .

RUN npm install && npm run build

FROM httpd:latest

COPY --from=build /opt/dist/ htdocs/

EXPOSE 80

CMD ["httpd", "-D", "FOREGROUND"]
```

---

# 🌍 Frontend Environment Configuration

Update `.env` file with your **EC2 Public IP**

```
VITE_API_URL="http://<EC2_PUBLIC_IP>:8080/api"
```

Example:

```
VITE_API_URL="http://13.201.43.34:8080/api"
```

---

# 🧱 Step 6: Docker Compose Configuration

Create a `docker-compose.yml` file in the project root.

```
services:
  database:
    image: mariadb:latest
    environment:
      MARIADB_ROOT_PASSWORD: "redhat"
      MYSQL_DATABASE: "studentapp"
    volumes:
      - "my-vol123:/var/lib/mysql"
    ports:
      - "3306:3306"
    healthcheck:
      test: ["CMD", "mariadb-admin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      DB_NAME: "studentapp"
      DB_USER: "root"
      DB_PASSWORD: "redhat"
      DB_HOST: "database"
    depends_on:
      database:
        condition: service_healthy
    restart: always

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    environment:
      PUBLIC_IP: "13.201.43.34"
    depends_on:
      - backend

volumes:
  my-vol123: {}
```

---

# ▶️ Step 7: Start Containers

Build and start all services using Docker Compose.

```
docker compose up -d --build
```

Verify running containers

```
docker ps
```

---

# 🌐 Step 8: Access Application

Once containers are running successfully, open your browser.

Frontend

```
http://<EC2_PUBLIC_IP>
```

Backend API

```
http://<EC2_PUBLIC_IP>:8080/api
```

---

# 📦 Running Containers

```
Frontend Container  → Port 80
Backend Container   → Port 8080
MariaDB Container   → Port 3306
```

---

# 📊 Useful Docker Commands

Check running containers

```
docker ps
```

View logs

```
docker logs <container_id>
```

Stop containers

```
docker compose down
```

---

# 🎯 Key Features

✔ Three-tier architecture
✔ Dockerized frontend, backend, and database
✔ Automated container orchestration using Docker Compose
✔ Persistent database storage using Docker volumes
✔ Health check for database container

---

# 👨‍💻 Author

**Yash Patel**

DevOps | Cloud | Docker | AWS | Kubernetes


