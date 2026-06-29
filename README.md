# Analyse-Dynamique-du-Ransomware-WannaCry

<p align="center">
  <img src="https://img.shields.io/badge/Malware-WannaCry-red?style=for-the-badge&logo=virustotal" alt="Malware"/>
  <img src="https://img.shields.io/badge/Analysis-Dynamic-blue?style=for-the-badge&logo=datadog" alt="Analysis"/>
  <img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge" alt="Status"/>
  <img src="https://img.shields.io/badge/Tools-ProcMon%20%7C%20Wireshark%20%7C%20REMnux-orange?style=for-the-badge&logo=wireshark" alt="Tools"/>
</p>

# 🧬 WannaCry Ransomware: In-Depth Dynamic Analysis

> **A comprehensive behavioral analysis of the WannaCry (WanaCrypt0r 2.0) ransomware, executed in a controlled, isolated sandbox environment.**

This repository contains the full forensic report and evidence artifacts from a dynamic malware analysis engagement. The objective was to map the complete infection chain—from initial delivery to the final ransom note—using industry-standard tools, without relying on static signature-based detection.

---

## 🎯 Executive Summary

WannaCry is a notorious ransomware worm that caused a global cyberattack in May 2017. This analysis successfully observed and documented its core behaviors:

- **Initial Access**: Delivery via a self-extracting RAR archive dropped into the user's `%TEMP%` directory.
- **Evasion**: The main process renamed itself to an MD5 hash and falsified its internal description to "DiskPart" to masquerade as a legitimate Windows utility.
- **Execution**: Launched **17+ parallel child processes** (masquerading as "Load PerfMon Counters") to encrypt files rapidly using multi-threading.
- **Impact**: Encrypted thousands of files, appending the `.WNCRY` extension, and replaced the desktop wallpaper with the `@WanaDecryptor@.bmp` ransom note.
- **C2 Communication**: Attempted to establish anonymous communication with its Command & Control (C2) infrastructure via the **Tor network** (TCP ports 9001, 443, and 80), though blocked by the isolated environment.

---

## 🖥️ Lab Environment & Toolset

The analysis was performed using an isolated host-only network to prevent accidental internet egress.

| Component | Role | IP Address | Tools Used |
| :--- | :--- | :--- | :--- |
| **Windows 10 (Victim)** | Target machine executing the malware | `192.168.100.10` | ProcMon, Regshot, Process Explorer |
| **REMnux (Gateway/Analyst)** | Simulated internet services & network capture | `192.168.100.1` | INetSim, Wireshark, tcpdump |

**Methodology**: Live behavior was captured in real-time using ProcMon and Wireshark, while Regshot provided a differential analysis of registry and filesystem changes (Before vs. After infection).

---

## 🚨 Key Indicators of Compromise (IOCs)

### 🖥️ Registry Modifications
| Path / Key | Action | Severity | Interpretation |
| :--- | :--- | :--- | :--- |
| `HKCU\...\BackgroundHistoryPath0` | Value Changed to `@WanaDecryptor@.bmp` | **CRITICAL** | Ransom note wallpaper applied; marks the end of the encryption cycle. |
| `HKLM\...\bam\State\UserSettings` | `RegSetValue` (REG_BINARY) | **HIGH** | Direct manipulation of Background Activity Moderator (BAM) for anti-forensics. |
| `HKCU\...\UserAssist` | Counters (ROT13 encoded) | **INFO** | Confirms execution of analysis tools (ProcExp, Regshot); native Windows behavior. |

### 📂 Filesystem Artifacts
| Path / Indicator | Action | Severity | Interpretation |
| :--- | :--- | :--- | :--- |
| `%TEMP%\Rar$...\@WanaDecryptor@.exe` | Created | **CRITICAL** | Self-extracting RAR delivery vector. |
| `*.WNCRY` | Created / Renamed | **CRITICAL** | Final encrypted file extension. |
| `*.WNCRYT` | Created (Temporary) | **CRITICAL** | Intermediate encryption state before final renaming. |
| `C:\Users\hp\Desktop\@WanaDecryptor@.bmp` | Created | **CRITICAL** | Ransom note image dropped on the Desktop. |
| `~SDxxxx.tmp` | Created | **HIGH** | Temporary buffer files used during encryption processing. |
| `b.wnry` / `00000000.res` | Extracted | **HIGH** | Embedded wallpaper bitmap and multilingual resources. |

### 🌐 Network Signatures
| Indicator | Observation | Severity | Interpretation |
| :--- | :--- | :--- | :--- |
| **Tor C2 Traffic** | TCP SYN attempts to multiple IPs (ports 9001, 443, 80). | **CRITICAL** | Hardcoded Tor node list attempts to establish an anonymous C2 channel. |
| **Kill-Switch** | DNS query for `iuqerfsodp9ifj...com` | **PENDING** | Domain check; resolves successfully = stops execution. |
| **SMB / EternalBlue** | Port 445 traffic. | **PENDING** | Lateral movement module (not triggered in this isolated single-host lab). |

---

## 🗺️ MITRE ATT&CK Framework Mapping

| Tactic | Technique (ID) | Observation |
| :--- | :--- | :--- |
| **Execution** | T1204 - User Execution | User manually extracted the self-extracting RAR archive. |
| **Execution** | T1106 - Native API | Spawned 17+ child processes for parallel encryption. |
| **Defense Evasion** | T1036 - Masquerading | Renamed file to MD5 hash and set description to "DiskPart" / "Load PerfMon". |
| **Defense Evasion** | T1070 - Indicator Removal | Directly modified the BAM registry key to obscure execution timestamps. |
| **Discovery** | T1083 - File & Directory Discovery | Recursively enumerated all user and system directories (OneDrive, Public, WER). |
| **C2** | T1090.003 - Multi-hop Proxy | Attempted communication via hardcoded Tor relay IPs. |
| **C2** | T1571 - Non-Standard Port | Used Tor ORPort (9001) and ports 443/80 to blend in. |
| **Impact** | T1486 - Data Encrypted for Impact | Encrypted files and renamed them with the `.WNCRY` extension. |
| **Impact** | T1491 - Defacement (Internal) | Replaced the desktop wallpaper with the ransom note image. |

---

## 🖼️ Visual Evidence & Artifacts

*Add your screenshots here by replacing the placeholder names with your actual image files (e.g., `images/process_masquerade.png`).*

**1. Process Masquerading (DiskPart)**
The main process (PID 6508) disguised as a Windows disk management tool.
![Main process masquerading as DiskPart](images/procmon_process_masquerade.png)

**2. Parallel Encryption (17+ Instances)**
Multiple child instances of `@WanaDecryptor@.exe` running simultaneously to maximize encryption speed.
![Parallel encryption instances](images/procexp_parallel_processes.png)

**3. Encrypted Files (.WNCRY)**
Files showing the intermediate `.WNCRYT` and final `.WNCRY` extensions.
![Encrypted files with WNCRY extension](images/explorer_wncry_files.png)

**4. Ransom Note Wallpaper**
The desktop background forcibly changed to the WannaCry ransom demand.
![Desktop wallpaper changed to ransom note](images/desktop_ransom_wallpaper.png)

**5. Tor C2 Traffic (Wireshark)**
SYN packets attempting to reach hardcoded Tor nodes (port 9001) with no response.
![Wireshark showing Tor C2 attempts](images/wireshark_tor_traffic.png)

---

## ⚠️ Analysis Limitations & Open Points

The following elements were not fully concluded during this session and require further investigation:

- **csrss.exe Interaction**: PID 716 was observed interacting with the malicious binary. Verification of its legitimate path and Microsoft signature is pending.
- **SMB/EternalBlue Propagation**: No secondary vulnerable host (Windows 7) was present on the isolated network, preventing observation of the worm's lateral movement.
- **Kill-Switch DNS Query**: A specific search for `iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com` in the PCAP is pending final confirmation.
- **Persistence Mechanism**: The `Run` registry key creation (typically `wbqguynnopc626`) was not confirmed in this specific run.

---

## 📄 Full Report

For the complete technical breakdown, including the detailed chronological infection timeline and complete ProcMon/Regshot logs, please refer to the attached document:

- **[Download the Full Analysis Report (DOCX)](Rapport_Final_WannaCry_Analyse_Dynamique.docx)**

---

## 🔒 Recommendations

Based on the analysis, the following defensive measures are recommended:

1.  **Deploy EDR (Endpoint Detection & Response)**: Focus on behavioral detections for mass-file encryption (high I/O write rates), rather than static filenames (easily bypassed).
2.  **Patch Management**: Ensure MS17-010 (EternalBlue) is applied to all legacy Windows systems. Consider disabling SMBv1 entirely.
3.  **Network Controls**: Monitor egress traffic for Tor-specific ports (9001, 9030) and known Tor relay IP ranges.
4.  **Backup Strategy**: Maintain offline, immutable backups to ensure recoverability in the event of a successful ransomware encryption.
5.  **User Awareness**: Train users to recognize and avoid executing unsolicited self-extracting archives received via email.

---

## 👩‍💻 Author

**Ouchahed Salma**  
*Cybersecurity Analyst / Reverse Engineering Enthusiast*  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=flat&logo=linkedin)](https://linkedin.com/in/your-profile)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?style=flat&logo=github)](https://github.com/your-profile)

---

## 📜 Disclaimer

This project was conducted strictly for **educational and defensive research purposes** in a controlled, air-gapped laboratory environment. The author does not condone the use of malware for any illegal or malicious activities. The sample was obtained from a public security research repository (GitHub) and handled in full compliance with best-practice isolation protocols.
