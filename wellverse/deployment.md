# WellVerse Deployment Guide

## Local Development Setup

### Prerequisites
- Python 3.12+
- Node.js 20+
- npm

### Backend Setup

```bash
cd backend
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Run the backend:

```bash
cd backend
PYTHONPATH=src .venv/bin/uvicorn wellverse.main:app --reload --port 8026
```

The backend will:
1. Create `wellverse.db` (SQLite) on first run
2. Seed the database with 6 verticals, 14 guides, 6 library items, 6 circles, 3 blog posts, and 1 admin user
3. Serve the API at `http://localhost:8026/api`

### Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

The frontend dev server runs on port 5190 with Vite proxy forwarding `/api` requests to `http://localhost:8026`.

### Quick Start

If a `start.sh` script exists in the project root:

```bash
./start.sh
```

This starts both the backend (port 8026) and frontend dev server.

### Default Admin Credentials

- Email: `admin@wellverse.app`
- Password: `admin123`

---

## Docker Build and Run

### Build

```bash
docker build -t wellverse .
```

The Dockerfile uses a multi-stage build:
1. **Stage 1** (Node 20): Installs frontend dependencies and builds the production bundle (`npm ci && npm run build`)
2. **Stage 2** (Python 3.12-slim): Installs Python dependencies, copies backend source and frontend dist

### Run

```bash
docker run -d \
  --name wellverse \
  -p 8026:8026 \
  -e WELLVERSE_SECRET_KEY="your-production-secret-key" \
  -e STRIPE_SECRET_KEY="sk_live_..." \
  -e STRIPE_WEBHOOK_SECRET="whsec_..." \
  -e STRIPE_PRICE_COMMITTED="price_..." \
  -e SENDGRID_API_KEY="SG...." \
  -e SENDGRID_FROM_EMAIL="hello@wellverse.app" \
  -e WELLVERSE_APP_URL="https://wellverse.satszone.link" \
  wellverse
```

In production, FastAPI serves the built frontend from `frontend/dist/` as static files at the root path.

### Persistent Data

SQLite database and uploads are stored inside the container. To persist across container restarts:

```bash
docker run -d \
  --name wellverse \
  -p 8026:8026 \
  -v /data/wellverse/db:/app/src/wellverse/../../wellverse.db \
  -v /data/wellverse/uploads:/app/uploads \
  wellverse
```

---

## GCP Deployment

### VM Setup

WellVerse runs as a Docker container on a GCP Compute Engine instance (consolidated instance hosting multiple apps).

```bash
# SSH into the VM
gcloud compute ssh <instance-name> --zone <zone>

# Pull and run
docker pull <registry>/wellverse:latest
docker run -d \
  --name wellverse \
  --restart unless-stopped \
  -p 8026:8026 \
  -e WELLVERSE_SECRET_KEY="..." \
  wellverse
```

### Port Assignment

WellVerse uses port **8026** on the consolidated instance.

---

## Nginx Configuration

Nginx serves as a reverse proxy with SSL termination on the consolidated instance.

### Site Configuration

```nginx
server {
    listen 443 ssl http2;
    server_name wellverse.satszone.link;

    ssl_certificate /etc/letsencrypt/live/wellverse.satszone.link/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/wellverse.satszone.link/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8026;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Stripe webhook needs raw body
    location /api/payments/webhook {
        proxy_pass http://127.0.0.1:8026;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Stripe-Signature $http_stripe_signature;
    }
}

server {
    listen 80;
    server_name wellverse.satszone.link;
    return 301 https://$server_name$request_uri;
}
```

### SSL Certificates

Using Let's Encrypt via Certbot:

```bash
sudo certbot --nginx -d wellverse.satszone.link
```

### Room Key Gate (Optional)

If using the Nginx room key gate pattern (auth_request), add the gate configuration before the main location block. See the project-level Nginx gate documentation for details.

---

## Route 53 DNS Setup

### DNS Record

| Record Type | Name | Value | TTL |
|-------------|------|-------|-----|
| A | wellverse.satszone.link | `<GCP VM External IP>` | 300 |

### Setup Steps

1. In AWS Route 53, navigate to the `satszone.link` hosted zone
2. Create an A record pointing `wellverse.satszone.link` to the GCP VM's external IP
3. Wait for DNS propagation (usually < 5 minutes with TTL 300)
4. Verify: `dig wellverse.satszone.link`

---

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `WELLVERSE_SECRET_KEY` | Yes (production) | `wellverse-dev-secret-key-change-in-production` | JWT signing key |
| `STRIPE_SECRET_KEY` | No | `""` | Stripe API secret key. If empty, payments run in dev mode. |
| `STRIPE_WEBHOOK_SECRET` | No | `""` | Stripe webhook signature verification secret |
| `STRIPE_PRICE_SEEKER` | No | `""` | Stripe Price ID for pay-per-session (not currently used) |
| `STRIPE_PRICE_COMMITTED` | No | `""` | Stripe Price ID for $49/month subscription |
| `SENDGRID_API_KEY` | No | `""` | SendGrid API key. If empty, emails are logged. |
| `SENDGRID_FROM_EMAIL` | No | `hello@wellverse.app` | Sender email address |
| `WELLVERSE_APP_URL` | No | `http://localhost:5190` | App URL for Stripe redirect URLs |

---

## Monitoring and Logs

### Health Check

```bash
curl https://wellverse.satszone.link/api/health
# Expected: {"status":"ok","app":"WellVerse","version":"2.0.0"}
```

### Docker Logs

```bash
# Tail logs
docker logs -f wellverse

# Last 100 lines
docker logs --tail 100 wellverse
```

### Key Log Signals

- `[EMAIL-DEV]` -- Email sent in dev mode (no SendGrid key)
- `Email send failed:` -- SendGrid API error
- Uvicorn access logs show all HTTP requests with status codes

### Container Management

```bash
# Restart
docker restart wellverse

# Stop and remove
docker stop wellverse && docker rm wellverse

# Check resource usage
docker stats wellverse
```

### Database Inspection

```bash
# Access SQLite database inside the container
docker exec -it wellverse sqlite3 /app/src/../../wellverse.db

# Or copy it out
docker cp wellverse:/app/wellverse.db ./wellverse-backup.db
```

### Stripe Webhook Testing

For local development, use the Stripe CLI:

```bash
stripe listen --forward-to localhost:8026/api/payments/webhook
```

This provides a temporary webhook secret to set as `STRIPE_WEBHOOK_SECRET`.
