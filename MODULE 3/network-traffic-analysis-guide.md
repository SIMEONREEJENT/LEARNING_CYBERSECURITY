# Guide to Network Traffic Analysis: From Packets to Threat Detection

Welcome to this comprehensive guide on **Network Traffic Analysis (NTA)**. Whether you are a beginner taking your first steps in cybersecurity or an IT professional setting up monitoring infrastructure, this guide will explain the core concepts of network data in a clear, highly structured, and beginner-friendly manner.

This documentation is designed to be saved as a Markdown (`.md`) file, making it perfect for your GitHub repository.

---

## Table of Contents
1. [Packets](#packets)
   - [The Postcard Analogy](#the-postcard-analogy)
   - [Packet Structure: Header vs. Payload](#packet-structure-header-vs-payload)
   - [Core Protocol Headers (Layers 3 & 4)](#core-protocol-headers-layers-3--4)
   - [Header Fields Comparison](#header-fields-comparison)
2. [Traffic Flows](#traffic-flows)
   - [What is a Traffic Flow?](#what-is-a-traffic-flow)
   - [The 5-Tuple Key](#the-5-tuple-key)
   - [Flow Monitoring Architecture](#flow-monitoring-architecture)
   - [Stateful Aggregation (NetFlow/IPFIX) vs. Stateless Sampling (sFlow)](#stateful-aggregation-netflowipfix-vs-stateless-sampling-sflow)
   - [Flow Exporter Timing and Timeouts](#flow-exporter-timing-and-timeouts)
   - [Flow Telemetry Comparison](#flow-telemetry-comparison)
3. [Network Visibility](#network-visibility)
   - [Physical Capture: SPAN Ports vs. Network TAPs](#physical-capture-span-ports-vs-network-taps)
   - [Cloud-Native Visibility: Traffic Mirroring & vTAPs](#cloud-native-visibility-traffic-mirroring--vtaps)
   - [Network Packet Brokers (NPBs): The Smart Traffic Controllers](#network-packet-brokers-npbs-the-smart-traffic-controllers)
   - [Traffic Capture Methods Comparison](#traffic-capture-methods-comparison)
4. [Log Generation](#log-generation)
   - [Syslog: The Classic Log Standard (RFC 3164 vs. RFC 5424)](#syslog-the-classic-log-standard-rfc-3164-vs-rfc-5424)
   - [Protocol-Specific Logs: Deep Packet Inspection (Zeek)](#protocol-specific-logs-deep-packet-inspection-zeek)
   - [SIEM Normalization (Splunk CIM vs. Elastic Common Schema)](#siem-normalization-splunk-cim-vs-elastic-common-schema)
   - [The SIEM Ingestion and Enrichment Pipeline](#the-siem-ingestion-and-enrichment-pipeline)
   - [Logging Dimension Comparison](#logging-dimension-comparison)
5. [Threat Indicators](#threat-indicators)
   - [Why Behavioral Threats Matter (Moving Beyond Static Hashes)](#why-behavioral-threats-matter-moving-beyond-static-hashes)
   - [1. Beaconing (C2 Communication)](#1-beaconing-c2-communication)
   - [2. DNS Tunneling (Covert Data Channels)](#2-dns-tunneling-covert-data-channels)
   - [3. Lateral Movement (Sideways Attacks)](#3-lateral-movement-sideways-attacks)
   - [Behavioral Threat Detection Summary](#behavioral-threat-detection-summary)

---

## Packets

### The Postcard Analogy
In computer networking, data is not sent as a single continuous block. Instead, it is broken down into small, manageable pieces called **packets** [91, 132, 231]. 

To understand how a packet works, think of it as a **postcard sent through the mail** [133]:
* **The Front of the Postcard (Header):** Contains the metadata—the sender's address, the recipient's address, and the stamp. This is what the postal service (or network routers) inspects to deliver the message [133, 134].
* **The Back of the Postcard (Payload):** Contains the actual message or data you are writing to the recipient. This is the content that the recipient reads [132, 133].

---

### Packet Structure: Header vs. Payload
Every network packet is physically structured into these two distinct sections [132, 133, 659]:

1. **The Packet Header:** This is the administrative section located at the front of the packet [133, 659]. It contains protocol-specific control information, routing instructions, and error-checking fields [134, 287]. 
2. **The Packet Payload:** This is the actual data cargo carried by the packet (such as a fragment of an image, part of an email, or a web page request) [132, 133].

As packets travel down the **Open Systems Interconnection (OSI) model** layers, protocols add their own headers in a process called *encapsulation* [91, 135].

---

### Core Protocol Headers (Layers 3 & 4)
When analyzing network traffic, security analysts primarily look at headers from the **Network Layer (Layer 3)** and the **Transport Layer (Layer 4)** [134]:

#### 1. Internet Protocol Version 4 (IPv4) — Layer 3 [94, 134]
IPv4 headers are responsible for logical addressing and routing across different networks [4, 94, 287]. An IPv4 header typically ranges from 20 to 60 bytes in size [94]. Its key fields include [287]:
* **Source IP Address (32-bit):** The network address of the sender [287].
* **Destination IP Address (32-bit):** The network address of the recipient [287].
* **Protocol Field (8-bit):** Identifies which upper-layer protocol (TCP, UDP, or ICMP) is carried in the packet's payload [274, 275].
* **Time to Live (TTL):** An 8-bit counter decremented by every router the packet traverses [287, 290]. If TTL hits 0, the packet is discarded, preventing packets from looping forever [290].

#### 2. Transmission Control Protocol (TCP) — Layer 4 [91, 94, 134]
TCP is a connection-oriented, reliable protocol used for steady, error-checked bidirectional communications (such as web browsing or file transfers) [91, 143, 661]. A standard TCP header is 20 to 60 bytes long [94] and includes [91]:
* **Source & Destination Ports (16-bit each):** Identify the specific software applications or services communicating on the hosts (e.g., port 443 for HTTPS) [91, 146].
* **Sequence & Acknowledgment Numbers (32-bit each):** Keep track of the order of bytes transmitted, enabling packet reassembly and session tracking [91, 94].
* **TCP Flags (9-bit):** Control bits that manage connection state (e.g., `SYN` to initiate a session, `ACK` to acknowledge data, `FIN` to gracefully close, or `RST` to force reset) [91, 94, 99, 333].

#### 3. User Datagram Protocol (UDP) — Layer 4 [94, 134]
UDP is a connectionless, stateless, and lightweight protocol designed for speed rather than guaranteed delivery (such as DNS queries or video streams) [94, 655, 661]. Its header is a fixed **8 bytes** in size [94] and contains only [94]:
* **Source Port & Destination Port** (16-bit each) [94].
* **Length** (16-bit) [94].
* **Checksum** (16-bit) [94].

#### 4. Internet Control Message Protocol (ICMP) — Layer 3/4 [92, 94, 655]
ICMP is an auxiliary diagnostic protocol used to report network errors (e.g., "Destination Unreachable") and perform diagnostics (such as ping or traceroute) [92, 290, 655]. Its **8-byte header** consists of [92, 659]:
* **Type (8-bit):** Defines the message class (e.g., Type 8 for Echo Request, Type 0 for Echo Reply, or Type 3 for Destination Unreachable) [659, 660].
* **Code (8-bit):** Provides sub-details on the error or status [659].
* **Checksum (16-bit):** Validates header integrity [659].
* **Type-Specific Fields:** Used for sequence or parameter details [92].

---

### Header Fields Comparison

| Protocol Layer | Protocol | Standard Header Length | Key Header Fields | Security Forensics Value | Primary Parsing Challenges |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Layer 4** | **TCP** | 20 – 60 Bytes [94] | Ports, Seq/Ack, Flags, Options [91, 94] | Reconstructs connection state, tracks sessions [91, 94, 146] | Dynamic options, out-of-order sequence tracking [94] |
| **Layer 4** | **UDP** | 8 Bytes (Fixed) [94] | Source/Dest Ports, Length, Checksum [94] | Fast state correlation, uncovers DNS tunnels [94] | Stateless nature requires artificial session tracking [94] |
| **Layer 3** | **IPv4** | 20 – 60 Bytes [94] | Version, TTL, Protocol, IPs, Checksum [287] | Geolocation mapping, hop limit limits [94] | IP fragmentation reassembly, optional fields [94] |
| **Layer 3** | **ICMP** | 8 Bytes Header + Variable Data [92, 94] | Type, Code, Checksum, Parameter fields [92, 659] | Path discovery, diagnostic mapping [94, 290] | Highly variable payload data [94] |

---

## Traffic Flows

### What is a Traffic Flow?
While parsing individual packets is critical for deep investigation, capturing and storing every single packet traversing a high-speed network is incredibly expensive and resource-intensive [97, 165, 171]. 

To manage network resources efficiently, analysts abstract packet streams into **traffic flows** [97, 415, 432]. 
* **The Phone Call Analogy:** Think of a network packet as a single spoken word, while a **traffic flow is the summary of the entire phone conversation** [97, 432]. Instead of recording every word, you simply note down who called whom, when the call started, how long it lasted, and how many words were exchanged [98, 432].

---

### The 5-Tuple Key
A traffic flow is a unidirectional sequence of packets that share key matching properties within a specific time window [97, 416, 722]. These matching properties are known as the **5-Tuple** [97, 416, 722]:
1. **Source IP Address** [97, 416, 722]
2. **Destination IP Address** [97, 416, 722]
3. **Source Port Number** [97, 416, 722]
4. **Destination Port Number** [97, 416, 722]
5. **IP Protocol (e.g., TCP or UDP)** [97, 416, 722]

Packets matching this 5-Tuple are aggregated together into a single flow entry [98, 419, 714].

---

### Flow Monitoring Architecture
Flow monitoring systems are organized into three primary structural layers [464, 721, 733]:
* **Flow Exporter (Sensor):** The network device (such as a core switch, router, or firewall) that observes the raw network traffic, groups packets into flows, and maintains counters in its local cache [713, 721, 722].
* **Flow Collector:** A dedicated server that receives the summarized flow records transmitted from one or more exporters over the network [464, 713, 721].
* **Flow Analyzer:** A software tool that queries the collected data to construct visualizations, generate dashboards, and trigger security alerts [464, 714, 721].

---

### Stateful Aggregation (NetFlow/IPFIX) vs. Stateless Sampling (sFlow)
Network flow protocols are divided into two fundamentally different architectural models [98, 100, 415]:

#### 1. Stateful Device-Side Aggregation (NetFlow & IPFIX) [98, 415, 419]
Developed initially by Cisco (NetFlow) and standardized by the IETF (IPFIX - RFC 7011), this model actively tracks connection state [98, 416, 417].
* **How it works:** The exporter maintains an active **flow cache** in its RAM [98, 419]. For every packet passing through, the device looks up the 5-Tuple [419]. If the flow already exists, the device simply increments the byte and packet counters [98]. If not, it creates a new cache entry [35, 419].
* **Pros:** Highly accurate; captures 100% of network conversations in unsampled mode [101, 266, 417].
* **Cons:** Demands significant CPU and RAM from the forwarding network hardware [101, 171, 266, 420].

#### 2. Stateless Device-Side Sampling (sFlow) [100, 415, 418]
Developed by InMon, sFlow (sampled flow) is completely stateless [100, 415, 418].
* **How it works:** The forwarding hardware ASIC randomly selects one packet out of a defined interval of $N$ packets (e.g., 1 out of every 1,000) [100, 418, 420]. It immediately copies the packet's initial header (usually the first 128 bytes) and encapsulates it into an sFlow UDP packet sent directly to the collector [100, 418, 420]. The switch does not maintain any active memory of network conversations [100, 418].
* **Pros:** Extremely low CPU and memory overhead, allowing it to scale effortlessly on high-speed datacenter switches operating at 100 Gbps and beyond [101, 418, 421].
* **Cons:** Statistical only [101, 418]. It can easily miss short-lived, low-volume communication streams, making it less suitable for precise security investigations [101, 418].

---

### Flow Exporter Timing and Timeouts
In stateful protocols (NetFlow/IPFIX), a flow record remains inside the hardware cache and is only exported to the collector when an expiration condition is met [99, 723]. The two primary timing parameters are [99]:
* **Inactive Timeout:** Triggers when a connection goes silent for a specified duration (typically 15 seconds) [99]. The exporter assumes the conversation has ended, exports the record, and clears the cache entry [99, 723].
* **Active Timeout:** Forces the exporter to export records for long-lived, continuous connections at regular intervals (typically 60 seconds) [99]. This prevents the collector from being starved of telemetry during massive, hours-long transfers [99]—a crucial setting for real-time DDoS detection, where active timeouts should never be left at default long intervals [859].

---

### Flow Telemetry Comparison

| Telemetry Metric | NetFlow v5 [23, 24] | NetFlow v9 [23, 24] | IPFIX (RFC 7011) [16, 23] | sFlow v5 [16, 24] |
| :--- | :--- | :--- | :--- | :--- |
| **Tracking State** | Stateful aggregation cache [98, 419] | Stateful aggregation cache [98, 419] | Stateful aggregation cache [98, 419] | Stateless hardware sampling [100, 418] |
| **Hardware Overhead** | Moderate CPU / High RAM [101] | Moderate CPU / High RAM [101] | Moderate CPU / High RAM [101] | Extremely Low CPU / RAM [101, 418] |
| **Data Completeness** | 100% of packets tracked [101] | 100% of packets tracked [101] | 100% of packets tracked [101] | 1-in-N sampled headers [100, 418] |
| **Format Structure** | Fixed, static fields [101, 435] | Extensible via templates [101, 436] | Extensible via templates [101, 417] | Non-extensible fixed format [101, 421] |
| **IPv6 Support** | No [101, 470] | Yes [101, 436] | Yes [101, 417] | Yes (via raw headers) [101] |
| **Transport Protocol** | UDP only [101] | UDP only [101] | UDP, TCP, SCTP [101, 265] | UDP only [101] |
| **Standardization** | Proprietary (Cisco) [101] | Proprietary (Cisco) [101] | Open standard (IETF) [101, 417] | Open standard (InMon) [101, 418] |

---

## Network Visibility

To analyze network traffic, security teams must first acquire reliable, non-intrusive access to raw data [13, 238, 516]. There are three main approaches to capturing network traffic across physical, virtual, and cloud-native environments [102, 104]:

### Physical Capture: SPAN Ports vs. Network TAPs

```
         PHYSICAL NETWORK TAP (Passive & Unidirectional)
         ===============================================
         [ Device A ] <=========( Inline Link )=========> [ Device B ]
                                     ||
                       (Optical Splitter - 100% Copy)
                                     ||
                                     \/
                             [ monitoring Tool ]


         SPAN PORT / PORT MIRRORING (Software-Defined Copy)
         ==================================================
         [ Device A ] <======( Production Switch )=======> [ Device B ]
                                     ||
                       (Switch CPU Copies Best Effort)
                                     ||
                                     \/
                             [ monitoring Tool ]
```

#### 1. SPAN (Switched Port Analyzer) / Port Mirroring [102, 103, 520]
SPAN is a software feature built into standard network switches [102, 517, 520]. 
* **How it works:** The administrator configures the switch to make software-based copies of packets from designated interfaces or VLANs and send them to an unused destination port where a monitoring tool is attached [103, 484, 520].
* **Limitations:** The switch's primary function is routing and forwarding production traffic [22, 485, 520]. Under heavy traffic loads or high CPU utilization, **the switch will drop SPAN frames** to prioritize standard production traffic [22, 103, 485]. Furthermore, SPAN ports routinely filter out malformed packets and physical Layer-1 errors, blinding analysts to potential hardware issues or stealthy attack techniques [477, 485].

#### 2. Network TAPs (Test Access Points) [14, 102, 475]
A physical Network TAP is a purpose-built, external hardware device spliced directly into an active cabling run [15, 102, 549].
* **How it works:** It acts as a dedicated, transparent gateway [486, 519]. 
  - **Passive Optical TAPs (for fiber links):** Physically split the light beam using a precise ratio (e.g., 70/30), sending 70% of the light down the production link and 30% directly to the monitoring port [17, 102, 334]. They require **zero power** and are completely immune to network load or software configuration changes [17, 102, 487].
  - **Active Copper TAPs (for copper links):** Require active power to duplicate electrical signals [16, 19, 103]. To protect the link, they feature mechanical **fail-safe bypass relays** [103]. In a power outage, these relays physically close to maintain immediate link continuity between network nodes, though monitoring traffic temporarily ceases [16, 103].

---

### Cloud-Native Visibility: Traffic Mirroring & vTAPs
In virtualized public cloud environments (such as AWS and Azure), physical network cables do not exist, making standard hardware TAPs impossible to deploy [104, 239]. Cloud providers instead implement **Virtual TAPs (vTAPs)** and cloud traffic mirroring [104, 239].
* **How it works:** Operating at the hypervisor level, the cloud infrastructure intercepts packets directly from designated virtual machine NICs [7, 104].
* **Delivery:** The captured packet (including its full payload) is encapsulated within a **Virtual Extensible LAN (VXLAN)** frame and forwarded over **UDP port 4789** to a target Network Virtual Appliance (NVA) or collector for analysis [7, 8, 9, 104].

---

### Network Packet Brokers (NPBs): The Smart Traffic Controllers
As networks scale, sending raw, unoptimized captured streams from dozens of TAPs directly to security tools will quickly overwhelm them [398, 517, 574]. **Network Packet Brokers (NPBs)** are placed as a centralized optimization layer between traffic access points (TAPs/SPANs) and security tools [20, 21, 398, 404].

NPBs execute several critical, hardware-accelerated functions [105, 106, 399]:
* **FPGA-Accelerated Deduplication:** Mirroring traffic at multiple hops can generate up to **80% redundant packet volume** [106, 574]. NPBs calculate real-time packet hashes at line-rate in hardware to identify and discard duplicates before they reach downstream tools, reducing SIEM ingest costs [106, 575, 577].
* **Packet Slicing:** Strips application payloads from packets while preserving the headers [106]. This reduces bandwidth consumption and helps comply with privacy mandates like **GDPR** by masking sensitive user data [23, 106].
* **Protocol Stripping:** Removes transport tunnel encapsulations (e.g., VXLAN, GRE, or MPLS) so legacy analysis tools can read the packets without choking [106].
* **Session-Aware Load Balancing:** Distributes high-volume traffic evenly across multiple monitoring tools while keeping related packets of a single communication session pinned to the same physical tool [106, 743, 758].

---

### Traffic Capture Methods Comparison

| Feature | Physical TAP [29, 31] | Physical SPAN [30, 31] | Azure vTAP [38, 39] | Network Packet Broker [41, 45] |
| :--- | :--- | :--- | :--- | :--- |
| **Operational Layer** | Layer 1 (Physical cabling) [107] | Layer 2/3 (Switch software) [107] | Hypervisor / Virtual Interface [107] | Centralized Inline or Out-of-Band [107] |
| **Power Dependency** | None for passive optical [107] | Relies on active switch power [103, 107] | Software allocation (virtual) [107] | Requires dual redundant power [107] |
| **Data Integrity** | 100% capture, including errors [23, 107] | Drops frames during congestion/errors [22, 107, 485] | 100% VM interface packets captured [107] | Cleans data; discards duplicate frames [106, 107] |
| **Cloud Compatibility** | None (On-premises only) [107] | Limited to internal switch loops [107] | Native cloud integration [107] | Supports virtual and physical [107] |
| **Security Risks** | Physically secure, unaddressable [107] | Vulnerable to misconfiguration [107] | Relies on cloud IAM role access [107] | Enforces role-based masking [107] |

---

## Log Generation

Once packets and traffic flows are captured, they must be formatted into readable, normalized records so that Security Information and Event Management (SIEM) platforms can store and analyze them [110, 112, 373].

---

### Syslog: The Classic Log Standard (RFC 3164 vs. RFC 5424)
Syslog is the oldest and most widely supported event-transmission framework [108, 278, 630]. Its structure has evolved across two primary standards [108, 603, 810]:

#### 1. Legacy BSD Syslog (RFC 3164) [108, 603]
Written in 2001 to document historical practices, RFC 3164 is a loose, unstructured format [108, 637]. It consists of three components [108]:
* **PRI (Priority):** Encodes the *Facility* (source system component, e.g., auth or mail) and *Severity* (urgency, 0-7) [110, 636]. It is calculated as [110]:
  $$\text{PRI} = (\text{Facility} \times 8) + \text{Severity}$$
* **HEADER:** Contains a timestamp (restricted to `Mmm dd hh:mm:ss`, lacking year, sub-second precision, or timezone) and the hostname [108, 109, 645].
* **MSG:** A free-form text payload describing the event [109, 636, 644].
* **Limitations:** Size limit of **1,024 bytes** [109], which easily truncates modern JSON strings or stack traces [109]. Timestamps without timezones cause severe parsing issues when correlating events globally [109, 809].

#### 2. Modern Structured Syslog (RFC 5424) [108, 603]
Published in 2009 to replace RFC 3164, this standard enforces a strict, machine-parsable format [45, 108, 637].
* **Format:** `VERSION TIMESTAMP HOSTNAME APP-NAME PROCID MSGID [STRUCTURED-DATA] MSG` [791]
* **Improvements:** Employs high-precision **ISO 8601 timestamps** with timezones [791]. It expands the maximum message size to **2,048+ bytes** and introduces **Structured Data**, allowing standardized key-value pairs (`[exampleSDID iut="3" eventID="1011"]`) that can be parsed reliably without complex regular expressions [113, 278, 791].

---

### Protocol-Specific Logs: Deep Packet Inspection (Zeek)
While syslog transports general system events, **Network Security Monitoring (NSM)** platforms like **Zeek** (formerly Bro) actively perform Deep Packet Inspection (DPI) at the application layer to generate highly detailed metadata logs [110, 296, 781]. 

Rather than generating full packet captures (PCAP), Zeek extracts crucial forensic metadata into structured, tab-delimited or JSON log files [110]:
* `conn.log`: Tracks Layer-3 and Layer-4 connections, detailing duration, protocols, IPs, ports, and bytes [111, 318, 319].
* `dns.log`: Documents application-layer DNS queries, response codes, record types (A, AAAA, TXT), and answers [111, 835].
* `ssl.log` and `x509.log`: Capture TLS negotiations, Server Name Indications (SNI), JA3/JA3S client-server fingerprints, and raw cryptographic certificate details [111, 223, 346, 863].

#### The Power of the Zeek Connection UID [111]
The core design feature of Zeek is the globally unique **connection identifier (`uid`)** [111, 840]. When a single network transaction spans multiple protocols—for example, a user connects via TCP, issues a DNS query, completes a TLS handshake, and downloads an executable over HTTP—Zeek stamps the **exact same `uid`** across every corresponding log file (`conn.log`, `dns.log`, `ssl.log`, `files.log`, and `pe.log`) [111, 219, 840]. This allows security analysts to instantly reconstruct the entire attack lifecycle by searching for a single string [111, 224, 842].

---

### SIEM Normalization (Splunk CIM vs. Elastic Common Schema)
When enterprise logs arrive at a centralized SIEM, they come from thousands of different vendors, each using completely different names for the same attributes (e.g., one firewall logs `src_ip`, while a proxy logs `client_address`) [112, 116, 646]. To allow search and correlation, the SIEM must normalize these logs into a standardized taxonomy [112, 376].

Two major normalization schemas are used [112, 113]:
* **Schema-on-Read (Splunk Common Information Model - CIM):** Raw logs are written directly to disk as unparsed text [112]. Field extraction and normalization occur dynamically via lookups at the exact moment the analyst runs a query [112]. This offers highly flexible ingestion, but search performance can degrade on massive datasets [112].
* **Schema-on-Write (Elastic Common Schema - ECS):** Incoming logs are parsed, transformed, and mapped into strict canonical field names (e.g., mapping both source IP fields to `source.ip`) *before* they are written to disk [113, 612]. This ensures consistent, high-speed queries, but requires substantial up-front engineering to build and maintain the ingestion pipelines [113, 116].

---

### The SIEM Ingestion and Enrichment Pipeline
A robust ingestion architecture transforms raw logs through five sequential stages [114]:
```
  [ Raw Log Ingest ] 
         ||
         \/
  ( 1. Normalization )   =====> Map vendor fields to standardized ECS/CIM taxonomies [114].
         ||
         \/
  ( 2. GeoIP Enrichment ) =====> Query MaxMind databases to append coordinates/country codes [114].
         ||
         \/
  ( 3. Identity Resolution ) ===> Query Active Directory/LDAP to map SIDs to human usernames [114].
         ||
         \/
  ( 4. Threat Intel Matching ) => Scan IPs against feeds (e.g., GreyNoise) to flag malicious actors [114].
         ||
         \/
  ( 5. Asset Enrichment ) ======> Pull CMDB context to prioritize high-risk, critical systems [114].
         ||
         \/
  [ Normalized Indexed Log ]
```

---

### Logging Dimension Comparison

| Logging Dimension | Syslog RFC 3164 [48] | Syslog RFC 5424 [46, 48] | Zeek Protocol Logs [57, 59] | Elastic Common Schema [62, 64] |
| :--- | :--- | :--- | :--- | :--- |
| **Telemetry Source** | Host operating systems and network devices [116] | Enterprise systems, cloud resources, and routers [116] | Raw network traffic parsing at application layer [116, 135] | Multi-source normalized enterprise security events [116] |
| **Metadata Parsing** | Manual regular expressions required [116, 644] | Standardized, predictable key-value blocks [116, 278] | Pre-parsed structured protocol files [116] | Globally aligned canonical fields (e.g., `source.ip`) [113, 116] |
| **Transmission Protocol**| UDP (Default Port 514) [116, 814] | UDP, TCP, TLS (Default Port 6514) [110, 116] | Local file system output or socket streaming [116] | REST APIs, beats forwarders, and log shippers [116] |
| **Forensic Utility** | **Low:** Lacks timezone precision and structured data [109, 116] | **Medium:** Provides high-precision time and structured IDs [116] | **High:** Correlates disparate protocols via connection UID [111, 116] | **High:** Combines host, network, cloud, and identity context [116] |

---

## Threat Indicators

Relying on static **Indicators of Compromise (IOCs)** (such as hard-coded IP addresses or file hashes) provides limited security value, as attackers can trivially alter these elements to bypass standard signature rules [117]. Modern cyber defense instead focuses on **Behavioral Threat Detection** to identify the fundamental patterns of malicious activity [49, 117].

---

### Why Behavioral Threats Matter (Moving Beyond Static Hashes)
Behavioral detections answer the question: *"Does this traffic pattern look like something malicious hosts do, even without a known signature?"* [49] By identifying anomalous traffic shapes, volumetric asymmetries, and protocol abuses, security teams can detect zero-day exploits and sophisticated attackers who are actively trying to evade detection [117, 144].

---

### 1. Beaconing (C2 Communication) [52]
Once a host is compromised, the installed malware implant must "phone home" to the attacker's **Command and Control (C2)** server to receive instructions [52, 587]. Because the operator is not constantly sitting at the console, the implant connects back at regular, periodic intervals (e.g., every 5 minutes) [52].

#### How to Detect It:
* **Time-Domain Analysis (Threshold-Based):** Detections flag traffic exhibiting uniform payload byte sizes (standard deviation $< 1,000$ bytes), very low server response (low server packet counts), and high connection frequency sustained over hours [52, 54, 55].
* **Frequency-Domain Analysis (Fast Fourier Transform - FFT):** Advanced attackers introduce randomization, or **jitter**, to break basic time-interval checks [120, 121]:
  $$\text{Next Delay} = \text{Interval} \pm (\text{Interval} \times \text{Jitter})$$
  To uncover jittered beacons, analysts convert time-series connection logs into the frequency domain using the **Discrete Fourier Transform (DFT)** [121]. High-frequency noise is filtered, and statistical thresholds are calculated [122]:
  $$\text{Threshold} = \mu + z\sigma$$
  Any frequency amplitude exceeding this threshold indicates a statistically significant periodic signal—exposing the hidden beacon against standard background network noise [122].

---

### 2. DNS Tunneling (Covert Data Channels) [72, 117]
Since local firewalls typically allow unrestricted outbound UDP port 53 (DNS) traffic to ensure basic internet functionality, threat actors exploit DNS as a covert path to exfiltrate data or establish interactive C2 channels [71, 72, 117].

```
  [ Infected Client ] =======( Obfuscated base64 Query )=======> [ Local DNS Server ]
                                                                       ||
                                                                       \/
  [ Attacker C2 Server ] <===( Authoritative Name Resolution )=== [ Internet Root ]
```

* **How it works:** Attackers encode binary payload bytes using base32 or base64 and append them as nested subdomains of a domain they control (e.g., `YWNrLWVuY29kZWQtcGF5bG9hZC1ibG9iLXYy.evil-tunnel.net`) [73, 76, 118]. The local DNS resolver forwards this query to the attacker's authoritative name server, thereby transmitting the exfiltrated data [71, 118]. The name server's response carries commands back to the implant [118].

#### How to Detect It:
Analysts monitor for several key behavioral indicators [118, 178]:
* **High-Entropy Query Names:** Legitimate domain names are human-readable [178]. Tunneled subdomains contain high-entropy, randomized character distributions [71, 118].
* **Extreme Subdomain Depth:** Tunneled requests stack multiple subdomain levels to bypass single DNS label size limits [71, 118, 178].
* **Unusual Record Types:** High volumes of `TXT`, `NULL`, or `CNAME` query types, which are abused to maximize the returned response payload size [71, 118, 178].
* **Abnormal Label Lengths:** Queries approaching the maximum limit of 253 characters [118].

---

### 3. Lateral Movement (Sideways Attacks) [567]
Lateral movement describes the techniques attackers deploy to navigate across an internal network after gaining an initial foothold [567, 688]. 

```
                                [ Compromised Laptop ] 
                                  ||              ||
         ( Horizontal Connection  ||              || ( Vertical Port Scan 
          Spray - Touch many IPs) \/              \/   On Single IP)
                           [ Host A ]            [ Host B ]
                           [ Host C ]            (Port 22, 443, 3389)
                           [ Host D ]
```

The lateral movement lifecycle generally unfolds in three distinct stages [687]:
1. **Reconnaissance:** Attackers probe the local environment to identify high-value targets (e.g., domain controllers) and open ports using network scanning [65, 687, 689].
   - *Vertical Port Scanning:* One source IP probing many different ports on a single destination IP [64].
   - *Horizontal Connection Spraying:* One source IP touching the same specific port (such as port 445 for SMB) across many different destination IPs in a subnet to propagate worms or ransomware [68, 71].
2. **Credential Dumping:** Attackers extract plaintext passwords, hashes, and Kerberos tickets from local memory (using tools like Mimikatz) to escalate their privileges [667, 690].
3. **Gaining Access / System Takeover:** Attackers leverage administrative protocols like RDP, SSH, PsExec, and Windows Management Instrumentation (WMI) to execute commands remotely and establish persistent footholds on target systems [119, 668].

#### How to Detect It:
* **East-West Traffic Anomalies:** Flagging communication pathways between internal machines that historically have never interacted (such as workstation-to-workstation administrative traffic) [119, 696].
* **User and Entity Behavior Analytics (UEBA):** Profiling standard system-account logins to spot anomalous administrative access during off-hours [664, 674].

---

### Behavioral Threat Detection Summary

| Detection Metric | Target Behavioral Threat | Core Mathematical Algorithm | Key Threshold Settings | MITRE ATT&CK Mapping |
| :--- | :--- | :--- | :--- | :--- |
| **Entropy Coefficient** [71] | DNS Tunneling (Data Exfiltration) [71, 72] | Shannon Entropy: $H(X) = -\sum P(x_i)\log_2 P(x_i)$ [71, 118] | Query entropy threshold $H(X) \geq 5.2$ [71] | **T1071.004:** Protocol: DNS [70] |
| **Spectral Peak** [80] | C2 Periodic Beaconing [17, 80] | Real-Input Fast Fourier Transform ($np.fft.rfft$) [121] | Frequency amplitude peak $\geq \mu + 4\sigma$ [122] | **T1071:** C2 Application Layer [116] |
| **Subsequence Changepoint** [83] | Intermittent/Burst Beaconing [123] | Changepoint detection of event-time intervals [123] | Log-likelihood ratio value $g \geq 3.5$ [123] | **T1071.001:** Web Protocols [116] |
| **Host Cardinality** [17] | Internal Network Scanning / Spraying [117] | Cardinality tracking of unique Destination IPs [117] | Dest IP Count $\geq 20$ within 60 seconds [123] | **T1046:** Network Service Discovery [123] |
| **Byte Asymmetry** [17] | Data Theft / Bulk Exfiltration [117, 119] | Linear ratio calculation of outgoing to incoming bytes [117] | Ratio $\geq 5:1$ with $\geq 10$ MB upload [117] | **T1048:** Exfiltration Over Protocol [123] |
