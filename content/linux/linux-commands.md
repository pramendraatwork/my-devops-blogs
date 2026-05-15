---
title: "Linux — Commands Every DevOps Engineer Uses Daily"
date: 2024-02-10
draft: false
description: "Linux commands I reach for every day: files, processes, networking, text processing, monitoring."
categories: ["linux"]
tags: ["linux", "bash", "shell", "sysadmin"]
showToc: true
---

## File operations

```bash
ls -la
find / -name "*.log" -mtime -7
find /var -type f -size +100M
df -h
du -sh * | sort -rh | head -10
chmod 755 script.sh
chown -R ubuntu:ubuntu /app/
```

## Text processing

```bash
grep "ERROR" app.log
grep -i "error" app.log
grep -r "TODO" ./src/
grep -v "DEBUG" app.log
grep -A 3 -B 3 "Exception" app.log
awk '{print $1, $3}' file.txt
awk -F',' '{print $2}' data.csv
sed 's/foo/bar/g' file.txt
sed -i 's/localhost/prod-db/g' config.yml
```

## Process management

```bash
ps aux | grep nginx
kill -9 1234
pkill -f "python worker.py"
systemctl status nginx
systemctl start nginx
systemctl enable nginx
journalctl -u nginx -f
journalctl -u nginx --since "1 hour ago"
```

## Networking

```bash
ss -tlnp
lsof -i :80
curl -I https://example.com
curl -o /dev/null -s -w "%{http_code}" https://example.com
ssh -L 8080:localhost:5432 user@bastion
dig +short example.com
```

## One-liners I always use

```bash
watch -n 2 "docker ps"
watch -n 1 "kubectl get pods"
sudo !!
python3 -m http.server 8080
openssl rand -base64 32
grep -v "^#\|^$" /etc/ssh/sshd_config
```
