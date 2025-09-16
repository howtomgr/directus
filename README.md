# directus Installation Guide

directus is a free and open-source data platform. Directus provides instant REST and GraphQL APIs for SQL database

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
  - Storage: 1GB for data
  - Network: HTTP/API access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8055 (default directus port)
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

# Install directus
sudo dnf install -y directus

# Enable and start service
sudo systemctl enable --now directus

# Configure firewall
sudo firewall-cmd --permanent --add-port=8055/tcp
sudo firewall-cmd --reload

# Verify installation
directus --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install directus
sudo apt install -y directus

# Enable and start service
sudo systemctl enable --now directus

# Configure firewall
sudo ufw allow 8055

# Verify installation
directus --version
```

### Arch Linux

```bash
# Install directus
sudo pacman -S directus

# Enable and start service
sudo systemctl enable --now directus

# Verify installation
directus --version
```

### Alpine Linux

```bash
# Install directus
apk add --no-cache directus

# Enable and start service
rc-update add directus default
rc-service directus start

# Verify installation
directus --version
```

### openSUSE/SLES

```bash
# Install directus
sudo zypper install -y directus

# Enable and start service
sudo systemctl enable --now directus

# Configure firewall
sudo firewall-cmd --permanent --add-port=8055/tcp
sudo firewall-cmd --reload

# Verify installation
directus --version
```

### macOS

```bash
# Using Homebrew
brew install directus

# Start service
brew services start directus

# Verify installation
directus --version
```

### FreeBSD

```bash
# Using pkg
pkg install directus

# Enable in rc.conf
echo 'directus_enable="YES"' >> /etc/rc.conf

# Start service
service directus start

# Verify installation
directus --version
```

### Windows

```bash
# Using Chocolatey
choco install directus

# Or using Scoop
scoop install directus

# Verify installation
directus --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/directus

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
directus --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable directus

# Start service
sudo systemctl start directus

# Stop service
sudo systemctl stop directus

# Restart service
sudo systemctl restart directus

# Check status
sudo systemctl status directus

# View logs
sudo journalctl -u directus -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add directus default

# Start service
rc-service directus start

# Stop service
rc-service directus stop

# Restart service
rc-service directus restart

# Check status
rc-service directus status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'directus_enable="YES"' >> /etc/rc.conf

# Start service
service directus start

# Stop service
service directus stop

# Restart service
service directus restart

# Check status
service directus status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start directus
brew services stop directus
brew services restart directus

# Check status
brew services list | grep directus
```

### Windows Service Manager

```powershell
# Start service
net start directus

# Stop service
net stop directus

# Using PowerShell
Start-Service directus
Stop-Service directus
Restart-Service directus

# Check status
Get-Service directus
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream directus_backend {
    server 127.0.0.1:8055;
}

server {
    listen 80;
    server_name directus.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name directus.example.com;

    ssl_certificate /etc/ssl/certs/directus.example.com.crt;
    ssl_certificate_key /etc/ssl/private/directus.example.com.key;

    location / {
        proxy_pass http://directus_backend;
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
    ServerName directus.example.com
    Redirect permanent / https://directus.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName directus.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/directus.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/directus.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8055/
    ProxyPassReverse / http://127.0.0.1:8055/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend directus_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/directus.pem
    redirect scheme https if !{ ssl_fc }
    default_backend directus_backend

backend directus_backend
    balance roundrobin
    server directus1 127.0.0.1:8055 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R directus:directus /etc/directus
sudo chmod 750 /etc/directus

# Configure firewall
sudo firewall-cmd --permanent --add-port=8055/tcp
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
sudo systemctl status directus

# View logs
sudo journalctl -u directus -f

# Monitor resource usage
top -p $(pgrep directus)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/directus"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/directus-backup-$DATE.tar.gz" /etc/directus /var/lib/directus

echo "Backup completed: $BACKUP_DIR/directus-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop directus

# Restore from backup
tar -xzf /backup/directus/directus-backup-*.tar.gz -C /

# Start service
sudo systemctl start directus
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u directus -n 100
sudo tail -f /var/log/directus/directus.log

# Check configuration
directus --version

# Check permissions
ls -la /etc/directus
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8055

# Test connectivity
telnet localhost 8055

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep directus)

# Check disk I/O
iotop -p $(pgrep directus)

# Check connections
ss -an | grep 8055
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  directus:
    image: directus:latest
    ports:
      - "8055:8055"
    volumes:
      - ./config:/etc/directus
      - ./data:/var/lib/directus
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update directus

# Debian/Ubuntu
sudo apt update && sudo apt upgrade directus

# Arch Linux
sudo pacman -Syu directus

# Alpine Linux
apk update && apk upgrade directus

# openSUSE
sudo zypper update directus

# FreeBSD
pkg update && pkg upgrade directus

# Always backup before updates
tar -czf /backup/directus-pre-update-$(date +%Y%m%d).tar.gz /etc/directus

# Restart after updates
sudo systemctl restart directus
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/directus

# Clean old logs
find /var/log/directus -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/directus
```

## Additional Resources

- Official Documentation: https://docs.directus.org/
- GitHub Repository: https://github.com/directus/directus
- Community Forum: https://forum.directus.org/
- Best Practices Guide: https://docs.directus.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
