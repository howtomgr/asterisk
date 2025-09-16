# Asterisk Installation Guide

Asterisk is a free and open-source PBX. Open source PBX and telephony toolkit

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 5060/5038 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5060/5038 (default asterisk port)
- **Dependencies**:
  - asterisk-core, asterisk-modules
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

# Install asterisk
sudo dnf install -y asterisk asterisk-core, asterisk-modules

# Enable and start service
sudo systemctl enable --now asterisk

# Configure firewall
sudo firewall-cmd --permanent --add-service=asterisk
sudo firewall-cmd --reload

# Verify installation
asterisk --version || systemctl status asterisk
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install asterisk
sudo apt install -y asterisk asterisk-core, asterisk-modules

# Enable and start service
sudo systemctl enable --now asterisk

# Configure firewall
sudo ufw allow 5060/5038

# Verify installation
asterisk --version || systemctl status asterisk
```

### Arch Linux

```bash
# Install asterisk
sudo pacman -S asterisk

# Enable and start service
sudo systemctl enable --now asterisk

# Verify installation
asterisk --version || systemctl status asterisk
```

### Alpine Linux

```bash
# Install asterisk
apk add --no-cache asterisk

# Enable and start service
rc-update add asterisk default
rc-service asterisk start

# Verify installation
asterisk --version || rc-service asterisk status
```

### openSUSE/SLES

```bash
# Install asterisk
sudo zypper install -y asterisk asterisk-core, asterisk-modules

# Enable and start service
sudo systemctl enable --now asterisk

# Configure firewall
sudo firewall-cmd --permanent --add-service=asterisk
sudo firewall-cmd --reload

# Verify installation
asterisk --version || systemctl status asterisk
```

### macOS

```bash
# Using Homebrew
brew install asterisk

# Start service
brew services start asterisk

# Verify installation
asterisk --version
```

### FreeBSD

```bash
# Using pkg
pkg install asterisk

# Enable in rc.conf
echo 'asterisk_enable="YES"' >> /etc/rc.conf

# Start service
service asterisk start

# Verify installation
asterisk --version || service asterisk status
```

### Windows

```powershell
# Using Chocolatey
choco install asterisk

# Or using Scoop
scoop install asterisk

# Verify installation
asterisk --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/asterisk

# Set up basic configuration
sudo tee /etc/asterisk/asterisk.conf << 'EOF'
# Asterisk Configuration
maxcalls=1000
EOF

# Test configuration
sudo asterisk -t || sudo asterisk configtest

# Reload service
sudo systemctl reload asterisk
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R asterisk:asterisk /etc/asterisk
sudo chmod 750 /etc/asterisk

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable asterisk

# Start service
sudo systemctl start asterisk

# Stop service
sudo systemctl stop asterisk

# Restart service
sudo systemctl restart asterisk

# Reload configuration
sudo systemctl reload asterisk

# Check status
sudo systemctl status asterisk

# View logs
sudo journalctl -u asterisk -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add asterisk default

# Start service
rc-service asterisk start

# Stop service
rc-service asterisk stop

# Restart service
rc-service asterisk restart

# Check status
rc-service asterisk status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'asterisk_enable="YES"' >> /etc/rc.conf

# Start service
service asterisk start

# Stop service
service asterisk stop

# Restart service
service asterisk restart

# Check status
service asterisk status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start asterisk
brew services stop asterisk
brew services restart asterisk

# Check status
brew services list | grep asterisk
```

### Windows Service Manager

```powershell
# Start service
net start asterisk

# Stop service
net stop asterisk

# Using PowerShell
Start-Service asterisk
Stop-Service asterisk
Restart-Service asterisk

# Check status
Get-Service asterisk
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/asterisk/asterisk.conf << 'EOF'
maxcalls=1000
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart asterisk
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream asterisk_backend {
    server 127.0.0.1:5060/5038;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name asterisk.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name asterisk.example.com;

    ssl_certificate /etc/ssl/certs/asterisk.example.com.crt;
    ssl_certificate_key /etc/ssl/private/asterisk.example.com.key;

    location / {
        proxy_pass http://asterisk_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName asterisk.example.com
    Redirect permanent / https://asterisk.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName asterisk.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/asterisk.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/asterisk.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5060/5038/
    ProxyPassReverse / http://127.0.0.1:5060/5038/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:5060/5038/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend asterisk_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/asterisk.pem
    redirect scheme https if !{ ssl_fc }
    default_backend asterisk_backend

backend asterisk_backend
    balance roundrobin
    option httpchk GET /health
    server asterisk1 127.0.0.1:5060/5038 check
    server asterisk2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R asterisk:asterisk /etc/asterisk
sudo chmod 750 /etc/asterisk

# Configure firewall
sudo firewall-cmd --permanent --add-service=asterisk
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/asterisk.conf << 'EOF'
[asterisk]
enabled = true
port = 5060/5038
filter = asterisk
logpath = /var/log/asterisk/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/asterisk.key \
    -out /etc/ssl/certs/asterisk.crt

# Configure SSL in asterisk
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE asterisk_db;
CREATE USER asterisk_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE asterisk_db TO asterisk_user;
EOF

# Configure asterisk to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE asterisk_db;
CREATE USER 'asterisk_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON asterisk_db.* TO 'asterisk_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Asterisk specific tuning
maxcalls=1000
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
asterisk soft nofile 65535
asterisk hard nofile 65535
asterisk soft nproc 32768
asterisk hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'asterisk'
    static_configs:
      - targets: ['localhost:5060/5038']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet asterisk; then
    echo "Asterisk is running"
    exit 0
else
    echo "Asterisk is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/asterisk << 'EOF'
/var/log/asterisk/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 asterisk asterisk
    postrotate
        systemctl reload asterisk > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Asterisk backup script
BACKUP_DIR="/backup/asterisk"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop asterisk

# Backup configuration
tar -czf "$BACKUP_DIR/asterisk-config-$DATE.tar.gz" /etc/asterisk

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/asterisk-data-$DATE.tar.gz" /var/lib/asterisk

# Start service
systemctl start asterisk

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop asterisk

# Restore configuration
sudo tar -xzf /backup/asterisk/asterisk-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/asterisk/asterisk-data-*.tar.gz -C /

# Set permissions
sudo chown -R asterisk:asterisk /etc/asterisk
sudo chown -R asterisk:asterisk /var/lib/asterisk

# Start service
sudo systemctl start asterisk
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u asterisk -n 100
sudo tail -f /var/log/asterisk/*.log

# Check configuration
sudo asterisk -t || sudo asterisk configtest

# Check permissions
ls -la /etc/asterisk
ls -la /var/lib/asterisk
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5060/5038
sudo netstat -tlnp | grep 5060/5038

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 5060/5038
nc -zv localhost 5060/5038
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep asterisk)
htop -p $(pgrep asterisk)

# Check connections
ss -ant | grep :5060/5038 | wc -l

# Monitor I/O
iotop -p $(pgrep asterisk)
```

### Debug Mode

```bash
# Run in debug mode
sudo asterisk -d
# or
sudo asterisk debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  asterisk:
    image: asterisk:latest
    container_name: asterisk
    ports:
      - "5060/5038:5060/5038"
    volumes:
      - ./config:/etc/asterisk
      - ./data:/var/lib/asterisk
    environment:
      - asterisk_CONFIG=/etc/asterisk/asterisk.conf
    restart: unless-stopped
    networks:
      - asterisk_net

networks:
  asterisk_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: asterisk
spec:
  replicas: 3
  selector:
    matchLabels:
      app: asterisk
  template:
    metadata:
      labels:
        app: asterisk
    spec:
      containers:
      - name: asterisk
        image: asterisk:latest
        ports:
        - containerPort: 5060/5038
        volumeMounts:
        - name: config
          mountPath: /etc/asterisk
      volumes:
      - name: config
        configMap:
          name: asterisk-config
---
apiVersion: v1
kind: Service
metadata:
  name: asterisk
spec:
  selector:
    app: asterisk
  ports:
  - port: 5060/5038
    targetPort: 5060/5038
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Asterisk
  hosts: all
  become: yes
  tasks:
    - name: Install asterisk
      package:
        name: asterisk
        state: present
    
    - name: Configure asterisk
      template:
        src: asterisk.conf.j2
        dest: /etc/asterisk/asterisk.conf
        owner: asterisk
        group: asterisk
        mode: '0640'
      notify: restart asterisk
    
    - name: Start and enable asterisk
      systemd:
        name: asterisk
        state: started
        enabled: yes
  
  handlers:
    - name: restart asterisk
      systemd:
        name: asterisk
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update asterisk

# Debian/Ubuntu
sudo apt update && sudo apt upgrade asterisk

# Arch Linux
sudo pacman -Syu asterisk

# Alpine Linux
apk update && apk upgrade asterisk

# openSUSE
sudo zypper update asterisk

# FreeBSD
pkg update && pkg upgrade asterisk

# Always backup before updates
tar -czf /backup/asterisk-pre-update-$(date +%Y%m%d).tar.gz /etc/asterisk

# Restart after updates
sudo systemctl restart asterisk
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/asterisk -name "*.log" -mtime +30 -delete

# Verify integrity
sudo asterisk --verify || sudo asterisk check

# Update databases (if applicable)
sudo asterisk-update-db

# Optimize performance
sudo asterisk-optimize

# Check for security updates
sudo asterisk --security-check
```

## Additional Resources

- Official Documentation: https://docs.asterisk.org/
- GitHub Repository: https://github.com/asterisk/asterisk
- Community Forum: https://forum.asterisk.org/
- Wiki: https://wiki.asterisk.org/
- Comparison vs FreeSWITCH, Kamailio, 3CX, FreePBX: https://docs.asterisk.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
