# ossec Installation Guide

ossec is a free and open-source host-based intrusion detection system. OSSEC provides log analysis, integrity checking, and rootkit detection, serving as an open-source alternative to commercial HIDS solutions

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
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 1GB for installation
  - Network: Agent communication
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 1514 (default ossec port)
  - Port 1515 for auth
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

# Install ossec
sudo dnf install -y ossec-hids

# Enable and start service
sudo systemctl enable --now ossec

# Configure firewall
sudo firewall-cmd --permanent --add-port=1514/tcp
sudo firewall-cmd --reload

# Verify installation
ossec-control info
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install ossec
sudo apt install -y ossec-hids

# Enable and start service
sudo systemctl enable --now ossec

# Configure firewall
sudo ufw allow 1514

# Verify installation
ossec-control info
```

### Arch Linux

```bash
# Install ossec
sudo pacman -S ossec-hids

# Enable and start service
sudo systemctl enable --now ossec

# Verify installation
ossec-control info
```

### Alpine Linux

```bash
# Install ossec
apk add --no-cache ossec-hids

# Enable and start service
rc-update add ossec default
rc-service ossec start

# Verify installation
ossec-control info
```

### openSUSE/SLES

```bash
# Install ossec
sudo zypper install -y ossec-hids

# Enable and start service
sudo systemctl enable --now ossec

# Configure firewall
sudo firewall-cmd --permanent --add-port=1514/tcp
sudo firewall-cmd --reload

# Verify installation
ossec-control info
```

### macOS

```bash
# Using Homebrew
brew install ossec-hids

# Start service
brew services start ossec-hids

# Verify installation
ossec-control info
```

### FreeBSD

```bash
# Using pkg
pkg install ossec-hids

# Enable in rc.conf
echo 'ossec_enable="YES"' >> /etc/rc.conf

# Start service
service ossec start

# Verify installation
ossec-control info
```

### Windows

```bash
# Using Chocolatey
choco install ossec-hids

# Or using Scoop
scoop install ossec-hids

# Verify installation
ossec-control info
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/ossec-hids

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
ossec-control info
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ossec

# Start service
sudo systemctl start ossec

# Stop service
sudo systemctl stop ossec

# Restart service
sudo systemctl restart ossec

# Check status
sudo systemctl status ossec

# View logs
sudo journalctl -u ossec -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ossec default

# Start service
rc-service ossec start

# Stop service
rc-service ossec stop

# Restart service
rc-service ossec restart

# Check status
rc-service ossec status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ossec_enable="YES"' >> /etc/rc.conf

# Start service
service ossec start

# Stop service
service ossec stop

# Restart service
service ossec restart

# Check status
service ossec status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start ossec-hids
brew services stop ossec-hids
brew services restart ossec-hids

# Check status
brew services list | grep ossec-hids
```

### Windows Service Manager

```powershell
# Start service
net start ossec

# Stop service
net stop ossec

# Using PowerShell
Start-Service ossec
Stop-Service ossec
Restart-Service ossec

# Check status
Get-Service ossec
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream ossec-hids_backend {
    server 127.0.0.1:1514;
}

server {
    listen 80;
    server_name ossec-hids.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ossec-hids.example.com;

    ssl_certificate /etc/ssl/certs/ossec-hids.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ossec-hids.example.com.key;

    location / {
        proxy_pass http://ossec-hids_backend;
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
    ServerName ossec-hids.example.com
    Redirect permanent / https://ossec-hids.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName ossec-hids.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ossec-hids.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/ossec-hids.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:1514/
    ProxyPassReverse / http://127.0.0.1:1514/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend ossec-hids_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/ossec-hids.pem
    redirect scheme https if !{ ssl_fc }
    default_backend ossec-hids_backend

backend ossec-hids_backend
    balance roundrobin
    server ossec-hids1 127.0.0.1:1514 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R ossec-hids:ossec-hids /etc/ossec-hids
sudo chmod 750 /etc/ossec-hids

# Configure firewall
sudo firewall-cmd --permanent --add-port=1514/tcp
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
sudo systemctl status ossec

# View logs
sudo journalctl -u ossec -f

# Monitor resource usage
top -p $(pgrep ossec-hids)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/ossec-hids"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/ossec-hids-backup-$DATE.tar.gz" /etc/ossec-hids /var/lib/ossec-hids

echo "Backup completed: $BACKUP_DIR/ossec-hids-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ossec

# Restore from backup
tar -xzf /backup/ossec-hids/ossec-hids-backup-*.tar.gz -C /

# Start service
sudo systemctl start ossec
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ossec -n 100
sudo tail -f /var/log/ossec-hids/ossec-hids.log

# Check configuration
ossec-control info

# Check permissions
ls -la /etc/ossec-hids
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 1514

# Test connectivity
telnet localhost 1514

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep ossec-hids)

# Check disk I/O
iotop -p $(pgrep ossec-hids)

# Check connections
ss -an | grep 1514
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  ossec-hids:
    image: ossec-hids:latest
    ports:
      - "1514:1514"
    volumes:
      - ./config:/etc/ossec-hids
      - ./data:/var/lib/ossec-hids
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update ossec-hids

# Debian/Ubuntu
sudo apt update && sudo apt upgrade ossec-hids

# Arch Linux
sudo pacman -Syu ossec-hids

# Alpine Linux
apk update && apk upgrade ossec-hids

# openSUSE
sudo zypper update ossec-hids

# FreeBSD
pkg update && pkg upgrade ossec-hids

# Always backup before updates
tar -czf /backup/ossec-hids-pre-update-$(date +%Y%m%d).tar.gz /etc/ossec-hids

# Restart after updates
sudo systemctl restart ossec
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/ossec-hids

# Clean old logs
find /var/log/ossec-hids -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/ossec-hids
```

## Additional Resources

- Official Documentation: https://docs.ossec-hids.org/
- GitHub Repository: https://github.com/ossec-hids/ossec-hids
- Community Forum: https://forum.ossec-hids.org/
- Best Practices Guide: https://docs.ossec-hids.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
