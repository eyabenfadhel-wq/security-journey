# HTB Starting Point — Crocodile

**Difficulty:** Very Easy
**Category:** FTP / Web / Credential Reuse

## Overview

Crocodile chains two weak points together: an FTP server that allows anonymous
access, and a web admin panel reachable through directory brute-forcing. The
credentials leaked via FTP end up being valid for the admin panel.

## 1. Enumeration

Start with a full version/script scan:

```bash
sudo nmap -sC -sV <target_IP>
```

Results:

| Port | Service | Version |
|------|---------|---------|
| 21/tcp | FTP | vsFTPd 3.0.3 |
| 80/tcp | HTTP | Apache httpd 2.4.41 (Ubuntu) |

The `ftp-anon` script flags that anonymous login is allowed (`FTP code 230`),
and the scan already lists two files sitting in the FTP root:
`allowed.userlist` and `allowed.userlist.passwd`.

## 2. FTP — Anonymous Access

```bash
ftp <target_IP>
Name: anonymous
Password: <blank>
```

Login succeeds with code `230`. Listing the directory (`dir`) confirms the two
files nmap already spotted. Download both:

```
ftp> get allowed.userlist
ftp> get allowed.userlist.passwd
ftp> exit
```

Reading them locally gives a list of usernames and a separate list of
passwords:

```bash
cat allowed.userlist
cat allowed.userlist.passwd
```

This produces four usernames and four passwords — not explicitly paired, so
they need to be tried against whatever login surface exists.

## 3. Trying the Credentials on FTP

Logging back into FTP with any of the harvested usernames fails with
`530 This FTP server is anonymous only`, so these credentials aren't meant for
FTP itself — they must belong to another service.

## 4. Web Enumeration

Port 80 serves a generic business/portfolio template built with Apache, PHP
and Bootstrap. Nothing on the visible pages hints at an admin area, so the
next step is directory brute-forcing:

```bash
gobuster dir --url http://<target_IP>/ \
  --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt \
  -x php,html
```

This turns up `/login.php` — a username/password form for what turns out to
be a "Server Manager" admin panel.

## 5. Credential Stuffing the Login Form

With only four usernames and four passwords, it's fast enough to try the
combinations by hand. The pair that works is:

```
admin : Supersecretpassword1
```

This logs into the Server Manager dashboard, where the flag is displayed
directly on the landing page.

## Takeaways

- Always check for anonymous FTP access — it's a common misconfiguration and
  an easy source of leaked credentials or leftover files.
- Small credential lists found during enumeration are worth testing by hand
  before reaching for an automated brute-force tool.
- Directory brute-forcing is essential when a web app doesn't expose its
  admin/login surface through visible navigation.
