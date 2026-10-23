# HTB Starting Point — Appointment

**Difficulty:** Very Easy
**Category:** Web / SQL Injection

## Overview

Appointment is an introduction to SQL injection authentication bypass. The
site's login form builds a raw SQL query from user input with no
sanitization, which lets an attacker log in without knowing a valid password.

## 1. Enumeration

```bash
sudo nmap -sC -sV <target_IP>
```

Only port 80 is open, running `Apache httpd 2.4.38 (Debian)`. The nmap script
output already identifies the page title as "Login."

## 2. Initial Recon

Browsing to the target shows a login form. There's no public registration or
visible admin link, so the only way in is through this form itself.

Directory brute-forcing was run for completeness:

```bash
gobuster dir --url http://<target_IP>/ \
  --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt \
  -x php,html
```

It only returns standard asset folders (`css`, `js`, `images`, `vendor`,
`fonts`) — nothing that changes the plan of attack.

## 3. Credential Guessing (Dead End)

Common default combinations (`admin:admin`, `guest:guest`, `root:root`, etc.)
were tried against the login form and all failed. At this point the form
itself becomes the target rather than a gate to get past with guesses.

## 4. SQL Injection Authentication Bypass

A typical vulnerable login query looks like this server-side:

```sql
SELECT * FROM users WHERE username='$username' AND password='$password';
```

If the username field isn't sanitized, a single quote lets an attacker break
out of the string literal, and a `#` comments out the rest of the query
(including the password check). Submitting:

```
Username: admin'#
Password: anything
```

turns the executed query into:

```sql
SELECT * FROM users WHERE username='admin'#' AND password='anything';
```

Everything after `#` is ignored, so the query becomes a lookup for any row
where `username = 'admin'`. Since an `admin` account exists, the query
returns exactly one row and the application logs the attacker in without
ever checking the password.

## 5. Result

After submitting the payload, the application redirects to a page confirming
a successful login and displaying the flag.

## Takeaways

- Never trust user input in a SQL query — this box is the textbook example
  of why parameterized queries / prepared statements exist.
- A single quote (`'`) and a comment marker (`#` or `--`) are often enough to
  probe a login form for classic SQLi.
- This specific technique only requires knowing (or guessing) one valid
  username — the password field becomes irrelevant once the rest of the
  query is commented out.
