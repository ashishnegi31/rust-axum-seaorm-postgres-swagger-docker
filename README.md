<p align="center">
  <img src="https://img.shields.io/github/license/asepindrak/rust-axum-seaorm-postgres-swagger-docker" />
  <img src="https://img.shields.io/badge/Rust-1.80+-orange?logo=rust" />
  <img src="https://img.shields.io/badge/Axum-0.8-blue?logo=rust" />
  <img src="https://img.shields.io/badge/SeaORM-1.1-green" />
  <img src="https://img.shields.io/badge/PostgreSQL-17-blue?logo=postgresql" />
  <img src="https://img.shields.io/badge/Docker-ready-0db7ed?logo=docker" />
</p>

---


# Rust Axum + SeaORM + PostgreSQL + PgAdmin + Swagger + Docker

A fully containerized boilerplate for building a scalable **Rust backend** using:

- **Axum 0.8** – async web framework  
- **SeaORM** – async ORM  
- **PostgreSQL 17**  
- **PgAdmin 4** – GUI database management  
- **Swagger UI + Utoipa** – OpenAPI documentation  
- **Docker Compose** – development environment with hot-reload  

---

## 📁 Repository

Source code repository:  
**https://github.com/asepindrak/rust-axum-seaorm-postgres-swagger-docker**

Clone the project:
```bash
git clone https://github.com/asepindrak/rust-axum-seaorm-postgres-swagger-docker.git
```

---

## 🚀 Features

- Layered architecture:  
  `routes → services → repositories → entities`
- Hot reload using `cargo watch`
- Auto OpenAPI generation via `utoipa`
- Built‑in Swagger UI at: **http://localhost:3000/swagger**
- PgAdmin at: **http://localhost:8080**
- PostgreSQL volume persistence
- Environment‑based configuration
- Fully dockerized setup

---

## 🏗️ Architecture Overview
              ┌───────────────────────────────┐
              │            Client              │
              └───────────────┬───────────────┘
                              │ HTTP (REST)
                              ▼
                ┌───────────────────────────┐
                │           Axum            │
                │        (Routes Layer)     │
                └───────────────┬──────────┘
                                │ calls
                                ▼
                ┌───────────────────────────┐
                │         Services          │
                │  (Business Logic Layer)   │
                └───────────────┬──────────┘
                                │ calls
                                ▼
                ┌───────────────────────────┐
                │       Repositories        │
                │        (Data Access)      │
                └───────────────┬──────────┘
                                │ SeaORM
                                ▼
                   ┌───────────────────────┐
                   │     PostgreSQL 17     │
                   └───────────────────────┘

            ┌────────────────────────────────────┐
            │            PgAdmin 4               │
            │ (Inspect database / backup / UI)   │
            └────────────────────────────────────┘



---

## 📁 Project Structure

```
rust-axum-seaorm-postgres-swagger-docker/
│
├── src/
│   ├── app_state.rs
│   ├── main.rs
│   ├── errors.rs
│   ├── openapi.rs
│   │
│   ├── routes/
│   │   ├── mod.rs
│   │   ├── health.rs
│   │   └── users.rs
│   │
│   ├── services/
│   │   ├── mod.rs
│   │   └── user_service.rs
│   │
│   ├── repositories/
│   │   ├── mod.rs
│   │   └── sea_user_repo.rs
│   │
│   ├── dto/
│   │   ├── mod.rs
│   │   └── user_dto.rs
│   │
│   └── entities/
│       ├── mod.rs
│       └── user.rs
│
├── migrations/
│   └── init.sql
│
├── pgadmin_storage/
├── Dockerfile.dev
├── Dockerfile
├── docker-compose.yml
├── Cargo.toml
└── README.md
```

---

## ⚙️ Environment Variables (`.env`)

```
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=axum_seaorm
POSTGRES_PORT=5435

DB_HOST=db
DATABASE_URL=postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${DB_HOST}:5432/${POSTGRES_DB}

RUST_LOG=axum_seaorm=debug,tower_http=debug

APP_PORT=3000

PGADMIN_EMAIL=admin@example.com
PGADMIN_PASSWORD=admin123
PGADMIN_PORT=8080

PGADMIN_EMAIL_ESCAPED=admin_example.com
```

---

## 🐳 Docker Compose (`docker-compose.yml`)

```
version: "3.8"

services:
  db:
    image: postgres:17
    container_name: axum-seaorm-db
    env_file:
      - ./.env
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "${POSTGRES_PORT}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./migrations/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5

  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: axum-seaorm-app
    ports:
      - "${APP_PORT}:3000"
    depends_on:
      db:
        condition: service_healthy
    env_file:
      - ./.env
    environment:
      DATABASE_URL: ${DATABASE_URL}
      RUST_LOG: ${RUST_LOG}
    volumes:
      - ./src:/app/src:ro
      - ./Cargo.toml:/app/Cargo.toml:ro
      - cargo_cache:/usr/local/cargo/registry
      - target_cache:/app/target
  
  pgadmin:
    image: dpage/pgadmin4:7
    container_name: axum-seaorm-pgadmin
    restart: unless-stopped
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_EMAIL:-admin@example.com}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD:-admin123}
    ports:
      - "${PGADMIN_PORT:-8080}:80"
    depends_on:
      - db

volumes:
  postgres_data:
  cargo_cache:
  target_cache:

```

---

## ▶️ Run the Project

### Start everything
```
docker compose up --build
```

### Swagger UI
👉 http://localhost:3000/swagger

### PgAdmin
👉 http://localhost:8080  
Login:
- Email: from `.env`
- Password: from `.env`

### Register PostgreSQL server in PgAdmin

```
Host: db
Port: 5432
Username: postgres
Password: postgres
```

---

## 📸 Swagger Screenshot

<img src="https://gcdnb.pbrd.co/images/NWzKLuHTrGHO.png" alt="Swagger Screenshot" style="max-width:100%; border:1px solid #ddd; border-radius:8px;" />

---
## 🚀 Production Build (Optimized Docker Image)

A separate production-ready Dockerfile is included.

### Build production image:
```bash
docker build -f Dockerfile -t axum-api .
```
```bash
docker run -p 3000:3000 --env-file .env axum-api
```

---
## 🧹 Clean Everything

```
docker compose down -v
```

---

## 📝 License
MIT
