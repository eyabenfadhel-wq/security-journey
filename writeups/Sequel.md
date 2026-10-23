# HTB Starting Point — Sequel

**Difficulty:** Very Easy
**Category:** Database (MySQL/MariaDB)

## Overview

Sequel is a straightforward database misconfiguration box: a MySQL/MariaDB
instance that accepts a passwordless `root` login from a remote host, which
is all that's needed to reach the flag.

## 1. Enumeration

```bash
sudo nmap -sC -sV <target_IP>
```

Only one port is open:

| Port | Service | Version |
|------|---------|---------|
| 3306/tcp | MySQL | MariaDB 5.5.5-10.3.27-MariaDB-0+deb10u1 |

## 2. Installing the Client

A local MySQL/MariaDB client is needed to talk to the remote service:

```bash
sudo apt update && sudo apt install mysql*
```

## 3. Testing for Passwordless Authentication

A common misconfiguration left over from deployment/testing is a database
that allows login as `root` with no password set. Worth testing first since
it costs nothing:

```bash
mysql -h <target_IP> -u root
```

The connection succeeds without a password prompt, dropping into a MariaDB
shell.

## 4. Exploring the Database

```sql
SHOW databases;
```

Alongside the default system databases, a custom one named `htb` stands out.

```sql
USE htb;
SHOW tables;
```

Two tables are present: `config` and `users`.

```sql
SELECT * FROM config;
```

The `config` table contains a `flag` row with the flag value directly in the
`value` column.

## Takeaways

- Database services exposed directly to the network (not just through an
  application layer) should always be tested for weak or missing
  authentication.
- A passwordless `root` account on a production-facing database is a severe
  misconfiguration — in a real environment this would expose every table,
  not just a flag.
- Basic SQL navigation (`SHOW databases`, `USE`, `SHOW tables`, `SELECT *`) is
  enough to fully enumerate a small, undocumented database.
