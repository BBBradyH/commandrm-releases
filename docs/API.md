# API Reference

CommandRM provides a REST API for programmatic access. All endpoints except authentication require a valid JWT token.

## Base URL

```
https://your-server:8080/api/v1
```

## Authentication

### Login

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password"
}
```

**Response:**
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "expires_in": 900,
  "user": {
    "id": "uuid",
    "username": "admin",
    "email": "admin@example.com",
    "is_admin": true,
    "two_factor_enabled": false
  }
}
```

If 2FA is enabled, the response includes `requires_2fa: true` and you must complete login via `/auth/2fa/login`.

### Refresh Token

```http
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refresh_token": "eyJ..."
}
```

### All Subsequent Requests

Include the access token in the Authorization header:

```http
Authorization: Bearer eyJ...
```

## Agents

### List Agents

```http
GET /api/v1/agents
```

Query parameters:
- `status` - Filter by status: `online`, `offline`, `pending`
- `group_id` - Filter by group
- `search` - Search by name or hostname
- `page` - Page number (default: 1)
- `page_size` - Items per page (default: 20)

### Get Agent

```http
GET /api/v1/agents/{id}
```

### Update Agent

```http
PUT /api/v1/agents/{id}
Content-Type: application/json

{
  "name": "New Name",
  "tags": ["production", "web"]
}
```

### Delete Agent

```http
DELETE /api/v1/agents/{id}
```

### Get Agent Metrics

```http
GET /api/v1/agents/{id}/metrics?start=2024-01-01T00:00:00Z&end=2024-01-02T00:00:00Z
```

### Get Latest Metrics

```http
GET /api/v1/agents/{id}/metrics/latest
```

## Groups

### List Groups

```http
GET /api/v1/groups
```

### Create Group

```http
POST /api/v1/groups
Content-Type: application/json

{
  "name": "Production Servers",
  "description": "All production servers"
}
```

### Update Group

```http
PUT /api/v1/groups/{id}
Content-Type: application/json

{
  "name": "Updated Name",
  "description": "Updated description"
}
```

### Delete Group

```http
DELETE /api/v1/groups/{id}
```

### Add Agents to Group

```http
POST /api/v1/groups/{id}/agents
Content-Type: application/json

{
  "agent_ids": ["uuid1", "uuid2"]
}
```

### Remove Agent from Group

```http
DELETE /api/v1/groups/{id}/agents/{agent_id}
```

## Alerts

### List Alert Rules

```http
GET /api/v1/alert-rules
```

### Create Alert Rule

```http
POST /api/v1/alert-rules
Content-Type: application/json

{
  "name": "High CPU",
  "metric": "cpu_percent",
  "operator": "gt",
  "threshold": 90,
  "duration_seconds": 300,
  "severity": "warning",
  "enabled": true
}
```

Operators: `gt` (greater than), `lt` (less than), `eq` (equal), `ne` (not equal)

Severity levels: `info`, `warning`, `critical`

### List Active Alerts

```http
GET /api/v1/alerts?status=active
```

Status options: `active`, `acknowledged`, `resolved`

### Acknowledge Alert

```http
POST /api/v1/alerts/{id}/acknowledge
```

### Resolve Alert

```http
POST /api/v1/alerts/{id}/resolve
```

## Scripts

### List Scripts

```http
GET /api/v1/scripts?category=maintenance&language=bash&search=backup
```

### Create Script

```http
POST /api/v1/scripts
Content-Type: application/json

{
  "name": "Backup Script",
  "description": "Backs up important files",
  "language": "bash",
  "category": "maintenance",
  "content": "#!/bin/bash\ntar -czf backup.tar.gz /data",
  "timeout_seconds": 300,
  "run_as_admin": true,
  "shared": true
}
```

Languages: `bash`, `powershell`, `python`, `batch`

### Execute Script

```http
POST /api/v1/scripts/{id}/execute
Content-Type: application/json

{
  "agent_ids": ["uuid1", "uuid2"],
  "group_ids": ["group-uuid"]
}
```

### Get Execution Details

```http
GET /api/v1/executions/{id}
```

## Software Inventory

### Get Agent Inventory

```http
GET /api/v1/agents/{id}/inventory?package_manager=apt&search=nginx
```

### Trigger Inventory Scan

```http
POST /api/v1/agents/{id}/inventory/scan
Content-Type: application/json

{
  "include_updates": true
}
```

### Global Inventory Search

```http
GET /api/v1/inventory?search=python&page=1&page_size=50
```

### Inventory Statistics

```http
GET /api/v1/inventory/stats
```

Response:
```json
{
  "total_packages": 15234,
  "by_package_manager": {
    "apt": 8521,
    "chocolatey": 6713
  },
  "agents_with_outdated_packages": 12,
  "total_outdated_packages": 156
}
```

## Patches

### List Available Updates

```http
GET /api/v1/updates?severity=critical
```

### Create Patch Deployment

```http
POST /api/v1/patches
Content-Type: application/json

{
  "name": "January Security Updates",
  "agent_ids": ["uuid1", "uuid2"],
  "packages": ["openssl", "nginx"],
  "reboot_policy": "if_required",
  "scheduled_at": "2024-01-15T02:00:00Z"
}
```

Reboot policies: `never`, `if_required`, `always`

### Execute Deployment

```http
POST /api/v1/patches/{id}/execute
```

### Get Deployment Results

```http
GET /api/v1/patches/{id}/results
```

## Deployment Policies

### List Policies

```http
GET /api/v1/policies?policy_type=auto_update&enabled=true
```

Policy types: `auto_update`, `scheduled_scan`, `script_schedule`

### Create Policy

```http
POST /api/v1/policies
Content-Type: application/json

{
  "name": "Daily Security Scan",
  "policy_type": "scheduled_scan",
  "schedule": {
    "minute": "0",
    "hour": "3",
    "day_of_week": "*",
    "day_of_month": "*"
  },
  "config": {
    "include_updates": true
  },
  "target_type": "all",
  "enabled": true
}
```

### Enable/Disable Policy

```http
POST /api/v1/policies/{id}/enable
POST /api/v1/policies/{id}/disable
```

## Files

### List Directory

```http
GET /api/v1/agents/{id}/files?path=/var/log
```

### Download File

```http
GET /api/v1/agents/{id}/files/download?path=/var/log/syslog
```

### Upload File

```http
POST /api/v1/agents/{id}/files/upload?path=/tmp/script.sh
Content-Type: multipart/form-data

file: [binary data]
```

### Delete File

```http
DELETE /api/v1/agents/{id}/files?path=/tmp/old-file.txt
```

### Create Directory

```http
POST /api/v1/agents/{id}/files/mkdir?path=/var/data/backup
```

## Terminal

### Start Terminal Session

```http
POST /api/v1/agents/{id}/terminal
Content-Type: application/json

{
  "rows": 24,
  "cols": 80
}
```

Response:
```json
{
  "session_id": "session-uuid",
  "ws_url": "/api/v1/terminal/ws?session_id=session-uuid&token=..."
}
```

### Close Terminal Session

```http
DELETE /api/v1/terminal/{session_id}
```

### List Terminal Recordings

```http
GET /api/v1/terminal/recordings?agent_id=uuid&page=1
```

### Playback Recording

```http
GET /api/v1/terminal/recordings/{id}/playback
```

Returns JSON Lines format with timestamped events.

## Remote Desktop

### Start Session

```http
POST /api/v1/agents/{id}/remote-desktop
Content-Type: application/json

{
  "quality": "medium"
}
```

Quality options: `low`, `medium`, `high`

### Stop Session

```http
DELETE /api/v1/agents/{id}/remote-desktop/{session_id}
```

## Users (Admin Only)

### List Users

```http
GET /api/v1/users
```

### Create User

```http
POST /api/v1/users
Content-Type: application/json

{
  "username": "newuser",
  "email": "user@example.com",
  "password": "SecurePassword123",
  "is_admin": false
}
```

### Update User

```http
PUT /api/v1/users/{id}
Content-Type: application/json

{
  "email": "newemail@example.com",
  "is_admin": true
}
```

### Delete User

```http
DELETE /api/v1/users/{id}
```

## Two-Factor Authentication

### Setup 2FA

```http
POST /api/v1/auth/2fa/setup
```

Response:
```json
{
  "secret": "JBSWY3DPEHPK3PXP",
  "qr_code": "data:image/png;base64,..."
}
```

### Verify 2FA Setup

```http
POST /api/v1/auth/2fa/verify
Content-Type: application/json

{
  "code": "123456"
}
```

Response includes backup codes.

### Complete 2FA Login

```http
POST /api/v1/auth/2fa/login
Content-Type: application/json

{
  "code": "123456"
}
```

### Disable 2FA

```http
POST /api/v1/auth/2fa/disable
Content-Type: application/json

{
  "password": "current-password"
}
```

## Enrollment Keys

### List Keys

```http
GET /api/v1/enrollment-keys
```

### Create Key

```http
POST /api/v1/enrollment-keys
Content-Type: application/json

{
  "description": "Production servers",
  "expires_at": "2024-12-31T23:59:59Z",
  "max_uses": 100
}
```

### Delete Key

```http
DELETE /api/v1/enrollment-keys/{id}
```

## Audit Logs (Admin Only)

### List Audit Logs

```http
GET /api/v1/audit-logs?action=delete&resource=agent&page=1
```

### Get Audit Log Entry

```http
GET /api/v1/audit-logs/{id}
```

## Dashboard

### Get Statistics

```http
GET /api/v1/dashboard/stats
```

Response:
```json
{
  "total_agents": 150,
  "online_agents": 142,
  "offline_agents": 8,
  "pending_agents": 0,
  "active_alerts": 3,
  "recent_activity": [...]
}
```

## Version

### Get Version Info

```http
GET /api/v1/version
```

Response:
```json
{
  "version": "1.0.0",
  "channel": "stable",
  "build_time": "2024-01-15T10:30:00Z",
  "git_commit": "abc123",
  "min_agent_version": "1.0.0"
}
```

### Check for Updates

```http
GET /api/v1/updates/available
```

## Health Checks

### Basic Health

```http
GET /health
```

### Readiness Check

```http
GET /ready
```

Checks database connectivity.

## Error Responses

All errors follow this format:

```json
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": {}
}
```

Common HTTP status codes:
- `400` - Bad Request (validation error)
- `401` - Unauthorized (missing/invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `429` - Too Many Requests (rate limited)
- `500` - Internal Server Error

## Rate Limits

- General API: 100 requests/minute
- Login endpoint: 5 requests/minute

Rate limit headers are included in responses:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705315200
```
