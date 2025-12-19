# Koinonia Server

WebSocket Secure signaling server for Koinonia P2P grocery list app.

## Quick Start

1. Clone this repository
2. Copy `.env.example` to `.env` and set your domain/email
3. Get initial certificates (see below)
4. Start the server

## Initial Certificate Setup (First-time Bootstrap)

Before starting with SSL, you need to obtain your first certificate. This is a one-time process.

**Important:** Make sure you've set `DOMAIN` and `EMAIL` in your `.env` file first!

The main nginx config requires SSL certificates to start, but we need nginx running to complete the ACME challenge. This bootstrap process uses a temporary HTTP-only config:

```bash
# Load environment variables
source .env

# Start temporary HTTP-only nginx for ACME challenge
docker run --rm -d \
  --name nginx-bootstrap \
  -p 80:80 \
  -v $(pwd)/nginx/nginx.bootstrap.conf:/etc/nginx/nginx.conf:ro \
  -v koinonia-server_certbot-webroot:/var/www/certbot \
  nginx:alpine

# Get certificate
docker run --rm \
  -v koinonia-server_certbot-etc:/etc/letsencrypt \
  -v koinonia-server_certbot-var:/var/lib/letsencrypt \
  -v koinonia-server_certbot-webroot:/var/www/certbot \
  certbot/certbot certonly \
    --webroot \
    --webroot-path=/var/www/certbot \
    -d ${DOMAIN} \
    --email ${EMAIL} \
    --agree-tos \
    --no-eff-email

# Stop bootstrap nginx and start full stack
docker stop nginx-bootstrap
docker compose up -d
```

**Note:** If you hit Let's Encrypt rate limits (common with free dynamic DNS domains), add `--staging` to the certbot command to test. Remove it once rate limits reset.

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
