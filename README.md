# Hoppscotch Self-Host — Setup Guide

## Prerequisites
- [Docker](https://docs.docker.com/get-docker/) installed and running

## Quick start (no account needed)

Just want to make API requests locally? No login or email setup required.

### 1. Get the files

Copy `docker-compose.yml` and `.env` from this repo (or the team share).

### 2. Run database migrations (first time only)

```bash
docker compose --profile default up -d hoppscotch-db
# Wait ~5 seconds, then:
docker run --rm \
  --network "$(basename "$PWD")_default" \
  --env-file .env \
  hoppscotch/hoppscotch:latest \
  sh -c "cd /dist/backend && npx prisma migrate deploy"
```

### 3. Start

```bash
docker compose --profile default up -d
```

### 4. Open the app

**http://localhost:3000** — ready to use, no login required.

### 5. Install the Agent (required for making API requests)

The browser interceptor is blocked by CORS on most APIs. The Agent runs locally and bypasses this.

**Linux:**
```bash
wget https://github.com/hoppscotch/agent-releases/releases/download/v0.1.16/Hoppscotch_Agent_linux_x64.deb
sudo dpkg -i Hoppscotch_Agent_linux_x64.deb
hoppscotch-agent
```

**Windows:** download and run `Hoppscotch_Agent_win_x64.msi`

**Mac:** download and run `Hoppscotch_Agent_mac_aarch64.dmg` (Apple Silicon) or `Hoppscotch_Agent_mac_x64.dmg` (Intel)

> All releases: https://github.com/hoppscotch/agent-releases/releases

Once the agent is running, go to the app → Settings → Interceptor → **Agent**, then paste the API key shown by the agent to pair them. This is local-only — no cloud involved.

---

## Accounts & team features (optional)

Login is only needed for server-saved collections, team workspaces, and sync across devices.
Authentication uses magic links sent via email. Since everything runs locally, Mailhog acts as a local mail catcher — no emails leave your machine.

To view magic link emails: **http://localhost:8025**

| | URL |
|---|---|
| App | http://localhost:3000 |
| Admin dashboard | http://localhost:3100 |
| Mail catcher | http://localhost:8025 |

---

## Creating your own `.env`

If you need to set up from scratch instead of copying the team's `.env`:

```env
# Database
DATABASE_URL=postgresql://postgres:testpass@hoppscotch-db:5432/hoppscotch?connect_timeout=300

# Secrets
JWT_SECRET=<openssl rand -base64 32>
SESSION_SECRET=<openssl rand -base64 32>
DATA_ENCRYPTION_KEY=<openssl rand -hex 16>   # must be exactly 32 chars

# Token settings
TOKEN_SALT_COMPLEXITY=10
MAGIC_LINK_TOKEN_VALIDITY=3
REFRESH_TOKEN_VALIDITY=604800000
ACCESS_TOKEN_VALIDITY=86400000

# URLs
REDIRECT_URL=http://localhost:3000
WHITELISTED_ORIGINS=http://localhost:3000,http://localhost:3100,http://localhost:3170
VITE_BASE_URL=http://localhost:3000
VITE_SHORTCODE_BASE_URL=http://localhost:3000
VITE_ADMIN_URL=http://localhost:3100
VITE_BACKEND_GQL_URL=http://localhost:3170/graphql
VITE_BACKEND_WS_URL=wss://localhost:3170/graphql
VITE_BACKEND_API_URL=http://localhost:3170/v1

# Auth
VITE_ALLOWED_AUTH_PROVIDERS=EMAIL
MAILER_SMTP_ENABLE=true
MAILER_USE_CUSTOM_CONFIGS=false
MAILER_ADDRESS_FROM=Hoppscotch <noreply@hoppscotch.local>
MAILER_SMTP_URL=smtp://mailhog:1025

# Rate limiting
RATE_LIMIT_TTL=60
RATE_LIMIT_MAX=100
```

Generate secrets:
```bash
openssl rand -base64 32   # JWT_SECRET
openssl rand -base64 32   # SESSION_SECRET
openssl rand -hex 16      # DATA_ENCRYPTION_KEY
```

> **Shared DB note:** If your team shares one database, everyone must use the **same** `DATA_ENCRYPTION_KEY`. Exchange it securely.

---

## Useful commands

```bash
# Stop
docker compose --profile default down

# View logs
docker logs hoppscotch-aio -f

# Restart
docker compose --profile default restart
```

**Important:** Never commit `.env` to version control.
