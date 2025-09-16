# rundeck Installation Guide

rundeck is a free and open-source job scheduler and runbook automation. Rundeck enables self-service operations and automates routine procedures, serving as an open-source alternative to commercial runbook automation platforms

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
  - CPU: 2+ cores recommended
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 5GB for installation
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4440 (default rundeck port)
  - SSH access to nodes
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

# Install rundeck
sudo dnf install -y rundeck

# Enable and start service
sudo systemctl enable --now rundeckd

# Configure firewall
sudo firewall-cmd --permanent --add-port=4440/tcp
sudo firewall-cmd --reload

# Verify installation
rd --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install rundeck
sudo apt install -y rundeck

# Enable and start service
sudo systemctl enable --now rundeckd

# Configure firewall
sudo ufw allow 4440

# Verify installation
rd --version
```

### Arch Linux

```bash
# Install rundeck
sudo pacman -S rundeck

# Enable and start service
sudo systemctl enable --now rundeckd

# Verify installation
rd --version
```

### Alpine Linux

```bash
# Install rundeck
apk add --no-cache rundeck

# Enable and start service
rc-update add rundeckd default
rc-service rundeckd start

# Verify installation
rd --version
```

### openSUSE/SLES

```bash
# Install rundeck
sudo zypper install -y rundeck

# Enable and start service
sudo systemctl enable --now rundeckd

# Configure firewall
sudo firewall-cmd --permanent --add-port=4440/tcp
sudo firewall-cmd --reload

# Verify installation
rd --version
```

### macOS

```bash
# Using Homebrew
brew install rundeck

# Start service
brew services start rundeck

# Verify installation
rd --version
```

### FreeBSD

```bash
# Using pkg
pkg install rundeck

# Enable in rc.conf
echo 'rundeckd_enable="YES"' >> /etc/rc.conf

# Start service
service rundeckd start

# Verify installation
rd --version
```

### Windows

```bash
# Using Chocolatey
choco install rundeck

# Or using Scoop
scoop install rundeck

# Verify installation
rd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/rundeck

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
rd --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable rundeckd

# Start service
sudo systemctl start rundeckd

# Stop service
sudo systemctl stop rundeckd

# Restart service
sudo systemctl restart rundeckd

# Check status
sudo systemctl status rundeckd

# View logs
sudo journalctl -u rundeckd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add rundeckd default

# Start service
rc-service rundeckd start

# Stop service
rc-service rundeckd stop

# Restart service
rc-service rundeckd restart

# Check status
rc-service rundeckd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'rundeckd_enable="YES"' >> /etc/rc.conf

# Start service
service rundeckd start

# Stop service
service rundeckd stop

# Restart service
service rundeckd restart

# Check status
service rundeckd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start rundeck
brew services stop rundeck
brew services restart rundeck

# Check status
brew services list | grep rundeck
```

### Windows Service Manager

```powershell
# Start service
net start rundeckd

# Stop service
net stop rundeckd

# Using PowerShell
Start-Service rundeckd
Stop-Service rundeckd
Restart-Service rundeckd

# Check status
Get-Service rundeckd
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream rundeck_backend {
    server 127.0.0.1:4440;
}

server {
    listen 80;
    server_name rundeck.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name rundeck.example.com;

    ssl_certificate /etc/ssl/certs/rundeck.example.com.crt;
    ssl_certificate_key /etc/ssl/private/rundeck.example.com.key;

    location / {
        proxy_pass http://rundeck_backend;
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
    ServerName rundeck.example.com
    Redirect permanent / https://rundeck.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName rundeck.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/rundeck.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/rundeck.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4440/
    ProxyPassReverse / http://127.0.0.1:4440/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend rundeck_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/rundeck.pem
    redirect scheme https if !{ ssl_fc }
    default_backend rundeck_backend

backend rundeck_backend
    balance roundrobin
    server rundeck1 127.0.0.1:4440 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R rundeck:rundeck /etc/rundeck
sudo chmod 750 /etc/rundeck

# Configure firewall
sudo firewall-cmd --permanent --add-port=4440/tcp
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
sudo systemctl status rundeckd

# View logs
sudo journalctl -u rundeckd -f

# Monitor resource usage
top -p $(pgrep rundeck)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/rundeck"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/rundeck-backup-$DATE.tar.gz" /etc/rundeck /var/lib/rundeck

echo "Backup completed: $BACKUP_DIR/rundeck-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop rundeckd

# Restore from backup
tar -xzf /backup/rundeck/rundeck-backup-*.tar.gz -C /

# Start service
sudo systemctl start rundeckd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u rundeckd -n 100
sudo tail -f /var/log/rundeck/rundeck.log

# Check configuration
rd --version

# Check permissions
ls -la /etc/rundeck
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4440

# Test connectivity
telnet localhost 4440

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep rundeck)

# Check disk I/O
iotop -p $(pgrep rundeck)

# Check connections
ss -an | grep 4440
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  rundeck:
    image: rundeck:latest
    ports:
      - "4440:4440"
    volumes:
      - ./config:/etc/rundeck
      - ./data:/var/lib/rundeck
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update rundeck

# Debian/Ubuntu
sudo apt update && sudo apt upgrade rundeck

# Arch Linux
sudo pacman -Syu rundeck

# Alpine Linux
apk update && apk upgrade rundeck

# openSUSE
sudo zypper update rundeck

# FreeBSD
pkg update && pkg upgrade rundeck

# Always backup before updates
tar -czf /backup/rundeck-pre-update-$(date +%Y%m%d).tar.gz /etc/rundeck

# Restart after updates
sudo systemctl restart rundeckd
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/rundeck

# Clean old logs
find /var/log/rundeck -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/rundeck
```

## Additional Resources

- Official Documentation: https://docs.rundeck.org/
- GitHub Repository: https://github.com/rundeck/rundeck
- Community Forum: https://forum.rundeck.org/
- Best Practices Guide: https://docs.rundeck.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
