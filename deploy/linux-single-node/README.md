# Linux Server Single-Node Deployment (4 GB)

This profile deploys the backend stack on one Linux server (Azure, AWS, GCP, Oracle Cloud, DigitalOcean, Hetzner, etc.) using Docker Compose:
- Spring Boot backend container
- MySQL container
- Redis container
- Nginx reverse proxy (HTTPS)

Frontend is hosted on Vercel:
- https://godamm.mraks.dev
- https://godamm.anjaliv.dev

Backend API domain:
- https://api.godamm.mraks.dev

## 1) Launch Linux Server

- Specification: 2 vCPU, 4 GB RAM minimum (e.g. Azure Standard_B2s, AWS t3.medium, GCP e2-medium, DigitalOcean 4GB)
- OS: Ubuntu 22.04 LTS or 24.04 LTS
- Storage: 20 GB minimum (30 GB recommended)
- Security Group / Firewall inbound: Ports 22 (SSH), 80 (HTTP), 443 (HTTPS), 8080 (Backend API)

## 2) Bootstrap host

On the remote server:

```bash
sudo bash deploy/linux-single-node/setup-server.sh
```

What it does:
- Installs Docker, Docker Compose plugin, Nginx, Certbot, and UFW
- Creates 4 GB swap and memory sysctl tuning
- Opens ports 22/80/443/8080 in UFW
- Clones or updates repository at /home/ubuntu/apps/GoDamm
- Copies .env.example to .env if missing

## 3) Configure environment

```bash
cd ~/apps/GoDamm
cp deploy/linux-single-node/backend.env.example .env
nano .env
```

Set production values at minimum:
- JWT_SECRET
- DB_URL
- DB_USERNAME
- DB_PASSWORD
- REDIS_PASSWORD
- MAIL_HOST
- MAIL_PORT
- MAIL_USERNAME
- MAIL_PASSWORD

Mail notes:
- Java Mail in Spring Boot sends OTP over SMTP, so `MAIL_HOST` and `MAIL_PORT` are required.
- Typical SMTP ports are `587` (STARTTLS) and `465` (SSL/TLS).

If using external managed MySQL (RDS, Azure Database for MySQL, Cloud SQL):
- Set DB_URL to your database endpoint, for example:
	- jdbc:mysql://<database-endpoint>:3306/godamm?useSSL=true&requireSSL=true&serverTimezone=UTC
- Set DB_USERNAME and DB_PASSWORD to your credentials.
- In network/firewall security rules, allow MySQL port 3306 from your Linux server IP.
- You can keep MYSQL_* values unchanged because backend will use DB_* values.

## 4) Start backend stack

```bash
cd ~/apps/GoDamm
bash deploy/linux-single-node/deploy-app.sh
```

This pulls the configured backend image tag (for example a commit SHA) and starts mysql, redis, and backend containers.

Deployment behavior:
- If DB_URL points to mysql container (local mode), deploy starts mysql + redis + backend.
- If DB_URL points to external host (managed DB mode), deploy starts redis + backend and skips mysql container.

## 5) Configure Nginx

```bash
sudo cp deploy/linux-single-node/nginx-inventory.conf /etc/nginx/sites-available/godamm-api
sudo ln -sf /etc/nginx/sites-available/godamm-api /etc/nginx/sites-enabled/godamm-api
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

## 6) Enable HTTPS with Let's Encrypt

```bash
sudo certbot --nginx -d api.godamm.mraks.dev
```

## 7) Optional auto-start with systemd

```bash
sudo cp deploy/linux-single-node/godamm-stack.service /etc/systemd/system/godamm-stack.service
sudo systemctl daemon-reload
sudo systemctl enable --now godamm-stack
```

## 8) Health checks

```bash
curl -fsS http://127.0.0.1:8080/actuator/health
curl -I https://api.godamm.mraks.dev/actuator/health
docker compose -f ~/apps/GoDamm/docker-compose.yml ps
free -h
```

## 9) 4 GB operations guardrails

- Keep backend heap around 1.0-1.2 GB
- Keep MySQL buffer pool around 512 MB
- Keep Redis maxmemory around 256 MB
- Keep swap at 4 GB as emergency headroom
- Upgrade to 8 GB if memory stays above 85% for sustained periods
