
📦 **Inventory Management System**

A full-stack inventory management application built with Spring Boot, MySQL, and React.js. It allows businesses to manage suppliers, products, and stock levels efficiently with features like CRUD operations, real-time updates, and a sell function to track transactions.

---

🚀 **Features**

🔹 **Secure Authentication** - Email registration/login with BCrypt password hashing and JWT sessions.

🔹 **OTP Verification** - 6-digit email OTP verification before first login.

🔹 **Protected APIs** - Product, supplier, sales, and order APIs require Bearer token authentication.

🔹 **Supplier Management** – Add, update, delete, and view suppliers.

🔹 **Product Management** – Manage products with supplier linkage.

🔹 **Sell Functionality** – Reduce stock automatically when a sale is made.

🔹 **Validation & Error Handling** – Prevent deletion of suppliers with linked products.

🔹 **Frontend-Backend Integration** – REST APIs consumed via React frontend.

🔹 **Database Persistence** – MySQL used for relational data storage.

---

🛠️ **Tech Stack**

**Backend:** Java, Spring Boot, Spring Data JPA, Spring Security, JWT, Java Mail, Flyway  
**Frontend:** React.js, TypeScript, Tailwind CSS, Framer Motion  
**Database:** MySQL  
**Build Tools:** Maven, npm  
**Other:** Git, Postman (for testing APIs)

---

⚙️ **Installation & Setup**

1️⃣ **Clone the Repository**
```powershell
git clone https://github.com/Anjan-Kumar-Sahoo/GoDamm.git
cd GoDamm
```

# Godamm Inventory Management System

Production-ready full-stack inventory platform with secure auth, OTP workflows, tenant-safe data isolation, Redis caching, and Linux VM / Vercel deployment support.

## Live Domains

- Frontend: https://godamm.mraks.dev
- Frontend (secondary): https://godamm.anjaliv.dev
- Backend API: https://api.godamm.mraks.dev

## Architecture

```mermaid
flowchart LR
		U[User Browser] --> V[Vercel Frontend\nReact + Vite]
		V -->|HTTPS /api + /auth| N[Nginx on Linux Server]
		N -->|Reverse proxy| B[Spring Boot Backend]
		B --> M[(MySQL)]
		B --> R[(Redis)]
		B --> S[SMTP Provider]
		G[GitHub Actions] --> D[Docker Hub]
		G --> E[Linux Server Deploy via SSH]
		E --> N
		E --> B
```

## Stack

- Backend: Java 17, Spring Boot 3, Spring Security, JWT, Flyway, Redis Cache, Actuator
- Frontend: React 18, TypeScript, Vite, Tailwind CSS, Framer Motion
- Data: MySQL 8, Redis 7
- DevOps: Docker, Docker Compose, GitHub Actions, Nginx, Certbot, Linux VM (Azure, AWS, GCP, etc.)

## Repository Structure

```text
backend/                    Spring Boot application
frontend/                   React + Vite application
.github/workflows/          CI/CD workflows
docker-compose.yml          Backend + MySQL + Redis stack
.env.example                Environment template
```

## Environment Variables

Copy `.env.example` to `.env` and fill real values.

### Required backend variables

- DB_URL
- DB_USERNAME
- DB_PASSWORD
- JWT_SECRET
- REDIS_HOST
- REDIS_PORT
- REDIS_PASSWORD
- MAIL_HOST
- MAIL_PORT
- MAIL_USERNAME
- MAIL_PASSWORD
- CORS_ALLOWED_ORIGINS

Mail notes:
- Spring Boot uses Java Mail over SMTP, so `MAIL_HOST` and `MAIL_PORT` are both required.
- Common provider ports are `587` (STARTTLS) and `465` (SSL/TLS).

### Required frontend variable (Vercel)

- VITE_API_BASE_URL=https://api.godamm.mraks.dev
- VITE_API_URL=https://api.godamm.mraks.dev (supported alias)

## Local Development

### Option A: Full stack with Docker (recommended)

```bash
cp .env.example .env
docker compose up -d --build
```

Backend health:

```bash
curl -fsS http://localhost:8080/actuator/health
```

### Option B: Backend and frontend separately

Backend:

```bash
cd backend
mvn spring-boot:run
```

Frontend:

```bash
cd frontend
npm install
npm run dev
```

## Dockerization

- Backend Dockerfile uses multi-stage build
	- Build stage: Maven + JDK 17
	- Runtime stage: openjdk:17-jdk-slim
- Health check endpoint: /actuator/health
- Compose stack includes:
	- backend
	- mysql (persistent volume)
	- redis (password-protected)

## CI/CD & Deployment

The application uses an automated continuous deployment pipeline powered by GitHub Actions, Docker Hub, and Docker Compose on an Azure Linux VM.

```text
GitHub (push to main)
        ↓
GitHub Actions (build & test)
        ↓
Build & Push Docker Image (aksahoo1097/godamm-backend:latest)
        ↓
SSH into Azure VM
        ↓
docker compose pull backend
        ↓
docker compose up -d backend
```

### Pipeline Steps (`.github/workflows/backend.yml`)

1. **Build & Test:** Compiles Java 17 and runs Maven unit and integration tests (`mvn clean verify`).
2. **Build & Push Image:** Builds the production Docker image and pushes `latest` and commit SHA tags to Docker Hub.
3. **SSH Deploy:** Connects to the Azure VM and executes:
   ```bash
   cd ~/apps/GoDamm
   docker compose pull backend
   docker compose up -d backend
   docker image prune -f
   ```

### Required GitHub Secrets

- `DOCKER_PASSWORD`
- `SERVER_HOST`
- `SERVER_USER`
- `SERVER_SSH_KEY`

## Security Hardening

- Stateless JWT security
- Strict env-driven CORS origin policy
- Redis-backed rate limiting
- Optional HTTPS enforcement via `REQUIRE_HTTPS=true`
- Input validation on DTOs and auth payloads
- Non-root app credentials for MySQL
- No hardcoded secrets in source

## Monitoring and Logging

Actuator endpoints:

- `/actuator/health`
- `/actuator/metrics`

Logging:

- Console logs
- File logs (`logs/inventory-app.log`)
- Structured JSON-like log pattern

## Performance Notes

- Redis cache for product/supplier/profit reads
- Cache eviction on writes
- HikariCP pool tuning via env vars
- 4 GB deployment profile tuned for production usage

## Testing

Run backend tests:

```bash
cd backend
mvn clean verify
```

CI fails on test/build failures by default.
