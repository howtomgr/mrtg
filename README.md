# mrtg Installation Guide

mrtg is a free and open-source traffic grapher. MRTG provides tool to monitor traffic load on network links

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
  - RAM: 256MB minimum
  - Storage: 5GB for RRDs
  - Network: SNMP/HTTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default mrtg port)
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

# Install mrtg
sudo dnf install -y mrtg

# Enable and start service
sudo systemctl enable --now mrtg

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
mrtg --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install mrtg
sudo apt install -y mrtg

# Enable and start service
sudo systemctl enable --now mrtg

# Configure firewall
sudo ufw allow 80

# Verify installation
mrtg --version
```

### Arch Linux

```bash
# Install mrtg
sudo pacman -S mrtg

# Enable and start service
sudo systemctl enable --now mrtg

# Verify installation
mrtg --version
```

### Alpine Linux

```bash
# Install mrtg
apk add --no-cache mrtg

# Enable and start service
rc-update add mrtg default
rc-service mrtg start

# Verify installation
mrtg --version
```

### openSUSE/SLES

```bash
# Install mrtg
sudo zypper install -y mrtg

# Enable and start service
sudo systemctl enable --now mrtg

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
mrtg --version
```

### macOS

```bash
# Using Homebrew
brew install mrtg

# Start service
brew services start mrtg

# Verify installation
mrtg --version
```

### FreeBSD

```bash
# Using pkg
pkg install mrtg

# Enable in rc.conf
echo 'mrtg_enable="YES"' >> /etc/rc.conf

# Start service
service mrtg start

# Verify installation
mrtg --version
```

### Windows

```bash
# Using Chocolatey
choco install mrtg

# Or using Scoop
scoop install mrtg

# Verify installation
mrtg --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/mrtg

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
mrtg --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable mrtg

# Start service
sudo systemctl start mrtg

# Stop service
sudo systemctl stop mrtg

# Restart service
sudo systemctl restart mrtg

# Check status
sudo systemctl status mrtg

# View logs
sudo journalctl -u mrtg -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add mrtg default

# Start service
rc-service mrtg start

# Stop service
rc-service mrtg stop

# Restart service
rc-service mrtg restart

# Check status
rc-service mrtg status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'mrtg_enable="YES"' >> /etc/rc.conf

# Start service
service mrtg start

# Stop service
service mrtg stop

# Restart service
service mrtg restart

# Check status
service mrtg status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start mrtg
brew services stop mrtg
brew services restart mrtg

# Check status
brew services list | grep mrtg
```

### Windows Service Manager

```powershell
# Start service
net start mrtg

# Stop service
net stop mrtg

# Using PowerShell
Start-Service mrtg
Stop-Service mrtg
Restart-Service mrtg

# Check status
Get-Service mrtg
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream mrtg_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name mrtg.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name mrtg.example.com;

    ssl_certificate /etc/ssl/certs/mrtg.example.com.crt;
    ssl_certificate_key /etc/ssl/private/mrtg.example.com.key;

    location / {
        proxy_pass http://mrtg_backend;
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
    ServerName mrtg.example.com
    Redirect permanent / https://mrtg.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName mrtg.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/mrtg.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/mrtg.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend mrtg_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/mrtg.pem
    redirect scheme https if !{ ssl_fc }
    default_backend mrtg_backend

backend mrtg_backend
    balance roundrobin
    server mrtg1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R mrtg:mrtg /etc/mrtg
sudo chmod 750 /etc/mrtg

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status mrtg

# View logs
sudo journalctl -u mrtg -f

# Monitor resource usage
top -p $(pgrep mrtg)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/mrtg"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/mrtg-backup-$DATE.tar.gz" /etc/mrtg /var/lib/mrtg

echo "Backup completed: $BACKUP_DIR/mrtg-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop mrtg

# Restore from backup
tar -xzf /backup/mrtg/mrtg-backup-*.tar.gz -C /

# Start service
sudo systemctl start mrtg
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u mrtg -n 100
sudo tail -f /var/log/mrtg/mrtg.log

# Check configuration
mrtg --version

# Check permissions
ls -la /etc/mrtg
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep mrtg)

# Check disk I/O
iotop -p $(pgrep mrtg)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  mrtg:
    image: mrtg:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/mrtg
      - ./data:/var/lib/mrtg
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update mrtg

# Debian/Ubuntu
sudo apt update && sudo apt upgrade mrtg

# Arch Linux
sudo pacman -Syu mrtg

# Alpine Linux
apk update && apk upgrade mrtg

# openSUSE
sudo zypper update mrtg

# FreeBSD
pkg update && pkg upgrade mrtg

# Always backup before updates
tar -czf /backup/mrtg-pre-update-$(date +%Y%m%d).tar.gz /etc/mrtg

# Restart after updates
sudo systemctl restart mrtg
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/mrtg

# Clean old logs
find /var/log/mrtg -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/mrtg
```

## Additional Resources

- Official Documentation: https://docs.mrtg.org/
- GitHub Repository: https://github.com/mrtg/mrtg
- Community Forum: https://forum.mrtg.org/
- Best Practices Guide: https://docs.mrtg.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
