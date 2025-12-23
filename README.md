# VPS-Checklist

Great idea! Let's first check your VPS health and what's currently running.

## **1. Check VPS Health Status**

### **System Overview:**
```bash
# Check uptime and load average
uptime

# Show system info
uname -a

# Check OS version
cat /etc/os-release
```

### **CPU & Memory Usage:**
```bash
# Real-time monitoring (install with: sudo apt install htop)
htop

# Or use these basic commands:
# CPU usage
top -bn1 | grep "Cpu(s)"

# Memory usage
free -h

# Detailed memory info
cat /proc/meminfo | head -10

# Disk usage
df -h

# Disk space in human readable format
df -h --total
```

### **Check Running Processes:**
```bash
# Top 10 CPU-consuming processes
ps aux --sort=-%cpu | head -11

# Top 10 memory-consuming processes
ps aux --sort=-%mem | head -11

# Check total number of processes
ps aux | wc -l
```

## **2. Check Ports Being Used**

### **Using netstat:**
```bash
# Install if not available (Ubuntu/Debian)
sudo apt update && sudo apt install net-tools

# Check all listening ports
sudo netstat -tulpn

# OR with ss (modern alternative)
sudo ss -tulpn

# Check specific port (e.g., 3000)
sudo netstat -tulpn | grep :3000
```

### **Using lsof:**
```bash
# Install lsof if needed
sudo apt install lsof

# Show all listening ports
sudo lsof -i -P -n | grep LISTEN

# Check what's using a specific port
sudo lsof -i :3000
sudo lsof -i :80
sudo lsof -i :443
```

### **Check Firewall Status:**
```bash
# If using UFW (Ubuntu)
sudo ufw status verbose
sudo ufw status numbered

# If using firewalld (CentOS/RHEL)
sudo firewall-cmd --list-all

# If using iptables
sudo iptables -L -n -v
```

## **3. Check PM2 Status & Processes**

```bash
# List all PM2 processes
pm2 list

# Show detailed info
pm2 show <app-name>

# Check PM2 logs
pm2 logs --lines 50

# Check PM2 startup status
pm2 startup

# Show PM2 status
pm2 status
```

## **4. Check Nginx/Apache Status**

### **For Nginx:**
```bash
# Check if Nginx is running
sudo systemctl status nginx

# Check Nginx configuration
sudo nginx -t

# List enabled sites
sudo ls -la /etc/nginx/sites-enabled/

# Check Nginx processes
ps aux | grep nginx
```

### **For Apache:**
```bash
sudo systemctl status apache2
# or
sudo systemctl status httpd
```

## **5. Comprehensive Check Script**

Run this script to get a complete overview:

```bash
#!/bin/bash
echo "===== VPS HEALTH CHECK ====="
echo ""
echo "=== System Info ==="
uptime
echo ""
echo "=== Memory Usage ==="
free -h
echo ""
echo "=== Disk Usage ==="
df -h --total
echo ""
echo "=== Top 5 CPU Processes ==="
ps aux --sort=-%cpu | head -6
echo ""
echo "=== Top 5 Memory Processes ==="
ps aux --sort=-%mem | head -6
echo ""
echo "=== Listening Ports ==="
sudo netstat -tulpn 2>/dev/null | grep LISTEN || sudo ss -tulpn
echo ""
echo "=== PM2 Status ==="
pm2 list
echo ""
echo "=== Nginx Status ==="
sudo systemctl status nginx --no-pager 2>/dev/null || echo "Nginx not found"
echo ""
echo "=== UFW Status ==="
sudo ufw status 2>/dev/null || echo "UFW not configured"
```

Save and run it:
```bash
nano check_vps.sh
chmod +x check_vps.sh
./check_vps.sh
```

## **6. Quick Health Dashboard Commands:**

**One-liner for quick overview:**
```bash
echo "CPU Load: $(uptime | awk -F'load average:' '{print $2}') | Memory: $(free -h | awk '/^Mem:/ {print $3"/"$2}') | Disk: $(df -h / | awk 'NR==2 {print $3"/"$2 " ("$5")"}')"
```

**Check specific services:**
```bash
# Check if a service is running
check_service() {
    if systemctl is-active --quiet $1; then
        echo "$1: ✅ RUNNING"
    else
        echo "$1: ❌ STOPPED"
    fi
}

check_service nginx
check_service pm2
check_service mysql  # if applicable
check_service redis  # if applicable
```

## **7. What to Look For:**

### **Healthy Indicators:**
- **Load average** < number of CPU cores
- **Memory usage** < 80-85%
- **Disk usage** < 85%
- All necessary ports are open and listening
- PM2 processes show "online" status

### **Red Flags:**
- Load average consistently > CPU cores × 2
- Memory usage > 90%
- Disk usage > 90%
- Port conflicts (same port used by multiple apps)
- Services failing to start

## **8. After Checking - Common Fixes:**

If you find issues:

```bash
# If memory is high:
pm2 list  # Check memory usage per app
pm2 monit  # Monitor in real-time

# If disk is full:
# Find large files
sudo du -sh /home/* 2>/dev/null
sudo du -sh /var/* 2>/dev/null

# Clear old logs
sudo journalctl --vacuum-time=7d
pm2 flush  # Clear PM2 logs

# If ports are conflicting:
# Check what's using the port
sudo lsof -i :PORT_NUMBER
# Kill process if needed (carefully!)
sudo kill -9 PID
```

**Please run these commands and share the output so I can help you identify:**

1. What ports are currently in use
2. How much resources are available
3. What applications are already running
4. If there are any conflicts or issues to resolve before adding a new app

Just copy-paste the output of these key commands:
```bash
free -h
df -h
sudo ss -tulpn
pm2 list
```
