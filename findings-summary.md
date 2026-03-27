# 📋 Executive Summary — Network Analysis Findings

**Analyst:** Lovedip Singh  
**Date:** March 22, 2026  
**Target:** Metasploitable2 (192.168.56.102)  
**Classification:** Educational Lab — Not Production

---

### Key Findings at a Glance

| Metric | Value |
|---|---:|
| Total Packets Captured | 2,626 |
| Open Ports Discovered | 21 |
| Critical Findings | 3 |
| High Severity Findings | 7 |
| Overall Risk Rating | CRITICAL |

---

### Top 5 Most Important Findings

- **vsftpd 2.3.4** — Historically associated with **CVE-2011-2523** and remote backdoor risk
- **Port 1524 root shell service** — Intended unauthenticated root shell exposure in Metasploitable2
- **Legacy remote shell protocols** — `rexec`, `rlogin`, and `rsh` transmit credentials in plaintext
- **Network-accessible MySQL and PostgreSQL services** — Reachable from the scanning host
- **Telnet enabled (port 23)** — Credentials transmitted in plaintext over the network

---

### Tools & Techniques Demonstrated

- Network host discovery with Nmap
- Service version detection
- Live packet capture with Wireshark
- Protocol filtering and analysis
- Port-scan detection
- ARP traffic analysis
- DNS query analysis
- Vulnerability identification and documentation

---

*This assessment was conducted in an isolated VirtualBox lab environment against a deliberately vulnerable machine for educational purposes only.*
