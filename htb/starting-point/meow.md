# Meow (HTB Starting Point, Tier 0)

> **Platform:** Hack The Box
> **Category:** Starting Point (Tier 0)
> **Status:** Completed on 2026-10-01

| | |
|---|---|
| **Difficulty** | Very Easy |
| **OS** | Linux |
| **Skills Practiced** | Port scanning, service enumeration, weak credentials |

> No flags, passwords, or hashes are published in this writeup, in line with platform rules.

---

## 1. Summary

Meow is the first machine in Hack The Box's Starting Point track and is aimed at complete beginners. An Nmap scan showed Telnet (port 23) as the only open service. Telnet is an old, unencrypted remote-login protocol, and the `root` account on this machine had no password set. Logging in as `root` with a blank password gave immediate full control of the system, with no exploit needed. The machine shows how a single misconfiguration on an exposed service can lead to a complete compromise.

---

## 2. Reconnaissance

### Port Scan

```bash
nmap -A <TARGET_IP>
```

**What I was looking for:** Open ports, service versions, and anything unusual.

**Result:**

| Port | Service | Notes |
|------|---------|---------|-------|
| 23/tcp | Telnet | Only open port; remote login service |

### Analysis

The scan showed Telnet (port 23) open. Telnet is an old, unencrypted remote-login protocol, so it was the obvious service to investigate first.

---

## 3. Initial Access (Foothold)

### Approach

Since Telnet was exposed, I tried logging in with common default usernames, starting with `root`.

### Steps

```bash
telnet <TARGET_IP>
```

When prompted for a username, I entered `root`. At the password prompt I pressed Enter, leaving it blank, and the login succeeded with a root shell.

---

## 4. Privilege Escalation

Not required. The `root` login already gave full access.

---

## 5. Flag

The flag was in `flag.txt`. I read it with `cat flag.txt` (value not shown).

---

## 6. Vulnerability Analysis

| | |
|---|---|
| **Weakness** | Telnet exposed with no password set on the root account |
| **Type** | Misconfiguration / Weak credentials |
| **Impact** | Full remote system access |
| **Related CWE** | CWE-258: Empty Password in Configuration File |

The machine was exploitable because it combined an unencrypted remote-access service with a privileged account that had no password, so no exploit was needed.

---

## 7. Mitigation

1. **Replace Telnet with SSH.** Telnet sends everything, including credentials, in plaintext. SSH encrypts the session, and Telnet should be disabled if it isn't needed.
2. **Set a strong password on every account, especially root.** A blank password meant anyone who reached the service had full control.
3. **Disable direct root login over the network.** Use a normal user with `sudo`, so an attacker needs a second step to gain admin rights.

---

## 8. Lessons Learned

- **Concepts:** Telnet transmits everything, including credentials, in plaintext, so it shouldn't be exposed to a network. I also learned that a service doesn't need a vulnerability to be exploitable: a weak or empty password on a privileged account is enough.
- **Mistakes / surprises:** I was surprised that logging in as `root` with a blank password actually worked, and that it gave full system access straight away.
- **Takeaway:** Enumeration comes first. A simple `nmap -A` scan showed me where to look, and trying default credentials is a basic step worth doing early on any exposed login service.

---

## 9. Tools Used

| Tool | Purpose |
|------|---------|
| nmap | Port and service scanning |
| telnet | Remote connection |

---

*Written by Adil Haneef. *Written by Adil Haneef |· [Hack The Box](https://profile.hackthebox.com/profile/019f98bb-4497-71d9-bb45-d8da415bd4db?utm_medium=copy_url) · [LinkedIn](https://linkedin.com/in/adil-haneef-263521343)*
*Attacks were performed only on authorised lab environments.*Attacks were performed only on authorised lab environments.*
