# valheim Installation Guide

valheim is a free and open-source Valheim dedicated server. Dedicated server for Valheim Viking survival game

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 4GB minimum
  - Storage: 10GB for worlds
  - Network: Steam protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 2456 (default valheim port)
  - Query on 2457
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install valheim
sudo dnf install -y valheim

# Enable and start service
sudo systemctl enable --now valheim

# Configure firewall
sudo firewall-cmd --permanent --add-port=2456/tcp
sudo firewall-cmd --reload

# Verify installation
valheim --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install valheim
sudo apt install -y valheim

# Enable and start service
sudo systemctl enable --now valheim

# Configure firewall
sudo ufw allow 2456

# Verify installation
valheim --version
```

### Arch Linux

```bash
# Install valheim
sudo pacman -S valheim

# Enable and start service
sudo systemctl enable --now valheim

# Verify installation
valheim --version
```

### Alpine Linux

```bash
# Install valheim
apk add --no-cache valheim

# Enable and start service
rc-update add valheim default
rc-service valheim start

# Verify installation
valheim --version
```

### openSUSE/SLES

```bash
# Install valheim
sudo zypper install -y valheim

# Enable and start service
sudo systemctl enable --now valheim

# Configure firewall
sudo firewall-cmd --permanent --add-port=2456/tcp
sudo firewall-cmd --reload

# Verify installation
valheim --version
```

### macOS

```bash
# Using Homebrew
brew install valheim

# Start service
brew services start valheim

# Verify installation
valheim --version
```

### FreeBSD

```bash
# Using pkg
pkg install valheim

# Enable in rc.conf
echo 'valheim_enable="YES"' >> /etc/rc.conf

# Start service
service valheim start

# Verify installation
valheim --version
```

### Windows

```bash
# Using Chocolatey
choco install valheim

# Or using Scoop
scoop install valheim

# Verify installation
valheim --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/valheim

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
valheim --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable valheim

# Start service
sudo systemctl start valheim

# Stop service
sudo systemctl stop valheim

# Restart service
sudo systemctl restart valheim

# Check status
sudo systemctl status valheim

# View logs
sudo journalctl -u valheim -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add valheim default

# Start service
rc-service valheim start

# Stop service
rc-service valheim stop

# Restart service
rc-service valheim restart

# Check status
rc-service valheim status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'valheim_enable="YES"' >> /etc/rc.conf

# Start service
service valheim start

# Stop service
service valheim stop

# Restart service
service valheim restart

# Check status
service valheim status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start valheim
brew services stop valheim
brew services restart valheim

# Check status
brew services list | grep valheim
```

### Windows Service Manager

```powershell
# Start service
net start valheim

# Stop service
net stop valheim

# Using PowerShell
Start-Service valheim
Stop-Service valheim
Restart-Service valheim

# Check status
Get-Service valheim
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream valheim_backend {
    server 127.0.0.1:2456;
}

server {
    listen 80;
    server_name valheim.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name valheim.example.com;

    ssl_certificate /etc/ssl/certs/valheim.example.com.crt;
    ssl_certificate_key /etc/ssl/private/valheim.example.com.key;

    location / {
        proxy_pass http://valheim_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName valheim.example.com
    Redirect permanent / https://valheim.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName valheim.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/valheim.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/valheim.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:2456/
    ProxyPassReverse / http://127.0.0.1:2456/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend valheim_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/valheim.pem
    redirect scheme https if !{ ssl_fc }
    default_backend valheim_backend

backend valheim_backend
    balance roundrobin
    server valheim1 127.0.0.1:2456 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R valheim:valheim /etc/valheim
sudo chmod 750 /etc/valheim

# Configure firewall
sudo firewall-cmd --permanent --add-port=2456/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status valheim

# View logs
sudo journalctl -u valheim -f

# Monitor resource usage
top -p $(pgrep valheim)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/valheim"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/valheim-backup-$DATE.tar.gz" /etc/valheim /var/lib/valheim

echo "Backup completed: $BACKUP_DIR/valheim-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop valheim

# Restore from backup
tar -xzf /backup/valheim/valheim-backup-*.tar.gz -C /

# Start service
sudo systemctl start valheim
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u valheim -n 100
sudo tail -f /var/log/valheim/valheim.log

# Check configuration
valheim --version

# Check permissions
ls -la /etc/valheim
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 2456

# Test connectivity
telnet localhost 2456

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep valheim)

# Check disk I/O
iotop -p $(pgrep valheim)

# Check connections
ss -an | grep 2456
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  valheim:
    image: valheim:latest
    ports:
      - "2456:2456"
    volumes:
      - ./config:/etc/valheim
      - ./data:/var/lib/valheim
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update valheim

# Debian/Ubuntu
sudo apt update && sudo apt upgrade valheim

# Arch Linux
sudo pacman -Syu valheim

# Alpine Linux
apk update && apk upgrade valheim

# openSUSE
sudo zypper update valheim

# FreeBSD
pkg update && pkg upgrade valheim

# Always backup before updates
tar -czf /backup/valheim-pre-update-$(date +%Y%m%d).tar.gz /etc/valheim

# Restart after updates
sudo systemctl restart valheim
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/valheim

# Clean old logs
find /var/log/valheim -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/valheim
```

## Additional Resources

- Official Documentation: https://docs.valheim.org/
- GitHub Repository: https://github.com/valheim/valheim
- Community Forum: https://forum.valheim.org/
- Best Practices Guide: https://docs.valheim.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
