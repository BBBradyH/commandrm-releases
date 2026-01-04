# CommandRM

A cross-platform remote management solution for IT administrators.

## What is CommandRM?

CommandRM is a comprehensive remote management platform that enables IT teams to:

- **Monitor Systems** - Real-time CPU, memory, disk, and network metrics with historical charts
- **Remote Terminal** - Web-based SSH-like access with session recording
- **File Management** - Browse, upload, and download files with chunked transfer support
- **Remote Desktop** - WebRTC-based screen sharing
- **Script Execution** - Create and run scripts across multiple machines
- **Software Inventory** - Track installed packages across all managed systems
- **Patch Management** - Deploy updates with staged rollouts
- **Alerts & Notifications** - Threshold-based alerts with email/webhook delivery

## Release Artifacts

### Server Components

| Binary | Description |
|--------|-------------|
| `commandrm-server-linux-amd64` | Server binary for Linux x64 |
| `commandrm-server-linux-arm64` | Server binary for Linux ARM64 |
| `commandrm-server-darwin-amd64` | Server binary for macOS Intel |
| `commandrm-server-darwin-arm64` | Server binary for macOS Apple Silicon |
| `commandrm-server-windows-amd64.exe` | Server binary for Windows x64 |

The server hosts the web interface, REST API, and WebSocket hub for agent connections.

### Agent Components

| Binary | Description |
|--------|-------------|
| `commandrm-agent-linux-amd64` | Agent binary for Linux x64 |
| `commandrm-agent-linux-arm64` | Agent binary for Linux ARM64 |
| `commandrm-agent-darwin-amd64` | Agent binary for macOS Intel |
| `commandrm-agent-darwin-arm64` | Agent binary for macOS Apple Silicon |
| `commandrm-agent-windows-amd64.exe` | Agent binary for Windows x64 |

The agent runs on managed machines, collecting metrics and executing commands.

### One-Click Installers

| Binary | Description |
|--------|-------------|
| `commandrm-server-setup-linux-amd64` | Interactive server installer for Linux |
| `commandrm-server-setup-darwin-amd64` | Interactive server installer for macOS |
| `commandrm-server-setup-windows-amd64.exe` | Interactive server installer for Windows |
| `commandrm-agent-setup-linux-amd64` | Agent installer for Linux (supports silent mode) |
| `commandrm-agent-setup-darwin-amd64` | Agent installer for macOS (supports silent mode) |
| `commandrm-agent-setup-windows-amd64.exe` | Agent installer for Windows (supports silent mode) |

Installers handle TLS certificates, database setup, service registration, and firewall configuration.

### Desktop Application

| Binary | Description |
|--------|-------------|
| `commandrm-desktop-linux-amd64` | Desktop app for Linux x64 |
| `commandrm-desktop-darwin-amd64.zip` | Desktop app for macOS Intel (.app bundle) |
| `commandrm-desktop-darwin-arm64.zip` | Desktop app for macOS Apple Silicon (.app bundle) |
| `commandrm-desktop-windows-amd64.exe` | Desktop app for Windows x64 |

Native desktop client with multi-server support, system tray, and notifications.

### Other Files

| File | Description |
|------|-------------|
| `commandrm-web.tar.gz` | Pre-built web frontend (for custom deployments) |
| `checksums.txt` | SHA256 checksums for all binaries |
| `checksums.txt.asc` | GPG signature for checksums (if signing is configured) |

## Quick Start

### Server Installation (One-Click)

```bash
# Linux/macOS
chmod +x commandrm-server-setup-linux-amd64
sudo ./commandrm-server-setup-linux-amd64

# Windows (Run as Administrator)
commandrm-server-setup-windows-amd64.exe
```

### Agent Installation

```bash
# Interactive
sudo ./commandrm-agent-setup-linux-amd64

# Silent (for mass deployment)
sudo ./commandrm-agent-setup-linux-amd64 --silent --server https://your-server:8080 --key YOUR_ENROLLMENT_KEY
```

## Verifying Downloads

Verify checksums before installing:

```bash
# Download checksums
curl -LO https://github.com/BBBradyH/commandrm-releases/releases/latest/download/checksums.txt

# Verify a specific binary
sha256sum -c checksums.txt --ignore-missing
```

## Documentation

- [Installation Guide](docs/INSTALL.md) - Detailed installation instructions
- [Configuration Reference](docs/CONFIG.md) - Environment variables and settings
- [API Documentation](docs/API.md) - REST API endpoints
- [Security Guide](docs/SECURITY.md) - Security best practices

## License

MIT License
