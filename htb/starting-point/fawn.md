# Fawn (HTB Starting Point, Tier 0)

> **Platform:** Hack The Box
> **Category:** Starting Point (Tier 0)
> **Status:** Completed on 2026-10-01

| | |
|---|---|
| **Difficulty** | Very Easy |
| **OS** | Linux |
| **Skills Practiced** | Port scanning, service enumeration, anonymous FTP access |

> No flags, passwords, or hashes are published in this writeup, in line with platform rules.

---

## 1. Summary

Fawn is the second machine in Hack The Box's Starting Point track and introduces FTP. An Nmap scan showed an FTP server on port 21, and a common misconfiguration is leaving anonymous login enabled. That was the case here, so I logged in without credentials and downloaded the flag file.

---

## 2. Reconnaissance

### Port Scan

```bash
nmap -sV <TARGET_IP>
```

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
Service Info: OS: Unix
```

**What I was looking for:** Open ports and service versions.

| Port | Service | Version | Notes |
|------|---------|---------|-------|
| 21/tcp | FTP | vsftpd 3.0.3 | Only open port |

### Analysis

The `-sV` scan showed FTP as the only open service, so it was the only place to start.

---

## 3. Initial Access (Foothold)

### Approach

FTP was the only exposed service, and a common FTP misconfiguration is allowing anonymous login, so I tried that first.

### Steps

```bash
ftp <TARGET_IP>
```

At the `Name` prompt I entered `anonymous`. I left the password blank by pressing Enter, and the server let me in.

**Note:** an FTP session isn't a normal shell, so `cat` doesn't work. It has its own commands, such as `ls`, `dir` and `get`.

---

## 4. Privilege Escalation

Not required. Anonymous access was enough to reach the flag file.

---

## 5. Flag

I ran `ls` to list the files, then downloaded the flag file to my machine with `get <filename>`. After logging out, I read it from my own terminal with `cat`. (The value isn't shown.)

---

## 6. Vulnerability Analysis

| | |
|---|---|
| **Weakness** | FTP server allowing anonymous login |
| **Type** | Misconfiguration |
| **Impact** | Unauthenticated access to files on the server |

Anonymous login lets anyone connect without credentials. Here it was enabled on a server that held a file that should not have been public.

---

## 7. Mitigation

1. **Disable anonymous login.** In vsftpd, set `anonymous_enable=NO` in the configuration file.
2. **Require authentication and encrypt the connection.** Plain FTP sends data in cleartext, so use SFTP or FTPS.
3. **Restrict access.** Limit port 21 to trusted IPs with a firewall, or disable the service if it isn't needed.
4. **Keep sensitive files out of publicly reachable directories.**

---

## 8. Lessons Learned

- **What anonymous FTP is:** It lets anyone access files on a server without a personal account, using the username `anonymous` and a blank password.
- **Why `cat` doesn't work:** FTP is designed for transferring files, not for running shell commands, so I had to download the file with `get` and read it locally.
- **Mistakes / surprises:** I was surprised by how easy it was. One small misconfiguration gave access to the server's files.

---

## 9. Tools Used

| Tool | Purpose |
|------|---------|
| nmap | Port and service scanning |
| ftp | Connecting to the FTP server and downloading the file |

---

*Written by Adil Haneef | [Hack The Box](https://profile.hackthebox.com/profile/019f98bb-4497-71d9-bb45-d8da415bd4db) · [LinkedIn](https://linkedin.com/in/adil-haneef-263521343)*
*Attacks were performed only on authorised lab environments.*
