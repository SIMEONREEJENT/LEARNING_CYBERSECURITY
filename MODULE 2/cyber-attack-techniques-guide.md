# Comprehensive Guide to Common Cyber Attack Techniques

This repository contains a highly detailed, operationally focused guide to common cyber attack techniques, designed for security analysts, incident responders, and students of cybersecurity. It breaks down five core threat vectors: **Reconnaissance**, **Social Engineering**, **Phishing**, **Credential Attacks**, and **Malware Delivery Mechanisms**, with full technical and tactical analysis aligned to real-world threats.

---

## Table of Contents
1. [Reconnaissance Activities (TA0043)](#1-reconnaissance-activities-ta0043)
2. [Social Engineering (The Human Perimeter)](#2-social-engineering-the-human-perimeter)
3. [Phishing (The Initial Access Vector)](#3-phishing-the-initial-access-vector)
4. [Credential Attacks (Perimeter Exploitation)](#4-credential-attacks-perimeter-exploitation)
5. [Malware Delivery Mechanisms](#5-malware-delivery-mechanisms)

---

## 1. Reconnaissance Activities (TA0043)

Reconnaissance represents the foundational, information-gathering phase of the attack lifecycle, mapping directly to tactic **TA0043** in the MITRE ATT&CK Enterprise Matrix [65, 218]. Adversaries systematically survey target infrastructure, personnel, and defensive posture before launching direct actions to identify high-value targets and vulnerable entry points.

```
                       ┌──────────────────────────────────────────┐
                       │   MITRE ATT&CK TA0043: Reconnaissance    │
                       └────────────────────┬─────────────────────┘
                                            │
                    ┌───────────────────────┴───────────────────────┐
                    ▼                                               ▼
     ┌──────────────────────────────┐                ┌──────────────────────────────┐
     │    Passive Reconnaissance    │                │    Active Reconnaissance     │
     │   (Non-Interactive / OSINT)  │                │   (Interactive & Direct)     │
     └──────────────┬───────────────┘                └──────────────┬───────────────┘
                    │                                               │
  - DNS Querying (dig, nslookup)                   - Port Scanning (Nmap)
  - Shodan & Censys Queries                        - Ping Sweeps & Traceroute
  - Social Media Scraping (LinkedIn)               - Active Directory Enumeration
```

### Passive Reconnaissance
Passive reconnaissance involves collecting data and technical specifications about the target **without directly interacting** with their networks, servers, or personnel.
*   **Operational Risk**: **Zero detection risk**. Because the adversary does not send network traffic to the victim’s perimeter, passive recon leaves no traces in the victim's intrusion detection logs or security information and event management (SIEM) systems [218].
*   **Tactics & Techniques**:
    *   **Open-Source Intelligence (OSINT)**: Scraping public data repositories, social media platforms (specifically LinkedIn), and corporate directories to map out employee names, roles, and internal organizational structures. This metadata is later used to craft targeted spear-phishing campaigns.
    *   **External Database Queries**: Using services like Shodan, Censys, passive DNS databases, and public WHOIS records to discover exposed assets, open ports, and registered IP blocks without initiating any direct scans.
    *   **Subdomain Enumeration**: Using open-source tools to pull subdomain listings from certificate transparency logs (e.g., search engines, public APIs) to map out the target's public-facing attack surface.

### Active Reconnaissance
Active reconnaissance requires the adversary to **directly interact** with the target’s network perimeter and hosts to harvest real-time technical configurations.
*   **Operational Risk**: **High detection risk**. Directly probing systems generates distinctive traffic patterns (e.g., rapid connection attempts across sequential ports) that trigger alerts in firewalls, Intrusion Detection/Prevention Systems (IDS/IPS), and perimeter logs [218].
*   **Tactics & Techniques**:
    *   **Port Scanning**: Using network discovery engines like `Nmap` to send packets (such as SYN, FIN, or NULL packets) to target ports, mapping which ports are open, closed, or filtered.
    *   **Banner Grabbing**: Establishing direct connections to active services (e.g., HTTP, SSH, FTP) to harvest raw responses containing the exact software name, version, and operating system details. This allows adversaries to pinpoint unpatched vulnerabilities.
    *   **Network Mapping**: Broadcasting ICMP probes (Ping Sweeps) and Traceroutes to visually chart the logical path and locate active routing nodes and live hosts.
    *   **Active Directory (AD) Enumeration**: Querying directory databases internally using administrative protocols (such as LDAP) or running visual paths via tools like `BloodHound` to locate privilege escalation lanes, domain admins, and high-value targets.

---

## 2. Social Engineering (The Human Perimeter)

Social engineering is the psychological manipulation of humans to trick them into performing unsafe actions, executing malicious payloads, or surrendering sensitive credentials. Rather than exploiting traditional code vulnerabilities, it targets cognitive biases, converting a well-intentioned employee into an unwitting entry point.

### The Human Exploitation Lifecycle
Adversaries execute social engineering campaigns using a structured, five-stage lifecycle:

```
 1. RECONNAISSANCE ──► 2. PRETEXT DEVELOPMENT ──► 3. ENGAGEMENT (THE HOOK)
   (Gathering OSINT      (Forging a believable      (Manipulating cognitive
    via LinkedIn)          scenario/identity)         biases of the target)
                                                               │
                                                               ▼
 5. EXECUTION & EXIT ◄─── 4. EXPLOITATION / ACTION ────────────┘
   (Covering footprints   (Victim executes command
    and removing access)    or transfers credentials)
```

1.  **Reconnaissance**: Scraping public profiles to identify target victims, their job roles, and organizational hierarchies.
2.  **Pretext Development**: Fabricating an intricate, believable scenario (a "pretext") and a spoofed identity (such as an IT technician, trusted supplier, or HR lead).
3.  **Engagement**: Contacting the target (via email, SMS, phone, or messaging platforms) and establishing an emotional or psychological "hook".
4.  **Exploitation**: Tricking the victim into performing the desired action—such as resetting their credentials, downloading a file, or initiating a financial transfer.
5.  **Execution & Cover**: Deploying the technical payload or exfiltrating the data while maintaining the illusion of legitimacy, then quietly exiting to prevent early detection.

### Core Psychological Triggers
*   **Authority**: Adversaries impersonate high-level figures (e.g., C-suite executives, legal advisors, or corporate IT engineers) to bypass typical verification procedures. Most employees are conditioned to respond rapidly to commands from perceived leadership.
*   **Urgency & Scarcity**: Creating artificial constraints (e.g., "MFA password expiration in 1 hour" or "pending contract cancellation") to induce panic, forcing the target to skip standard verification channels.
*   **Liking & Connection**: Establishing a sense of familiarity or rapport. Attackers may construct complex recuiting pretexts over weeks on professional networks before dropping a malicious payload.
*   **Reciprocity**: Offering a small favor (such as a complimentary IT health check or a free gift card) to psychologically obligate the victim to perform a requested action in return (e.g., answering sensitive internal system questions).

### Advanced Social Engineering Paradigms
*   **Business Email Compromise (BEC)**: A highly targeted fraud campaign where attackers compromise or spoof the email accounts of senior executives or trusted external vendors [3, 8]. The attacker uses their assumed authority to guide financial teams to route payments to fraudulent bank accounts.
*   **MFA Fatigue (Push Bombing)**: Bombarding a target's mobile device with dozens of continuous multi-factor authentication (MFA) push prompts [3, 8]. Simultaneously, attackers call or message the victim as "IT Helpdesk," instructing them to accept the prompt to stop the annoying notifications.
*   **Deepfakes**: Utilizing real-time generative AI models to clone voices and video streams of high-profile company leadership. This has been used to trick financial departments into executing multimillion-dollar fund transfers during live virtual meetings.

---

## 3. Phishing (The Initial Access Vector)

Phishing represents the tactical application of social engineering delivered through electronic communication channels. It remains the most common method of gaining initial entry into secure networks [238, 240].

```
                       ┌──────────────────────────────────────────┐
                       │          Phishing Methodologies          │
                       └────────────────────┬─────────────────────┘
                                            │
         ┌───────────────────┬──────────────┴──────┬───────────────────┐
         ▼                   ▼                     ▼                   ▼
  ┌──────────────┐    ┌──────────────┐      ┌──────────────┐    ┌──────────────┐
  │Bulk Phishing │    │Spear-Phishing│      │   Whaling    │    │   Quishing   │
  │(Mass volume, │    │(Personalized,│      │ (Targeting   │    │(Malicious QR │
  │ untargeted)  │    │ OSINT-driven)│      │  C-Suite)    │    │ codes in PDF)│
  └──────────────┘    └──────────────┘      └──────────────┘    └──────────────┘
```

### Key Phishing Classifications
*   **Bulk Phishing**: Automated, low-cost, mass-volume email distributions sent to millions. These rely purely on broad statistics rather than personalization.
*   **Spear-Phishing**: A highly personalized, custom-tailored campaign targeted at a specific individual, department, or organization [89, 90]. It leverages prior reconnaissance (such as vendor relationships or active project names) to construct highly convincing lures [276, 278].
*   **Whaling**: A high-priority sub-discipline of spear-phishing targeting executive leadership (C-suite, board members) to steal highly privileged credentials or initiate wire transfers.
*   **Quishing (QR-Code Phishing)**: Embedding malicious links inside QR graphics within PDF attachments or email bodies [238, 240]. Gateway security filters struggle to parse and scan links rendered inside image layers, allowing these messages to bypass traditional inbound perimeters.

### Core Technical Indicators of Phishing Campaigns
*   **Typosquatting & Domain Spoofing**: Registering domain names that look nearly identical to trusted entities (e.g., substituting characters like `mac-safer.com` or `@microsoft-billing.co`) to deceive targets [238, 240].
*   **Hyperlink Mismatching**: Writing anchor text that displays a trusted URL (e.g., `https://trusted-bank.com`) while the underlying HTML `href` attribute directs the browser to a completely different, attacker-controlled harvesting site [238, 240].
*   **HTML Smuggling**: A sophisticated fileless delivery technique that embeds base64-encoded payload variables within HTML attachments or web pages [238, 240]. When the user opens the attachment, the browser decodes and reconstructs the payload locally. Because the actual malicious file is compiled *after* passing through email gateway filters, standard perimeter scanners see only benign HTML and JavaScript [238, 240].

---

## 4. Credential Attacks (Perimeter Exploitation)

Once reconnaissance has mapped the target’s external perimeters and harvested username configurations, adversaries attempt to bypass authentication interfaces.

| Attack Vector | Input Requirements | Technical Execution Strategy | Lockout Risk | Evasion Capabilities |
| :--- | :--- | :--- | :--- | :--- |
| **Traditional Brute Force** | 1 Username + 1 Dictionary of millions of passwords [25, 26]. | Sequentially testing thousands of password guesses against a single target account [25, 26]. | **High**: Rapidly triggers account lockout thresholds or blocks the source IP. | None. This attack is noisy, predictable, and easily mitigated. |
| **Password Spraying (OAT-007)** | List of valid enterprise usernames + a single common password [61, 62]. | Testing a single highly probable password (e.g., *Summer2024!*) horizontally across thousands of accounts [61, 62]. | **Low**: Evades lockouts by making only one login attempt per account within a given time window [61, 62]. | Implements random time delays and rotates source IPs across commercial proxies. |
| **Credential Stuffing (OAT-008)** | Massive databases of username-password pairs from historical third-party breaches [25, 26]. | Automated botnets attempt login credentials against unrelated web portals, exploiting user password reuse [25, 26, 29, 30]. | **Low**: Attempts only one or two matching sets per account before moving on [25, 26]. | Bots rotate user-agent strings and use residential IP networks to mimic organic user traffic. |

### NIST Password Guidelines (SP 800-63B)
Modern security policies align with **NIST SP 800-63B (Revision 4)**, which prioritizes empirical, evidence-based security over outdated checklist policies [139, 145].

```
           LEGACY CHECKLIST POLICIES                    NIST SP 800-63B ALIGNED
        ┌──────────────────────────────┐            ┌──────────────────────────────┐
        │ ❌ Complexity Rules          │            │ ✅ Length Over Complexity    │
        │ ❌ Mandatory 90-Day Rotation │            │ ✅ Check Against Breach Lists│
        │ ❌ Account Lockouts          │            │ ✅ Smart Rate Limiting       │
        │ ❌ Security Questions / Hints │            │ ✅ Phishing-Resistant MFA    │
        └──────────────────────────────┘            └──────────────────────────────┘
```

*   **Length Over Complexity**: NIST SP 800-63B requires systems to support passwords of up to at least 64 characters and explicitly prohibits legacy complexity rules (e.g., upper/lowercase, numbers, special characters) [178, 186]. Composition rules force users into highly predictable substitution workarounds (e.g., *Password1!*), which lowers overall entropy [150, 165].
*   **Banning Mandatory Periodic Rotation**: Forcing users to change passwords every 30, 60, or 90 days results in predictable, incremental changes (e.g., *Spring2024!* to *Summer2024!*) [178, 186]. Rotation should only be forced upon verified compromise.
*   **Mandatory Breach List Checking**: Verifiers must dynamically screen newly created passwords against a compiled database of compromised, leaked, or highly common keyboard patterns (e.g., *qwerty*) during registration and asynchronously [178, 186].
*   **Deprecating Hints and Knowledge-Based Authentication (KBA)**: Ban legacy hints and KBA security questions (e.g., "What was your first school?") [178, 186]. This information is easily harvested via passive reconnaissance, OSINT, and social media scraping.
*   **MFA Strength Tiers**: Enforce multi-factor authentication, prioritizing phishing-resistant authenticators (WebAuthn/FIDO security keys) over less secure tiers like SMS-based OTPs, which are vulnerable to SIM swapping and interception [176, 177].

---

## 5. Malware Delivery Mechanisms

Adversaries rely on specialized, highly evasion-oriented delivery mechanisms to transport code into memory while bypassing endpoint controls (like EDRs).

### Watering Hole Attacks
Instead of targeting high-security networks directly, an attacker compromises a trusted, industry-specific third-party website frequently visited by the target group (e.g., government regulatory portals, financial compliance blogs, or niche partner forums) [95, 96].
*   **Mechanics**: The attacker infects the trusted site with a malicious redirect or exploit script [316, 317]. When employees from the targeted organization browse the compromised page, their systems are silently scanned and exploited.
*   **Advantage**: Firewalls and gateway email filters offer zero protection, as the communication is initiated by the internal user navigating voluntarily to an established, trusted external domain.

### Drive-By Downloads
Drive-by downloads occur when malicious files are automatically downloaded and executed on a victim's device without their active consent, file execution, or knowledge [319, 322].
*   **Mechanics**: The attacker injects exploit scripts into compromised web advertising networks (malvertising) or vulnerable web pages [319, 322]. When the user simply browses the page, the script silently exploits unpatched client-side browser, plug-in, or operating system vulnerabilities to execute shellcode.

### ClickFix / InstallFix Techniques
ClickFix is a modern interactive social engineering vector that guides targets to execute commands themselves, exploiting human troubleshooting habits while completely bypassing standard browser-level download scanning tools (such as Google Safe Browsing) [89, 92, 257, 270].

```
┌────────────────────────────────────────────────────────────────────────┐
│                        CHROME DOCUMENT ERROR                           │
│  An error occurred while loading this PDF. Click the button below      │
│  to copy the fixing command, then press Win+R and paste to fix.       │
│                                                                        │
│                      [ Copy PowerFix Command ]                         │
└────────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼ (Victim Copies base64 Script)
┌────────────────────────────────────────────────────────────────────────┐
│  Victim executes command in Windows Run box:                           │
│  powershell.exe -nop -w hidden -enc ZnVuY3Rpb24gR2V0LUNv...             │
└────────────────────────────────────────────────────────────────────────┘
```

1.  **The Lure**: Compromised websites or phishing pages display a realistic, fake error overlay—such as a Google Meet connection failure, missing document certificate, web browser crash, or fake CAPTCHA check [59, 74, 256, 269].
2.  **The Interaction**: The site guides the user through clear instructions: click a button to "copy the automatic fix code", press `Win + R` to open the Windows *Run* prompt, and paste the code directly into the dialog box or PowerShell terminal [89, 92].
3.  **The Exploit**: Because the victim executes the code directly through local administrative consoles, standard security boundaries are bypassed, and obfuscated shellcode executes instantly [89, 92].

### Fileless Payloads (LOLBins & Injection)
To bypass traditional security tools that analyze executable files saved to disk, modern loaders (e.g., *DeepLoad*, *SmartRAT*) transition to fileless execution [21, 27, 248, 261].
*   **Living-off-the-Land Binaries (LOLBins)**: Adversaries execute scripts using trusted, pre-installed operating system administration utilities (such as `mshta.exe`, `powershell.exe`, or WMI scripting engines) [107, 108, 248, 261].
*   **Memory Injection**: Payloads inject code directly into the active memory space of legitimate running processes (like the Windows lock screen host, `LockAppHost.exe` or `msedge.exe`), avoiding leaving physical, scanning-vulnerable files on the physical storage drive [20, 26, 248, 261].
*   **PowerShell Variable Obfuscation**: In PowerShell, anything inside `${...}` is treated as a variable name [107, 108]. Loaders like *DeepLoad* use this feature to write thousands of meaningless variable assignments containing legitimate domains (e.g., `${windowsupdate.microsoft.com}`) as decoys, confusing automated sandboxes and static analysis scanners [107, 109].

---

## Real-World Campaigns & Indicators of Compromise (IoCs)

To aid in defensive hunt teams, the following indicators from recent campaigns (observed in 2024-2025) should be monitored:

### 1. ClickFix / SmartRAT Infrastructure
Discovered by ThreatLabz delivering *SmartRAT* payloads via fake Google Meet/Microsoft error templates [20, 21, 27, 28]:
*   **Stealth PowerShell Dropper**: Fetches `st.txt` from direct IP addresses, downloads the payload, and saves it as a decoy file (`msedge.txt`) in `C:\Users\Public\Documents\` [20, 26].
*   **Persistence Vectors**: Copies itself to `%APPDATA%\Microsoft\Diagnosis\ETW\msedgeupdate.txt` and schedules a logon task named `MicrosoftEdgeUpdateCore` [23, 29].
*   **Debug Logs**: Client debug logs are written to `C:\ProgramData\Microsoft\Diagnosis\ETW\client_debug.log` or `%APPDATA%\Microsoft\Diagnosis\ETW\client_debug.log` [22, 28].

### 2. Threat Actor Group TTPs (Scattered Spider / TA571)
*   **Scattered Spider**: Leverages active SSH tunneling, abuse of remote management monitoring (RMM) tools, and data exfiltration to public platforms (MEGA, Google Drive) using tools like `Rclone` [98, 99].
*   **TA571**: Relies heavily on deep-nested zip archives containing malicious shortcut `.lnk` files [238, 240]. In May 2024, they were observed using PowerShell clipboard payloads to deploy *NetSupport RAT* and *DarkGate* malware [67, 68].

---

*Operational guide compiled using threat intelligence and regulatory standards strictly grounded in project reference material.*
