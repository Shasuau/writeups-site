---
# ============================================
# COPY THIS FILE FOR EVERY NEW WRITEUP
# Filename format: YYYY-MM-DD-box-name.md
# Place in: _posts/
# ============================================

layout: writeup
title: "Box Name Here"
date: 2026-03-26

# Card metadata (shown on homepage)
platform: HackTheBox          # HackTheBox | HTB Academy
difficulty: Easy               # Easy | Medium | Hard
os: Linux                      # Linux | Windows | Other
status: Retired                # Always Retired before publishing

# The one-liner shown on the homepage card (keep it honest and human)
stuck_summary: "Short description of where you got stuck — shown on the card"

# Tags — used for filtering and SEO. Include: os, tools, techniques, CVEs
tags:
  - linux
  - smb
  - metasploit
  - cve-2007-2447
  - privesc
---

## TL;DR

Just the commands, no fluff. For people who are almost there and need a nudge.

```bash
nmap -sC -sV -oN nmap/initial 10.10.10.X
# command 2
# command 3
```

---

## Given Info

**Box description:** One sentence on what the box is about.

**Recommended modules:**
- Module 1
- Module 2

---

## Where I Got Stuck

> Be honest here. This is the most valuable section for other people.

- Spent X minutes on Y before realising Z
- Kept getting [error] — turned out to be [reason]
- Wrong assumption: I thought X meant Y, it actually meant Z

---

## Enumeration

### Nmap

```bash
nmap -sC -sV -p- --min-rate 5000 -oN nmap/full 10.10.10.X
```

**Results:**

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
22/tcp open  ssh     OpenSSH 4.7p1
139/tcp open  netbios-ssn Samba 3.X
445/tcp open  microsoft-ds Samba 3.X
```

**Interesting findings:**
- Port X is running Y — worth investigating because Z
- Version number suggests possible CVE — [how you spotted it]

### Gobuster / ffuf

```bash
gobuster dir -u http://10.10.10.X -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

**Results:**
- `/admin` — [what you found]
- `/uploads` — [what you found]

---

## Attack Plan

### Option A: [Name of approach] ✗

**Why you tried it:**

1. Command you ran:
```bash
your command here
```
Result:
```
output here
```

2. Next step:
```bash
your command here
```
Result:
```
output here
```

**Why it failed:** Explain what went wrong so readers don't repeat your mistake.

---

### Option B: [Name of approach] ✓

**Why this worked:**

1. Step one:
```bash
your command here
```
Result:
```
output here
```

2. Step two:
```bash
your command here
```
Result:
```
output here — shell obtained / foothold established
```

---

## Post Exploitation / Privilege Escalation

1. Enumerate for privesc:
```bash
your command here
```

2. Found [X] — exploited via:
```bash
your command here
```

Result: `root shell obtained`

---

## Flags

<details>
<summary>🚩 User Flag — click to reveal</summary>

[paste user flag hash here]

</details>

<details>
<summary>🚩 Root Flag — click to reveal</summary>

[paste root flag hash here]

</details>
