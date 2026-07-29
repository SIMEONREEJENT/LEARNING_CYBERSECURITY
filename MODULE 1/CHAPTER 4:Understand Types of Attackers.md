# Understand Types of Attackers

The study of cybersecurity requires a structured understanding of the threat landscape and the various adversaries that operate within it. Threat modeling categorizes attackers by analyzing their technical capability, intent, and target selection to establish repeatable risk assessments and coordinated defenses.

Adversarial threat agents are systematically classified into five primary types: script kiddies, cyber criminals, insider threats, hacktivists, and nation-state actors, each defined by distinct motivations, skill levels, and operational methodologies.

## A. Script Kiddies

### Definition

A **script kiddie** (also referred to as a skid, skiddie, or lammer) is a pejorative classification for an unskilled individual who executes cyberattacks utilizing pre-built scripts, automated exploit kits, or publicly accessible hacking tools developed by advanced programmers.

### Explanation

Script kiddies lack formal information technology training and do not possess the technical expertise required to write custom code or comprehend underlying system architectures and network protocols. Their operational style is highly opportunistic, reactive, and noisy; instead of targeting specific entities, they indiscriminately scan the internet for known, unpatched vulnerabilities or weak administrative credentials. Their primary motivations are rooted in curiosity, peer recognition, attention-seeking, or the thrill of causing disruption. Due to their lack of knowledge, script kiddies exhibit poor operational security, frequently reusing handles, failing to cover their digital tracks, or falling victim to backdoored tools themselves.

### Key Terminology

* **Exploit Kit:** A pre-packaged software toolkit that automates the discovery and exploitation of system vulnerabilities.
* **Intrusion Detection System (IDS):** Security software that script kiddies frequently trigger due to their generation of noisy, easily detectable network traffic.
* **Operational Security (OPSEC):** The practice of concealing one's identity and digital footprint, a defensive measure that script kiddies routinely fail to maintain.

### Real-World Examples

* The 2015 breach of TalkTalk, a United Kingdom telecommunications company, which was orchestrated by a 17-year-old utilizing readily available automated tools.
* The 2014 Distributed Denial of Service (DDoS) attacks against the PlayStation Network and Xbox Live, conducted by a script kiddie collective known as "Lizard Squad".
* The deployment of automated malware generators to compile and propagate global disruptions, such as the ILOVEYOU and Anna Kournikova viruses.

## B. Cyber Criminals

### Definition

**Cyber criminals** are highly professional, commercially structured, and financially driven threat actors operating illicit enterprises.

### Explanation

Unlike amateur hackers, modern cyber criminals operate within organized structures that directly mirror legitimate corporate entities. They utilize marketing campaigns, service-level agreements (SLAs), and customer support channels to optimize their operations. Financial gain remains the primary motivator, driving approximately 86% of all cybercriminal operations. To achieve scale and targeted exploitation, these actors rely on sophisticated methodologies to distribute malware, harvest credentials, and extort organizations.

### Key Terminology

* **Ransomware-as-a-Service (RaaS):** A business model in which criminal organizations develop complex ransomware and lease it to affiliate attackers. The developers handle technical malware creation, while affiliates execute the target selection and ransom negotiation.
* **Business Email Compromise (BEC):** An attack methodology where corporate email accounts are compromised to intercept and conduct unauthorized financial transfers.
* **Spear-Phishing:** Highly targeted phishing campaigns directed at specific individuals or organizations, utilizing deception to trick targets into revealing sensitive information.

### Real-World Examples

* Mid-2024 ransomware attacks targeting the largest payment processor for U.S. healthcare transactions, causing extended delays in accessing electronic health records and forcing ambulances to divert patients.

## C. Insider Threats

### Definition

An **insider threat** is a security risk originating from individuals who currently have, or previously had, authorized access to an organization's systems, data, or facilities, such as employees, contractors, or business partners.

### Explanation

Insiders pose an elevated danger because they already possess legitimate access credentials, allowing them to bypass traditional perimeter defenses. The taxonomy of insider threats includes four main variations. **Malicious insiders** intentionally steal data or sabotage systems for revenge, financial gain, or corporate espionage. **Negligent or accidental insiders** cause harm through careless policy violations or human error, such as clicking phishing links. **Compromised insiders** are legitimate users whose credentials have been hijacked by external attackers. **Collusive insiders** are employees who partner with external competitors or state entities to bypass controls.

### Key Terminology

* **Shadow IT:** Unauthorized software, applications, or devices deployed by employees without IT department approval, frequently leading to negligent data exposure.
* **Zero-Trust Network:** A security framework that assumes all users and applications—even those already inside the corporate network—are potential threats, requiring strict, continuous access verification.
* **Data Loss Prevention (DLP):** Security solutions and strategies designed to detect and block the unauthorized transfer or exfiltration of sensitive information outside the corporate perimeter.

### Real-World Examples

* An IT administrator maliciously deleting critical corporate assets and causing massive operational downtime as an act of retaliation after being terminated.
* A contractor utilizing privileged system access to steal intellectual property and trade secrets to sell to a rival competitor.
* An employee inadvertently exposing sensitive customer data by forwarding unencrypted emails, representing a negligent threat.

## D. Hacktivists

### Definition

**Hacktivists** are threat actors who combine computer hacking techniques with activism to promote a specific political, social, religious, or ideological cause.

### Explanation

Hacktivists operate as digital protesters whose primary goal is not financial enrichment or military advantage, but rather to raise public awareness, express grievances, inflict reputational damage, or provoke systemic change. Their technical capabilities range from low-skilled individuals deploying pre-made tools to highly adept programmers. They frequently target public administrations, corporations, and governments whose policies run counter to the hacktivists' ideological beliefs.

### Key Terminology

* **Distributed Denial of Service (DDoS):** An attack technique that overloads targeted web servers with excessive internet traffic to render services inaccessible, providing a high-profile, symbolic means of disruption.
* **Website Defacement:** The unauthorized alteration of a target's website appearance to display political messaging or embarrass the victim organization.

### Real-World Examples

* Coordinated operations by decentralized collectives such as Anonymous and Lizard Squad, which have launched high-profile digital protests and cyberattacks against organizations they perceive as unethical.
* Massive, politically motivated DDoS campaigns deployed against European public administrations to disrupt citizen access to government resources during geopolitical conflicts.

## E. Nation-State Actors

### Definition

**Nation-state actors** are the most sophisticated, well-funded, and persistent threat class in cyberspace, operating with direct funding, direction, and protection from a national government to execute strategic geopolitical objectives.

### Explanation

These state-sponsored attackers engage in cyber warfare, military espionage, intellectual property theft, critical infrastructure disruption, and influence operations. They employ advanced, custom software engineering, utilizing zero-day vulnerabilities and supply-chain compromises. The primary state adversaries tracked globally operate with specific mandates: Russia focuses on aggressive disruption and wiper malware; China engages in prolific cyber-espionage and intellectual property theft; Iran utilizes cyber tools for regional deterrence and infrastructure targeting; and North Korea integrates state espionage with massive financial cybercrime (e.g., cryptocurrency theft) to fund state operations.

### Key Terminology

* **Advanced Persistent Threat (APT):** A highly skilled and determined adversary that establishes a prolonged, stealthy presence within a target network to continuously exfiltrate data or position itself for future sabotage.
* **Zero-Day Exploit:** An attack that takes advantage of a software vulnerability previously unknown to the vendor, for which no patch or mitigation currently exists.
* **Supply-Chain Compromise:** An attack vector where adversaries inject malicious code into the software or hardware of a trusted third-party vendor to indirectly compromise the vendor's downstream clients or government targets.

### Real-World Examples

* The 2022 Ukraine Electric Power Attack, attributed to the Russian state-sponsored APT known as "Sandworm Team," which specifically targeted and disrupted critical energy infrastructure.
* The SolarWinds and Hafnium supply-chain compromises, which allowed state actors to inject malicious updates and breach thousands of corporate and government networks globally.
* Operations by "MuddyWater," an Iranian APT group documented targeting telecommunications, government, and oil companies across the Middle East, Europe, and North America.

