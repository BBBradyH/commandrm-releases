# Security Guide

This document covers security best practices for deploying and operating CommandRM.

## Table of Contents

- [Security Architecture](#security-architecture)
- [Authentication](#authentication)
- [Transport Security](#transport-security)
- [Agent Security](#agent-security)
- [Network Security](#network-security)
- [Data Security](#data-security)
- [Hardening Checklist](#hardening-checklist)
- [Security Updates](#security-updates)
- [Reporting Vulnerabilities](#reporting-vulnerabilities)

## Security Architecture

CommandRM implements defense-in-depth with multiple security layers:

```
┌──────────────────────────────────────────────────────────────┐
│                     Security Layers                          │
├──────────────────────────────────────────────────────────────┤
│  TLS Encryption          All traffic encrypted in transit    │
│  JWT Authentication      Token-based user authentication     │
│  Certificate Auth        mTLS for agent authentication       │
│  Rate Limiting           Protection against brute force      │
│  CORS Protection         Cross-origin request control        │
│  Audit Logging           All mutations logged                │
│  2FA Support             Optional TOTP second factor         │
│  GPG Signatures          Signed release binaries             │
└──────────────────────────────────────────────────────────────┘
```

## Authentication

### User Authentication

CommandRM uses JWT (JSON Web Tokens) for user authentication:

- **Access tokens**: Short-lived (15 minutes default), used for API requests
- **Refresh tokens**: Longer-lived (7 days default), used to obtain new access tokens
- **Secure storage**: Tokens stored in httpOnly cookies or local storage

### JWT Secret

The `AUTH_JWT_SECRET` is critical for security:

```bash
# Generate a secure secret (Linux/macOS)
openssl rand -base64 32

# Generate a secure secret (PowerShell)
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Maximum 256 }) -as [byte[]])
```

**Requirements:**
- Minimum 32 characters
- Use cryptographically random generation
- Never reuse across environments
- Rotate periodically (requires all users to re-authenticate)

### Two-Factor Authentication

Enable 2FA for all admin accounts:

1. Go to Settings > Security
2. Click "Enable 2FA"
3. Scan QR code with authenticator app (Google Authenticator, Authy, etc.)
4. Enter verification code
5. Save backup codes securely

2FA is required for:
- Remote terminal access
- Remote desktop access
- Script execution
- User management operations

### Password Requirements

Default password policy:
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character

## Transport Security

### TLS Configuration

Always enable TLS in production:

```bash
SERVER_TLS_ENABLED=true
SERVER_TLS_CERT=/etc/commandrm/certs/server.crt
SERVER_TLS_KEY=/etc/commandrm/certs/server.key
```

**Recommended TLS settings:**
- TLS 1.2 minimum (TLS 1.3 preferred)
- Strong cipher suites only
- HSTS enabled (automatic with TLS)

### Certificate Options

1. **Let's Encrypt** (Recommended for public servers)
   ```bash
   # During installation
   ./commandrm-server-setup --tls-mode letsencrypt --domain commandrm.example.com
   ```

2. **Self-Signed** (Internal networks)
   ```bash
   # Auto-generated during installation
   ./commandrm-server-setup --tls-mode self-signed
   ```

3. **Custom Certificate**
   ```bash
   # Use existing certificate
   ./commandrm-server-setup --tls-mode custom \
     --tls-cert /path/to/cert.pem \
     --tls-key /path/to/key.pem
   ```

### WebSocket Security

WebSocket connections are secured via:
- Same TLS encryption as HTTP
- JWT token validation on connect
- Certificate authentication for agents
- Connection timeout handling

## Agent Security

### Certificate Authentication

For high-security environments, use certificate authentication:

```bash
# Server configuration
AGENT_AUTH_MODE=certificate  # Only accept certificate auth
AGENT_AUTO_GENERATE_CA=true
AGENT_CERT_VALIDITY_DAYS=365
```

Certificate authentication provides:
- Mutual TLS (mTLS) verification
- No shared secrets to manage
- Certificate revocation capability
- Unique identity per agent

### Enrollment Keys

Best practices for enrollment keys:

1. **Short-lived keys**: Set expiration dates
   ```bash
   ./commandrm-server enrollment create \
     -d "New deployment" \
     --expires 24h
   ```

2. **Limited uses**: Restrict number of enrollments
   ```bash
   ./commandrm-server enrollment create \
     -d "Single server" \
     --max-uses 1
   ```

3. **Descriptive names**: Track key purpose
4. **Immediate revocation**: Delete unused keys
5. **Audit key usage**: Monitor enrollment events

### Agent Hardening

1. Run agent as non-root user (except for privileged operations)
2. Use filesystem permissions to protect config files
3. Limit network access to server only
4. Enable logging for audit trail

## Network Security

### Firewall Configuration

**Server:**
```bash
# Allow only necessary ports
ufw allow 443/tcp  # HTTPS
ufw allow 80/tcp   # HTTP (for Let's Encrypt only)
ufw deny all       # Deny everything else
```

**Agent:**
```bash
# Outbound to server only
iptables -A OUTPUT -d server-ip -p tcp --dport 443 -j ACCEPT
iptables -A OUTPUT -j DROP
```

### Reverse Proxy

Using nginx as a reverse proxy adds:
- Additional TLS termination point
- Request filtering
- Rate limiting
- DDoS protection

```nginx
server {
    listen 443 ssl http2;
    server_name commandrm.example.com;

    ssl_certificate /etc/letsencrypt/live/commandrm.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/commandrm.example.com/privkey.pem;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        limit_req zone=api burst=20 nodelay;
    }
}
```

### CORS Configuration

Restrict allowed origins in production:

```bash
# Allow specific origins only
CORS_ALLOWED_ORIGINS=https://commandrm.example.com,https://admin.example.com

# Never use * in production
# CORS_ALLOWED_ORIGINS=*  # DON'T DO THIS
```

## Data Security

### Database Security

**PostgreSQL:**
```sql
-- Create dedicated user with minimal privileges
CREATE USER commandrm WITH PASSWORD 'secure-password';
CREATE DATABASE commandrm OWNER commandrm;

-- Use SSL connections
ALTER SYSTEM SET ssl = on;
```

**SQLite:**
- Store database file with restricted permissions (600)
- Use encrypted filesystem for additional protection
- Regular backups with encryption

### Secrets Management

Store secrets securely:

1. **Environment variables**: Use systemd or container secrets
2. **Files**: Restrict permissions to 600
3. **Never commit secrets**: Use .gitignore, scan for leaks
4. **Rotate regularly**: Especially JWT secrets

### Audit Logging

Enable and protect audit logs:

```bash
AUDIT_LOG_ENABLED=true
LOG_FILE=/var/log/commandrm/audit.log
```

Logs capture:
- All authentication events (success/failure)
- User management operations
- Agent enrollment/deletion
- Configuration changes
- Script executions
- File operations

Forward logs to SIEM for monitoring:
```bash
# Example: forward to syslog
LOG_FORMAT=json
# Configure rsyslog or similar to forward logs
```

## Hardening Checklist

### Pre-Deployment

- [ ] Generate strong JWT secret (32+ chars)
- [ ] Obtain TLS certificates
- [ ] Set up PostgreSQL with SSL (for large deployments)
- [ ] Configure firewall rules
- [ ] Review default settings

### Production Configuration

- [ ] `PRODUCTION_MODE=true`
- [ ] `SERVER_TLS_ENABLED=true`
- [ ] `CORS_ALLOWED_ORIGINS` set to specific origins
- [ ] `RATE_LIMIT_LOGIN=5` or lower
- [ ] `AUDIT_LOG_ENABLED=true`

### User Management

- [ ] Create separate admin accounts per person
- [ ] Enable 2FA for all admin accounts
- [ ] Remove default/test accounts
- [ ] Document access policies

### Agent Deployment

- [ ] Use certificate authentication in high-security environments
- [ ] Set enrollment key expiration
- [ ] Limit enrollment key uses
- [ ] Monitor agent enrollments

### Ongoing Operations

- [ ] Monitor audit logs
- [ ] Review access regularly
- [ ] Apply updates promptly
- [ ] Rotate secrets periodically
- [ ] Test backup restoration
- [ ] Conduct security reviews

## Security Updates

### Verifying Releases

All releases are GPG-signed. Verify before deploying:

```bash
# Import public key
gpg --import commandrm-release-key.asc

# Verify signature
gpg --verify checksums.txt.asc checksums.txt

# Verify binary checksum
sha256sum -c checksums.txt --ignore-missing
```

### Update Process

1. Review release notes for security fixes
2. Download and verify new binaries
3. Test in staging environment
4. Schedule maintenance window
5. Deploy update
6. Verify functionality
7. Monitor for issues

### Agent Updates

Server can push updates to agents automatically:

```bash
# Enable auto-updates
AGENT_UPDATE_ROLLBACK_ENABLED=true  # Auto-rollback on failure
AGENT_UPDATE_MAX_CONCURRENT=10      # Staged rollout
```

## Reporting Vulnerabilities

If you discover a security vulnerability:

1. **Do not** disclose publicly
2. Email security details to the maintainers
3. Include:
   - Description of vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

We will:
- Acknowledge within 48 hours
- Provide fix timeline
- Credit reporter (unless anonymity requested)
- Publish advisory after fix is available

## Common Security Issues

### Weak JWT Secret

**Symptom:** Server starts with warning about default secret

**Fix:**
```bash
# Generate and set strong secret
AUTH_JWT_SECRET=$(openssl rand -base64 32)
```

### Exposed API

**Symptom:** API accessible without authentication

**Fix:**
- Verify firewall rules
- Check CORS settings
- Ensure TLS is enabled

### Agent Enrollment Abuse

**Symptom:** Unauthorized agents appearing

**Fix:**
- Delete unused enrollment keys
- Set key expiration
- Limit key uses
- Monitor audit logs

### Session Hijacking

**Prevention:**
- Always use HTTPS
- Set appropriate token expiry
- Enable 2FA for sensitive operations
- Monitor concurrent sessions
