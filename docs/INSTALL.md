# Installation Guide

This guide covers installing CommandRM server and agents on various platforms.

## Table of Contents

- [System Requirements](#system-requirements)
- [Server Installation](#server-installation)
  - [One-Click Installer](#one-click-installer)
  - [Manual Installation](#manual-installation)
  - [Docker](#docker)
- [Agent Installation](#agent-installation)
  - [Interactive Installation](#interactive-installation)
  - [Silent/Automated Installation](#silentautomated-installation)
- [Post-Installation](#post-installation)
- [Troubleshooting](#troubleshooting)

## System Requirements

### Server

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 2 cores | 4 cores |
| RAM | 2 GB | 4 GB |
| Disk | 10 GB | 50 GB |
| OS | Windows 10+, Ubuntu 20.04+, macOS 12+ | Same |

### Agent

| Resource | Minimum |
|----------|---------|
| CPU | 1 core |
| RAM | 512 MB |
| Disk | 100 MB |
| OS | Windows 10+, Ubuntu 18.04+, macOS 11+ |

## Server Installation

### One-Click Installer

The easiest way to install the CommandRM server.

#### Download

Get the latest installer from [GitHub Releases](https://github.com/BBBradyH/commandrm-releases/releases):

- `commandrm-server-setup-linux-amd64` - Linux x64
- `commandrm-server-setup-darwin-amd64` - macOS Intel
- `commandrm-server-setup-windows-amd64.exe` - Windows x64

#### Linux/macOS

```bash
# Download (example for Linux)
curl -LO https://github.com/BBBradyH/commandrm-releases/releases/latest/download/commandrm-server-setup-linux-amd64
chmod +x commandrm-server-setup-linux-amd64

# Run installer (requires root)
sudo ./commandrm-server-setup-linux-amd64
```

#### Windows

1. Download `commandrm-server-setup-windows-amd64.exe`
2. Right-click and "Run as administrator"
3. Follow the installation wizard

#### Installation Modes

The installer supports three modes:

**Express Mode** (`--express`): Quick installation with sensible defaults
- SQLite database
- Self-signed TLS certificate
- Auto-generated CA for agent certificates
- Installs as system service

**Standard Mode** (default): Guided installation with prompts
- Choose database (SQLite or PostgreSQL)
- Configure TLS certificate (self-signed, Let's Encrypt, or custom)
- Set server address and port
- Create admin user

**Advanced Mode**: Full control over all settings
- All standard options plus
- Custom paths, logging, retention settings
- Nginx reverse proxy configuration
- Fine-grained security settings

#### Express Installation Example

```bash
# Linux/macOS
sudo ./commandrm-server-setup --express

# Windows PowerShell (as admin)
.\commandrm-server-setup.exe --express
```

### Manual Installation

For custom deployments or development:

```bash
# Clone repository
git clone https://github.com/BBBradyH/commandrm.git
cd commandrm

# Build server
go build -o commandrm-server ./cmd/server

# Build web frontend
cd web && npm install && npm run build && cd ..

# Create configuration
cat > .env << 'EOF'
SERVER_ADDRESS=:8080
DATABASE_DRIVER=sqlite
DATABASE_SQLITE_PATH=./commandrm.db
AUTH_JWT_SECRET=your-32-character-secret-here!!!
PRODUCTION_MODE=true
EOF

# Run database migrations
./commandrm-server migrate

# Create admin user
./commandrm-server user create -u admin -e admin@example.com -p SecurePassword123 --admin

# Create enrollment key for agents
./commandrm-server enrollment create -d "Production agents"

# Start server
./commandrm-server serve
```

### Docker

```bash
# Using docker-compose
docker-compose up -d

# Or manually
docker build -t commandrm-server .
docker run -d \
  -p 8080:8080 \
  -v commandrm-data:/data \
  -e AUTH_JWT_SECRET=your-secret-here \
  commandrm-server
```

## Agent Installation

### Interactive Installation

```bash
# Download agent installer
curl -LO https://github.com/BBBradyH/commandrm-releases/releases/latest/download/commandrm-agent-setup-linux-amd64
chmod +x commandrm-agent-setup-linux-amd64

# Run installer
sudo ./commandrm-agent-setup-linux-amd64
```

The installer will prompt for:
1. Server URL (e.g., `https://your-server.com:8080`)
2. Enrollment key
3. Optional: Custom agent name

### Silent/Automated Installation

For mass deployment via GPO, MDM, Ansible, etc.:

```bash
# Linux/macOS
sudo ./commandrm-agent-setup \
  --silent \
  --server https://your-server.com:8080 \
  --key YOUR_ENROLLMENT_KEY \
  --name "web-server-01"

# Windows PowerShell
.\commandrm-agent-setup.exe `
  --silent `
  --server https://your-server.com:8080 `
  --key YOUR_ENROLLMENT_KEY `
  --name "workstation-42"
```

#### Ansible Example

```yaml
- name: Install CommandRM Agent
  hosts: all
  become: yes
  vars:
    server_url: "https://commandrm.example.com:8080"
    enrollment_key: "{{ lookup('env', 'COMMANDRM_KEY') }}"
  tasks:
    - name: Download agent installer
      get_url:
        url: "https://github.com/BBBradyH/commandrm-releases/releases/latest/download/commandrm-agent-setup-linux-amd64"
        dest: /tmp/commandrm-agent-setup
        mode: '0755'

    - name: Run installer
      command: >
        /tmp/commandrm-agent-setup
        --silent
        --server {{ server_url }}
        --key {{ enrollment_key }}
        --name {{ inventory_hostname }}
```

#### Group Policy (Windows)

1. Create a network share with the installer
2. Create a startup script:

```batch
@echo off
if exist "C:\Program Files\CommandRM\agent.exe" exit /b 0
\\server\share\commandrm-agent-setup.exe --silent --server https://commandrm.local:8080 --key YOUR_KEY
```

3. Deploy via Computer Configuration > Policies > Windows Settings > Scripts > Startup

## Post-Installation

### Verify Server

```bash
# Check service status
# Linux
sudo systemctl status commandrm-server

# Windows
sc query commandrm-server

# macOS
sudo launchctl list | grep commandrm
```

### Access Web Interface

Open your browser to:
- HTTP: `http://your-server:8080`
- HTTPS: `https://your-server:8080`

### Create Enrollment Keys

From the web interface: Settings > Enrollment Keys > Create New

Or via CLI:
```bash
./commandrm-server enrollment create -d "Department X agents"
```

### Verify Agent Connection

After installing agents, they should appear in the web interface within 60 seconds. Check the Agents page to verify:
- Agent appears in list
- Status shows "Online"
- Metrics are being collected

## Troubleshooting

### Server Won't Start

1. Check logs:
   ```bash
   # Linux
   sudo journalctl -u commandrm-server -f

   # Windows
   Get-Content "C:\Program Files\CommandRM\logs\server.log" -Wait
   ```

2. Verify port is available:
   ```bash
   # Linux
   ss -tlnp | grep 8080

   # Windows
   netstat -an | findstr 8080
   ```

3. Check permissions on data directory

### Agent Won't Connect

1. Verify server URL is reachable:
   ```bash
   curl https://your-server:8080/health
   ```

2. Check firewall rules (port 8080 or your custom port)

3. Verify enrollment key is valid and not expired

4. Check agent logs:
   ```bash
   # Linux
   sudo journalctl -u commandrm-agent -f

   # Windows
   Get-Content "C:\Program Files\CommandRM\logs\agent.log" -Wait
   ```

### TLS Certificate Issues

If using Let's Encrypt:
1. Ensure port 80 is accessible for HTTP-01 challenge
2. Verify domain DNS points to server

If using self-signed:
1. Agents accept self-signed certs by default
2. Browsers will show security warning (expected)

### Database Connection Failed

For PostgreSQL:
1. Verify PostgreSQL is running
2. Check credentials in configuration
3. Ensure database exists
4. Check pg_hba.conf allows connections

### Performance Issues

1. For >100 agents, use PostgreSQL instead of SQLite
2. Increase `METRICS_RETENTION_DAYS` cleanup frequency
3. Consider running server behind nginx for caching
