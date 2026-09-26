# Windows Security Testing Guide: Telegram Desktop Session Hijacking

![License](https://img.shields.io/badge/License-MIT-blue)
![Platform](https://img.shields.io/badge/Platform-Windows-lightgrey)
![Focus](https://img.shields.io/badge/Focus-OSINT%20%2F%20Red%20Teaming-red)

## 📖 Introduction

This repository provides a practical, academic-led guide for security testing on Windows systems, focusing on **Application Data Storage Vulnerabilities**. While much of cybersecurity research focuses on server-side exploits and network protocols, this project highlights how local file structure and default user behaviors can lead to significant unauthorized access.

The core case study examines **Telegram Desktop**, demonstrating how the theft of local session data (the `tdata` folder) allows attackers to impersonate victims without needing passwords or bypassing Two-Factor Authentication (2FA).

---

## 🎯 Case Study: Telegram Desktop `tdata` Theft

### The Vulnerability

During installation, Telegram Desktop prompts users to select a directory for saving local data. Most users accept the default location (`%APPDATA%\Telegram Desktop\tdata`). This folder contains critical session keys and database files that identify the user to the Telegram servers.

**Key Insight:** This is treated as a **design characteristic/tradeoff** rather than a single patched CVE. Because the client stores long-lived session tokens, local file access to these files equates to full account control.

### Attack Vector

1. **Impersonation:** An attacker who obtains this folder can load it into their own Telegram Desktop instance and appear as the victim.
2. **Session Persistence:** Session tokens do not expire automatically. A stolen `tdata` folder remains usable indefinitely until the victim manually terminates all active sessions.
3. **No Password/2FA Required:** Since the attacker possesses the valid session key, they bypass login credentials entirely.

---

## 🗺️ Attack Surface Overview

This vulnerability can be reached through several broad categories of access, each with a different risk profile for the attacker and detectability for the defender:

- **Physical/local access** — anyone who can briefly access an unlocked or unsecured machine can reach the data folder directly.
- **Automated tooling** — commodity infostealer malware and scripts are known to search for and collect this folder as part of broader credential-harvesting routines (see Real-World Evidence below).
- **Detection evasion** — as with most local data-theft techniques, attacker tooling in the wild attempts to minimize forensic footprint and antivirus detection during collection and exfiltration.
- **Hardware-based delivery** — physical delivery mechanisms (e.g., programmable USB devices) are a known vector for automatically triggering local scripts once physical access is obtained.

This project does not document the operational specifics of these techniques. The **Real-World Evidence** section below cites documented cases of each category being used in practice, which is more useful for risk assessment than a reproducible how-to.

---

## 🔍 Real-World Evidence & Threat Intelligence

This vulnerability is not theoretical. It is actively exploited by malware families and supply-chain attacks.

### Notable Malware & Tools

| Tool | Description |
|---|---|
| **PupkinStealer** | A .NET infostealer specifically targeting `tdata`. It locates the folder, terminates Telegram to release file locks, and exfiltrates session data. |
| **RoboThiefClient** | Older tool that kills Telegram, compresses `tdata` into a password-protected zip, and restarts the app to avoid detection. |
| **TeleShadow** | A distributed tool marketed explicitly for stealing Telegram Desktop sessions by replacing the victim's local `tdata` folder. |

### Supply-Chain & Campaign Evidence

- **Imperva Threat Research:** Documented a malicious PyPI package designed to steal `tdata` folders, with stolen identities sold in underground markets.
- **PowerShell Campaigns:** A campaign used a Pastebin-hosted script disguised as a "Windows Telemetry Update" to locate `%APPDATA%\Telegram Desktop\tdata`, kill the process, and exfiltrate via the Telegram Bot API.
- **SANS ISC Honeypot Diary:** Recorded an attacker pivoting from cryptojacking to hunting for `tdata` folders and SMS logs (to reset passwords if sessions expire).

---

## 🛡️ Mitigations & Best Practices

How can users and analysts defend against this design tradeoff?

1. **Session Management:** Regularly check "Devices" in Telegram Settings and terminate unknown or old sessions.
2. **Data Location:** Move the `tdata` folder to a non-default location during installation (advanced users) or use portable versions with encrypted storage if available.
3. **SMS Reset Hardening:** Since SMS is often used for recovery, ensure your SIM card is secure against SIM-swapping attacks.
4. **Antivirus Monitoring:** Configure EDR/AV solutions to alert on the creation of password-protected ZIP files containing `tdata` folders or the termination of `Telegram.exe` by unknown processes.
5. **Full Disk Encryption:** BitLocker or VeraCrypt prevents attackers from accessing `tdata` via Live USB if they cannot unlock the drive.

---

## ⚖️ Ethical Considerations

This project is strictly for **educational use and research** within legal and ethical boundaries.

- **Authorization:** Always ensure you have explicit permission to test any system.
- **No CVE Assigned:** This is a local-access/malware problem, not a server-side vulnerability in Telegram's code.
- **Scope:** This guide documents publicly reported threat intelligence and defensive practices — it does not provide operational attack tooling or evasion instructions.

---

## 💛 Contributing

If you want to contribute additional case studies, defensive tooling, or mitigation strategies, feel free to:

1. Open an Issue for discussion.
2. Submit a Pull Request with your findings.

For questions regarding the research methodology, contact the project maintainers.

---

*Disclaimer: The author is not responsible for any misuse of the information provided in this repository.*
