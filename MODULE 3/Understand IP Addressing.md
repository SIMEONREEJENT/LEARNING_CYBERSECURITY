# 4. Understand IP Addressing

This guide serves as a comprehensive, beginner-friendly reference for understanding network addressing. It covers the architectural design, mathematics, and operational mechanisms of both IPv4 and IPv6, as well as the transition mechanisms used to bridge them.

---

## Table of Contents
1. [a. IPv4](#a-ipv4)
   - [Address Structure and Dot-Decimal Notation](#address-structure-and-dot-decimal-notation)
   - [The Network vs. Host Analogy](#the-network-vs-host-analogy)
   - [Historical Classful Addressing (Classes A, B, C, D, E)](#historical-classful-addressing-classes-a-b-c-d-e)
2. [b. IPv6](#b-ipv6)
   - [The Motivation Behind IPv6](#the-motivation-behind-ipv6)
   - [Structure and Hexadecimal Representation](#structure-and-hexadecimal-representation)
   - [Address Compression Rules](#address-compression-rules)
   - [IPv4 vs. IPv6 Header Comparison](#ipv4-vs-ipv6-header-comparison)
3. [c. Public IPs](#c-public-ips)
4. [d. Private IPs](#d-private-ips)
   - [IPv4 Private Addressing (RFC 1918)](#ipv4-private-addressing-rfc-1918)
   - [IPv6 Private Addressing (Unique Local Addresses - ULA)](#ipv6-private-addressing-unique-local-addresses---ula)
   - [Network Address Translation (NAT) & PAT](#network-address-translation-nat--pat)
5. [e. Subnetting fundamentals](#e-subnetting-fundamentals)
   - [What is Subnetting and Why Do We Use It?](#what-is-subnetting-and-why-do-we-use-it)
   - [Subnet Masks and CIDR Notation](#subnet-masks-and-cidr-notation)
   - [The Network and Broadcast Address Rule](#the-network-and-broadcast-address-rule)
   - [Variable Length Subnet Masking (VLSM)](#variable-length-subnet-masking-vlsm)
   - [The Magic Number Method (Step-by-Step Walkthrough)](#the-magic-number-method-step-by-step-walkthrough)

---

## a. IPv4

**Internet Protocol Version 4 (IPv4)** is a connectionless network layer protocol (OSI Layer 3) defined in **RFC 791** that enables communication between network hosts by carrying data in packets [1]. Every host on an IPv4 network is assigned a numerical address that acts similarly to a postal address on an envelope, ensuring data reaches the correct destination [1].

### Address Structure and Dot-Decimal Notation
An IPv4 address is exactly **32 bits** long (composed of 1s and 0s) [1]. For human readability, these 32 bits are divided into four 8-bit groups called **octets** (or bytes) separated by periods—a format known as **dotted-decimal notation** [1]. 

* **Binary Representation:** `11000000.10101000.00000000.00000101` [1]
* **Decimal Equivalent:** `192.168.0.5` [1]

Because each octet is 8 bits, its value can range from **0 to 255** [1]. The total number of unique addresses available in IPv4 is \\(2^{32}\\), which equals **4,294,967,296** (approximately 4.3 billion) addresses [1, 101].

### The Network vs. Host Analogy
Every IP address is divided into two logical sections [214]:
1. **The Network Portion:** Identifies the specific network the device belongs to [214].
2. **The Host Portion:** Identifies the specific device within that network [214, 218].

**The Analogy:** Think of an IP address as a street address (e.g., *123 Main Street*). "Main Street" represents the **network portion** (the shared street that connects all the houses), while "123" represents the **host portion** (the unique house number on that street).

### Historical Classful Addressing (Classes A, B, C, D, E)
Originally, IPv4 addresses were allocated using a rigid, bit-pattern system called **classful addressing** [93, 214]. The class of an address was defined by its leading bits [214]:

| Class | Leading Bits | First Octet Range | Default Subnet Mask | Typical Use / Allocation |
| :---: | :---: | :--- | :--- | :--- |
| **A** | `0` [215] | `0.0.0.0` to `127.255.255.255` [215] | `255.0.0.0` (/8) [4] | Massive networks (16M+ hosts) [4] |
| **B** | `10` [215] | `128.0.0.0` to `191.255.255.255` [215] | `255.255.0.0` (/16) [4] | Medium networks (65k hosts) [4] |
| **C** | `110` [215] | `192.0.0.0` to `223.255.255.255` [215] | `255.255.255.0` (/24) [4] | Small networks (254 hosts) [4] |
| **D** | `1110` [215] | `224.0.0.0` to `239.255.255.255` [215] | *N/A* [215] | Multicast groups (one-to-many) [215, 216] |
| **E** | `1111` [215] | `240.0.0.0` to `255.255.255.255` [215] | *N/A* [215] | Experimental / Reserved [215, 216] |

#### Reserved Class A Ranges
Within Class A, two blocks are carved out and are not available for general host assignment [215]:
* **`0.0.0.0/8`:** The "this network" address, used as a source address before a host acquires its own IP [215].
* **`127.0.0.0/8`:** The loopback range, reserved for hosts to test their own local network stack (typically using `127.0.0.1`) [112, 215].

#### Why Classful Addressing is Deprecated
This system was highly inefficient [94]. For example, an organization needing 300 IP addresses was too large for a Class C block (254 hosts) but had to be assigned a Class B block (65,536 hosts), wasting over 65,000 addresses [4, 234]. Classful addressing was officially deprecated in 1993 and replaced by **Classless Inter-Domain Routing (CIDR)** under **RFC 4632**, which allows arbitrary prefix lengths to match actual requirements [93, 95, 216].

---

## b. IPv6

### The Motivation Behind IPv6
Because of the explosive growth of internet-connected devices, smartphones, smart grids, and IoT, the 4.3 billion addresses provided by IPv4 were completely exhausted [43, 283]. **Internet Protocol Version 6 (IPv6)**, standardized in **RFC 8200**, was developed to solve address exhaustion and optimize routing [126, 283].

### Structure and Hexadecimal Representation
IPv6 uses a **128-bit address space** [428]. This provides \\(2^{128}\\) (approximately \\(3.4 \times 10^{38}\\)) unique addresses, meaning we have more than enough addresses to satisfy global demand for generations [102, 283].

An IPv6 address is written as eight groups of four **hexadecimal** digits (each group represents 16 bits, often called a **hextet** or segment), separated by colons [297]:
`2001:0db8:85a3:0000:0000:8a2e:0370:7334` [297]

### Address Compression Rules
To make long IPv6 addresses easier to write and read, you can apply two strictly defined compression rules [246, 297]:

1. **Omit Leading Zeros:** Within any 16-bit segment, you can drop any zeros that appear at the beginning of the group [246].
   * *Example:* `:0db8:` becomes `:db8:`, and `:0000:` becomes `:0:` [246, 297].
2. **The Double Colon (::) Rule:** A contiguous sequence of all-zero groups can be replaced with a single double colon `::` [229, 246].
   * *Critical Restriction:* You can **only use `::` once** in an address [246, 297]. If used twice, there is no way to mathematically determine the size of each omitted block of zeros, making the address ambiguous [246].

#### Compression Walkthrough:
* **Fully Expanded Address:** `2001:0db8:0000:0000:0008:0800:200c:417a` [429]
* **Rule 1 (Omit leading zeros):** `2001:db8:0:0:8:800:200c:417a` [429]
* **Rule 2 (Double Colon):** `2001:db8::8:800:200c:417a` [429]

### IPv4 vs. IPv6 Header Comparison
The IPv6 header was redesigned from the ground up to be simpler and faster for routers to process [112, 150]. Instead of a variable-length header, IPv6 uses a streamlined, **fixed 40-byte header** [112, 150].

| Header Feature | IPv4 Header | IPv6 Header | Architectural Reason for Change |
| :--- | :--- | :--- | :--- |
| **Address Size** | 32 bits (4 bytes) [112] | 128 bits (16 bytes) [112] | Solves global address exhaustion [43, 283]. |
| **Header Size** | **Variable** (20 to 60 bytes) [112] | **Fixed** (40 bytes) [112] | Fixed size allows routers to process packets much faster in hardware without parsing options [112, 150]. |
| **Header Checksum** | Present [245] | **Removed** [150] | Error checking is handled at Layer 2 (Ethernet) and Layer 4 (TCP/UDP), eliminating redundant per-hop checksum updates [112, 150]. |
| **Broadcast Support** | Supported [247] | **None** (Replaced by Multicast) [118, 247] | Broadcasts interrupt every device on the segment. IPv6 replaces broadcasts with targeted multicast groups (such as `ff02::1` for all nodes) [118, 292]. |
| **Fragmentation Fields** | Present in main header [150] | **Removed** (Moved to Extension Headers) [112, 150] | In IPv6, fragmentation can only be performed by the source node, not by transit routers, speeding up the forwarding path [111, 150]. |
| **Options Field** | Present (variable length) [112, 245] | **Removed** (Replaced by Extension Headers) [112, 150] | Optional data is placed in separate, chainable extension headers only when needed [112, 150]. |

---

## c. Public IPs

* **Definition:** Globally unique IP addresses that are registered and routed on the public internet [11, 404].
* **Allocation:** Regulated internationally by the Internet Assigned Numbers Authority (IANA) and regional registries (like ARIN or RIPE) [10, 260]. No two devices connected directly to the internet can share the same public IP address [11].

---

## d. Private IPs

Because of IPv4 address scarcity, certain address spaces are set aside for internal use within private networks (like home LANs, offices, and enterprise datacentres) [374, 584]. These are called **Private IP addresses**.

* **The Routing Rule:** Private IP addresses **must never** be routed across the public internet [213, 225]. Internet border routers are explicitly configured to drop any transit packets containing a private source or destination IP on their WAN interfaces [117, 158].

### IPv4 Private Address Ranges (RFC 1918)
Under **RFC 1918**, three blocks of IPv4 address space are reserved for private networks [157, 213]:

| Private Block | Address Range | CIDR Prefix | Total Addresses | Default Mask |
| :---: | :--- | :--- | :--- | :--- |
| **Class A** | `10.0.0.0` to `10.255.255.255` [11] | `10.0.0.0/8` [11] | 16,777,216 [4] | `255.0.0.0` [4] |
| **Class B** | `172.16.0.0` to `172.31.255.255` [11, 158] | `172.16.0.0/12` [11] | 1,048,576 [4] | `255.240.0.0` [4] |
| **Class C** | `192.168.0.0` to `192.168.255.255` [11] | `192.168.0.0/16` [11] | 65,536 [4] | `255.255.0.0` [4] |

Any organization can reuse these private ranges inside their local networks without notifying or paying any authority [10, 11].

### IPv6 Private Addressing (Unique Local Addresses - ULA)
In IPv6, traditional flat private addresses are replaced by **Unique Local Addresses (ULA)** defined in **RFC 4193** [378, 528].
* **Address Range:** Starts with prefix **`fc00::/7`** [420, 528].
* **Locally Assigned Block (`fd00::/8`):** In current practice, the L-bit is set to 1, meaning all locally assigned private prefixes start with **`fd00::/8`** [378, 420, 529].
* **Global ID Uniqueness:** When deploying ULA, organizations generate a **40-bit random string** to append to `fd00::/8`, creating a unique `/48` prefix [421, 529]. Because this 40-bit string is generated randomly, the probability of two private networks having the same address range is virtually zero [422, 528]. This completely eliminates IP address conflicts when two corporate networks merge or connect via VPN [418, 422].

### Network Address Translation (NAT) & PAT
Since private IP addresses cannot travel across the internet, a gateway device (like a router or firewall) uses **Network Address Translation (NAT)** to rewrite the IP headers as they cross network borders [124, 214].

```
               +--------------------------------------+
               |          The Public Internet         |
               |          (Globally Routable)         |
               +-------------------+------------------+
                                   | [Public IP: 138.76.28.4]
                             [NAT Router] (Translates Private to Public)
                                   | [Private IP: 10.0.0.1]
               +-------------------+------------------+
               |         Your Local Private LAN       |
               |         (Non-Globally Routable)      |
               |   [Device A: 10.0.0.10]              |
               +--------------------------------------+
```

* **Static NAT (One-to-One):** Maps a single private address to a single public address in a permanent relationship [12, 13]. Typically used for internal servers (like web or mail servers) that must be directly reachable from the outside [13, 479].
* **Dynamic NAT (Many-to-Many):** Maps private IP addresses to a pool of registered public IP addresses on an as-needed basis [12, 404].
* **Port Address Translation (PAT) / NAT Overload (Many-to-One):** Maps **thousands of private IP devices** to **a single public IP address** [371, 402, 406]. It accomplishes this by appending a unique **Layer 4 port number** (TCP or UDP) to each connection, allowing the router to keep track of which return packet belongs to which specific internal host [16, 371, 406].

---

## e. Subnetting fundamentals

### What is Subnetting and Why Do We Use It?
**Subnetting** is the logical division of a large IP network block into smaller, isolated subnetworks [2]. We subnet networks for three primary reasons [3]:
1. **Limiting Broadcast Storms:** On a flat network, broadcast frames (like ARP requests) hit every connected machine [275, 560]. This uses up processing power and bandwidth [292]. Subnets contain broadcasts within smaller boundaries [560].
2. **Security Segmentation:** Separating critical segments (such as database servers) from general segments (such as guest Wi-Fi) so firewall rules can restrict access [3, 231].
3. **Optimizing Address Allocation:** Sizing networks precisely to match host requirements, preventing IP waste [3, 213].

### Subnet Masks and CIDR Notation
To determine where the "network portion" of an IP address ends and the "host portion" begins, a **subnet mask** is applied [122]. 
* A subnet mask is a 32-bit number where continuous binary **1s represent the network portion** and continuous **0s represent the host portion** [98]. Under **RFC 4632**, modern subnet masks must have contiguous 1-bits followed by contiguous 0-bits [98].
* **CIDR Notation:** A compact shorthand that appends a slash (`/`) followed by the count of consecutive leading 1-bits in the subnet mask [94, 97].

```text
CIDR Prefix:  /24
Binary Mask:  11111111 . 11111111 . 11111111 . 00000000
Decimal Mask: 255      . 255      . 255      . 0
```

### The "Network" and "Broadcast" Address Rule
In any standard IPv4 subnet, two IP addresses are strictly reserved and **cannot** be assigned to a host device [101]:
1. **The Network Address (First IP):** Where all host bits are set to binary **0** [220]. It is used to identify the subnet itself [101].
2. **The Broadcast Address (Last IP):** Where all host bits are set to binary **1** [220]. A packet sent here is received by every device in that subnet [101].

* **Formula for Usable Hosts:** 
  \\[\text{Usable Hosts} = 2^{\text{host bits}} - 2\\] [101]
  * *Example (/24 network):* There are 8 host bits remaining (32 - 24 = 8) [3]. This gives \\(2^8 = 256\\) total addresses [4]. Subtract 2, and you get exactly **254 usable host addresses** [4, 101].
  * *(Note: RFC 3021 defines an exception for /31 point-to-point links, where both addresses are usable because no broadcast is needed on a simple router-to-router connection) [101, 102].*

### Variable Length Subnet Masking (VLSM)
Historically, traditional subnetting required all subnets of a network block to be the same size, leading to massive address waste [94, 96]. **Variable Length Subnet Masking (VLSM)** is "subnetting your subnets" [2]. It allows you to carve a parent block into subnets of different, power-of-two sizes to perfectly fit your actual host requirements [2, 96, 553].

* **The Core Rule of VLSM:** To prevent address ranges from overlapping, you must calculate and allocate subnets in **descending order of size** (largest host requirement first, down to the smallest point-to-point links) [523, 534, 536].

---

### The Magic Number Method (Step-by-Step Walkthrough)
The **Magic Number Method** is a popular mental shortcut that simplifies subnetting calculations by avoiding tedious decimal-to-binary conversions [193, 339, 341].

#### The Core Formulas:
1. **Interesting Octet:** The octet in the subnet mask containing a value other than `255` or `0` (where the network-to-host boundary falls) [194, 195].
2. **Magic Number (Block Size):** 
   \\[\text{Magic Number} = 256 - \text{Interesting Mask Octet}\\] [123, 194]

---

#### Step-by-Step Practical Example:
Let's find the network boundary, usable host range, and broadcast address for:
**`172.20.50.100 /21`** [194]

##### Step 1: Find the Subnet Mask and Interesting Octet
A `/21` prefix means the first 21 bits of the mask are 1s [194]:
* **Binary Mask:** `11111111.11111111.11111000.00000000`
* **Dotted-Decimal Mask:** `255.255.248.0` [4, 194]

The **interesting octet** is the **3rd octet** because its value is `248` (neither 255 nor 0) [194].

##### Step 2: Calculate the Magic Number (Block Size)
\\[\text{Magic Number} = 256 - 248 = 8\\] [194]
This means our networks jump in increments (blocks) of **8** in the 3rd octet [194].

##### Step 3: Find the Subnet Address (Subnet ID)
Look at the 3rd octet of our IP address (`172.20.50.100`), which is `50` [194].
List the multiples of our Magic Number (8):
`0, 8, 16, 24, 32, 40, 48, 56, 64...` [194]

Identify the largest multiple of 8 that is **less than or equal to** `50`. That number is **`48`** [194].
* **Subnet Address:** **`172.20.48.0`** [194]
*(Note: All octets to the right of the interesting octet are zeroed out in the network address) [346].*

##### Step 4: Calculate the Broadcast Address
The next subnet block starts at `48 + 8 = 56` in the 3rd octet (which is `172.20.56.0`) [194].
Our broadcast address is exactly one IP address *before* the next subnet starts [194, 339]:
* **Broadcast Address:** **`172.20.55.255`** [194]
*(Note: All octets to the right of the interesting octet are set to 255 in the broadcast address) [195, 366].*

##### Step 5: Determine the Usable Host Range
The usable host range falls cleanly between the Subnet Address (+1) and the Broadcast Address (-1) [356]:
* **First Usable Host Address:** **`172.20.48.1`**
* **Last Usable Host Address:** **`172.20.55.254`**

---

*This guide was compiled from verified standards and official RFC documents to serve as an accurate reference for networks engineering practices [21, 401, 416, 426].*
