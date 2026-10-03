# Dancing (HTB Starting Point, Tier 0)

> **Platform:** Hack The Box
> **Category:** Starting Point (Tier 0)
> **Status:** Completed on 2026-10-03

| | |
|---|---|
| **Difficulty** | Easy |
| **OS** | Windows |
| **Skills Practiced** | Port scanning, service enumeration, SMB share enumeration, null session access |

> No flags, passwords, or hashes are published in this writeup, in line with platform rules.

---

## 1. Summary

Dancing is a Starting Point machine that introduces SMB (Server Message Block). An Nmap scan showed SMB running on a Windows host, and enumerating the available shares revealed one, `WorkShares`, that could be accessed without credentials. From there I navigated the share and retrieved the flag.

---

## 2. Reconnaissance

### Port Scan

```bash
nmap -sV <TARGET_IP>
```

**Result:**

```text
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

| Port | Service | Notes |
|------|---------|-------|
| 135/tcp | msrpc | Microsoft Windows RPC |
| 139/tcp | netbios-ssn | NetBIOS session service |
| 445/tcp | microsoft-ds | SMB over TCP |
| 5985/tcp | http | WinRM (HTTPAPI) |

### Analysis

Port 445 running `microsoft-ds` confirmed SMB was exposed. Combined with the Windows OS fingerprint, this meant the next step was enumerating SMB shares rather than looking at the other open ports.

---

## 3. Initial Access (Foothold)

### Approach

With SMB confirmed open, I listed the available shares, then tried connecting to each one to see which allowed access without credentials.

### Steps

```bash
smbclient -L <TARGET_IP>
```

```text
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk
```

I then tried connecting to each share:

```bash
smbclient //<TARGET_IP>/ADMIN$ -N
# tree connect failed: NT_STATUS_ACCESS_DENIED

smbclient //<TARGET_IP>/C$ -N
# tree connect failed: NT_STATUS_ACCESS_DENIED

smbclient //<TARGET_IP>/IPC$ -N
# connects, but IPC$ is for inter-process communication, not file browsing

smbclient //<TARGET_IP>/WorkShares -N
# connects successfully
```

`ADMIN$` and `C$` require administrator credentials and refused the anonymous login. `IPC$` accepted the connection but isn't a file share. `WorkShares` accepted the anonymous (`-N`) login and gave me a file listing.

Once inside `WorkShares`, I navigated it using standard `smbclient` commands (`ls`, `cd`, `get`) until I located the flag file, then downloaded it with `get`.

### What Didn't Work

- `ADMIN$` and `C$` both refused anonymous access (`NT_STATUS_ACCESS_DENIED`), which told me those required real credentials rather than being misconfigured like `WorkShares`.

---

## 4. Vulnerability Analysis

| | |
|---|---|
| **Weakness** | SMB share (`WorkShares`) accessible via anonymous/null session, with no credentials required |
| **Type** | Misconfiguration |
| **Impact** | Unauthenticated access to files on the share |

---

## 5. Mitigation

1. **Enforce authentication on shares.** `WorkShares` should require a valid domain or local user account to connect — it should not accept a null/anonymous session. Once authenticated, that account should only have access to the specific folders it needs, rather than broad read/write access across the whole share.
2. **Disable anonymous SMB access at the OS level**, so no share on the host accepts a blank-credential connection by default.
3. **Restrict SMB exposure on the network.** Ports 139/445 shouldn't be reachable from untrusted networks. Limit access to the hosts and subnets that actually need file sharing (internal LAN only), and never expose SMB to the internet.

---

## 6. Lessons Learned

SMB can be an easy way to extract data if a share is left open — even someone with very little SMB knowledge can get in. It's one of the first things a beginner learns, which is exactly what makes it dangerous when misconfigured. During recon, the first thing I'd check now is whether open ports and services point to a situation like this: a small misconfiguration, like anonymous login being enabled, is enough to let anyone in.

`IPC$` was also open, but it's a default, virtual Windows share used for inter-process communication — it doesn't point to an actual folder. `WorkShares`, on the other hand, is a normal human-created share meant for people to edit, copy, or open documents and media, which is why it actually exposed files.

- **Mistakes / surprises:** It's surprising how little knowledge someone needs to try this. Just recognizing "port 445 open, OS Windows, service microsoft-ds" is enough for someone to attempt access, which shows how low the bar is for this kind of attack.

---

## 7. Tools Used

| Tool | Purpose |
|------|---------|
| nmap | Port and service scanning |
| smbclient | Enumerating and connecting to SMB shares |

---

*Written by Adil Haneef | [Hack The Box](https://profile.hackthebox.com/profile/019f98bb-4497-71d9-bb45-d8da415bd4db) · [LinkedIn](https://linkedin.com/in/adil-haneef-263521343)*
*Attacks were performed only on authorised lab environments.*
