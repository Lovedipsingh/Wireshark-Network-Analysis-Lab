# 📋 Executive Summary — Network Analysis Findings

**Analyst:** Lovedip Singh
**Date:** March 22, 2026
**Target:** Metasploitable2 (192.168.56.102)
**Classification:** Educational Lab — NOT Production

---

## Key Findings At A Glance

| Metric | Value |
|---|---|
| Total Packets Captured | 2,626 |
| Open Ports Discovered | 21 |
| Critical Vulnerabilities | 5 |
| High Severity Issues | 8 |
| Overall Risk Rating | CRITICAL |

---

## Top 5 Critical Issues

1. **vsftpd 2.3.4 Backdoor** — Remote root access via FTP username trick
2. **Root Bindshell (Port 1524)** — Unauthenticated root shell open to network
3. **Legacy Remote Shell Protocols** — rexec/rlogin/rsh transmit credentials in plaintext
4. **UnrealIRCd Backdoor** — IRC server with known backdoor vulnerability
5. **Exposed Database Services** — MySQL and PostgreSQL accessible from network

---

## Tools & Techniques Demonstrated

- Network host discovery with Nmap
- Service version detection
- Live packet capture with Wireshark
- Protocol filtering and analysis
- Port scan detection
- ARP traffic analysis
- DNS query analysis
- Vulnerability identification and documentation

---

*This assessment was conducted in an isolated VirtualBox lab environment against a deliberately vulnerable machine for educational purposes only.*
