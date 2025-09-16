# gitlab-runner Installation Guide

gitlab-runner is a free and open-source multi-platform build agent for GitLab CI/CD. GitLab Runner executes CI/CD jobs defined in GitLab pipelines, providing a FOSS alternative to GitHub Actions runners or CircleCI executors

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
  - CPU: 1 core minimum (2+ for concurrent jobs)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 10GB for builds and caches
  - Network: HTTPS access to GitLab instance
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port N/A (default gitlab-runner port)
  - Metrics on port 9252
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

# Install gitlab-runner
sudo dnf install -y gitlab-runner

# Enable and start service
sudo systemctl enable --now gitlab-runner

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
gitlab-runner --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install gitlab-runner
sudo apt install -y gitlab-runner

# Enable and start service
sudo systemctl enable --now gitlab-runner

# Configure firewall
sudo ufw allow N/A

# Verify installation
gitlab-runner --version
```

### Arch Linux

```bash
# Install gitlab-runner
sudo pacman -S gitlab-runner

# Enable and start service
sudo systemctl enable --now gitlab-runner

# Verify installation
gitlab-runner --version
```

### Alpine Linux

```bash
# Install gitlab-runner
apk add --no-cache gitlab-runner

# Enable and start service
rc-update add gitlab-runner default
rc-service gitlab-runner start

# Verify installation
gitlab-runner --version
```

### openSUSE/SLES

```bash
# Install gitlab-runner
sudo zypper install -y gitlab-runner

# Enable and start service
sudo systemctl enable --now gitlab-runner

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
gitlab-runner --version
```

### macOS

```bash
# Using Homebrew
brew install gitlab-runner

# Start service
brew services start gitlab-runner

# Verify installation
gitlab-runner --version
```

### FreeBSD

```bash
# Using pkg
pkg install gitlab-runner

# Enable in rc.conf
echo 'gitlab-runner_enable="YES"' >> /etc/rc.conf

# Start service
service gitlab-runner start

# Verify installation
gitlab-runner --version
```

### Windows

```bash
# Using Chocolatey
choco install gitlab-runner

# Or using Scoop
scoop install gitlab-runner

# Verify installation
gitlab-runner --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/gitlab-runner

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
gitlab-runner --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable gitlab-runner

# Start service
sudo systemctl start gitlab-runner

# Stop service
sudo systemctl stop gitlab-runner

# Restart service
sudo systemctl restart gitlab-runner

# Check status
sudo systemctl status gitlab-runner

# View logs
sudo journalctl -u gitlab-runner -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add gitlab-runner default

# Start service
rc-service gitlab-runner start

# Stop service
rc-service gitlab-runner stop

# Restart service
rc-service gitlab-runner restart

# Check status
rc-service gitlab-runner status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'gitlab-runner_enable="YES"' >> /etc/rc.conf

# Start service
service gitlab-runner start

# Stop service
service gitlab-runner stop

# Restart service
service gitlab-runner restart

# Check status
service gitlab-runner status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start gitlab-runner
brew services stop gitlab-runner
brew services restart gitlab-runner

# Check status
brew services list | grep gitlab-runner
```

### Windows Service Manager

```powershell
# Start service
net start gitlab-runner

# Stop service
net stop gitlab-runner

# Using PowerShell
Start-Service gitlab-runner
Stop-Service gitlab-runner
Restart-Service gitlab-runner

# Check status
Get-Service gitlab-runner
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream gitlab-runner_backend {
    server 127.0.0.1:N/A;
}

server {
    listen 80;
    server_name gitlab-runner.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name gitlab-runner.example.com;

    ssl_certificate /etc/ssl/certs/gitlab-runner.example.com.crt;
    ssl_certificate_key /etc/ssl/private/gitlab-runner.example.com.key;

    location / {
        proxy_pass http://gitlab-runner_backend;
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
    ServerName gitlab-runner.example.com
    Redirect permanent / https://gitlab-runner.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName gitlab-runner.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/gitlab-runner.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/gitlab-runner.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:N/A/
    ProxyPassReverse / http://127.0.0.1:N/A/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend gitlab-runner_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/gitlab-runner.pem
    redirect scheme https if !{ ssl_fc }
    default_backend gitlab-runner_backend

backend gitlab-runner_backend
    balance roundrobin
    server gitlab-runner1 127.0.0.1:N/A check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R gitlab-runner:gitlab-runner /etc/gitlab-runner
sudo chmod 750 /etc/gitlab-runner

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
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
sudo systemctl status gitlab-runner

# View logs
sudo journalctl -u gitlab-runner -f

# Monitor resource usage
top -p $(pgrep gitlab-runner)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/gitlab-runner"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/gitlab-runner-backup-$DATE.tar.gz" /etc/gitlab-runner /var/lib/gitlab-runner

echo "Backup completed: $BACKUP_DIR/gitlab-runner-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop gitlab-runner

# Restore from backup
tar -xzf /backup/gitlab-runner/gitlab-runner-backup-*.tar.gz -C /

# Start service
sudo systemctl start gitlab-runner
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u gitlab-runner -n 100
sudo tail -f /var/log/gitlab-runner/gitlab-runner.log

# Check configuration
gitlab-runner --version

# Check permissions
ls -la /etc/gitlab-runner
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep N/A

# Test connectivity
telnet localhost N/A

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep gitlab-runner)

# Check disk I/O
iotop -p $(pgrep gitlab-runner)

# Check connections
ss -an | grep N/A
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  gitlab-runner:
    image: gitlab-runner:latest
    ports:
      - "N/A:N/A"
    volumes:
      - ./config:/etc/gitlab-runner
      - ./data:/var/lib/gitlab-runner
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update gitlab-runner

# Debian/Ubuntu
sudo apt update && sudo apt upgrade gitlab-runner

# Arch Linux
sudo pacman -Syu gitlab-runner

# Alpine Linux
apk update && apk upgrade gitlab-runner

# openSUSE
sudo zypper update gitlab-runner

# FreeBSD
pkg update && pkg upgrade gitlab-runner

# Always backup before updates
tar -czf /backup/gitlab-runner-pre-update-$(date +%Y%m%d).tar.gz /etc/gitlab-runner

# Restart after updates
sudo systemctl restart gitlab-runner
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/gitlab-runner

# Clean old logs
find /var/log/gitlab-runner -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/gitlab-runner
```

## Additional Resources

- Official Documentation: https://docs.gitlab-runner.org/
- GitHub Repository: https://github.com/gitlab-runner/gitlab-runner
- Community Forum: https://forum.gitlab-runner.org/
- Best Practices Guide: https://docs.gitlab-runner.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
