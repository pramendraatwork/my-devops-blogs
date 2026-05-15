+++
title = 'Linux Introduction'
date = 2026-05-15T08:59:24Z
draft = true
+++
---
title: "Linux — The Complete Beginner to DevOps Guide"
date: 2024-02-10
draft: false
description: "Everything you need to know about Linux: folder structure, user management, file management with real commands and diagrams."
categories: ["linux"]
tags: ["linux", "bash", "shell", "filesystem", "users", "files"]
showToc: true
TocOpen: true
---

## 1. What is Linux? 🐧

Linux is the **backbone of the internet**. Almost every server, cloud instance, container, and supercomputer runs Linux. As a DevOps engineer, Linux is not optional — it IS your work environment.

> 💡 **Fun fact**: Android, AWS EC2, Docker containers, Kubernetes nodes — all running Linux under the hood.

### Why Linux over Windows for DevOps?

| Feature | Linux | Windows |
|---|---|---|
| **Cost** | Free & open source | Paid license |
| **Servers** | 96% of world's servers | ~4% |
| **Shell** | Powerful bash/zsh | Limited (PowerShell) |
| **Containers** | Native support | Emulated |
| **Performance** | Lightweight, fast | Heavy |
| **Automation** | Everything scriptable | Limited |

---

## 2. Linux Folder Structure 🗂️

Linux has one single tree starting from `/` (called **root**). Everything — disks, devices, files — lives under it.

```
/  ← Root of everything
├── bin/        → Essential user commands (ls, cp, mv, cat...)
├── boot/       → Bootloader files, Linux kernel
├── dev/        → Device files (disks, USB, terminals)
├── etc/        → System configuration files
├── home/       → User home directories
│   ├── pramendra/
│   └── john/
├── lib/        → Shared libraries for binaries
├── media/      → Auto-mounted removable drives (USB, CD)
├── mnt/        → Manually mounted filesystems
├── opt/        → Optional/third-party software
├── proc/       → Virtual filesystem — running processes info
├── root/       → Home directory for root user
├── run/        → Runtime data (PIDs, sockets)
├── sbin/       → System binaries (for root — fdisk, iptables)
├── srv/        → Data for services (web, FTP)
├── sys/        → Virtual filesystem — kernel & hardware info
├── tmp/        → Temporary files (cleared on reboot)
├── usr/        → User programs and data
│   ├── bin/    → Most user commands live here
│   ├── lib/    → Libraries for /usr/bin
│   └── local/  → Locally installed software
└── var/        → Variable data — logs, databases, mail
    ├── log/    → System and app logs
    └── www/    → Web server files
```

> 🔑 **Key rule**: In Linux, **everything is a file** — even hardware devices, processes, and network sockets.

---

## 3. Directory Deep Dive 🔍

### 📁 Important System Directories

#### `/etc` — The Brain of Configuration
Every service's config file lives here. Change something here → it affects the whole system.

```bash
/etc/
├── passwd          → User account info (NOT passwords)
├── shadow          → Encrypted passwords (root only)
├── group           → Group definitions
├── hostname        → System hostname
├── hosts           → Static DNS mappings
├── fstab           → Filesystem mount table
├── crontab         → Scheduled tasks
├── ssh/            → SSH server configuration
│   └── sshd_config → SSH daemon settings
├── nginx/          → Nginx web server config
├── systemd/        → Systemd service configs
└── apt/            → Package manager config (Ubuntu/Debian)
```

```bash
# Useful commands for /etc
cat /etc/hostname           # see system hostname
cat /etc/hosts              # see host-to-IP mappings
cat /etc/passwd             # list all users
cat /etc/os-release         # Linux distribution info
```

---

#### `/var` — Where Things Change
Logs, databases, mail, print queues — anything that grows over time.

```bash
/var/
├── log/
│   ├── syslog          → General system log
│   ├── auth.log        → Authentication events (logins, sudo)
│   ├── kern.log        → Kernel messages
│   └── nginx/
│       ├── access.log  → Every HTTP request
│       └── error.log   → Nginx errors
├── lib/                → Persistent app data (databases)
├── cache/              → App cache files
└── tmp/                → Temp files that survive reboots
```

```bash
# Useful log commands
tail -f /var/log/syslog              # follow system log live
tail -f /var/log/auth.log            # watch login attempts
grep "Failed password" /var/log/auth.log   # find failed logins
journalctl -f                        # follow systemd journal
journalctl -u nginx --since "1 hour ago"   # service logs
```

---

#### `/proc` — The Window Into the Kernel
A virtual filesystem — nothing is actually stored on disk. It's the kernel exposing runtime info.

```bash
cat /proc/cpuinfo       # CPU details
cat /proc/meminfo       # RAM details  
cat /proc/uptime        # how long system has been running
cat /proc/1/status      # info about process with PID 1
ls /proc/               # each number = a running process PID
```

---

#### `/dev` — Every Device is a File
```bash
/dev/
├── sda         → First hard disk
├── sda1        → First partition of sda
├── sdb         → Second hard disk
├── tty         → Terminal
├── null        → Black hole (discard output)
├── zero        → Infinite zeros
└── random      → Random number generator
```

```bash
# Fun examples
echo "hello" > /dev/null    # discard output (no error shown)
dd if=/dev/zero of=file bs=1M count=100   # create 100MB file of zeros
cat /dev/random | head -c 16 | base64     # random bytes
```

---

### 📁 User & Application Specific Directories

```bash
~/ or /home/username/
├── .bashrc         → Bash config (aliases, env vars) loaded on each shell
├── .bash_profile   → Loaded on login shell
├── .ssh/           → SSH keys and config
│   ├── id_rsa      → Private key (never share!)
│   ├── id_rsa.pub  → Public key (safe to share)
│   └── authorized_keys → Keys allowed to SSH in
├── .config/        → App configuration files
├── .local/         → User-specific installed apps
└── .cache/         → App cache
```

```bash
# Application installations
/usr/local/bin/         → Manually installed binaries (e.g. kubectl, helm)
/opt/myapp/             → Self-contained third-party apps
~/.local/bin/           → User-installed binaries (no sudo needed)
```

> 💡 Files starting with `.` are **hidden**. Use `ls -la` to see them.

---

## 4. User Management in Linux 👥

Linux is a **multi-user system**. Every process, file, and action belongs to a user.

### User Types

```
┌─────────────────────────────────────────────┐
│  ROOT (UID 0)                               │
│  The superuser. Can do ANYTHING.            │
│  Like Administrator on Windows but scarier. │
├─────────────────────────────────────────────┤
│  SYSTEM USERS (UID 1-999)                   │
│  Created by services (nginx, mysql, www-data│
│  No login shell. Just run services safely.  │
├─────────────────────────────────────────────┤
│  REGULAR USERS (UID 1000+)                  │
│  You, your team members.                    │
│  Limited permissions. Must sudo for admin.  │
└─────────────────────────────────────────────┘
```

### Essential User Commands

```bash
# ── View users ──────────────────────────────────────────────
whoami                          # who am I right now?
id                              # my UID, GID and groups
id username                     # info about another user
cat /etc/passwd                 # all users (format: name:x:UID:GID:info:home:shell)
getent passwd username          # info about specific user
w                               # who is logged in right now
last                            # login history

# ── Create users ────────────────────────────────────────────
sudo useradd john                           # create user (no home dir)
sudo useradd -m -s /bin/bash john           # create with home dir + bash shell
sudo useradd -m -G sudo,docker john         # create + add to groups
sudo passwd john                            # set password for john

# ── Modify users ────────────────────────────────────────────
sudo usermod -aG docker john        # add john to docker group (-a = append!)
sudo usermod -aG sudo john          # give john sudo access
sudo usermod -s /bin/zsh john       # change shell to zsh
sudo usermod -l newname oldname     # rename user
sudo usermod -L john                # lock account (disable login)
sudo usermod -U john                # unlock account

# ── Delete users ────────────────────────────────────────────
sudo userdel john                   # delete user (keep home dir)
sudo userdel -r john                # delete user AND home directory

# ── Switch users ────────────────────────────────────────────
su - john                           # switch to john (full login)
sudo -i                             # switch to root shell
sudo -u john command                # run command as john
sudo command                        # run command as root
```

### Groups — Controlling Access

```bash
# View groups
cat /etc/group                  # all groups
groups                          # groups I belong to
groups john                     # groups john belongs to

# Create and manage groups
sudo groupadd developers        # create a group
sudo groupadd -g 1500 devops    # create with specific GID
sudo groupdel developers        # delete group
sudo gpasswd -d john developers # remove john from developers
```

### The `/etc/passwd` File Explained

```bash
# Format: username:x:UID:GID:comment:home:shell
pramendra:x:1000:1000:Pramendra Rajput:/home/pramendra:/bin/bash
#    │     │  │    │        │               │              │
#  name  pwd UID  GID    full name        home dir       shell
#        (x = in /etc/shadow)
```

### sudo — Running as Root Safely

```bash
sudo command                    # run one command as root
sudo -i                         # open root shell (be careful!)
sudo visudo                     # safely edit sudoers file

# Give a user sudo access
sudo usermod -aG sudo username  # Ubuntu/Debian
sudo usermod -aG wheel username # CentOS/RHEL

# Run as another user
sudo -u www-data ls /var/www/
```

> ⚠️ **Rule of least privilege**: never run apps as root. Create a dedicated user for each service.

---

## 5. File Management in Linux 📄

### File Permissions — The Most Important Concept

Every file has 3 sets of permissions for 3 types of users:

```
-rwxr-xr--  1  pramendra  devops  4096  Jan 15  script.sh
│└─┬──┘└─┬─┘└──────┬─────────┘
│  │     │         └── Owner: pramendra | Group: devops
│  │     └── Others: r-- (read only)
│  └── Group: r-x (read + execute)
└── Owner: rwx (read + write + execute)
│
└── File type: - = file, d = directory, l = symlink
```

```
Permission  Symbol  Number
Read        r       4
Write       w       2
Execute     x       1
None        -       0

Common combinations:
rwx = 4+2+1 = 7  (full access)
rw- = 4+2+0 = 6  (read/write)
r-x = 4+0+1 = 5  (read/execute)
r-- = 4+0+0 = 4  (read only)
```

### File Permission Commands

```bash
# ── View permissions ─────────────────────────────────────────
ls -la                          # list with permissions
stat file.txt                   # detailed file info
namei -l /path/to/file          # permissions of entire path

# ── Change permissions ───────────────────────────────────────
chmod 755 script.sh             # rwxr-xr-x (owner all, others read+exec)
chmod 644 config.txt            # rw-r--r-- (owner rw, others read)
chmod 600 ~/.ssh/id_rsa         # rw------- (owner only — required for SSH keys!)
chmod +x script.sh              # add execute for everyone
chmod -w file.txt               # remove write for everyone
chmod -R 755 /var/www/          # recursive — entire directory

# ── Change ownership ─────────────────────────────────────────
chown pramendra file.txt            # change owner
chown pramendra:devops file.txt     # change owner and group
chown -R www-data:www-data /var/www/ # recursive ownership
chgrp devops file.txt               # change group only
```

### Essential File Commands

```bash
# ── Create ───────────────────────────────────────────────────
touch file.txt                  # create empty file / update timestamp
touch file1.txt file2.txt       # create multiple files
mkdir mydir                     # create directory
mkdir -p a/b/c/d                # create nested dirs (no error if exists)

# ── Copy ─────────────────────────────────────────────────────
cp file.txt backup.txt          # copy file
cp -r dir/ backup-dir/          # copy directory recursively
cp -p file.txt dest/            # preserve permissions + timestamps
cp -u *.txt dest/               # copy only if newer

# ── Move & Rename ────────────────────────────────────────────
mv file.txt /tmp/               # move file
mv oldname.txt newname.txt      # rename file
mv -i file.txt dest/            # ask before overwriting

# ── Delete ───────────────────────────────────────────────────
rm file.txt                     # delete file
rm -f file.txt                  # force delete (no error if missing)
rm -r mydir/                    # delete directory
rm -rf mydir/                   # force delete directory (CAREFUL!)
rmdir emptydir/                 # delete empty directory only

# ── View files ───────────────────────────────────────────────
cat file.txt                    # print entire file
less file.txt                   # scroll through file (q to quit)
head -20 file.txt               # first 20 lines
tail -20 file.txt               # last 20 lines
tail -f /var/log/app.log        # follow file live (Ctrl+C to stop)
wc -l file.txt                  # count lines
wc -w file.txt                  # count words

# ── Find files ───────────────────────────────────────────────
find / -name "nginx.conf"                   # find by name
find /var -type f -name "*.log"             # find log files
find /home -type f -size +10M              # files larger than 10MB
find . -mtime -7                            # modified in last 7 days
find . -perm 777                            # files with 777 permissions
find / -user pramendra -type f             # all files owned by user
locate nginx.conf                           # fast search (uses index)
which python3                               # find command location
whereis nginx                               # find binary + man page

# ── Links ────────────────────────────────────────────────────
ln file.txt hardlink.txt                    # hard link (same inode)
ln -s /path/to/file symlink.txt             # symbolic link (like shortcut)
ln -s /usr/local/bin/python3 /usr/bin/python # common pattern
```

### File Archiving & Compression

```bash
# tar — most common
tar -czf archive.tar.gz folder/     # create compressed archive
tar -xzf archive.tar.gz             # extract archive
tar -xzf archive.tar.gz -C /tmp/    # extract to specific directory
tar -tzf archive.tar.gz             # list contents without extracting

# zip
zip -r archive.zip folder/          # create zip
unzip archive.zip                   # extract zip
unzip -l archive.zip                # list contents

# Quick reference
# c = create  x = extract  z = gzip  j = bzip2  v = verbose  f = filename
```

### Viewing Disk Usage

```bash
df -h                           # disk space of all filesystems
df -h /var                      # disk space of specific path
du -sh /var/log/                # size of a directory
du -sh * | sort -rh | head -10  # top 10 largest items
du -ah /home/ | sort -rh | head # all files sorted by size
lsblk                           # list block devices (disks + partitions)
```

---

## Quick Reference Cheatsheet 📋

```bash
# USER MANAGEMENT
whoami                          # current user
sudo useradd -m -s /bin/bash username   # create user
sudo passwd username            # set password
sudo usermod -aG groupname username     # add to group
sudo userdel -r username        # delete user + home

# FILE PERMISSIONS
chmod 755 file      # rwxr-xr-x
chmod 644 file      # rw-r--r--
chmod 600 file      # rw------- (private)
chown user:group file
chmod -R 755 dir/   # recursive

# FILE OPERATIONS
ls -la              # list all with permissions
find . -name "*.log"
cp -r src/ dest/
mv old new
rm -rf dir/
tar -czf out.tar.gz dir/
tar -xzf archive.tar.gz

# NAVIGATION
cd ~                # go home
cd -                # go back
pwd                 # where am I
tree -L 2           # visual tree
```

---

## What's Next? 🚀

Now that you understand Linux fundamentals, the next steps in your DevOps journey:

- **Git & GitHub** — version control your code and configs
- **Shell Scripting** — automate repetitive Linux tasks
- **Networking** — understand how Linux handles network traffic
- **Systemd** — manage services like nginx, docker, postgresql

> 💪 **Practice tip**: Spin up a free EC2 instance on AWS or use WSL on Windows and practice these commands daily. Muscle memory is everything in Linux.