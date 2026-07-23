# The CIA Triad: Foundational Information Security Model

> The CIA Triad—standing for Confidentiality, Integrity, and Availability—is the foundational information security model that guides organizations in protecting their data, analyzing risks, and maintaining reliable access to their systems.

Here is a breakdown of the CIA Triad, real-world examples, and how security objectives are defined:

## 1. Breakdown of the CIA Triad

### a. Confidentiality
Confidentiality ensures that sensitive information is not made available or disclosed to unauthorized individuals, entities, or processes. It serves to protect data from prying eyes and guarantees that only those with the proper authorization can view or read the information.
*   **Key Protections:** Confidentiality is typically enforced through encryption (scrambling data so it cannot be read without a key), access controls (such as multi-factor authentication and the principle of least privilege), and physical controls (like security cameras and locked doors)

### b. Integrity
Integrity refers to maintaining and assuring the accuracy, completeness, and consistency of data over its entire lifecycle. It guarantees that information is reliable and has not been corrupted, tampered with, or modified in an unauthorized manner.
*   **Types of Integrity:** Integrity encompasses physical integrity (safeguarding data from hardware failures or environmental disasters) and logical integrity (ensuring data remains accurate and consistent across relational databases via specific constraints).
*   **Key Protections:** Organizations preserve integrity using cryptographic hashes or checksums (to verify data hasn't changed), transaction logs and audit trails (to track changes), and regular data backups

### c. Availability
Availability guarantees that computing systems, data, and communication channels are accessible to authorized users whenever they are needed. It is often considered the most operationally visible property of the triad, because when availability fails, users immediately notice the disruption.
*   **Metrics:** Availability is tracked using specific operational metrics such as Mean Time Between Failures (MTBF), Mean Time To Recover (MTTR), Recovery Time Objective (RTO), and Recovery Point Objective (RPO).
*   **Key Protections:** Ensuring availability involves deploying redundant systems (like backup power and clustered servers), load balancers, and comprehensive incident response and disaster recovery plans.

---

## 2. Real-world examples

*   🔴 **Confidentiality Breaches:** Unauthorized access occurs through data breaches, insider threats, and social engineering (like phishing). A massive real-world example occurred in 2025 when a business associate, Conduent Business Services, experienced a breach that exposed the protected health information—including Social Security numbers and claims data—of over 62.2 million individuals. Simple examples also include laptop theft or sending sensitive emails to the wrong person.
*   🟠 **Integrity Breaches:** Integrity can be broken by data corruption, tampering, or malicious software modifying data. A prominent example is the 2020 SolarWinds supply chain attack, where attackers injected a SUNBURST backdoor into the Orion software's source code, altering the system in a way that went completely undetected during the software development lifecycle.
*   🟡 **Availability Breakdowns:** Threats to availability include hardware failures, power outages, and Denial-of-Service (DDoS) attacks. In July 2024, a faulty update to the CrowdStrike Falcon sensor caused a massive global outage, triggering a "Blue Screen of Death" for millions of Microsoft Windows devices and disrupting highly critical sectors like aviation, healthcare, and banking. Ransomware attacks, such as the 2024 attack on Change Healthcare that shut down IT systems for hospitals and pharmacies nationwide, also severely impact availability.

---

## 3. Security objectives define

While the CIA Triad focuses strictly on confidentiality, integrity, and availability, modern frameworks and authors have proposed additional overarching security objectives to account for the complexity of today's digital landscape. These include properties like:

*   ✅ **Authenticity:** Validating that a transaction or communication genuinely comes from the claimed source.
*   ✅ **Possession or Control:** Addressing the physical or logical control of an asset (e.g., a stolen encrypted laptop means a loss of possession, even if confidentiality remains intact).
*   ✅ **Utility:** Ensuring information remains in a usable format.
*   ✅ **Non-repudiation:** Ensuring a party cannot deny the authenticity of their signature or sending a message.

---

## 4. Threats

> Just as the CIA Triad serves as the foundation and goal for defenders to secure a system, the **DAD Triad** represents the attacker's goal to compromise it. The DAD Triad stands for Disclosure, Alteration, and Denial, and each component represents a specific method used by hackers to break one of the pillars of the CIA Triad:

### a. Disclosure (Breaks Confidentiality)
Disclosure occurs when an attacker accesses or leaks sensitive information without permission. By exposing this data, the attacker defeats the defender's goal of keeping private information out of the hands of unauthorized users.
*   *Real-world example:* A hacker stealing credit card details from an e-commerce website and leaking them online.

### b. Alteration (Breaks Integrity)
Alteration happens when an attacker changes, modifies, manipulates, or corrupts data without authorization. This destroys the trustworthiness and accuracy of the data, completely undermining the integrity of the system.
*   *Real-world example:* An attacker gaining access to a hospital system and secretly changing a patient's medical prescription.

### c. Denial (Breaks Availability)
Denial occurs when an attacker makes a system or service difficult or impossible to access for legitimate users. Instead of a system being available 24/7 as intended by defenders, the attacker intentionally forces a disruption.
*   *Real-world example:* A hacker launching a Denial-of-Service (DDoS) attack to bring down an online banking service, preventing customers from accessing their own money.

---

## 5. Detailed Breakdown of Threats

### 1. Threats to Confidentiality (Disclosure)
Threats to confidentiality involve unauthorized individuals or systems gaining access to sensitive data. Common threats include:
*   **Data breaches:** Attackers gaining unauthorized access to databases that store sensitive information, such as credit card numbers or Personally Identifiable Information (PII).
*   **Insider threats:** Employees who intentionally or carelessly access and leak sensitive organizational information.
*   **Social engineering and Phishing:** Deceptive tactics used by attackers to trick users into revealing sensitive information, such as login credentials.
*   **Eavesdropping and Interception:** Attackers eavesdropping on network communications to intercept passwords, files, or messages in transit.
*   **Physical theft and human error:** The physical theft of devices like laptops, password theft, or simply sending a sensitive email or text message to the wrong person.
*   **Brute force attacks:** Automated attempts to guess passwords and gain unauthorized access to secure systems.

### 2. Threats to Integrity (Alteration)
Threats to integrity occur when data is tampered with, corrupted, or modified in an unauthorized manner, destroying its accuracy and trustworthiness. Common threats include:
*   **Tampering and unauthorized modification:** An unauthorized user accessing a system to physically or logically alter, delete, or add false information. This also includes modifying configuration files or editing and removing system logs to cover their tracks.
*   **Malicious software (Malware):** Attackers injecting viruses or malware that secretly change data without the user's consent. A prominent example is supply chain attacks, where attackers inject malicious code into legitimate software updates (e.g., the SolarWinds SUNBURST backdoor).
*   **Man-in-the-Middle (MitM) attacks:** Attackers intercepting data while it is in transit to maliciously alter it, such as changing the destination bank account or amount in a payment request, or modifying an email before it reaches the recipient.
*   **Data corruption:** Non-malicious threats such as software bugs or hardware malfunctions that cause data to be transmitted or stored incorrectly, resulting in errors or inconsistencies.

### 3. Threats to Availability (Denial)
Threats to availability prevent authorized users from accessing the systems, networks, or data they need, when they need them. Common threats include:
*   **Denial-of-Service (DoS) and Distributed Denial-of-Service (DDoS) attacks:** Malicious attacks where hackers flood a target system, website, or network with an overwhelming amount of traffic, making it incredibly slow or completely crashing it for legitimate users.
*   **Ransomware:** Malicious software that encrypts an organization's production data or locks down critical systems, making them entirely unusable until a ransom is paid.
*   **Faulty software deployments and misconfigurations:** Aggressive vulnerability remediation (like patching without adequate testing), misconfigured firewall rules blocking legitimate traffic, or flawed software updates. A massive real-world example of this was the July 2024 CrowdStrike outage, where a faulty update to the Falcon sensor triggered a global "Blue Screen of Death" for millions of Windows devices.
*   **Hardware and software failures:** Physical device failures, such as crashing servers, malfunctioning network equipment, or failing storage disks.
*   **Natural disasters and environmental factors:** Events completely outside of human control, such as fires, floods, earthquakes, or severe power outages that physically destroy or disrupt data centers and critical infrastructure.
*   **Network congestion or outages:** General connectivity disruptions that prevent users from accessing data or systems hosted over the internet

---

## 6. Prevention

### 1. Preventing Threats to Confidentiality (Defending against Disclosure)
To prevent unauthorized individuals or systems from accessing or leaking sensitive data, organizations rely on:
*   🛡️ **Encryption:** Converting data into an unreadable format using cryptographic algorithms. This protects data "at rest" (stored on drives) and "in transit" (moving across networks) so that even if it is intercepted, it cannot be read without the decryption key.
*   🛡️ **Access Controls and IAM:** Implementing Identity and Access Management (IAM) and the Principle of Least Privilege, which ensures users are only granted the minimum access rights necessary to perform their jobs.
*   🛡️ **Strong Authentication:** Using Multi-Factor Authentication (MFA) or biometric authentication to verify that a user is genuinely who they claim to be before granting access, effectively neutralizing stolen passwords.
*   🛡️ **Data Masking and Tokenization:** Hiding parts of sensitive data (such as only showing the last four digits of a credit card number) or replacing sensitive data with non-sensitive placeholders (tokens) to protect privacy during testing or daily operations.
*   🛡️ **Data Loss Prevention (DLP):** Implementing software that monitors network traffic and prevents sensitive information from leaving the organization. DLP can even be used to block physical USB ports on workstations to prevent data theft.
*   🛡️ **Security Awareness Training:** Educating employees on how to recognize and report social engineering attacks and phishing campaigns, reducing the risk of human error.

### 2. Preventing Threats to Integrity (Defending against Alteration)
To ensure that data remains accurate, complete, and free from malicious tampering or accidental corruption, organizations use:
*   🔒 **Hashing and Checksums:** Generating a unique mathematical string (a "hash" or "fingerprint") for a file. If even a single character in the file is altered by an attacker or corrupted, the hash will change entirely, immediately alerting defenders to the tampering.
*   🔒 **File Integrity Monitoring (FIM):** Deploying software that continuously monitors critical system files and configuration logs. If a file is modified, added, or deleted without authorization, the FIM system raises an immediate alert.
*   🔒 **Digital Signatures and Certificates:** Using cryptographic signatures to guarantee that files, emails, or transactions genuinely came from the claimed source and have not been altered in transit.
*   🔒 **Audit Logs and Version Control:** Keeping a detailed, immutable record of who accessed or modified data. This allows administrators to track unauthorized changes and revert data back to a safe, original state.
*   🔒 **Change Management Processes:** Enforcing strict, authorized procedures for updating systems. This ensures that no configurations or data can be altered without proper testing and approval.

### 3. Preventing Threats to Availability (Defending against Denial)
To guarantee that authorized users can consistently access computing systems and avoid catastrophic downtime, organizations implement:
*   ⚡ **Redundancy and High Availability:** Deploying duplicate systems, such as clustered servers and redundant network paths. If one piece of hardware fails or is attacked, the backup system seamlessly takes over.
*   ⚡ **Load Balancing:** Distributing incoming network traffic evenly across multiple servers. This prevents any single system from becoming overwhelmed by legitimate traffic or sudden spikes in demand.
*   ⚡ **DDoS Protection:** Utilizing managed services (like AWS Shield) and firewalls to detect and block volumetric traffic floods before they can crash a website or application.
*   ⚡ **Backups and Disaster Recovery:** Maintaining regular, encrypted backups of critical data (which helps recover from ransomware attacks that lock production data) and establishing comprehensive Business Continuity and Disaster Recovery (DR) plans.
*   ⚡ **Power Resiliency:** Installing Uninterruptible Power Supplies (UPS) and backup generators to keep data centers and critical systems running smoothly during physical power outages.
*   ⚡ **Software Deployment Testing:** Using safe release techniques—such as "canary deployments" (releasing updates to a tiny fraction of users first) or sandbox testing—to catch software bugs before they can cause massive global outages, such as the July 2024 CrowdStrike incident.
