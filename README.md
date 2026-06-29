# 🔒 WannaCry Ransomware — Dynamic Behavioral Analysis

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Profile-blue)](YOUR_LINKEDIN_LINK)

Complete dynamic analysis of the WannaCry ransomware (WCry / WanaCrypt0r 2.0), conducted in an isolated lab environment, combined with complementary static analysis performed as a team project.

> Academic project for defensive and educational purposes only. No malicious binary is included in this repository.
> 📄 **Note:** the full detailed report is written in French (see link below). This README provides a complete summary in English.

---

## 🎯 Skills Demonstrated

- Behavioral malware analysis (ProcMon, Process Explorer)
- Registry/filesystem before-after comparison (Regshot)
- Network traffic capture and analysis (Wireshark, tcpdump)
- Controlled network service simulation (INetSim)
- Noise/signal filtering methodology across millions of system events
- MITRE ATT&CK mapping
- Network configuration troubleshooting (DNS binding, VM routing)
- Technical investigation reporting and oral presentation

---

## 🧰 Lab Environment & Tools

| Category | Tools |
|---|---|
| Behavioral analysis | Process Monitor, Process Explorer |
| System analysis | Regshot |
| Network analysis | Wireshark, tcpdump, INetSim |
| Infrastructure | Windows 10 VM (victim), Windows 7 VM (secondary target), REMnux VM (network analysis) |

---

## 🚨 Key Findings

### Evasion techniques
- Main process renamed as an MD5 hash
- Internal metadata spoofing: "DiskPart" description (parent process) and "Load PerfMon Counters" (child instances), fake "Microsoft Corporation" company attribute

![Process Tree](screenshots/procmon/05-process-tree.png)

### Anti-forensics
- Direct manipulation of the **BAM** (Background Activity Moderator) registry key: reading then overwriting the execution timestamp value

![BAM anti-forensics](screenshots/procmon/02-bam-anti-forensique.png)

### Encryption mechanism
- Two-stage pipeline: `.WNCRYT` (in progress) → `.WNCRY` (finalized)
- No selectivity: user files and internal application files (OneDrive, system caches) treated identically

![OneDrive encryption](screenshots/procmon/04-energie-onedrive-wncry.png)

### Network — C2 communication and kill switch
- C2 connections confirmed toward **Tor** nodes (port 9001), hardcoded IPs
- **Absence of a functional kill switch**
