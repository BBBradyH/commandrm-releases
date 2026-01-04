# Configuration Reference

CommandRM is configured via environment variables. You can set these in a `.env` file in the server directory or export them in your shell.

## Server Configuration

### Core Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `SERVER_ADDRESS` | `:8080` | Address and port to listen on |
| `SERVER_TLS_ENABLED` | `false` | Enable HTTPS |
| `SERVER_TLS_CERT` | | Path to TLS certificate file |
| `SERVER_TLS_KEY` | | Path to TLS private key file |
| `PRODUCTION_MODE` | `false` | Enable production mode (stricter validation) |

### Database

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_DRIVER` | `sqlite` | Database driver: `sqlite` or `postgres` |
| `DATABASE_SQLITE_PATH` | `./commandrm.db` | Path to SQLite database file |
| `DATABASE_URL` | | PostgreSQL connection string |

**PostgreSQL URL Format:**
```
postgres://user:password@host:5432/database?sslmode=disable
```

### Redis (Optional)

| Variable | Default | Description |
|----------|---------|-------------|
| `REDIS_ENABLED` | `false` | Enable Redis for caching/sessions |
| `REDIS_URL` | `redis://localhost:6379` | Redis connection URL |

### Authentication

| Variable | Default | Description |
|----------|---------|-------------|
| `AUTH_JWT_SECRET` | `change-me` | JWT signing secret (32+ characters required in production) |
| `AUTH_ACCESS_TOKEN_EXPIRY` | `15` | Access token expiry in minutes |
| `AUTH_REFRESH_TOKEN_EXPIRY` | `168` | Refresh token expiry in hours (default 7 days) |

**Important:** In production mode, the server will refuse to start if `AUTH_JWT_SECRET` is set to the default value.

### Agent Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `AGENT_AUTO_APPROVE` | `true` | Auto-approve new agent enrollments |
| `AGENT_METRICS_INTERVAL` | `30` | Metrics collection interval in seconds |
| `AGENT_HEARTBEAT_TIMEOUT` | `90` | Seconds before marking agent offline |

### Agent Certificate Authentication

| Variable | Default | Description |
|----------|---------|-------------|
| `AGENT_AUTH_MODE` | `both` | Auth mode: `enrollment_key`, `certificate`, or `both` |
| `AGENT_CA_CERT` | | Path to CA certificate (optional if auto-generate) |
| `AGENT_CA_KEY` | | Path to CA private key (optional if auto-generate) |
| `AGENT_AUTO_GENERATE_CA` | `true` | Auto-generate CA if not provided |
| `AGENT_CERT_VALIDITY_DAYS` | `365` | Agent certificate validity period |

### Security

| Variable | Default | Description |
|----------|---------|-------------|
| `CORS_ALLOWED_ORIGINS` | | Comma-separated allowed origins (empty = none, `*` = all) |
| `RATE_LIMIT_REQUESTS` | `100` | General API rate limit per minute |
| `RATE_LIMIT_LOGIN` | `5` | Login endpoint rate limit per minute |
| `AUDIT_LOG_ENABLED` | `true` | Enable audit logging |

### Metrics & Cleanup

| Variable | Default | Description |
|----------|---------|-------------|
| `METRICS_RETENTION_DAYS` | `7` | How long to keep detailed metrics |
| `METRICS_CLEANUP_INTERVAL_MINS` | `60` | Cleanup job interval in minutes |

### Inventory Scanning

| Variable | Default | Description |
|----------|---------|-------------|
| `INVENTORY_SCAN_ENABLED` | `true` | Enable automatic inventory scanning |
| `INVENTORY_SCAN_INTERVAL_HOURS` | `1` | How often to check for agents needing scans |
| `INVENTORY_SCAN_MAX_AGE_HOURS` | `24` | Max age before agent needs rescan |
| `INVENTORY_INCLUDE_UPDATES` | `true` | Include available updates in scheduled scans |

### Update System

| Variable | Default | Description |
|----------|---------|-------------|
| `UPDATE_ENABLED` | `true` | Enable update checking |
| `UPDATE_CHECK_INTERVAL_HOURS` | `6` | How often to check for updates |
| `UPDATE_CHANNEL` | `stable` | Update channel: `stable` or `beta` |
| `UPDATE_AUTO_CHECK` | `true` | Auto-check on server start |
| `UPDATE_GPG_PUBLIC_KEY` | | Path to GPG public key for verification |
| `UPDATE_HISTORY_RETENTION_DAYS` | `90` | How long to keep update history |

### Agent Updates

| Variable | Default | Description |
|----------|---------|-------------|
| `AGENT_UPDATE_DELIVERY` | `push` | Delivery method: `push`, `pull`, or `both` |
| `AGENT_UPDATE_CHUNK_SIZE` | `1048576` | Chunk size in bytes (default 1MB) |
| `AGENT_UPDATE_MAX_CONCURRENT` | `10` | Max simultaneous agent updates |
| `AGENT_UPDATE_RETRY_ATTEMPTS` | `3` | Retry attempts for failed updates |
| `AGENT_UPDATE_ROLLBACK_ENABLED` | `true` | Enable automatic rollback on failure |

### Logging

| Variable | Default | Description |
|----------|---------|-------------|
| `LOG_LEVEL` | `info` | Log level: `debug`, `info`, `warn`, `error` |
| `LOG_FORMAT` | `text` | Log format: `text` or `json` |
| `LOG_FILE` | | Log file path (empty = stdout only) |

### Storage

| Variable | Default | Description |
|----------|---------|-------------|
| `STORAGE_TYPE` | `local` | Storage backend: `local` or `s3` |
| `STORAGE_LOCAL_PATH` | `./data` | Local storage path |

### Notifications (Optional)

| Variable | Default | Description |
|----------|---------|-------------|
| `SMTP_ENABLED` | `false` | Enable email notifications |
| `SMTP_HOST` | | SMTP server hostname |
| `SMTP_PORT` | `587` | SMTP server port |
| `SMTP_USERNAME` | | SMTP username |
| `SMTP_PASSWORD` | | SMTP password |
| `SMTP_FROM` | | From email address |
| `SMTP_TLS` | `true` | Use TLS for SMTP |

## Agent Configuration

The agent reads configuration from environment variables or a config file at `/etc/commandrm/agent.conf` (Linux) or `C:\Program Files\CommandRM\agent.conf` (Windows).

| Variable | Default | Description |
|----------|---------|-------------|
| `COMMANDRM_SERVER` | | Server URL (required) |
| `COMMANDRM_KEY` | | Enrollment key (required for first connection) |
| `COMMANDRM_NAME` | hostname | Agent display name |
| `COMMANDRM_CERT` | | Path to agent certificate |
| `COMMANDRM_CERT_KEY` | | Path to agent certificate key |
| `COMMANDRM_CA_CERT` | | Path to CA certificate |
| `COMMANDRM_INSECURE` | `false` | Skip TLS verification (not recommended) |
| `COMMANDRM_METRICS_INTERVAL` | `30` | Metrics collection interval in seconds |
| `COMMANDRM_SCRIPTS_DIR` | `./scripts` | Directory for custom metric scripts |

## Example Configurations

### Development

```bash
# .env for development
SERVER_ADDRESS=:8080
DATABASE_DRIVER=sqlite
DATABASE_SQLITE_PATH=./dev.db
AUTH_JWT_SECRET=development-secret-not-for-prod
LOG_LEVEL=debug
```

### Production (SQLite)

```bash
# .env for small production deployment
SERVER_ADDRESS=:443
SERVER_TLS_ENABLED=true
SERVER_TLS_CERT=/etc/commandrm/certs/server.crt
SERVER_TLS_KEY=/etc/commandrm/certs/server.key
DATABASE_DRIVER=sqlite
DATABASE_SQLITE_PATH=/var/lib/commandrm/commandrm.db
AUTH_JWT_SECRET=your-very-long-random-secret-here-32-chars
PRODUCTION_MODE=true
CORS_ALLOWED_ORIGINS=https://commandrm.example.com
```

### Production (PostgreSQL)

```bash
# .env for larger production deployment
SERVER_ADDRESS=:443
SERVER_TLS_ENABLED=true
SERVER_TLS_CERT=/etc/commandrm/certs/server.crt
SERVER_TLS_KEY=/etc/commandrm/certs/server.key
DATABASE_DRIVER=postgres
DATABASE_URL=postgres://commandrm:password@localhost:5432/commandrm?sslmode=require
AUTH_JWT_SECRET=your-very-long-random-secret-here-32-chars
PRODUCTION_MODE=true
CORS_ALLOWED_ORIGINS=https://commandrm.example.com
REDIS_ENABLED=true
REDIS_URL=redis://localhost:6379
METRICS_RETENTION_DAYS=30
```

### Docker Compose Environment

```yaml
# docker-compose.yml
services:
  server:
    image: commandrm-server
    environment:
      - SERVER_ADDRESS=:8080
      - DATABASE_DRIVER=postgres
      - DATABASE_URL=postgres://postgres:password@db:5432/commandrm
      - AUTH_JWT_SECRET=${JWT_SECRET}
      - PRODUCTION_MODE=true
    ports:
      - "8080:8080"
    depends_on:
      - db

  db:
    image: postgres:14
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=commandrm
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

## Configuration Precedence

1. Environment variables (highest priority)
2. `.env` file in working directory
3. Default values (lowest priority)

## Validating Configuration

Run the server with the `--validate` flag to check configuration without starting:

```bash
./commandrm-server serve --validate
```

This will:
- Check all required settings are present
- Validate database connection
- Verify TLS certificates (if enabled)
- Check JWT secret strength in production mode
