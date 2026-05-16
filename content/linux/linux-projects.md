---
title: "Linux Projects — Beginner to Advanced (3 Real Projects)"
date: 2024-03-20
draft: false
description: "3 hands-on Linux projects: beginner system monitor script, intermediate log analyzer, and advanced automated server setup tool. Build real skills!"
categories: ["linux"]
tags: ["linux", "bash", "scripting", "projects", "devops", "automation"]
showToc: true
TocOpen: true
---

## Why Projects Matter 🎯

Reading about Linux is one thing. **Building something real** is how you actually learn. These 3 projects go from simple to production-ready — each one teaches real skills used by DevOps engineers every day.

```
PROJECT ROADMAP:
──────────────────────────────────────────────────────────────
Project 1 (Beginner)     → System Health Monitor Script
Project 2 (Intermediate) → Automated Log Analyzer & Reporter
Project 3 (Advanced)     → Server Setup Automation Tool
──────────────────────────────────────────────────────────────
```

---

## Project 1: System Health Monitor 🟢 Beginner

### What You'll Build

A bash script that checks CPU, memory, disk, services — and shows a color-coded dashboard.

```
OUTPUT EXAMPLE:
──────────────────────────────────────────────────────────────
╔══════════════════════════════════════════╗
║        SYSTEM HEALTH MONITOR             ║
║        2024-03-20 14:30:22               ║
╚══════════════════════════════════════════╝

── SYSTEM INFO ─────────────────────────────
  🖥️  Hostname:  ubuntu-server
  ⏱️  Uptime:    3 days, 4 hours
  👤  Users:    2 logged in
  🐧  Kernel:   5.15.0-91-generic

── CPU ─────────────────────────────────────
  CPU Usage:  ████░░░░░░ 42%  ✅ OK

── MEMORY ──────────────────────────────────
  RAM Usage:  ██████░░░░ 61%  ✅ OK

── DISK USAGE ──────────────────────────────
  /           ███░░░░░░░ 34%  ✅ OK
  /data       ████████░░ 82%  ⚠️  WARNING

── SERVICES ────────────────────────────────
  ✅ nginx:       Running
  ✅ docker:      Running
  ❌ postgresql:  Stopped
  ✅ ssh:         Running
──────────────────────────────────────────────
```

### Skills You'll Learn

- Bash scripting fundamentals
- Reading system info (`top`, `free`, `df`, `ps`)
- Functions, colors, loops in bash
- Cron jobs for automation

### The Script

```bash
#!/bin/bash
# ============================================================
# Project 1: System Health Monitor
# File: health-monitor.sh
# Usage: bash health-monitor.sh
# ============================================================

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
WHITE='\033[1;37m'
NC='\033[0m'

# Thresholds
CPU_WARN=70
CPU_CRIT=90
MEM_WARN=80
MEM_CRIT=95
DISK_WARN=80
DISK_CRIT=95

# Services to check
SERVICES=("nginx" "docker" "ssh" "cron" "postgresql")

# ── Progress bar ─────────────────────────────────────────────
progress_bar() {
    local percent=$1
    local filled=$((percent / 10))
    local empty=$((10 - filled))
    local bar=""
    for ((i=0; i<filled; i++)); do bar+="█"; done
    for ((i=0; i<empty; i++)); do bar+="░"; done
    echo -n "$bar"
}

# ── Status color ─────────────────────────────────────────────
get_status() {
    local value=$1
    local warn=$2
    local crit=$3
    if [ "$value" -ge "$crit" ]; then
        echo -e "${RED}❌ CRITICAL${NC}"
    elif [ "$value" -ge "$warn" ]; then
        echo -e "${YELLOW}⚠️  WARNING${NC}"
    else
        echo -e "${GREEN}✅ OK${NC}"
    fi
}

# ── Header ───────────────────────────────────────────────────
print_header() {
    clear
    echo -e "${CYAN}╔══════════════════════════════════════════╗${NC}"
    echo -e "${CYAN}║${WHITE}        SYSTEM HEALTH MONITOR             ${CYAN}║${NC}"
    echo -e "${CYAN}║${WHITE}        $(date '+%Y-%m-%d %H:%M:%S')           ${CYAN}║${NC}"
    echo -e "${CYAN}╚══════════════════════════════════════════╝${NC}"
}

# ── System Info ──────────────────────────────────────────────
show_system_info() {
    echo -e "\n${BLUE}── SYSTEM INFO ─────────────────────────────${NC}"
    echo -e "  🖥️  Hostname:  ${WHITE}$(hostname)${NC}"
    echo -e "  ⏱️  Uptime:    ${WHITE}$(uptime -p)${NC}"
    echo -e "  👤  Users:    ${WHITE}$(who | wc -l) logged in${NC}"
    echo -e "  📊  Load:     ${WHITE}$(uptime | awk -F'load average:' '{print $2}' | xargs)${NC}"
    echo -e "  🐧  Kernel:   ${WHITE}$(uname -r)${NC}"
}

# ── CPU ──────────────────────────────────────────────────────
show_cpu() {
    echo -e "\n${BLUE}── CPU ─────────────────────────────────────${NC}"

    local cpu_usage=$(grep 'cpu ' /proc/stat | \
        awk '{usage=($2+$4)*100/($2+$4+$5)} END {print int(usage)}')

    local bar=$(progress_bar $cpu_usage)
    local status=$(get_status $cpu_usage $CPU_WARN $CPU_CRIT)

    echo -e "  CPU Usage:  ${bar} ${WHITE}${cpu_usage}%${NC}  $status"
    echo -e "  CPU Cores:  ${WHITE}$(nproc)${NC}"
    echo -e "\n  ${YELLOW}Top processes by CPU:${NC}"
    ps aux --sort=-%cpu | awk 'NR>1 && NR<=4 {
        printf "  %-25s %s%%\n", $11, $3}'
}

# ── Memory ───────────────────────────────────────────────────
show_memory() {
    echo -e "\n${BLUE}── MEMORY ──────────────────────────────────${NC}"

    local mem_total=$(free -m | awk 'NR==2{print $2}')
    local mem_used=$(free -m | awk 'NR==2{print $3}')
    local mem_free=$(free -m | awk 'NR==2{print $4}')
    local mem_percent=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')

    local bar=$(progress_bar $mem_percent)
    local status=$(get_status $mem_percent $MEM_WARN $MEM_CRIT)

    echo -e "  RAM Usage:  ${bar} ${WHITE}${mem_percent}%${NC}  $status"
    echo -e "  Total:  ${WHITE}${mem_total} MB${NC}  |  Used: ${WHITE}${mem_used} MB${NC}  |  Free: ${WHITE}${mem_free} MB${NC}"

    # Swap
    local swap_total=$(free -m | awk 'NR==3{print $2}')
    local swap_used=$(free -m | awk 'NR==3{print $3}')
    if [ "$swap_total" -gt 0 ]; then
        echo -e "  Swap: ${WHITE}${swap_used}/${swap_total} MB${NC}"
    fi

    echo -e "\n  ${YELLOW}Top processes by Memory:${NC}"
    ps aux --sort=-%mem | awk 'NR>1 && NR<=4 {
        printf "  %-25s %s%%\n", $11, $4}'
}

# ── Disk ─────────────────────────────────────────────────────
show_disk() {
    echo -e "\n${BLUE}── DISK USAGE ──────────────────────────────${NC}"

    df -h | grep -v "tmpfs\|udev\|overlay\|snap" | awk 'NR>1' | \
    while read line; do
        local mount=$(echo $line | awk '{print $6}')
        local used_pct=$(echo $line | awk '{print $5}' | tr -d '%')
        local avail=$(echo $line | awk '{print $4}')

        local bar=$(progress_bar $used_pct)
        local status=""

        if [ "$used_pct" -ge "$DISK_CRIT" ]; then
            status="${RED}❌ CRITICAL${NC}"
        elif [ "$used_pct" -ge "$DISK_WARN" ]; then
            status="${YELLOW}⚠️  WARNING${NC}"
        else
            status="${GREEN}✅ OK${NC}"
        fi

        printf "  %-12s %s ${WHITE}%s%%${NC}  $(echo -e $status)  (${WHITE}%s${NC} free)\n" \
            "$mount" "$bar" "$used_pct" "$avail"
    done
}

# ── Network ──────────────────────────────────────────────────
show_network() {
    echo -e "\n${BLUE}── NETWORK ─────────────────────────────────${NC}"

    ip -brief addr show | grep -v "lo" | while read line; do
        local iface=$(echo $line | awk '{print $1}')
        local state=$(echo $line | awk '{print $2}')
        local ip=$(echo $line | awk '{print $3}' | cut -d'/' -f1)

        if [ "$state" = "UP" ]; then
            echo -e "  ${GREEN}●${NC} $iface  ${WHITE}${ip:-no IP}${NC}  ${GREEN}UP${NC}"
        else
            echo -e "  ${RED}●${NC} $iface  ${RED}DOWN${NC}"
        fi
    done
}

# ── Services ─────────────────────────────────────────────────
show_services() {
    echo -e "\n${BLUE}── SERVICES ────────────────────────────────${NC}"

    for service in "${SERVICES[@]}"; do
        if systemctl is-active --quiet "$service" 2>/dev/null; then
            echo -e "  ${GREEN}✅${NC} ${WHITE}$service${NC}:  ${GREEN}Running${NC}"
        elif systemctl list-units --all | grep -q "$service" 2>/dev/null; then
            echo -e "  ${RED}❌${NC} ${WHITE}$service${NC}:  ${RED}Stopped${NC}"
        else
            echo -e "  ${YELLOW}⚪${NC} ${WHITE}$service${NC}:  ${YELLOW}Not installed${NC}"
        fi
    done
}

# ── Docker ───────────────────────────────────────────────────
show_docker() {
    if command -v docker &>/dev/null && docker info &>/dev/null 2>&1; then
        echo -e "\n${BLUE}── DOCKER ──────────────────────────────────${NC}"
        local running=$(docker ps -q 2>/dev/null | wc -l)
        local total=$(docker ps -aq 2>/dev/null | wc -l)
        local images=$(docker images -q 2>/dev/null | wc -l)

        echo -e "  Running: ${GREEN}$running${NC}  |  Total: ${WHITE}$total${NC}  |  Images: ${WHITE}$images${NC}"

        if [ "$running" -gt 0 ]; then
            docker ps --format "  🐳 {{.Names}}  {{.Status}}" 2>/dev/null
        fi
    fi
}

# ── Footer ───────────────────────────────────────────────────
print_footer() {
    echo -e "\n${CYAN}══════════════════════════════════════════${NC}"
    echo -e "  Report: $(date)"
    echo -e "${CYAN}══════════════════════════════════════════${NC}\n"
}

# ── Main ─────────────────────────────────────────────────────
main() {
    print_header
    show_system_info
    show_cpu
    show_memory
    show_disk
    show_network
    show_services
    show_docker
    print_footer
}

main
```

### Setup & Run

```bash
# Create the script
mkdir -p ~/projects/linux-monitor
cd ~/projects/linux-monitor
nano health-monitor.sh
# paste the script above

# Make executable
chmod +x health-monitor.sh

# Run it
./health-monitor.sh

# Watch live — updates every 5 seconds
watch -n 5 ./health-monitor.sh

# Schedule with cron — run every hour
crontab -e
# Add this line:
# 0 * * * * /home/ubuntu/projects/linux-monitor/health-monitor.sh >> /var/log/health.log 2>&1
```

### Test It

```bash
# Install stress tool
sudo apt install stress -y

# Spike CPU to 100% for 30 seconds
stress --cpu 4 --timeout 30s

# In another terminal — run monitor and watch CPU go red!
./health-monitor.sh

# Fill disk to trigger disk warning
dd if=/dev/zero of=/tmp/testfile bs=1M count=2000

# Stop a service to test service check
sudo systemctl stop nginx
./health-monitor.sh    # nginx shows as stopped ❌

# Clean up
rm /tmp/testfile
sudo systemctl start nginx
```

### What You Learned

- ✅ Bash functions and variables
- ✅ Reading CPU, memory, disk from Linux
- ✅ Color output in terminal
- ✅ Cron job scheduling
- ✅ Service status checking

---

## Project 2: Automated Log Analyzer 🟡 Intermediate

### What You'll Build

A script that scans Nginx and system logs, finds errors, counts patterns, detects suspicious IPs, and saves a report. Real sysadmins run scripts like this daily.

```
REPORT OUTPUT EXAMPLE:
──────────────────────────────────────────────────────────────
📊 LOG ANALYSIS REPORT — 2024-03-20
══════════════════════════════════════

📁 File: /var/log/nginx/access.log
   Total requests:    45,231
   Successful (2xx):  41,892 (92.6%)
   Client errors(4xx): 2,891 ( 6.4%)
   Server errors(5xx):   448 ( 1.0%)

🔥 Top 5 IPs:
   192.168.1.100   2341 requests
   10.0.0.45       1892 requests
   172.16.0.22      934 requests

🚨 Suspicious IPs (>1000 requests):
   192.168.1.100   2341 requests  ← possible bot!

📈 Top requested URLs:
   /api/users      8921 hits
   /api/products   6234 hits
   /health          923 hits

❌ Recent errors (last 50):
   [error] connect() failed (111: Connection refused)
   ...
──────────────────────────────────────────────────────────────
```

### Skills You'll Learn

- Advanced `grep`, `awk`, `sed`
- File processing and parsing
- Generating reports
- Email notifications
- Cron automation

### The Script

```bash
#!/bin/bash
# ============================================================
# Project 2: Automated Log Analyzer
# File: log-analyzer.sh
# Usage: bash log-analyzer.sh [logfile]
# ============================================================

# ── Config ───────────────────────────────────────────────────
NGINX_ACCESS="/var/log/nginx/access.log"
NGINX_ERROR="/var/log/nginx/error.log"
AUTH_LOG="/var/log/auth.log"
SYSLOG="/var/log/syslog"
REPORT_DIR="/var/log/reports"
REPORT_FILE="$REPORT_DIR/report-$(date +%Y-%m-%d).txt"
SUSPICIOUS_THRESHOLD=1000      # flag IPs with more than this many requests
EMAIL="pramendraatwork@gmail.com"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
WHITE='\033[1;37m'
NC='\033[0m'

# ── Setup ────────────────────────────────────────────────────
setup() {
    mkdir -p "$REPORT_DIR"
    echo "📊 LOG ANALYSIS REPORT" > "$REPORT_FILE"
    echo "Generated: $(date)" >> "$REPORT_FILE"
    echo "================================================" >> "$REPORT_FILE"
}

# ── Log line to both terminal and file ───────────────────────
log() {
    echo -e "$1" | tee -a "$REPORT_FILE"
}

# ── Analyze Nginx Access Log ─────────────────────────────────
analyze_nginx_access() {
    if [ ! -f "$NGINX_ACCESS" ]; then
        log "${YELLOW}⚠️  Nginx access log not found: $NGINX_ACCESS${NC}"
        return
    fi

    log "\n${CYAN}── NGINX ACCESS LOG ─────────────────────────${NC}"
    log "📁 File: $NGINX_ACCESS"

    # Total requests
    local total=$(wc -l < "$NGINX_ACCESS")
    log "   Total requests:    ${WHITE}$total${NC}"

    # Status codes
    local success=$(awk '$9 ~ /^2/' "$NGINX_ACCESS" | wc -l)
    local redirect=$(awk '$9 ~ /^3/' "$NGINX_ACCESS" | wc -l)
    local client_err=$(awk '$9 ~ /^4/' "$NGINX_ACCESS" | wc -l)
    local server_err=$(awk '$9 ~ /^5/' "$NGINX_ACCESS" | wc -l)

    local success_pct=$(awk "BEGIN {printf \"%.1f\", $success*100/$total}")
    local err_pct=$(awk "BEGIN {printf \"%.1f\", $server_err*100/$total}")

    log "   ${GREEN}✅ Successful (2xx):  $success ($success_pct%)${NC}"
    log "   ${YELLOW}↪️  Redirects  (3xx):  $redirect${NC}"
    log "   ${YELLOW}⚠️  Client err (4xx):  $client_err${NC}"
    log "   ${RED}❌ Server err (5xx):  $server_err ($err_pct%)${NC}"

    # Top IPs
    log "\n   ${YELLOW}🔥 Top 10 IPs by requests:${NC}"
    awk '{print $1}' "$NGINX_ACCESS" | \
        sort | uniq -c | sort -rn | head -10 | \
        while read count ip; do
            log "   $ip  ${WHITE}$count requests${NC}"
        done

    # Suspicious IPs
    log "\n   ${RED}🚨 Suspicious IPs (>$SUSPICIOUS_THRESHOLD requests):${NC}"
    local suspicious=0
    awk '{print $1}' "$NGINX_ACCESS" | \
        sort | uniq -c | sort -rn | \
        while read count ip; do
            if [ "$count" -gt "$SUSPICIOUS_THRESHOLD" ]; then
                log "   ${RED}⚠️  $ip  $count requests ← investigate!${NC}"
                suspicious=1
            fi
        done

    # Top URLs
    log "\n   ${YELLOW}📈 Top 10 requested URLs:${NC}"
    awk '{print $7}' "$NGINX_ACCESS" | \
        sort | uniq -c | sort -rn | head -10 | \
        while read count url; do
            log "   ${WHITE}$count${NC}  $url"
        done

    # Top 404s
    log "\n   ${YELLOW}🔍 Top 404 Not Found URLs:${NC}"
    awk '$9 == 404 {print $7}' "$NGINX_ACCESS" | \
        sort | uniq -c | sort -rn | head -5 | \
        while read count url; do
            log "   ${RED}$count${NC}  $url"
        done

    # Bandwidth usage
    log "\n   ${YELLOW}📊 Bandwidth by hour:${NC}"
    awk '{
        split($4, dt, ":")
        hour=dt[2]
        bytes+=$10
        count[hour]++
    }
    END {
        for (h in count) {
            printf "   Hour %s: %d requests\n", h, count[h]
        }
    }' "$NGINX_ACCESS" | sort | head -5 | while read line; do
        log "$line"
    done
}

# ── Analyze Nginx Error Log ───────────────────────────────────
analyze_nginx_error() {
    if [ ! -f "$NGINX_ERROR" ]; then
        log "${YELLOW}⚠️  Nginx error log not found${NC}"
        return
    fi

    log "\n${CYAN}── NGINX ERROR LOG ──────────────────────────${NC}"

    local total_errors=$(wc -l < "$NGINX_ERROR")
    log "   Total errors: ${RED}$total_errors${NC}"

    # Error types
    log "\n   ${YELLOW}Error breakdown:${NC}"
    grep -oP '\[.*?\]' "$NGINX_ERROR" | \
        sort | uniq -c | sort -rn | head -5 | \
        while read count level; do
            log "   $level  ${WHITE}$count${NC}"
        done

    # Recent errors
    log "\n   ${YELLOW}Recent 10 errors:${NC}"
    tail -10 "$NGINX_ERROR" | while read line; do
        log "   ${RED}$line${NC}"
    done
}

# ── Analyze Auth Log ─────────────────────────────────────────
analyze_auth() {
    if [ ! -f "$AUTH_LOG" ]; then
        log "${YELLOW}⚠️  Auth log not found${NC}"
        return
    fi

    log "\n${CYAN}── AUTHENTICATION LOG ───────────────────────${NC}"

    # Failed logins
    local failed=$(grep "Failed password" "$AUTH_LOG" 2>/dev/null | wc -l)
    local success=$(grep "Accepted password\|Accepted publickey" "$AUTH_LOG" 2>/dev/null | wc -l)
    local invalid=$(grep "Invalid user" "$AUTH_LOG" 2>/dev/null | wc -l)

    log "   ${GREEN}✅ Successful logins: $success${NC}"
    log "   ${RED}❌ Failed logins:     $failed${NC}"
    log "   ${RED}🚫 Invalid users:     $invalid${NC}"

    # IPs with most failed attempts
    log "\n   ${YELLOW}🔥 Top IPs with failed login attempts:${NC}"
    grep "Failed password" "$AUTH_LOG" 2>/dev/null | \
        awk '{print $(NF-3)}' | \
        sort | uniq -c | sort -rn | head -5 | \
        while read count ip; do
            if [ "$count" -gt 10 ]; then
                log "   ${RED}⚠️  $ip  $count attempts ← possible brute force!${NC}"
            else
                log "   $ip  ${WHITE}$count attempts${NC}"
            fi
        done

    # Root login attempts
    local root_attempts=$(grep "Failed password for root" "$AUTH_LOG" 2>/dev/null | wc -l)
    if [ "$root_attempts" -gt 0 ]; then
        log "\n   ${RED}🚨 ROOT LOGIN ATTEMPTS: $root_attempts ← SECURITY ALERT!${NC}"
    fi
}

# ── Analyze Syslog ───────────────────────────────────────────
analyze_syslog() {
    if [ ! -f "$SYSLOG" ]; then
        log "${YELLOW}⚠️  Syslog not found${NC}"
        return
    fi

    log "\n${CYAN}── SYSTEM LOG ───────────────────────────────${NC}"

    # Today's errors
    local today=$(date +"%b %d")
    local errors=$(grep "$today" "$SYSLOG" | grep -i "error\|fail\|critical" | wc -l)
    local warnings=$(grep "$today" "$SYSLOG" | grep -i "warn" | wc -l)

    log "   Today's errors:   ${RED}$errors${NC}"
    log "   Today's warnings: ${YELLOW}$warnings${NC}"

    # OOM killer events
    local oom=$(grep "Out of memory\|OOM killer" "$SYSLOG" 2>/dev/null | wc -l)
    if [ "$oom" -gt 0 ]; then
        log "   ${RED}🚨 OOM Killer events: $oom ← memory issues!${NC}"
    fi

    # Recent critical messages
    log "\n   ${YELLOW}Recent critical messages:${NC}"
    grep -i "critical\|emerg\|alert" "$SYSLOG" 2>/dev/null | \
        tail -5 | while read line; do
        log "   ${RED}$line${NC}"
    done
}

# ── Disk Space Check ─────────────────────────────────────────
check_disk_space() {
    log "\n${CYAN}── DISK SPACE ALERT CHECK ───────────────────${NC}"

    df -h | grep -v "tmpfs\|udev" | awk 'NR>1' | while read line; do
        local mount=$(echo $line | awk '{print $6}')
        local pct=$(echo $line | awk '{print $5}' | tr -d '%')
        local avail=$(echo $line | awk '{print $4}')

        if [ "$pct" -ge 90 ]; then
            log "   ${RED}🚨 CRITICAL: $mount is ${pct}% full! Only $avail free${NC}"
        elif [ "$pct" -ge 80 ]; then
            log "   ${YELLOW}⚠️  WARNING: $mount is ${pct}% full. $avail remaining${NC}"
        else
            log "   ${GREEN}✅ $mount: ${pct}% used, $avail free${NC}"
        fi
    done
}

# ── Summary ──────────────────────────────────────────────────
print_summary() {
    log "\n${CYAN}════════════════════════════════════════════${NC}"
    log "${WHITE}📋 SUMMARY${NC}"
    log "${CYAN}════════════════════════════════════════════${NC}"
    log "  Report saved to: ${WHITE}$REPORT_FILE${NC}"
    log "  Generated at:    ${WHITE}$(date)${NC}"
    log "  Hostname:        ${WHITE}$(hostname)${NC}"
    log "${CYAN}════════════════════════════════════════════${NC}"
}

# ── Send Email Report ────────────────────────────────────────
send_email() {
    if command -v mail &>/dev/null; then
        mail -s "Log Report - $(hostname) - $(date +%Y-%m-%d)" \
            "$EMAIL" < "$REPORT_FILE"
        echo "📧 Report emailed to $EMAIL"
    else
        echo "💡 Install mailutils to enable email: sudo apt install mailutils"
    fi
}

# ── Main ─────────────────────────────────────────────────────
main() {
    echo -e "${CYAN}🔍 Starting log analysis...${NC}"
    setup
    analyze_nginx_access
    analyze_nginx_error
    analyze_auth
    analyze_syslog
    check_disk_space
    print_summary

    # Uncomment to send email:
    # send_email

    echo -e "\n${GREEN}✅ Analysis complete! Report: $REPORT_FILE${NC}"
}

main
```

### Setup & Run

```bash
# Create project
mkdir -p ~/projects/log-analyzer
cd ~/projects/log-analyzer
nano log-analyzer.sh
chmod +x log-analyzer.sh

# Run it
sudo ./log-analyzer.sh

# View report
cat /var/log/reports/report-$(date +%Y-%m-%d).txt

# Schedule daily at 6am
crontab -e
# Add:
# 0 6 * * * sudo /home/ubuntu/projects/log-analyzer/log-analyzer.sh
```

### What You Learned

- ✅ Advanced `grep`, `awk`, `sed` for log parsing
- ✅ Detecting security threats (brute force, bots)
- ✅ Generating text reports
- ✅ Cron scheduling
- ✅ Email notifications

---

## Project 3: Server Setup Automation Tool 🔴 Advanced

### What You'll Build

A complete server setup script that provisions a fresh Ubuntu server — installs packages, creates users, hardens SSH, sets up firewall, configures monitoring, and generates a setup report. This is what DevOps engineers run on new servers.

```
WHAT IT DOES:
──────────────────────────────────────────────────────────────
✅ Updates system packages
✅ Installs: nginx, docker, git, curl, vim, htop, ufw
✅ Creates deploy user with sudo access
✅ Sets up SSH key authentication
✅ Disables root SSH login (security!)
✅ Configures UFW firewall (allow 22, 80, 443 only)
✅ Sets up fail2ban (blocks brute force)
✅ Configures automatic security updates
✅ Sets up log rotation
✅ Installs and configures Docker
✅ Sets system timezone
✅ Generates setup report
──────────────────────────────────────────────────────────────
```

### Skills You'll Learn

- User management automation
- SSH hardening
- Firewall configuration
- Package management
- System security
- Error handling in bash

### The Script

```bash
#!/bin/bash
# ============================================================
# Project 3: Server Setup Automation Tool
# File: server-setup.sh
# Usage: sudo bash server-setup.sh
# ============================================================

set -euo pipefail    # exit on error, undefined var, pipe fail

# ── Config — Edit these! ────────────────────────────────────
DEPLOY_USER="deploy"
DEPLOY_PASSWORD="$(openssl rand -base64 12)"
SSH_PORT=22
TIMEZONE="Asia/Kolkata"
YOUR_SSH_PUBLIC_KEY=""    # paste your public key here!
ALLOWED_PORTS=(22 80 443)

# ── Colors ───────────────────────────────────────────────────
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
WHITE='\033[1;37m'
NC='\033[0m'

# ── Log file ─────────────────────────────────────────────────
LOGFILE="/var/log/server-setup.log"
exec > >(tee -a "$LOGFILE") 2>&1

# ── Helper functions ─────────────────────────────────────────
info()    { echo -e "${GREEN}[INFO]${NC}    $1"; }
warn()    { echo -e "${YELLOW}[WARN]${NC}    $1"; }
error()   { echo -e "${RED}[ERROR]${NC}   $1"; }
section() { echo -e "\n${CYAN}══ $1 ══${NC}"; }

# Check if running as root
check_root() {
    if [ "$EUID" -ne 0 ]; then
        error "Run as root: sudo bash server-setup.sh"
        exit 1
    fi
}

# Check OS
check_os() {
    if ! grep -q "Ubuntu" /etc/os-release 2>/dev/null; then
        warn "This script is designed for Ubuntu. Proceed with caution."
    fi
    info "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"
}

# ── Step 1: System Update ────────────────────────────────────
update_system() {
    section "STEP 1: SYSTEM UPDATE"
    info "Updating package list..."
    apt-get update -qq

    info "Upgrading packages..."
    apt-get upgrade -y -qq

    info "Installing essential packages..."
    apt-get install -y -qq \
        curl \
        wget \
        git \
        vim \
        nano \
        htop \
        tree \
        unzip \
        net-tools \
        ufw \
        fail2ban \
        nginx \
        certbot \
        python3-certbot-nginx \
        logrotate \
        unattended-upgrades \
        apt-transport-https \
        ca-certificates \
        gnupg \
        lsb-release

    info "✅ System updated and packages installed"
}

# ── Step 2: Create Deploy User ───────────────────────────────
create_deploy_user() {
    section "STEP 2: CREATE DEPLOY USER"

    if id "$DEPLOY_USER" &>/dev/null; then
        warn "User $DEPLOY_USER already exists. Skipping."
        return
    fi

    info "Creating user: $DEPLOY_USER"
    useradd -m -s /bin/bash "$DEPLOY_USER"
    echo "$DEPLOY_USER:$DEPLOY_PASSWORD" | chpasswd

    info "Adding $DEPLOY_USER to sudo group"
    usermod -aG sudo "$DEPLOY_USER"

    info "Adding $DEPLOY_USER to docker group"
    usermod -aG docker "$DEPLOY_USER" 2>/dev/null || true

    # Setup SSH directory
    mkdir -p /home/$DEPLOY_USER/.ssh
    chmod 700 /home/$DEPLOY_USER/.ssh

    # Add SSH public key if provided
    if [ -n "$YOUR_SSH_PUBLIC_KEY" ]; then
        echo "$YOUR_SSH_PUBLIC_KEY" >> /home/$DEPLOY_USER/.ssh/authorized_keys
        chmod 600 /home/$DEPLOY_USER/.ssh/authorized_keys
        chown -R $DEPLOY_USER:$DEPLOY_USER /home/$DEPLOY_USER/.ssh
        info "✅ SSH key added for $DEPLOY_USER"
    else
        warn "No SSH key provided. Set YOUR_SSH_PUBLIC_KEY in config!"
    fi

    info "✅ User $DEPLOY_USER created (password saved in report)"
}

# ── Step 3: SSH Hardening ────────────────────────────────────
harden_ssh() {
    section "STEP 3: SSH HARDENING"

    local sshd_config="/etc/ssh/sshd_config"

    # Backup original config
    cp "$sshd_config" "${sshd_config}.backup.$(date +%Y%m%d)"
    info "SSH config backed up"

    # Apply hardening settings
    cat >> "$sshd_config" << EOF

# ── Security hardening by server-setup.sh ──
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
LoginGraceTime 20
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PermitEmptyPasswords no
Protocol 2
EOF

    # If SSH key was added, disable password auth
    if [ -n "$YOUR_SSH_PUBLIC_KEY" ]; then
        info "Disabling password authentication (key auth only)"
        sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' "$sshd_config"
    else
        warn "SSH key not set — keeping password auth enabled for now"
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' "$sshd_config"
    fi

    # Restart SSH
    systemctl restart sshd
    info "✅ SSH hardened — root login disabled"
}

# ── Step 4: Firewall ─────────────────────────────────────────
setup_firewall() {
    section "STEP 4: FIREWALL (UFW)"

    # Reset UFW
    ufw --force reset

    # Default policies
    ufw default deny incoming
    ufw default allow outgoing

    # Allow configured ports
    for port in "${ALLOWED_PORTS[@]}"; do
        ufw allow "$port/tcp"
        info "Allowed port: $port/tcp"
    done

    # Enable firewall
    ufw --force enable
    ufw status verbose

    info "✅ Firewall configured — only ports ${ALLOWED_PORTS[*]} open"
}

# ── Step 5: Fail2Ban ─────────────────────────────────────────
setup_fail2ban() {
    section "STEP 5: FAIL2BAN (Brute Force Protection)"

    cat > /etc/fail2ban/jail.local << EOF
[DEFAULT]
bantime  = 3600
findtime  = 600
maxretry = 5
backend = systemd

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 86400

[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 10
EOF

    systemctl enable fail2ban
    systemctl restart fail2ban
    info "✅ Fail2ban configured — SSH max 3 attempts, 24h ban"
}

# ── Step 6: Install Docker ───────────────────────────────────
install_docker() {
    section "STEP 6: DOCKER"

    if command -v docker &>/dev/null; then
        info "Docker already installed: $(docker --version)"
        return
    fi

    info "Installing Docker..."

    # Add Docker GPG key
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
        gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

    # Add Docker repository
    echo "deb [arch=$(dpkg --print-architecture) \
        signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
        https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) stable" | \
        tee /etc/apt/sources.list.d/docker.list > /dev/null

    apt-get update -qq
    apt-get install -y -qq \
        docker-ce \
        docker-ce-cli \
        containerd.io \
        docker-compose-plugin

    systemctl enable docker
    systemctl start docker

    info "✅ Docker installed: $(docker --version)"
}

# ── Step 7: Automatic Security Updates ───────────────────────
setup_auto_updates() {
    section "STEP 7: AUTOMATIC SECURITY UPDATES"

    cat > /etc/apt/apt.conf.d/20auto-upgrades << EOF
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
EOF

    cat > /etc/apt/apt.conf.d/50unattended-upgrades << EOF
Unattended-Upgrade::Allowed-Origins {
    "\${distro_id}:\${distro_codename}";
    "\${distro_id}:\${distro_codename}-security";
    "\${distro_id}ESMApps:\${distro_codename}-apps-security";
    "\${distro_id}ESM:\${distro_codename}-infra-security";
};
Unattended-Upgrade::Mail "$YOUR_EMAIL";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
EOF

    systemctl enable unattended-upgrades
    info "✅ Automatic security updates enabled"
}

# ── Step 8: System Settings ──────────────────────────────────
configure_system() {
    section "STEP 8: SYSTEM SETTINGS"

    # Set timezone
    timedatectl set-timezone "$TIMEZONE"
    info "Timezone set to: $TIMEZONE"

    # Set hostname prompt
    echo "PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '" \
        >> /etc/bash.bashrc

    # Increase file limits
    cat >> /etc/security/limits.conf << EOF
* soft nofile 65536
* hard nofile 65536
EOF

    # Kernel parameters for performance
    cat >> /etc/sysctl.conf << EOF
# Performance tuning
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
fs.file-max = 2097152
EOF

    sysctl -p &>/dev/null
    info "✅ System settings configured"
}

# ── Step 9: Nginx Basic Setup ────────────────────────────────
setup_nginx() {
    section "STEP 9: NGINX"

    # Basic security headers
    cat > /etc/nginx/conf.d/security.conf << EOF
add_header X-Frame-Options SAMEORIGIN;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
add_header Referrer-Policy "strict-origin-when-cross-origin";
server_tokens off;
EOF

    nginx -t && systemctl restart nginx
    info "✅ Nginx configured with security headers"
}

# ── Step 10: Generate Report ─────────────────────────────────
generate_report() {
    section "SETUP COMPLETE — REPORT"

    local report="/root/server-setup-report.txt"

    cat > "$report" << EOF
════════════════════════════════════════════
SERVER SETUP REPORT
$(date)
════════════════════════════════════════════

SERVER INFO:
  Hostname:  $(hostname)
  IP:        $(curl -s ifconfig.me 2>/dev/null || echo "check manually")
  OS:        $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)
  Kernel:    $(uname -r)

CREATED USER:
  Username:  $DEPLOY_USER
  Password:  $DEPLOY_PASSWORD
  Groups:    sudo, docker
  SSH:       Key-based auth $([ -n "$YOUR_SSH_PUBLIC_KEY" ] && echo "ENABLED" || echo "NOT SET")

SECURITY:
  Root SSH login:     DISABLED
  Password SSH auth:  $([ -n "$YOUR_SSH_PUBLIC_KEY" ] && echo "DISABLED" || echo "ENABLED")
  Firewall (UFW):     ENABLED
  Open ports:         ${ALLOWED_PORTS[*]}
  Fail2ban:           ENABLED (3 attempts = 24h ban)
  Auto updates:       ENABLED

INSTALLED PACKAGES:
  nginx, docker, git, curl, wget, vim, htop
  fail2ban, ufw, certbot, unattended-upgrades

NEXT STEPS:
  1. Test SSH: ssh $DEPLOY_USER@$(hostname -I | awk '{print $1}')
  2. Get SSL cert: certbot --nginx -d yourdomain.com
  3. Deploy your app in /var/www/
  4. Check fail2ban: sudo fail2ban-client status sshd
  5. Check UFW: sudo ufw status verbose

IMPORTANT - SAVE THIS FILE SECURELY!
════════════════════════════════════════════
EOF

    cat "$report"
    info "✅ Report saved to: $report"
}

# ── Main ─────────────────────────────────────────────────────
main() {
    echo -e "${CYAN}"
    echo "╔══════════════════════════════════════════╗"
    echo "║      SERVER SETUP AUTOMATION TOOL        ║"
    echo "║      $(date '+%Y-%m-%d %H:%M:%S')             ║"
    echo "╚══════════════════════════════════════════╝"
    echo -e "${NC}"

    check_root
    check_os

    echo -e "${YELLOW}This will set up a production server. Continue? (yes/no)${NC}"
    read -r confirm
    if [ "$confirm" != "yes" ]; then
        echo "Cancelled."
        exit 0
    fi

    update_system
    create_deploy_user
    harden_ssh
    setup_firewall
    setup_fail2ban
    install_docker
    setup_auto_updates
    configure_system
    setup_nginx
    generate_report

    echo -e "\n${GREEN}🎉 Server setup complete!${NC}"
    echo -e "${YELLOW}⚠️  Log out and log back in as '$DEPLOY_USER' to verify access!${NC}"
    echo -e "${WHITE}📄 Setup report: /root/server-setup-report.txt${NC}"
    echo -e "${WHITE}📋 Setup log:    $LOGFILE${NC}"
}

main
```

### Setup & Run

```bash
# On a fresh Ubuntu server (EC2, DigitalOcean, VirtualBox)

# 1. Copy your SSH public key first
cat ~/.ssh/id_rsa.pub    # copy this

# 2. Edit the config section of the script
nano server-setup.sh
# Set YOUR_SSH_PUBLIC_KEY="paste your key here"
# Set TIMEZONE="Asia/Kolkata"

# 3. Run as root
sudo bash server-setup.sh

# 4. Check the report
cat /root/server-setup-report.txt

# 5. Test new user login in a NEW terminal first!
# Don't close your current session until you verify access
ssh deploy@your-server-ip
```

### What You Learned

- ✅ Full server automation
- ✅ User management at scale
- ✅ SSH security hardening
- ✅ Firewall with UFW
- ✅ Fail2ban for brute force protection
- ✅ Docker installation via script
- ✅ Error handling (`set -euo pipefail`)
- ✅ Generating setup reports
- ✅ Production-grade bash scripting

---

## Summary 📋

| Project | Level | What You Built | Key Skills |
|---|---|---|---|
| Health Monitor | 🟢 Beginner | System dashboard | bash basics, cron |
| Log Analyzer | 🟡 Intermediate | Log parser & reporter | awk, grep, automation |
| Server Setup Tool | 🔴 Advanced | Full server provisioner | security, users, firewall |

> 💪 **Challenge**: Run all 3 on a real server (AWS EC2 free tier). Put the code on GitHub. That's 3 real projects for your portfolio — all from Linux alone!