# komga Installation Guide

komga is a free and open-source comics/ebooks server. Komga provides a media server for comics, manga, and ebooks with OPDS support

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
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 1GB for app
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default komga port)
  - None
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

# Install komga
sudo dnf install -y komga

# Enable and start service
sudo systemctl enable --now komga

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
komga --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install komga
sudo apt install -y komga

# Enable and start service
sudo systemctl enable --now komga

# Configure firewall
sudo ufw allow 8080

# Verify installation
komga --version
```

### Arch Linux

```bash
# Install komga
sudo pacman -S komga

# Enable and start service
sudo systemctl enable --now komga

# Verify installation
komga --version
```

### Alpine Linux

```bash
# Install komga
apk add --no-cache komga

# Enable and start service
rc-update add komga default
rc-service komga start

# Verify installation
komga --version
```

### openSUSE/SLES

```bash
# Install komga
sudo zypper install -y komga

# Enable and start service
sudo systemctl enable --now komga

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
komga --version
```

### macOS

```bash
# Using Homebrew
brew install komga

# Start service
brew services start komga

# Verify installation
komga --version
```

### FreeBSD

```bash
# Using pkg
pkg install komga

# Enable in rc.conf
echo 'komga_enable="YES"' >> /etc/rc.conf

# Start service
service komga start

# Verify installation
komga --version
```

### Windows

```bash
# Using Chocolatey
choco install komga

# Or using Scoop
scoop install komga

# Verify installation
komga --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/komga

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
komga --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable komga

# Start service
sudo systemctl start komga

# Stop service
sudo systemctl stop komga

# Restart service
sudo systemctl restart komga

# Check status
sudo systemctl status komga

# View logs
sudo journalctl -u komga -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add komga default

# Start service
rc-service komga start

# Stop service
rc-service komga stop

# Restart service
rc-service komga restart

# Check status
rc-service komga status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'komga_enable="YES"' >> /etc/rc.conf

# Start service
service komga start

# Stop service
service komga stop

# Restart service
service komga restart

# Check status
service komga status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start komga
brew services stop komga
brew services restart komga

# Check status
brew services list | grep komga
```

### Windows Service Manager

```powershell
# Start service
net start komga

# Stop service
net stop komga

# Using PowerShell
Start-Service komga
Stop-Service komga
Restart-Service komga

# Check status
Get-Service komga
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream komga_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name komga.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name komga.example.com;

    ssl_certificate /etc/ssl/certs/komga.example.com.crt;
    ssl_certificate_key /etc/ssl/private/komga.example.com.key;

    location / {
        proxy_pass http://komga_backend;
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
    ServerName komga.example.com
    Redirect permanent / https://komga.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName komga.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/komga.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/komga.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend komga_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/komga.pem
    redirect scheme https if !{ ssl_fc }
    default_backend komga_backend

backend komga_backend
    balance roundrobin
    server komga1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R komga:komga /etc/komga
sudo chmod 750 /etc/komga

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
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
sudo systemctl status komga

# View logs
sudo journalctl -u komga -f

# Monitor resource usage
top -p $(pgrep komga)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/komga"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/komga-backup-$DATE.tar.gz" /etc/komga /var/lib/komga

echo "Backup completed: $BACKUP_DIR/komga-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop komga

# Restore from backup
tar -xzf /backup/komga/komga-backup-*.tar.gz -C /

# Start service
sudo systemctl start komga
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u komga -n 100
sudo tail -f /var/log/komga/komga.log

# Check configuration
komga --version

# Check permissions
ls -la /etc/komga
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep komga)

# Check disk I/O
iotop -p $(pgrep komga)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  komga:
    image: komga:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/komga
      - ./data:/var/lib/komga
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update komga

# Debian/Ubuntu
sudo apt update && sudo apt upgrade komga

# Arch Linux
sudo pacman -Syu komga

# Alpine Linux
apk update && apk upgrade komga

# openSUSE
sudo zypper update komga

# FreeBSD
pkg update && pkg upgrade komga

# Always backup before updates
tar -czf /backup/komga-pre-update-$(date +%Y%m%d).tar.gz /etc/komga

# Restart after updates
sudo systemctl restart komga
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/komga

# Clean old logs
find /var/log/komga -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/komga
```

## Additional Resources

- Official Documentation: https://docs.komga.org/
- GitHub Repository: https://github.com/komga/komga
- Community Forum: https://forum.komga.org/
- Best Practices Guide: https://docs.komga.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
