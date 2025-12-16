# Koinonia Server

WebSocket Secure signaling server for Koinonia P2P grocery list app.

## Quick Start

1. Clone this repository
2. Copy `.env.example` to `.env` and set your domain/email
3. Get initial certificates (see below)
4. Start the server

## Initial Certificate Setup

Before starting with SSL, get your first certificate.

**Important:** Make sure you've set `DOMAIN` and `EMAIL` in your `.env` file first!

```bash
# Load environment variables
source .env

# Start nginx temporarily for HTTP challenge
docker compose up -d nginx

# Get certificate (uses DOMAIN and EMAIL from .env)
docker compose run --rm certbot certonly \
  --webroot \
  --webroot-path=/var/www/certbot \
  -d ${DOMAIN} \
  --email ${EMAIL} \
  --agree-tos \
  --no-eff-email

# Restart with full config
docker compose down
docker compose up -d
```

## Running

```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f

# Stop
docker compose down
```

## Certificate Renewal

Certificates auto-renew via the certbot container (checks every 12 hours).

## Update Koinonia App

Update your Koinonia app settings to use this server (replace with your actual domain):

```typescript
signalingServers: ['wss://your-domain.com']
```

## How It Works

The nginx configuration automatically uses the `DOMAIN` environment variable from your `.env` file. No manual editing of configuration files is required - just set your domain in `.env` and everything else is handled automatically.
