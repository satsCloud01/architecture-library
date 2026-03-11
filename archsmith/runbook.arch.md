---
name: "Deployment Runbook"
project: "ArchSmith"
project_slug: "archsmith"
project_url: "https://archsmith.satszone.link"
github: "https://github.com/satsCloud01/archsmith"
category: "architecture-devops"
type: "runbook"
icon: "🏗️"
tags: [Docker, Nginx, AWS]
---

# Deployment Runbook

## Local Development

### Prerequisites
- Python 3.12+
- Node.js 18+
- Git

### Quick Start
```bash
git clone https://github.com/satsCloud01/archsmith.git
cd archsmith
./start.sh
```

Open http://localhost:5175

### Manual Start
```bash
# Backend
cd backend
python3.12 -m venv .venv
.venv/bin/pip install -r requirements.txt
PYTHONPATH=src .venv/bin/uvicorn archsmith.main:app --reload --port 8002

# Frontend (new terminal)
cd frontend
npm install
npm run dev   # starts at :5175, proxies /api → :8002
```

### Environment Variables
```bash
# backend/.env (optional — AI features work via UI key entry)
ANTHROPIC_API_KEY=sk-ant-...   # Optional
```

---

## AWS EC2 Production Deployment

### Architecture
```
Internet ──HTTPS──► Nginx :443 ──static──► /home/ubuntu/archsmith/frontend/dist/
                               ──proxy──► uvicorn 127.0.0.1:8002
```

### Instance Details
| Property | Value |
|----------|-------|
| AMI | Ubuntu 22.04 LTS |
| Type | t2.micro (1 vCPU, 1GB RAM) |
| Region | us-east-1 |
| URL | https://archsmith.satszone.link |
| SSH | `ssh -i ~/.ssh/archsmith-key.pem ubuntu@<IP>` |

### Security Groups
| Rule | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | 22 | Your IP |
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

### Initial Server Setup
```bash
ssh -i ~/.ssh/archsmith-key.pem ubuntu@<IP>

# System packages
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3.12 python3.12-venv python3-pip nodejs npm nginx certbot python3-certbot-nginx git

# Clone repository
cd /home/ubuntu
git clone https://github.com/satsCloud01/archsmith.git
cd archsmith

# Backend setup
cd backend
python3.12 -m venv .venv
.venv/bin/pip install -r requirements.txt

# Frontend build
cd ../frontend
npm install
npm run build   # outputs to dist/
```

### Systemd Service
```bash
sudo tee /etc/systemd/system/archsmith.service << 'EOF'
[Unit]
Description=ArchSmith API
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/archsmith/backend
ExecStart=/home/ubuntu/archsmith/backend/.venv/bin/uvicorn archsmith.main:app --host 127.0.0.1 --port 8002
Restart=always
RestartSec=5
Environment=PYTHONPATH=src

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable archsmith
sudo systemctl start archsmith
sudo systemctl status archsmith
```

### Nginx Configuration
```bash
sudo tee /etc/nginx/sites-available/archsmith << 'EOF'
server {
    listen 80;
    server_name archsmith.satszone.link;

    root /home/ubuntu/archsmith/frontend/dist;
    index index.html;

    location /api/ {
        proxy_pass http://127.0.0.1:8002;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /health {
        proxy_pass http://127.0.0.1:8002;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/archsmith /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

### SSL Certificate
```bash
sudo certbot --nginx -d archsmith.satszone.link
# Follow prompts — auto-renews via systemd timer
```

### Deployment Updates
```bash
ssh -i ~/.ssh/archsmith-key.pem ubuntu@<IP>
cd /home/ubuntu/archsmith
git pull
cd frontend && npm run build
sudo systemctl restart archsmith
```

---

## Service Management

```bash
sudo systemctl status archsmith       # check status
sudo systemctl restart archsmith      # restart after code changes
sudo journalctl -u archsmith -f       # tail logs
sudo systemctl stop archsmith         # stop
```

---

## Database

SQLite file at `backend/archsmith.db` — auto-created on first run.

```bash
# Reset database
rm backend/archsmith.db && sudo systemctl restart archsmith
```

---

## Monitoring & Health

```bash
# Health check
curl https://archsmith.satszone.link/health

# Backend logs
sudo journalctl -u archsmith --since "1 hour ago"

# Nginx access logs
sudo tail -f /var/log/nginx/access.log
```
