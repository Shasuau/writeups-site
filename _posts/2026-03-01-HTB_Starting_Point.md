---
layout: writeup
title: "HTB Starting Point - Tier 0"
date: 2026-03-01

platform: HackTheBox
difficulty: Easy
os: Linux
status: Retired

stuck_summary: "Tier 0 exists to teach one concept per box — if you're stuck, the answer is almost always the simplest thing you haven't tried yet"

tags:
  - ftp
  - rdp
  - telnet
  - smb
  - redis
  - mongodb
  - rsync
  - gobuster
  - anonymous-access
  - default-credentials
  - starting-point
---

## What is Tier 0?

Tier 0 is HTB's Starting Point — eight short labs designed to introduce one concept each. No exploit development, no CVEs, no privilege escalation. Just foundational service knowledge.

Each box teaches a single lesson: **default credentials exist**, **anonymous access is common**, **services talk on predictable ports**. These habits — checking for anonymous access, trying default credentials, reading service banners — carry into every box you'll ever do.

Flags are hidden per box below. Commands are real, explanations are included.

---

## Meow — Telnet No Authentication

**Lesson:** Some services have no authentication at all.

**Port:** 23 (Telnet)

Telnet is an ancient remote access protocol — think SSH but with zero encryption and sometimes zero authentication. Still found on embedded devices, old routers, and HTB labs.

```bash
nmap -sV 10.129.x.x
# 23/tcp open telnet

telnet 10.129.x.x
# Username: root
# Password: (blank — just press enter)
```

Logged straight in as root. No password required.

> Telnet has no encryption and frequently ships with blank credentials on embedded hardware. In the real world this shows up on routers, IoT devices, and industrial control systems. Always check port 23 with a blank password before moving on.

<details>
<summary>🚩 Flag — click to reveal</summary>

HTB Starting Point flags rotate — yours will appear in `/root/flag.txt` after logging in.

</details>

---

## Fawn — FTP Anonymous Login

**Lesson:** FTP commonly allows anonymous access by default.

**Port:** 21 (FTP)

FTP (File Transfer Protocol) is used to transfer files between systems. Many FTP servers are configured to allow "anonymous" login — a convention that lets anyone connect without real credentials.

```bash
nmap -sC -sV 10.129.x.x
# 21/tcp open ftp vsftpd x.x
# Anonymous FTP login allowed (confirmed by -sC)

ftp 10.129.x.x
# Username: anonymous
# Password: (blank)

ls
# flag.txt

get flag.txt
# Downloaded locally

cat flag.txt
```

> The `-sC` flag runs Nmap's default scripts — one of which specifically checks for anonymous FTP. Always run both `-sV` and `-sC` together. Anonymous FTP is surprisingly common on misconfigured servers and often contains sensitive files left there by accident.

<details>
<summary>🚩 Flag — click to reveal</summary>

Retrieved via `cat flag.txt` after downloading with `get`.

</details>

---

## Dancing — SMB Anonymous Share

**Lesson:** SMB shares are often accessible without credentials.

**Port:** 445 (SMB)

SMB (Server Message Block) is Windows' file sharing protocol. Shares can be configured to allow anonymous (null session) access — meaning no username or password required to browse and download files.

```bash
nmap -sV 10.129.x.x
# 445/tcp open microsoft-ds

# List available shares — -N means no password
smbclient -L //10.129.x.x -N
# ADMIN$, C$, IPC$, WorkShares

# Connect to the accessible share
smbclient //10.129.x.x/WorkShares -N

# Navigate and find the flag
ls
cd James.P
ls
# flag.txt

get flag.txt
exit

cat flag.txt
```

> `ADMIN$` and `C$` are default administrative shares — they typically require credentials. `WorkShares` (or similar named shares) are custom shares that are more likely to be misconfigured. Always check non-default share names first.

<details>
<summary>🚩 Flag — click to reveal</summary>

Retrieved via `get flag.txt` inside the WorkShares/James.P directory.

</details>

---

## Explosion — RDP Default Credentials

**Lesson:** Default or blank credentials work more often than they should.

**Port:** 3389 (RDP)

RDP (Remote Desktop Protocol) gives you a full graphical desktop on a Windows machine. Like any service, it's only as secure as its credentials. Administrator accounts with blank passwords are a common misconfiguration.

```bash
nmap -sV 10.129.x.x
# 3389/tcp open ms-wbt-server (RDP)

xfreerdp /v:10.129.x.x /u:Administrator /cert:ignore
# Password: (blank — just press enter)
```

A full Windows desktop appears. Flag is sitting on the desktop.

> `xfreerdp` is the Linux RDP client. `/cert:ignore` bypasses the certificate warning — fine for labs, never do this in production. The Administrator account with a blank password is one of the first things to check on any Windows service. Also try `admin`, `guest`, and `user` with blank passwords.

<details>
<summary>🚩 Flag — click to reveal</summary>

Flag is visible on the Windows desktop after connecting.

</details>

---

## Preignition — Gobuster + Default Web Credentials

**Lesson:** Hidden web pages exist, and default credentials are everywhere.

**Port:** 80 (HTTP/Nginx)

This box chains two concepts: directory enumeration to find a hidden admin page, then default credentials to log in.

```bash
nmap -sV 10.129.x.x
# 80/tcp open http nginx 1.14.2

# Enumerate directories — -x php checks for PHP files specifically
gobuster dir -u http://10.129.x.x -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php
# /admin.php (Status: 200)
```

Navigated to `/admin.php` — login form appeared.

Tried `admin:admin` — logged in. Flag on the dashboard.

> Always add `-x php` (or `-x php,html,txt`) when gobuster finds nothing obvious. Web servers hide a lot behind file extensions. Default credentials (`admin:admin`, `admin:password`, `admin:1234`) should always be your first attempt on any login form before trying anything else.

<details>
<summary>🚩 Flag — click to reveal</summary>

Visible on the dashboard after logging in with admin:admin.

</details>

---

## Redeemer — Redis Unauthenticated Access

**Lesson:** Databases exposed to the network without authentication are an immediate critical finding.

**Port:** 6379 (Redis)

Redis is an in-memory data store — think a fast key-value database often used for caching. It's designed to run on internal networks behind a firewall. When exposed publicly with no authentication, everything inside is readable.

```bash
nmap -sV -p 6379 10.129.x.x
# 6379/tcp open redis Redis 5.0.7

# Connect directly — no credentials needed
redis-cli -h 10.129.x.x

# Check server info
INFO

# List all keys in the default database
SELECT 0
KEYS *
# flag

# Read the flag
GET flag
```

> Redis with no authentication on a public port is one of the most common misconfigurations in cloud environments. Exposed Redis instances have been used in real-world attacks to write SSH keys, execute code, and steal data. If you see port 6379 open, always try `redis-cli -h {ip}` immediately.

<details>
<summary>🚩 Flag — click to reveal</summary>

Retrieved via `GET flag` in redis-cli.

</details>

---

## Mongod — MongoDB Unauthenticated Access

**Lesson:** Same problem as Redis, different database. Unauthenticated databases are critical findings.

**Port:** 27017 (MongoDB)

MongoDB is a NoSQL document database. Like Redis, it's designed for internal use and frequently misconfigured to accept connections without authentication.

```bash
nmap -sV -p 27017 10.129.x.x
# 27017/tcp open mongodb MongoDB 3.6.8

# Connect — no credentials needed
mongosh 10.129.x.x
# Note: version mismatch warnings are normal, connection still works

# List databases
show dbs
# sensitive_information

# Switch to it
use sensitive_information

# List collections (like tables)
show collections
# flag

# Read the flag
db.flag.find()
```

> If `mongosh` gives version errors, try `mongo` instead (older client). The database structure mirrors SQL concepts: databases → collections → documents. `show dbs`, `show collections`, and `db.collection.find()` are your three core recon commands.

<details>
<summary>🚩 Flag — click to reveal</summary>

Retrieved via `db.flag.find()` in the sensitive_information database.

</details>

---

## Synced — Rsync Anonymous Access

**Lesson:** File sync services can expose everything if misconfigured.

**Port:** 873 (Rsync)

Rsync is a file synchronisation tool — commonly used for backups. Like FTP, it can be configured to allow anonymous access. Unlike FTP, most people forget it exists on port 873.

```bash
nmap -sV -p 873 10.129.x.x
# 873/tcp open rsync protocol version 31

# List available shares — double colon syntax
rsync --list-only 10.129.x.x::
# public

# List contents of the public share
rsync --list-only 10.129.x.x::public
# flag.txt

# Download the flag
rsync 10.129.x.x::public/flag.txt flag.txt

cat flag.txt
```

> The `::` syntax is rsync-specific — it means "list shares on this server" the same way `smbclient -L` lists SMB shares. Rsync on port 873 is easy to miss during enumeration. If you're doing a full port scan (`-p-`) you'll catch it. Another reason to never rely on the default top-1000 port scan alone.

<details>
<summary>🚩 Flag — click to reveal</summary>

Retrieved via `rsync {ip}::public/flag.txt` then `cat flag.txt`.

</details>

---

## What Tier 0 Actually Teaches

Each box here demonstrates the same underlying truth: **services are only as secure as their configuration**. Anonymous FTP, blank RDP passwords, unauthenticated Redis — none of these are vulnerabilities in the traditional sense. They're misconfigurations. And misconfigurations are responsible for the majority of real-world breaches.

The habits built here — check for anonymous access, try default credentials, read service banners carefully, enumerate everything — apply to every box above Easy and every real engagement you'll ever do.

**Key commands to remember:**

| Service | Quick check |
|---------|-------------|
| FTP | `ftp {ip}` → username: anonymous |
| SMB | `smbclient -L //{ip} -N` |
| Telnet | `telnet {ip}` → try root/blank |
| RDP | `xfreerdp /v:{ip} /u:Administrator /cert:ignore` |
| Redis | `redis-cli -h {ip}` → `KEYS *` |
| MongoDB | `mongosh {ip}` → `show dbs` |
| Rsync | `rsync --list-only {ip}::` |
| Web | `gobuster dir` + try `admin:admin` |
