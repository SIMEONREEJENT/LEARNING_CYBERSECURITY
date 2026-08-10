# 5. Understand Network Devices

This guide provides a comprehensive, beginner-friendly introduction to the core hardware and software components that power and secure modern enterprise networks: **Routers**, **Switches**, **Firewalls**, **Intrusion Detection Systems (IDS)**, and **Intrusion Prevention Systems (IPS)** [268]. 

---

## Table of Contents
1. [Introduction to Network Devices](#introduction-to-network-devices)
2. [a. Routers](#a-routers)
   - [Analogy: The Global Postal Service](#analogy-the-global-postal-service)
   - [Core Functions](#core-functions)
   - [Control Plane vs. Data Plane](#control-plane-vs-data-plane)
   - [Collision and Broadcast Domains](#collision-and-broadcast-domains)
3. [b. Switches](#b-switches)
   - [Analogy: The Office Intercom System](#analogy-the-office-intercom-system)
   - [Layer 2 Switches](#layer-2-switches)
   - [Virtual Local Area Networks (VLANs)](#virtual-local-area-networks-vlans)
   - [Layer 3 Switches (Multilayer Switches)](#layer-3-switches-multilayer-switches)
   - [Key Differences: Layer 3 Switch vs. Dedicated Router](#key-differences-layer-3-switch-vs-dedicated-router)
4. [c. Firewalls](#c-firewalls)
   - [Analogy: The Security Gate and ID Checker](#analogy-the-security-gate-and-id-checker)
   - [The Evolution of Firewalls](#the-evolution-of-firewalls)
   - [Stateful Connection Tracking](#stateful-connection-tracking)
   - [Boundary Security: Perimeter vs. Internal Segmentation Firewalls (ISFW)](#boundary-security-perimeter-vs-internal-segmentation-firewalls-isfw)
   - [Demilitarized Zones (DMZ)](#demilitarized-zones-dmz)
5. [d. Intrusion Detection Systems (IDS)](#d-intrusion-detection-systems-ids)
   - [Analogy: The Security Camera System](#analogy-the-security-camera-system)
   - [How It Works (Passive Out-of-Band)](#how-it-works-passive-out-of-band)
   - [NIDS vs. HIDS](#nids-vs-hids)
   - [Detection Methodologies](#detection-methodologies)
   - [Inherent Limitations](#inherent-limitations)
6. [e. Intrusion Prevention Systems (IPS)](#e-intrusion-prevention-systems-ips)
   - [Analogy: The Active Bouncer](#analogy-the-active-bouncer)
   - [How It Works (Active Inline)](#how-it-works-active-inline)
   - [IPS Classifications](#ips-classifications)
   - [The Operational Danger of Inline Tools](#the-operational-danger-of-inline-tools)
   - [Rerouting Failures: Hardware Bypass Switches (Bypass TAPs)](#rerouting-failures-hardware-bypass-switches-bypass-taps)
7. [Device Summary and Comparison Matrix](#device-summary-and-comparison-matrix)

---

## Introduction to Network Devices

Computer networks are made of many devices that must communicate reliably, quickly, and securely. In the early days of networking, simple devices like **hubs** were used to connect computers [180]. However, a hub is "dumb": when it receives data from one computer, it simply repeats (floods) that data out of every single port, meaning all other computers have to listen to it and drop it if it is not addressed to them [5, 180, 259]. This causes immense network traffic congestion and collisions [180, 259].

To solve these problems, modern networks rely on specialized devices [180]. Understanding how **Routers**, **Switches**, **Firewalls**, **IDS**, and **IPS** interact is the foundation of network engineering and cybersecurity [565, 573].

---

## a. Routers

### Analogy: The Global Postal Service
Imagine you want to mail a letter from London to Tokyo. You don't walk to Tokyo yourself; instead, you drop the letter into a mailbox. The local post office reads the destination address, decides which regional center to send it to, and forwards it [676]. This chain repeats across international transport systems until the letter reaches the recipient's local mailbox [676]. 

A **Router** is the postal service of the internet [676]. It acts as a gateway that connects completely different, geographically separated networks (such as your home network to the Internet, or separate corporate offices) [278, 488].

```
                 +-------------------+
                 |      Internet     |
                 +---------+---------+
                           |
                     [ WAN Interface ]
                           |
                     +-----+-----+
                     |  ROUTER   |  <-- Connects LAN to WAN, Routes packets by IP
                     +-----+-----+
                           |
                     [ LAN Interface ]
                           |
                 +---------+---------+
                 |   Local Network   |
                 +-------------------+
```

### Core Functions
- **Operates at Layer 3 (Network Layer):** Routers make forwarding decisions based on logical **Internet Protocol (IP) addresses**, which can change dynamically over time [450, 488].
- **Path Determination:** Routers look at the destination IP address of each packet, consult an internal **Routing Table (RIB)**, and dynamically calculate the "best next hop" to push the packet closer to its final destination [25, 453, 491].
- **Edge Security and Services:** Because routers sit at the border of networks, they often handle critical edge functions such as **Network Address Translation (NAT)** (which shields internal private IP addresses from the public internet), stateful firewalling, and secure tunnels (like VPNs) to connect different offices securely [116, 284, 285].

### Control Plane vs. Data Plane
Modern routers enforce a strict division between their administrative brain and their high-speed forwarding muscles [279, 975]:
1. **Control Plane (The Brain):** Runs in software on the router's main CPU [279]. It runs complex routing protocols (like OSPF, EIGRP, and BGP) and builds the **Routing Information Base (RIB)**—the master map of the network [25, 279].
2. **Data Plane / Forwarding Plane (The Muscles):** Moves packets from an incoming port to an outgoing port as fast as possible [280]. The optimal routes from the RIB are pre-compiled into a simplified **Forwarding Information Base (FIB)** and pushed directly into hardware tables [280, 282]. Packets flowing through the router use this hardware-accelerated path, bypassing the CPU entirely to achieve wire-speed throughput [280, 976].

### Collision and Broadcast Domains
- **Collision Domain:** A physical network segment where data packets can crash (collide) with one another when sent at the same time [5, 181]. Routers break up collision domains [185, 662].
- **Broadcast Domain:** A logical division of a network where all devices receive "broadcast" messages (like ARP or DHCP requests) [5, 7, 272]. Broadcast messages are like shouting in a room; everyone inside has to listen [181]. If a broadcast domain is too big, the constant "shouting" slows down devices (a broadcast storm) [272, 647]. 
- **The Router Boundary:** **Routers do not forward Layer 2 broadcast traffic [3].** This means a router forms a strict boundary that separates broadcast domains, keeping network traffic localized, organized, and secure [3, 7].

---

## b. Switches

### Analogy: The Office Intercom System
Imagine an office building where the manager wants to talk to a specific employee in Room 102. Instead of broadcasting his voice over a giant loudspeaker to the entire building (like a hub does), he dials Room 102 directly on an intercom. The conversation is private, and no other room is disturbed.

A **Switch** is the intelligent intercom system of a Local Area Network (LAN) [259, 268]. It connects local devices (computers, printers, servers) together within the same physical building or office [268, 488].

```
                     +-------------+
                     |   SWITCH    |  <-- Connects local devices in a LAN
                     +--+---+---+--+
                        |   |   |
         +--------------+   |   +--------------+
         |                  |                  |
   +-----+-----+      +-----+-----+      +-----+-----+
   | Computer  |      |  Printer  |      |  IP Camera |  <-- Each port is an isolated
   +-----------+      +-----------+      +-----------+      collision domain
```

### Layer 2 Switches
Traditional switches operate at **Layer 2 (Data Link Layer)** of the OSI model and move traffic using physical **Media Access Control (MAC) addresses** [260, 277].
- **MAC Address Table (CAM Table):** The switch maintains an internal lookup table (Content Addressable Memory) mapping physical MAC addresses to their connected port numbers [260, 269].
- **Dynamic Learning:** When a device sends data, the switch reads the source MAC address to learn which port it is plugged into [259, 269]. It then reads the destination MAC address [269]. If it knows the port, it sends the data *only* to that port [259, 269]. If the address is unknown or is a broadcast, the switch floods it to all ports [269].
- **Microsegmentation (Eliminating Collisions):** **Every single port on a switch is its own separate collision domain [1, 3].** By separating ports physically and supporting **full-duplex mode** (electrically separating the sending and receiving wires), switches completely eliminate physical collisions, letting devices send and receive data at the same time [10, 271, 655].

### Virtual Local Area Networks (VLANs)
Although Layer 2 switches segment collision domains, they do not block broadcast traffic [260, 272]. By default, all devices plugged into a switch belong to the same broadcast domain [260, 272]. 
To solve this, switches support **VLANs** [260, 273]. VLANs let administrators logically carve a single physical switch into multiple isolated virtual networks [273]. A broadcast sent on VLAN 10 will never reach a device on VLAN 20 [273]. To allow these different VLANs to talk to each other, you must pass traffic through a Layer 3 routing mechanism [3, 273].

### Layer 3 Switches (Multilayer Switches)
A **Layer 3 Switch** is a hybrid device that merges the rapid frame-forwarding of a Layer 2 switch with the routing capabilities of a router [273, 489].
- **ASIC-Driven Routing:** Traditional routers route packets using software on a CPU [273, 504]. Layer 3 switches perform routing in hardware using specialized chips called **Application-Specific Integrated Circuits (ASICs)** [273, 504].
- **Route Once, Switch Many:** The very first packet traveling between different VLANs is sent to the switch's CPU to determine the path [509, 512]. Once the path is determined, the route is programmed into the ASIC's hardware forwarding table [507, 509]. All subsequent packets are switched directly in hardware at wire speed with minimal latency [261, 510].
- **Logical Gateways:** Inter-VLAN routing is represented on the switch by **Switch Virtual Interfaces (SVIs)**, which act as the default gateways for their respective VLAN subnets [274].

### Key Differences: Layer 3 Switch vs. Dedicated Router
While a Layer 3 switch is incredibly fast for routing internal LAN traffic, it cannot fully replace a dedicated router at the edge of a network [284, 443]. 

| Feature | Layer 3 Switch [277, 285, 496] | Dedicated Enterprise Router [285, 496] |
| :--- | :--- | :--- |
| **Primary Domain** | Internal local network (LAN) [496] | External wide area network (WAN) & Internet [496] |
| **Hardware Engine** | Fast, hard-wired ASICs [285] | Specialized Network Processors or CPUs [285] |
| **Routing Table Size** | Limited (thousands of local LAN routes) [284, 285] | Massive (millions of global BGP routes) [284, 285] |
| **Interface Types** | Ethernet and Fiber Uplinks only [285, 495] | Diverse (Ethernet, T1/T3, Serial, SONET) [285, 495] |
| **Packet Buffer Depth** | Shallow (designed for low-latency LAN) [285] | Deep (designed to handle WAN congestion) [284, 285] |
| **Advanced Edge Services**| Rarely supports NAT, VPNs, or Tunneling [284, 285] | Full support (NAT, IPSec, GRE, Firewalls) [284, 285] |
| **Traffic Control** | Coarse Traffic Policing [285] | Granular Traffic Shaping and QoS [284, 285] |

---

## c. Firewalls

### Analogy: The Security Gate and ID Checker
Imagine a secure corporate building. A visitor cannot just walk in; they must stop at the security gate. The guard checks their ID, verifies they have an active appointment, logs their entry, and monitors them. If they try to enter an unauthorized area, the guard stops them.

A **Firewall** is the security gate of your network [292, 534]. It sits at network boundaries and monitors all incoming and outgoing traffic, deciding whether to allow or block packets based on a defined set of security rules [292, 521, 534].

```
     Untrusted                   Monitored Zone                  Protected
     (Internet)                      (DMZ)                     (Private LAN)
         |                             |                             |
   +-----+-----+                 +-----+-----+                 +-----+-----+
   |  External | --(Firewall)--> | Web/Email | --(Firewall)--> |  Internal |
   |  Network  |                 |  Servers  |                 |  Network  |
   +-----------+                 +-----------+                 +-----------+
```

### The Evolution of Firewalls
Firewall technology has evolved through three distinct generations to handle increasingly sophisticated cyber threats [286]:

1. **First-Generation: Packet Filtering Firewalls (Stateless)**
   - **How it works:** Operates at Layers 3 and 4 of the OSI model [286]. It inspects each packet in complete isolation, checking its header against a static access control list (ACL) [286]. It only looks at the source IP, destination IP, protocol, and port numbers [286, 534].
   - **Limitation:** Extremely blind [286]. It cannot tell if an incoming packet is a legitimate reply to a request you sent, or an attacker trying to break in [535, 864]. It is highly vulnerable to IP spoofing and payload-based attacks [286, 535].

2. **Second-Generation: Stateful Inspection Firewalls**
   - **How it works:** Operates at Layers 3 and 4, but tracks connection context [287, 801]. It maintains a dynamic **State Table** of all active, established network connections [207, 287].
   - **The Hands-on Handshake:** For TCP traffic, the firewall monitors the **TCP Three-Way Handshake** (SYN, SYN-ACK, ACK) [207, 288]. It logs the connection as `NEW` [207, 288]. Once the handshake completes, the connection state becomes `ESTABLISHED` [207, 288].
   - **Context-Aware Security:** Any incoming packet is checked against the State Table [848]. If it belongs to an established conversation, it is let through automatically [287, 532]. If not, it is compared against the security rules [538, 848]. This stops connection hijacking and unsolicited incoming scans [207, 289, 532].
   - **Pseudo-State for UDP:** Because connectionless protocols like UDP do not have state flags, stateful firewalls create a virtual "pseudo-state" entry with a custom inactivity timer (such as when you send a DNS query on UDP port 53, temporarily allowing the response back) [207, 290, 684].

3. **Third-Generation: Next-Generation Firewalls (NGFW)**
   - **How it works:** Combines traditional stateful packet filtering with deep analysis up to Layer 7 (the Application Layer) [294, 767, 777].
   - **Key Capabilities [542, 544]:**
     - **Deep Packet Inspection (DPI):** Looks inside the packet payload, not just the headers, to identify hidden malware and malicious scripts [540, 781].
     - **Application Awareness (App-ID):** Recognizes the actual application generating traffic regardless of port (e.g., separating Dropbox traffic from general web browsing on port 443) [544, 782].
     - **User Identity Awareness (User-ID):** Integrates with directory services (like Active Directory) to tie security policies to specific users or roles rather than raw IP addresses [544, 783].
     - **SSL/TLS Decryption:** Decrypts, inspects, and re-encrypts encrypted traffic to catch hidden malware and command-and-control channels [544, 784].
     - **Integrated Security Services:** Combines sandboxing, URL filtering, and Intrusion Prevention Systems (IPS) into a single physical appliance [523, 753, 778].

### Boundary Security: Perimeter vs. Internal Segmentation Firewalls (ISFW)
- **Perimeter Firewalls:** Deployed at the edge of the corporate network to inspect "North-South" traffic (data entering or leaving the network from the internet) [292, 809].
- **Internal Segmentation Firewalls (ISFW):** Deployed deep inside the local network [292, 403]. Traditional security assumed the internal network was safe, leaving it "flat" and open [402, 403]. If an attacker breached the perimeter, they could move laterally ("East-West" traffic) completely unchecked [292, 403, 423]. ISFWs sit in front of high-value internal assets (like customer databases or finance servers) to enforce Zero Trust boundaries at multi-gigabit speeds [292, 403, 414].

### Demilitarized Zones (DMZ)
A **Demilitarized Zone (DMZ)** is an isolated subnetwork that acts as a secure buffer checkpoint between the untrusted public internet and your trusted internal private network [55, 293, 886]. 
You place your public-facing servers (web servers, mail servers, public DNS) inside the DMZ [55, 293, 738].
- **Containment Philosophy:** If an attacker compromises your public web server, the DMZ's strict firewall rules prevent them from moving laterally into your private local network [55, 293, 738, 887].
- **DMZ Architectures:**
  1. **Single-Firewall DMZ:** A single firewall appliance configured with three network interfaces: Inside (trusted), Outside (untrusted), and DMZ (semi-trusted public services) [110, 192, 743].
  2. **Dual-Firewall DMZ (Screened Subnet):** The most secure approach, using two separate firewalls [194, 743, 890]. An **External Firewall** sits between the internet and the DMZ, and an **Internal Firewall** sits between the DMZ and the private local network [194, 890, 897]. An attacker must compromise *both* firewalls to reach your private data [194, 743].

---

## d. Intrusion Detection Systems (IDS)

### Analogy: The Security Camera System
Imagine a retail store. The owner installs high-definition security cameras throughout the building. The cameras watch everything, record footage, and can use software to flag when someone enters a restricted area. If a shoplifter steals an item, the camera records the act and triggers a silent alarm to alert the store manager. However, the camera cannot physically jump off the wall and grab the thief's arm; it only provides visibility, alerts, and evidence.

An **Intrusion Detection System (IDS)** is the security camera of your network [391, 571]. It monitors systems or network traffic passively, analyzing data to identify potential threats or policy violations, and triggers alerts for security teams to investigate [352, 353, 382].

```
                     +-------------+
                     |   SWITCH    |
                     +--+---+---+--+
                        |   |   |
         +--------------+   |   +--------------+
         |                  |                  |
   +-----+-----+      +-----+-----+      +-----+-----+
   | Computer  |      |   Server  |      |  SPAN/TAP |  <-- Mirrors a copy of traffic
   +-----------+      +-----------+      +-----+-----+      out-of-band to the IDS
                                               |
                                         +-----+-----+
                                         |    IDS    |  <-- Analyzes data & alerts,
                                         +-------------+      does NOT block traffic
```

### How It Works (Passive Out-of-Band)
- **Passive Monitoring:** Unlike firewalls, which sit directly in the middle of traffic, an IDS operates **out-of-band** [295, 391]. Traffic does not physically flow through the IDS [362].
- **Traffic Mirroring:** The IDS receives a copied stream of network traffic from a physical **network TAP** or a **SPAN (Switch Port Analyzer) port** on a switch [156, 295, 384].
- **Zero Traffic Impact:** Because it analyzes a copy, a failure of the IDS has zero impact on network performance or availability, and it introduces no processing latency to the live network [295, 391].
- **No Direct Prevention:** An IDS cannot drop packets, block traffic, or alter connection payloads on its own [295, 352, 391].

### NIDS vs. HIDS
IDS implementations are classified by where they gather their security data [356, 384]:

1. **Network-Based IDS (NIDS):** Deployed at strategic network choke points [356, 369]. NIDS places its network interface card into **promiscuous mode**, allowing the adapter to capture and analyze all packets traversing that entire network segment, regardless of the destination MAC address [295, 872].
2. **Host-Based IDS (HIDS):** Installed as a software agent directly on individual high-value devices (like servers or laptops) [295, 356, 873]. It monitors host-level activity, including operating system logs, running processes, file access attempts, and critical system file integrity (by taking snapshots of system files and alerting if they are modified or deleted) [295, 356, 873, 874].

### Detection Methodologies
To identify threats, the IDS analysis engine uses three primary detection techniques [298, 355, 372]:
- **Signature-Based Detection:** Compares network patterns against a database of known threat fingerprints (like specific byte sequences or malware file hashes) [298, 355]. It is highly accurate and efficient for known attacks but completely blind to zero-day (novel) exploits [298, 355, 389].
- **Statistical Anomaly-Based Detection:** Uses machine learning to establish a baseline of "normal" network behavior (average bandwidth, protocol usage, user login times) [298, 355, 389]. If network activity deviates significantly from this baseline, it triggers an alert [298, 389]. This is great for spotting new threats but suffers from high **false-positive rates** if the baseline is poorly configured [298, 355, 594].
- **Stateful Protocol Analysis:** Compares real-time protocol exchanges against strict RFC standards to detect deviations or manipulations of the protocol stack [298, 358, 372].

### Inherent Limitations
While valuable, an IDS suffers from several core challenges [361]:
- **Alert Fatigue (Noise):** High rates of false alarms caused by minor bugs, corrupted packets, or network changes can cause real attacks to be missed or ignored [361, 388].
- **Outdated Signature Databases:** Signature-based engines require constant database updates to remain effective [361, 586].
- **Encryption Blind Spot:** Most NIDS engines cannot read or process encrypted payloads, letting malicious traffic hide inside TLS streams [361, 361].

---

## e. Intrusion Prevention Systems (IPS)

### Analogy: The Active Bouncer
Imagine a premier nightclub. Instead of just installing security cameras to record bad behavior (like an IDS), the club hires a professional bouncer and positions him directly at the front door. Every guest must walk past him. If he spots an underage ID or a known troublemaker, he physically blocks them from entering. If a fight breaks out inside, he immediately grabs the instigators and throws them out.

An **Intrusion Prevention System (IPS)** is the active network bouncer [391]. It proactively monitors network traffic and, upon detecting a threat, takes immediate, automated action to block, drop, or mitigate the attack in real-time [352, 366, 386].

```
     Live                                                              Live
    Traffic ---> [ Network Port 1 ] --> (  IPS  ) --> [ Network Port 2 ] ---> Traffic
                                           |
                                    [Active Control]
                                           |
                                    - Drops bad packets
                                    - Resets TCP sessions
                                    - Blocks offending IPs
```

### How It Works (Active Inline)
- **Active Inline Deployment:** Unlike an IDS, an IPS is placed **inline** directly in the physical data path of live network traffic [296, 386, 706]. Every packet must physically traverse the IPS interfaces to reach its destination [296, 709].
- **Real-Time Prevention:** Because it is inline, the IPS can take immediate prevention actions before a packet can reach its target [296, 353, 386]:
  - **Drop packets:** Discards malicious or unauthorized packets on a packet-by-packet basis [296, 352, 362].
  - **Terminate TCP sessions:** Injects TCP reset (RST) packets to both the client and server to tear down an active exploit session instantly [296, 359].
  - **Reconfigure firewalls:** Dynamically instructs firewalls to block the offending attacker's IP address [358, 359, 386].
  - **Sanitize payloads:** Strip out malicious attachments or headers and rewrite packets in transit [296, 359, 709].

### IPS Classifications
- **Network-Based IPS (NIPS):** Inspects and blocks malicious traffic across the entire subnet in real-time [360, 371].
- **Host-Based IPS (HIPS):** Installed on individual devices to analyze code behavior, particularly useful for protecting encrypted local files and preventing data exfiltration (like PII or PHI) [360, 371].
- **Wireless IPS (WIPS):** Scans the radio frequency spectrum to detect and block rogue access points, MAC spoofing, and wireless man-in-the-middle attacks [360, 371, 388].
- **Network Behavior Analysis (NBA):** Anomaly-focused prevention that monitors network flows to find deviations from baseline patterns [297, 360, 371].

### The Operational Danger of Inline Tools
Because an active IPS operates inline, its physical failure represents a severe operational threat [299, 711]. If the IPS software crashes, suffers power loss, or becomes overwhelmed under heavy traffic load, the physical network link is broken [229, 299, 711]. **Traffic stops entirely, and your network goes down [229, 299, 707].** 

To resolve this issue, security appliances have built-in fail behaviors, but neither is ideal:
- **Fail-Open:** Keeps the network running if the device fails, but leaves the network completely unprotected [711, 712].
- **Fail-Closed:** Shuts down the network entirely to prevent any uninspected traffic, causing catastrophic business downtime [711, 712].

### Rerouting Failures: Hardware Bypass Switches (Bypass TAPs)
To eliminate this single point of failure, organizations implement external **Hardware Bypass Switches** (also known as **Bypass TAPs**) [13, 299, 707].

```
                  +-----------------------------------+
                  |            Bypass TAP             |
                  |                                   |
    Network In -->| [Net Port A] ===[Relay]=== [Port B] |--> Network Out
                  +----+-------------------------+----+
                       |                         ^
                 (Monitor Out)              (Monitor In)
                       v                         |
                  +----+-------------------------+----+
                  |           INLINE IPS              |
                  +-----------------------------------+
```

- **How it works:** The Bypass TAP is a dedicated hardware device placed directly in the physical network path [299, 642]. Under normal conditions, it passes live traffic through the inline IPS [17, 230, 642].
- **Heartbeat Monitoring:** The Bypass TAP continuously injects small **heartbeat test packets** through the IPS inline path [15, 299, 716]. 
- **Fail-Open Action:** If the inline IPS crashes or reboots, it fails to return the heartbeat packets [15, 19, 299, 644]. The Bypass TAP instantly detects this and triggers a hardware relay, bypassing the IPS to keep live traffic flowing unimpeded [15, 17, 299, 644].
- **Dynamic Recovery:** The Bypass TAP continues to send heartbeats [17, 19, 644]. Once the IPS recovers and returns the heartbeats, the TAP automatically restores the inline path [17, 17, 644].
- **Physical Fail-Safe Protection:** If the Bypass TAP itself loses power, it uses internal copper micro-relays or normally closed optical switches to create a direct physical link, ensuring the network link never drops [14, 18, 645, 715].

---

## Device Summary and Comparison Matrix

The table below summarizes the key attributes of each network device:

| Device | Operating OSI Layer | Primary Forwarding/Filtering Identifier | Traffic Path Placement | Action on Threat Detection | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Router** [277, 285] | Layer 3 (Network) [277] | Logical IP Address [277, 488] | Inline boundary [278, 491] | Drops packet (if routing rule/ACL matches) [285, 534] | Connecting separate networks and routing WAN/Internet traffic [278, 488, 496] |
| **Switch (L2)** [277] | Layer 2 (Data Link) [277] | Physical MAC Address [277, 450] | Local LAN connections [268, 488] | Drops frame (if Port Security violation) [88, 138, 277] | High-speed, local device connectivity and VLAN segmentation within a LAN [260, 449, 452] |
| **Firewall (Stateful)** [294, 533] | Layers 3 and 4 [294, 533] | IP address, port, and connection State [333, 533] | Inline boundary or segment [292, 415] | Blocks traffic, drops packets, or resets connection [294, 359, 687] | Guarding network boundaries and monitoring connection session states [292, 532, 537] |
| **IDS** [295, 391, 533] | Layers 3 through 7 [295, 533] | Known signatures or behavior baselines [298, 355] | Passive Out-of-Band (mirrored TAP/SPAN) [295, 384, 391] | Generates alerts and logs event data [295, 352, 382] | Passive traffic visibility, security monitoring, and rule validation [381, 383, 385] |
| **IPS** [296, 391, 533] | Layers 3 through 7 [296, 533] | Known signatures or behavior baselines [298, 358] | Active Inline [296, 386, 391] | Blocks packets, resets TCP, or alters payload [296, 359, 386] | Real-time active threat blocking and automated vulnerability mitigation [296, 352, 386] |
