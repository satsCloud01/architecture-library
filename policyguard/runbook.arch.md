---
name: "Deployment Runbook"
project: "PolicyGuard"
project_slug: "policyguard"
project_url: "https://policyguard.satszone.link"
github: "https://github.com/satsCloud01/policyguard"
category: "architecture-devops"
type: "runbook"
icon: "🔐"
tags: [Docker, Nginx, AWS]
---

# Deployment Runbook

> Step-by-step guide to deploying PolicyGuard on an AWS EC2 t2.micro instance with Nginx reverse proxy and Let's Encrypt SSL.

---

## Prerequisites

- AWS EC2 t2.micro, Ubuntu 22.04 LTS, us-east-1
- Elastic IP or static public IP
- Route 53 A record: `policyguard.satszone.link` → EC2 public IP
- SSH key pair: `policyguard-key.pem`
- Domain registrar: `satszone.link` hosted zone in Route 53

---

## 1. Connect to the Instance

```bash
ssh -i ~/.ssh/policyguard-key.pem ubuntu@<EC2_PUBLIC_IP>
```

---

## 2. System Setup

```bash
# Update packages
sudo apt-get update && sudo apt-get upgrade -y

# Install Python 3.12, pip, venv, nginx, certbot
sudo apt-get install -y python3.12 python3.12-venv python3-pip nginx certbot python3-certbot-nginx git curl

# Install Node.js 20 LTS (for frontend build)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

---

## 3. Clone Repository

```bash
cd /opt
sudo git clone https://github.com/satsCloud01/policyguard.git
sudo chown -R ubuntu:ubuntu /opt/policyguard
cd /opt/policyguard
```

---

## 4. Backend Setup

```bash
cd /opt/policyguard/backend

# Create virtual environment
python3.12 -m venv .venv

# Activate and install dependencies
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt

# Create environment file
cat > .env << 'EOF'
DATABASE_URL=sqlite+aiosqlite:///./policyguard.db
EOF

# Verify startup (seeds DB on first run)
PYTHONPATH=src .venv/bin/uvicorn policyguard.main:app --host 127.0.0.1 --port 8003 --workers 1
# Ctrl+C after confirming startup
```

---

## 5. Frontend Build

```bash
cd /opt/policyguard/frontend

# Install dependencies
npm install

# Build production bundle
npm run build
# Output: frontend/dist/
```

---

## 6. Systemd Service

Create the service file:

```bash
sudo nano /etc/systemd/system/policyguard.service
```

Paste:

```ini
[Unit]
Description=PolicyGuard - Dynamic Policy Access Engine
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/opt/policyguard/backend
Environment="PYTHONPATH=src"
ExecStart=/opt/policyguard/backend/.venv/bin/uvicorn policyguard.main:app --host 127.0.0.1 --port 8003 --workers 1
Restart=always
RestartSec=5
StandardOutput=append:/var/log/policyguard/app.log
StandardError=append:/var/log/policyguard/error.log

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo mkdir -p /var/log/policyguard
sudo chown ubuntu:ubuntu /var/log/policyguard
sudo systemctl daemon-reload
sudo systemctl enable policyguard
sudo systemctl start policyguard
sudo systemctl status policyguard
```

---

## 7. Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/policyguard
```

Paste:

```nginx
server {
    listen 80;
    server_name policyguard.satszone.link;

    # Redirect HTTP to HTTPS (Certbot will add this automatically)
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name policyguard.satszone.link;

    # SSL certificates (filled by Certbot)
    # ssl_certificate /etc/letsencrypt/live/policyguard.satszone.link/fullchain.pem;
    # ssl_certificate_key /etc/letsencrypt/live/policyguard.satszone.link/privkey.pem;

    # Frontend static files
    root /opt/policyguard/frontend/dist;
    index index.html;

    # SPA fallback — serve index.html for all non-API routes
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API proxy to FastAPI
    location /api/ {
        proxy_pass http://127.0.0.1:8003;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
    }

    # Gzip compression
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/policyguard /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 8. SSL Certificate (Let's Encrypt)

```bash
sudo certbot --nginx -d policyguard.satszone.link --non-interactive --agree-tos -m admin@satszone.link
```

Certbot automatically:
- Generates certificates in `/etc/letsencrypt/live/policyguard.satszone.link/`
- Updates the Nginx config with SSL directives
- Reloads Nginx
- Creates a systemd timer for auto-renewal (runs twice daily)

Verify auto-renewal:
```bash
sudo certbot renew --dry-run
```

---

## 9. Verify Deployment

```bash
# Backend health
curl http://127.0.0.1:8003/api/dashboard/summary

# Full stack (HTTPS)
curl https://policyguard.satszone.link/api/dashboard/summary

# Frontend
curl -I https://policyguard.satszone.link/
```

Expected: HTTP 200 from all three.

---

## Service Management

```bash
# Status
sudo systemctl status policyguard

# Start / Stop / Restart
sudo systemctl start policyguard
sudo systemctl stop policyguard
sudo systemctl restart policyguard

# View logs
tail -f /var/log/policyguard/app.log
tail -f /var/log/policyguard/error.log

# Nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

---

## Updating the Deployment

```bash
cd /opt/policyguard

# Pull latest code
git pull

# Rebuild frontend
cd frontend && npm install && npm run build && cd ..

# Restart backend service
sudo systemctl restart policyguard

# Verify
sudo systemctl status policyguard
```

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | No (default: SQLite) | SQLAlchemy connection string |
| `PYTHONPATH` | Yes | Set to `src` so module imports resolve |

> The Anthropic API key is stored in the `settings` table (configured via Settings UI), not in environment variables.

---

## Rollback

If a deployment breaks, rollback with:

```bash
cd /opt/policyguard
git log --oneline -10  # find the last good commit
git checkout <commit_hash>
cd frontend && npm run build && cd ..
sudo systemctl restart policyguard
```

---

## Security Notes

- Port 8003 is bound to `127.0.0.1` only — not accessible from the public internet
- All external traffic goes through Nginx on 443
- HTTP on port 80 redirects to HTTPS
- `policyguard.db` file permissions: `ubuntu:ubuntu 600`
- EC2 security group: allow 443 (HTTPS) and 22 (SSH) inbound only

---

## Instance Details

| Property | Value |
|---|---|
| Instance type | t2.micro (1 vCPU, 1 GB RAM) |
| OS | Ubuntu 22.04 LTS |
| Region | us-east-1 |
| URL | https://policyguard.satszone.link |
| SSH Key | `~/.ssh/policyguard-key.pem` |
| Backend port | 8003 (internal only) |
| Service name | `policyguard` |
