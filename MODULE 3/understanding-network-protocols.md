# Understanding Network Protocols

This document provides a detailed, beginner-friendly, and well-structured overview of core internet protocols. It is designed to be used directly for GitHub documentation (such as in a repository wiki or `README.md` file).

---

## Table of Contents
1. [HTTP (Hypertext Transfer Protocol)](#http)
2. [HTTPS (Hypertext Transfer Protocol Secure)](#https)
3. [DNS (Domain Name System)](#dns)
4. [DHCP (Dynamic Host Configuration Protocol)](#dhcp)
5. [FTP (File Transfer Protocol)](#ftp)
6. [SMTP (Simple Mail Transfer Protocol)](#smtp)
7. [SSH (Secure Shell)](#ssh)

---

## HTTP

### 1. What is HTTP?
**HTTP** stands for **Hypertext Transfer Protocol**. It is the primary technical standard used for distributed, collaborative, and hypermedia information systems [40]. In simple terms, it is the foundational language used by your web browser (the client) and web servers to talk to each other so you can load web pages.

### 2. How Does It Work?
When you type a website address or click a link, your browser sends a request to a server, and the server sends back a response containing the website's content.

### 3. The Big Security Vulnerability
Historically, legacy HTTP operated in **plaintext** directly over the raw **Transport Control Protocol (TCP)** [40]. 
* **The Plaintext Problem:** "Plaintext" means the data is sent as raw, unencrypted text. Because the data is unencrypted, the payload contents, headers, and metadata are highly vulnerable to:
  * **Passive Eavesdropping:** Anyone sitting on the network path (like a hacker on public Wi-Fi) can easily read your data [40].
  * **Active Injection Attacks:** Malicious actors can intercept your connection and inject harmful code or alter the data being sent [40].

Due to these critical vulnerabilities, the internet has largely transitioned to **HTTPS**.

---

## HTTPS

### 1. What is HTTPS?
**HTTPS** stands for **Hypertext Transfer Protocol Secure**. It was introduced to solve HTTP's massive security flaws by encapsulating standard HTTP traffic within a cryptographically secure **Transport Layer Security (TLS)** tunnel [40]. 

### 2. How Does It Work?
HTTPS provides three core guarantees [743]:
1. **Encryption:** Converts plaintext into scrambled ciphertext, preventing eavesdropping on data in transit.
2. **Authentication:** Verifies that the server (and optionally the client) is who it claims to be using digital certificates.
3. **Integrity:** Detects any tampering or modification of data during transmission.

### 3. The Cryptographic Mix: Asymmetric & Symmetric Encryption
To be both secure and fast, HTTPS combines two types of cryptography [862]:
* **The Handshake (Asymmetric Encryption):** When your browser first connects, it performs a "handshake." It uses asymmetric keys (a public/private key pair, such as RSA or an elliptic-curve pair) to safely authenticate the server and negotiate a "shared secret" key [862]. This mathematically heavy process happens only once [862].
* **Bulk Data Transfer (Symmetric Encryption):** Once the handshake is complete, both sides use the shared secret key to run a fast symmetric cipher (like AES-GCM or ChaCha20-Poly1305) to encrypt all the actual data you send and receive [862]. 
* **Tamper Prevention (AEAD):** Each chunk of data is authenticated with an **AEAD (Authenticated Encryption with Associated Data)** tag [862]. If a single bit is altered or a fake packet is injected, it fails verification and the connection is immediately torn down [862].

### 4. The Leap to TLS 1.3
While legacy systems used TLS 1.2, the industry is currently undergoing a wholesale migration to **TLS 1.3 (RFC 8446)**, which brings major performance and security enhancements [40, 753]:
* **Streamlined Handshake (1-RTT):** In TLS 1.2, establishing a connection required two round trips of data (2-RTT) [753]. TLS 1.3 speculatively sends key shares in the first message, cutting this to a single round trip (1-RTT) [771, 863]. This reduces connection setup times by 30% to 50% [763].
* **0-RTT Resumption:** Returning clients can send encrypted data in their very first message, before the handshake officially finishes, though this is limited to specific "idempotent" requests to avoid "replay" security risks [769, 774].
* **Mandatory Forward Secrecy:** TLS 1.3 forces ephemeral key exchange (ECDHE) [753, 769]. It completely removes static RSA key exchange [769]. This means that even if a server's long-term private key is stolen in the future, past sessions cannot be decrypted [764].
* **Handshake Privacy:** Unlike TLS 1.2, the server's certificate is completely **encrypted** during the handshake, shielding your identity from passive observers [65, 771].
* **Encrypted Client Hello (ECH):** Historically, observers could see the name of the website you were visiting via the plaintext SNI (Server Name Indication) in your initial connection [865]. ECH encrypts this entire initial message, making your browsing truly private when paired with DNS-over-HTTPS (DoH) [769, 775, 776].

---

## DNS

### 1. What is DNS?
**DNS** stands for **Domain Name System** (standardized in RFC 1034 and RFC 1035) [41]. It acts as the "phonebook" of the internet [298]. Since computers communicate using numerical **IP addresses**, but humans prefer easy-to-remember domain names (like `example.com`), DNS translates human-friendly hostnames into computer-routable IP addresses [41, 197].

### 2. The Cast of Characters
A single DNS lookup is a team effort involving several players [235]:
* **Stub Resolver:** A lightweight piece of software built into your device's operating system (such as `systemd-resolved` on Linux) [236]. It doesn't know how to navigate the internet itself; it simply packages your query, hands it to a recursive resolver, and waits for the final answer [236].
* **Recursive Resolver (DNS Resolver):** A high-throughput server (typically run by your ISP, or a public service like Cloudflare's `1.1.1.1` or Google's `8.8.8.8`) [236]. It takes full ownership of the lookup process [42]. If it doesn't have the answer cached, it goes out and searches the global hierarchy on your behalf [236].
* **Root Name Servers:** 13 logical servers (representing hundreds of physical machines globally using anycast routing) that sit at the top of the DNS tree [236]. Every resolver has a "root hints" file with their permanent IP addresses to bootstrap searches [237, 304]. They refer resolvers to the top-level domain (TLD) servers [42, 192].
* **TLD (Top-Level Domain) Servers:** Servers responsible for top-level suffixes like `.com`, `.org`, or `.edu` [192, 301, 856]. They direct the resolver to the authoritative nameservers of the specific domain [192].
* **Authoritative Name Server:** The final destination [192]. This nameserver maintains the definitive master "zone file" for the specific website and returns the actual IP address [42, 854].

### 3. Recursive vs. Iterative Query Resolution
DNS uses two different query methods to divide the labor [42]:
* **Recursive Query ("Find the answer for me"):** The client sends a query to the recursive resolver, commanding it: *"Find me the IP address, and do not return until you have it"* [190]. The resolver must take complete charge and return either the final IP or an explicit error [42, 236].
* **Iterative Query ("Point me in the right direction"):** The recursive resolver queries other servers hierarchically [883]. If a server doesn't know the answer, it responds with a **referral** (e.g., *"I don't know example.com, but you should ask the .com TLD server at this IP address"*) [192, 193]. The resolver iteratively follows these clues until it reaches the authoritative server [193, 306].

### 4. What is a DNS Zone & the Start of Authority (SOA) Record?
The global DNS namespace is partitioned into administrative boundaries called **zones** [41]. 
* **The SOA Record:** Every valid DNS zone file **must begin with exactly one Start of Authority (SOA) record** [43, 618]. The SOA record is the administrative and operational "source of truth" for the zone [43]. 

An SOA record consists of seven structural fields [43]:
1. **MNAME (Primary Name Server):** Specifies the primary authoritative nameserver holding the master copy of the zone data [869, 875].
2. **RNAME (Responsible Party):** The email of the zone administrator (formatted with a dot instead of `@`, e.g., `admin.example.com`) [824, 875].
3. **SERIAL:** A version number that incremented on every edit (triggering secondary servers to refresh) [867].
4. **REFRESH:** How often secondary servers should poll the primary for updates [876].
5. **RETRY:** How long to wait before retrying a failed zone refresh [827].
6. **EXPIRE:** The limit after which secondary servers must stop serving the zone if the primary remains unreachable [421].
7. **MINIMUM (Negative Caching TTL):** Controls how long servers cache a negative response (e.g., "this domain does not exist"), optimizing performance [431, 620].

### 5. Common DNS Record Types
Inside a zone file, DNS maps properties using various **Resource Records (RRs)** [41]:
* **A:** Maps hostnames to 32-bit IPv4 addresses [44].
* **AAAA:** Maps hostnames to 128-bit IPv6 addresses [44].
* **CNAME (Canonical Name):** Creates an alias directing one hostname to another canonical name (e.g., mapping `www.example.com` to `example.com`) [44, 802]. *Note:* CNAMEs cannot coexist with other records for the same name and **cannot** be used at the zone apex (root domain) [23, 798, 803].
* **NS (Name Server):** Delegates authority over a zone to physical nameservers [44, 302].
* **MX (Mail Exchanger):** Specifies authoritative mail servers accepting email for the domain, utilizing preference numbers [44, 817].
* **PTR (Pointer):** Maps an IP address back to its canonical hostname, used primarily for reverse DNS zones (e.g., `in-addr.arpa`) [44].
* **TXT (Text):** Holds arbitrary text, used extensively to verify domain ownership and publish email authentication policies (SPF, DKIM, DMARC) [44].

### 6. Encrypting the Last Mile: DoT vs. DoH
Plaintext DNS runs over UDP/TCP Port 53, making it vulnerable to eavesdropping, cache poisoning, and censorship [45, 46]. Modern setups use encrypted DNS transport protocols [45]:
* **DNS over TLS (DoT):** Encrypts queries using TLS directly over a dedicated **Port 853** [46]. Because it uses a dedicated port, network administrators can easily identify, monitor, and configure policy controls on it [207].
* **DNS over HTTPS (DoH):** Tunnels queries inside standard HTTPS traffic over **Port 443** [46, 198]. Because it blends in with regular web traffic, it is extremely difficult for firewalls to selectively block, inspect, or censor [46, 208].

---

## DHCP

### 1. What is DHCP?
**DHCP** stands for **Dynamic Host Configuration Protocol** (standardized in RFC 2131) [47]. It is a critical client-server protocol designed to automatically assign IP addresses, subnet masks, default gateways, and DNS server configurations to connecting devices [47, 125]. It replaces the laborious and error-prone process of manually configuring networking settings [123].

### 2. Why UDP and Not TCP?
DHCP operates entirely over the connectionless **User Datagram Protocol (UDP)** [48]. This is due to a simple logical trap:
* At the moment a device joins a network, it has **no IP address** [127].
* TCP requires a three-way handshake to establish a connection, which requires both devices to have known IP addresses [127].
* Since the client does not yet have an IP, establishing a TCP handshake is impossible [127]. UDP allows connectionless broadcasting to the entire network [127].

### 3. UDP Port Separation
DHCP uses two dedicated UDP ports:
* **Port 67:** Used by DHCP servers and relay agents to listen for incoming client requests [48, 128].
* **Port 68:** Used by DHCP clients to listen for server responses [48, 131].
Using separate ports ensures that DHCP traffic is handled in an orderly, unambiguous way without conflicts on shared systems [48, 131].

### 4. The 4-Step DORA Process
When a device joins a network, it obtains its network settings through a four-step sequence abbreviated as **DORA** [49]:

```
Client (Port 68)                           DHCP Server (Port 67)
       │                                             │
       │ ─── 1. Discover (Broadcast 255.255.255) ──> │
       │                                             │
       │ <── 2. Offer (Unicast or Broadcast) ─────── │
       │                                             │
       │ ─── 3. Request (Broadcast - selects offer) ─> │
       │                                             │
       │ <── 4. Acknowledge (Finalizes lease) ────── │
       ▼                                             ▼
```

1. **Discover (Client → Server):** The unconfigured client broadcasts a `DHCPDISCOVER` packet on the local subnet (source IP: `0.0.0.0`, destination IP: `255.255.255.255`) [49, 133]. It includes its physical hardware MAC address to identify itself [49].
2. **Offer (Server → Client):** DHCP servers on the subnet check their address pools and respond with a `DHCPOFFER` packet containing a proposed IP address, subnet mask, lease duration, default gateway, and DNS servers [49, 134].
3. **Request (Client → Server):** The client typically selects the first offer it receives and broadcasts a `DHCPREQUEST` [134]. It is still broadcast so that all other DHCP servers see which offer was chosen, allowing them to release their tentatively reserved IPs back into their pools [49, 134].
4. **Acknowledge (Server → Client):** The selected server commits the lease to its database and sends a `DHCPACK` containing the finalized settings [49, 135]. The client performs a final check (typically using an ARP probe to ensure no one else is using that IP) and officially configures its interface [49, 135].

*What if DHCP fails?* If a client cannot reach a DHCP server, it falls back to an **APIPA (Automatic Private IP Addressing)** address in the `169.254.x.x` range to maintain local subnet connectivity [50, 140].

### 5. Lease Renewals & Timers
IP configurations are not permanently owned; they are **leased** for a defined period [137]. To maintain its address, the client goes through lease renewal states [137]:
* **T1 Timer (Renewal - 50% Lease Mark):** At the 50% mark, the client enters the **RENEWING** state [50]. It sends a **unicast** `DHCPREQUEST` directly to the original server that granted the lease [50, 137].
* **T2 Timer (Rebinding - 87.5% Lease Mark):** If the original server is offline and does not respond, at the 87.5% mark, the client transitions to the **REBINDING** state [50]. It now **broadcasts** a `DHCPREQUEST` to any available server on the local subnet [50].
* **Lease Expiration (100%):** If the lease expires completely without a response, the client must immediately drop the IP address and restart the DORA process from the INIT state [50].

### 6. Subnet Traversals: Option 82
Normally, broadcast packets cannot cross routers, meaning you would need a DHCP server on every single subnet [173, 529]. To avoid this, networks use **DHCP Relay Agents** (or IP helpers) co-located on interconnecting routers or switches [47, 529].
* **Option 82 (Relay Agent Information Option):** When a relay agent intercepts a client's broadcast, it inserts Option 82 (formatted starting with `0x52`) into the DHCP packet payload before forwarding it as a unicast packet to a central DHCP server on a different subnet [51, 173].
* Option 82 adds physical attachment details:
  * **Circuit ID:** Identifies the physical port/VLAN on the switch where the client is attached [157, 527].
  * **Remote ID:** Identifies the physical relay agent or modem [527].
* The server inspects Option 82 alongside the relay gateway address (`giaddr`) to determine the correct IP subnet range to allocate from [102, 527].

---

## FTP

### 1. What is FTP?
**FTP** stands for **File Transfer Protocol** (standardized in RFC 959) [51]. It is a legacy, connection-oriented protocol designed to move files between systems over TCP networks [51]. 

### 2. Dual-Channel Architecture
Unlike modern protocols that send control instructions and file data over a single stream, FTP employs a unique **dual-channel architecture** [51]:
* **The Control Channel (Command Channel):** The client initiates a TCP connection to the server's **Port 21** [52]. All control commands (like USER, PASS, PORT, PASV) and server replies travel exclusively over this path [18, 52]. No file contents travel here [31].
* **The Data Channel:** A separate TCP connection opened on demand solely for transmitting file bytes and directory listings [29, 31]. The way this channel is established depends on whether the FTP connection is configured to use **Active** or **Passive** mode [52].

### 3. Active Mode FTP (using PORT)
Active mode was the original design from 1971 [33, 34].

```
Client                                                   Server
  │ ─── 1. Control connection to Server Port 21 ─────────> │
  │ ─── 2. Client PORT command (Client IP + Port N+1) ───> │
  │                                                        │
  │ <── 3. Server initiates Data connection from Port 20 ── │ (Server actively connects!)
  ▼                                                        ▼
```

1. **Control Connection:** The client establishes a control connection to the server's Port 21 from a random unprivileged port ($N > 1024$) [14].
2. **PORT Command:** When a data transfer is required, the client opens a local listening socket on port $N+1$ and sends a `PORT` command containing its own IP address and the port number ($N+1$) over the control channel to the server [14, 34].
3. **Server Connects Back:** The server **actively** initiates a fresh TCP connection from its local **Port 20** to the client's listening port ($N+1$) [14, 52].

*The Problem:* Active mode is highly problematic for client-side firewalls and NAT routers [52]. Client firewalls view the server's inbound connection attempt as an unsolicited intrusion from the outside and block it by default [15, 35]. NAT traversal also fails because the client's internal private IP is embedded inside the `PORT` command payload, which the external server cannot route to [52].

### 4. Passive Mode FTP (using PASV)
Passive mode is the modern default, created to solve client-side firewall issues [30, 33].

```
Client                                                   Server
  │ ─── 1. Control connection to Server Port 21 ─────────> │
  │ ─── 2. Client sends PASV command ────────────────────> │
  │ <── 3. Server PASV reply (Server IP + Port P) ─────── │
  │                                                        │
  │ ─── 4. Client initiates Data connection to Port P ────> │ (Client initiates!)
  ▼                                                        ▼
```

1. **Control Connection:** The client connects to the server's Port 21 [14].
2. **PASV Command:** The client sends the `PASV` command over the control channel [19, 30].
3. **PASV Reply:** The server opens a random ephemeral listening port ($P > 1024$) on its end and replies to the client with this port number [19, 30].
4. **Client Connects:** The client **initiates** the data connection, connecting from its port $N+1$ directly to the server's specified port $P$ [19, 52].

*The Trade-off:* While passive mode is seamless for clients (since both connections are outbound), it shifts the complexity to the server administrator [22, 52]. The server must allow inbound connections to a broad range of random, high-numbered ephemeral ports, requiring complicated firewall rules [19, 22]. Additionally, NAT traversal breaks if the server returns its internal private IP address in the `PASV` reply [52].

---

## SMTP

### 1. What is SMTP?
**SMTP** stands for **Simple Mail Transfer Protocol** (standardized under RFC 5321) [53]. It is the universal standard protocol used to push, route, and deliver email messages across TCP networks [53, 73]. It functions as a stateful, text-based request-response protocol [53].

### 2. ESMTP (Extended SMTP)
Modern email infrastructure uses **Extended SMTP (ESMTP)** [75]. ESMTP clients initiate the session using the `EHLO` command instead of the legacy `HELO` [75, 78]. When a server receives `EHLO`, it responds with a list of supported capabilities and service extensions, such as `AUTH` (for secure credentials), `STARTTLS` (for transport encryption), and `SIZE` (declaring maximum message limits) [75].

### 3. Step-by-Step Mail Transaction Sequence
A standard SMTP mail transaction moves through a strict linear sequence of client text commands and numeric server response codes [53, 54]:
1. **Connection Greeting:** The client establishes a TCP connection [54]. The server greets it with a `220` ready code [54]. The client introduces itself with `EHLO clientdomain.com` [54].
2. **Sender Identification:** The client initiates the transaction with `MAIL FROM:<sender@domain.com>` [54]. This sets the **Envelope Sender** (or Return-Path), where non-delivery bounce reports are sent [82]. The server responds with `250 OK` [54].
3. **Recipient Identification:** The client sends `RCPT TO:<recipient@domain.com>` [54]. For multiple recipients, a separate `RCPT TO` command is sent for each [54, 83]. This allows the server to accept some valid local mailboxes with a `250 OK` while rejecting non-existent local addresses immediately with a `550 User Not Found` error [54].
4. **Data Stream:** The client sends the `DATA` command [54]. The server replies with `354 Start Mail Input` [54, 92]. The client then streams the mail headers (From, To, Subject) and the message body [54]. The client terminates the stream by sending a **single dot on its own line (`\r\n.\r\n`)** [54, 56]. The server accepts the mail with `250 OK` [54].
5. **Termination:** The client sends `QUIT` [54]. The server responds with `221 Bye` and terminates the connection [54].

### 4. SMTP Ports & Encryption
* **Port 25 (SMTP Relay):** The original standard port, used strictly for server-to-server mail relaying [55, 94]. Most ISPs and cloud hosting providers block outbound Port 25 by default to prevent spambots from sending massive amounts of unauthenticated spam [94].
* **Port 587 (SMTP Submission):** The modern, recommended port for secure client-to-server mail submission (e.g., configuring an email app or custom API) [55, 94, 601]. It mandates SMTP authentication (`AUTH`) and upgrades the plain-text connection to secure TLS encryption via the `STARTTLS` command [55, 86, 94].
* **Port 465 (Implicit SMTPS):** A legacy, non-RFC compliant option where a secure TLS/SSL tunnel is established immediately upon connection, before any SMTP text commands can be exchanged [55, 94, 604].
* **Port 2525:** A non-standard alternative provided by ESPs to bypass Port 25/587 blocks in highly restricted networks [55, 267].

### 5. The Email Authentication Triad (SPF, DKIM, and DMARC)
Because legacy SMTP has no built-in mechanism to verify who is actually sending an email (allowing anyone to write a fake address in the From header), the modern internet relies on three DNS-based security standards to protect domain reputations and stop phishing [56, 179]:

| Standard | How it Works | Primary Purpose | Key Limitation |
| :--- | :--- | :--- | :--- |
| **SPF** (Sender Policy Framework) | The domain owner publishes a list of authorized mail-sending IP addresses in a DNS **TXT** record [56]. Receiving servers look up this record to verify the physical IP of the sending mail server [56]. | Validates authorized sending servers [185, 644]. | Breaks easily when emails are forwarded [185, 644]. |
| **DKIM** (DomainKeys Identified Mail) | The sending server adds a digital signature (`DKIM-Signature` header) to outgoing emails, created using the domain's private key [57, 60]. The receiver verifies it using the public key published in the domain's DNS [57, 630]. | Ensures email content integrity (confirms it wasn't altered in transit) [57, 180, 185]. | Does not verify the visible "From" address [644]. |
| **DMARC** (Domain-based Message Authentication...) | Combines SPF and DKIM [179]. It mandates **alignment** (the domain in the visible "From" header must match the domains authenticated by SPF and/or DKIM) [58, 643]. It publishes policies (none, quarantine, reject) instructing receivers what to do with failed mail, and enables reporting [58, 182, 643]. | Enforces alignment, prevents visible "From" spoofing, provides XML reports [58, 182]. | Only as strong as your underlying SPF and DKIM setups [185]. |

---

## SSH

### 1. What is SSH?
**SSH** stands for **Secure Shell** (standardized under RFC 4251) [59]. It is a cryptographic network protocol used to run interactive terminals, execute commands, and securely transfer files over unsecured TCP/IP networks [59, 702, 706]. Listening traditionally on **Port 22**, it was designed to replace insecure, plaintext utilities like Telnet, rlogin, and rsh [59, 65, 702].

### 2. Three-Layer Protocol Architecture
SSH-2 is engineered with three independent, highly cohesive, and layered components [59, 60]:
* **The Transport Layer Protocol (RFC 4253):** Sits directly on top of raw TCP to establish a cryptographically secure, encrypted tunnel [60]. It handles server authentication, algorithm negotiation, symmetric encryption, and integrity [60]. It enforces a **key re-exchange** (typically after 1 GB of data has passed or 1 hour of elapsed time) to limit the amount of ciphertext generated under a single session key [60].
* **The User Authentication Protocol (RFC 4252):** Handles client-side user authentication over the established secure transport tunnel [60]. It supports methods like standard passwords, public keys (e.g. RSA, ECDSA, or Ed25519), keyboard-interactive prompts, and GSSAPI (Kerberos) [60, 561, 577].
* **The Connection Protocol (RFC 4254):** Multiplexes the single secure transport tunnel into multiple independent logical data channels [60]. These channels are used for interactive shells, exec commands, and port forwarding tunnels [60, 709].

### 3. The Cryptographic Handshake & Key Exchange
When a client connects to an SSH server, a multi-stage cryptographic handshake occurs [61, 68]:
1. **Protocol Version Exchange:** Both parties exchange ASCII strings defining their protocol versions and software releases [61].
2. **Algorithm Negotiation (`SSH_MSG_KEXINIT`):** Both parties broadcast a list of supported algorithms in order of preference [61]. They negotiate key exchange algorithms (e.g., Curve25519 or ECDH), server host key algorithms (e.g., Ed25519 or RSA-PSS), symmetric encryption (e.g., AES-GCM or ChaCha20-Poly1305), and message integrity verification [61].
3. **Key Exchange Execution (ECDHE):** Modern configurations leverage **Ephemeral Elliptic Curve Diffie-Hellman (ECDHE)** to negotiate a session key while guaranteeing perfect forward secrecy [61].
   * The client speculatively generates an ephemeral keypair and sends its public key to the server using the `SSH_MSG_KEX_ECDH_INIT` message [61].
   * The server generates its own ephemeral keypair and mathematically calculates the shared secret key ($K$) [61].
   * The server constructs the **Exchange Hash ($H$)**, which acts as a unique cryptographic fingerprint of the entire handshake transcript [61, 62].
   * The server signs the Exchange Hash ($H$) using its private host key and sends the signature back to the client [61]. The client verifies the signature using the server's known public key to prove server identity [61, 539].

Once completed, symmetric encryption is enabled immediately, and the client securely authenticates its username and credentials over the encrypted channel [60].

### 4. Multiplexed Tunneling (Port Forwarding)
One of SSH's most powerful capabilities is multiplexed port forwarding, which lets you securely route other network protocols (like HTTP, databases, or SMTP) through an encrypted SSH tunnel [536, 551]:

| Forwarding Type | OpenSSH Flag | How It Routes Traffic | Primary Use Case |
| :--- | :--- | :--- | :--- |
| **Local Port Forwarding** | `-L` | Client host opens a local listening port [63, 676]. Any connections made to this port are encrypted, sent through the SSH tunnel to the server, and forwarded by the server to a specified static target host and port [63]. | Accessing a remote database or admin console protected behind an enterprise firewall [63]. |
| **Remote Port Forwarding** | `-R` | Opens a listening port on the remote SSH server host [63, 673]. Connections made to this remote port are routed back through the existing SSH tunnel to the client machine, which forwards them to a local destination [63]. | Exposing a local development web server on your laptop to the public internet [63, 672]. |
| **Dynamic Port Forwarding** | `-D` | Client opens a local port acting as a SOCKS proxy [63, 674]. Application traffic routed to this proxy is dynamically forwarded through the tunnel, where the SSH server performs DNS resolution and initiates connections dynamically [63]. | Secure, encrypted web browsing over public Wi-Fi or reaching multiple internal VPC microservices [63, 674]. |
