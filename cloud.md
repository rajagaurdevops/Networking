# The OSI 7-Layer Model: A Complete Real-World Reference

The Open Systems Interconnection (OSI) model is a conceptual framework that standardizes the network communication functions of a telecommunication or computing system.

While modern networks run on the simplified **TCP/IP stack**, the OSI model remains the gold standard for learning, system design, and debugging.

---

## Table of Contents

1. [The 7 Layers of OSI vs. TCP/IP Stack](#the-7-layers-of-osi-vs-tcpip-stack)
2. [The TCP/IP Model and Merged Layers](#the-tcpip-model-and-merged-layers)
3. [Real-World Scenario: Typing https://google.com](#real-world-scenario-typing-httpsgooglecom)
4. [Layer 7 — Application Layer](#layer-7--application-layer)
   - [DNS Resolution: Under the Hood](#dns-resolution-under-the-hood)
   - [DNS Hijacking: Intercepting the Traffic](#dns-hijacking-intercepting-the-traffic)
   - [HTTP Methods and Status Codes](#http-methods-and-status-codes)
5. [Layer 6 — Presentation Layer](#layer-6--presentation-layer)
6. [Layer 5 — Session Layer](#layer-5--session-layer)
7. [Layer 4 — Transport Layer](#layer-4--transport-layer)
   - [Ports, Sockets, and IP Addresses](#ports-sockets-and-ip-addresses-the-multiplexing-magic)
   - [TCP vs. UDP](#tcp-vs-udp-key-differences)
   - [The TCP Header Structure](#the-tcp-header-structure)
   - [TCP Three-Way Handshake](#tcp-three-way-handshake-establishing-a-connection)
   - [TCP Connection Termination](#tcp-connection-termination-the-four-way-handshake)
8. [Layer 3 — Network Layer](#layer-3--network-layer)
   - [The IPv4 Header Structure](#the-ipv4-header-structure)
   - [IPv6: The Next-Generation Protocol](#ipv6-the-next-generation-protocol)
   - [Subnetting and CIDR Notation](#subnetting-and-cidr-notation)
   - [NAT (Network Address Translation)](#nat-network-address-translation)
9. [Layer 2 — Data Link Layer](#layer-2--data-link-layer)
   - [ARP: Mapping IPs to MACs](#arp-address-resolution-protocol-mapping-ips-to-macs)
   - [Switching, VLANs, and Broadcast Domains](#switching-vlans-and-broadcast-domains)
   - [DHCP DORA](#dhcp-dora-obtaining-an-ip-address)
10. [Layer 1 — Physical Layer](#layer-1--physical-layer)
11. [Summary of the Communication Flow](#summary-of-the-communication-flow)
12. [Load Balancing: Layer 4 vs. Layer 7](#load-balancing-layer-4-vs-layer-7)
13. [Firewalls and Security Devices by Layer](#firewalls-and-security-devices-by-layer)
14. [VPNs and Tunneling](#vpns-and-tunneling)
15. [Common Network Attacks by Layer](#common-network-attacks-by-layer)
16. [Common Port Numbers Reference](#common-port-numbers-reference)
17. [DevOps Reference: Troubleshooting Tools by Layer](#devops-reference-troubleshooting-tools-by-layer)
18. [Glossary of Key Terms](#glossary-of-key-terms)

---

## The 7 Layers of OSI vs. TCP/IP Stack

Here is how the seven OSI layers map to the modern TCP/IP model:

| OSI Layer | Name | Common Protocols | TCP/IP Layer Equivalent | Unit of Data (PDU) |
| :--- | :--- | :--- | :--- | :--- |
| **L7** | Application | HTTP, HTTPS, DNS, SSH, SMTP, FTP | Application | Data |
| **L6** | Presentation | TLS, SSL, ASCII, JPEG, GIF | Application | Data |
| **L5** | Session | NetBIOS, Sockets, RPC | Application | Data |
| **L4** | Transport | TCP, UDP, QUIC | Transport | Segment / Datagram |
| **L3** | Network | IPv4, IPv6, ICMP, IPsec | Network / Internet | Packet |
| **L2** | Data Link | MAC, Ethernet, Wi-Fi (802.11), ARP, VLAN (802.1Q) | Network Access | Frame |
| **L1** | Physical | Fiber, Coax, Ethernet cables, Radio, Bits | Network Access | Bits |

> [!NOTE]
> A handy mnemonic for remembering the layers top-down (L7→L1) is **"All People Seem To Need Data Processing."**

---

## The TCP/IP Model and Merged Layers

The **TCP/IP model** (also known as the Internet Protocol Suite) is a 4-layer model designed specifically for real-world protocol implementations. Unlike the theoretical 7-layer OSI model, it consolidates functionalities to match how modern operating systems and hardware actually process network traffic.

```
       OSI 7-LAYER MODEL                       TCP/IP 4-LAYER MODEL
    ┌──────────────────────────┐
L7  │ Application              │┐
    ├──────────────────────────┤│
L6  │ Presentation             │├──────────>  │ Application Layer        │
    ├──────────────────────────┤│
L5  │ Session                  │┘
    ├──────────────────────────┤
L4  │ Transport                │───────────>  │ Transport Layer          │
    ├──────────────────────────┤
L3  │ Network                  │───────────>  │ Internet Layer           │
    ├──────────────────────────┤
L2  │ Data Link                │┐
    ├──────────────────────────┤├──────────>  │ Network Access Layer     │
L1  │ Physical                 │┘             │ (Link Layer)             │
    └──────────────────────────┘
```

---

## Real-World Scenario: Typing https://google.com

Let's trace what happens when you open your browser, type `https://google.com`, and press **Enter**.

```
    Your Laptop                          Home Router                          Internet                           Google
[ 192.168.1.10 ] ═══════════════> [ Default Gateway ] ═══════════════> [ Public Routing ] ═══════════════> [ Google Server ]
```

As the request moves from your browser down to the physical cable, and then back up on Google's server, it flows through all seven layers.

---

## Layer 7 — Application Layer

### What It Does
This layer is closest to the end user. It provides network services directly to client applications (like web browsers or email clients) and defines the protocols they use to communicate.

### In Our Example
1. Your web browser initiates the request for `https://google.com`.
2. It first uses **DNS** (Domain Name System) to resolve the hostname `google.com` to its corresponding IP address.
3. Once the IP is known, the browser crafts an **HTTPS** request (`GET / HTTP/1.1`) to retrieve the homepage.

> [!NOTE]
> L7 is where most business logic bugs reside. When troubleshooting, if you can `curl` the endpoint but get a `4xx` or `5xx` status code, or a malformed JSON payload, your lower-level connectivity is fully working. The issue lies entirely at the application layer.

### DNS Resolution: Under the Hood

Before your browser can send an HTTPS request to `https://google.com`, it must translate the human-readable domain name `google.com` into a machine-readable IP address. This translation is performed by the **Domain Name System (DNS)**.

#### The Step-by-Step DNS Resolution Process

If the IP address is not already cached, the DNS lookup traverses a hierarchical system of servers:

```
[ Browser ] ──(1. Check Local Cache)──> [ Browser/OS/Router Cache ]
     │                                               │ (Miss)
     │ (IP Found)                                    ▼
     └───────────────────────────────> [ DNS Recursive Resolver ] (e.g., 8.8.8.8)
                                                     │
                                   ┌─────────────────┼─────────────────┐
                                   │ (2. Query)      │ (4. Query)      │ (6. Query)
                                   ▼                 ▼                 ▼
                         ┌─────────────────┐ ┌───────────────┐ ┌───────────────┐
                         │ Root Server (.) │ │  TLD Server   │ │ Authoritative │
                         │                 │ │    (.com)     │ │  Server (IP)  │
                         └────────┬────────┘ └───────┬───────┘ └───────┬───────┘
                                  │                  │                 │
                                  └─(3. Point TLD)───┴─(5. Point Auth)─┴─(7. Return IP)
```

1. **Step 1: Check Local Cache**
   * **Browser Cache:** The browser first checks its own record history for `google.com`.
   * **Operating System Cache:** If the browser doesn't have it, it makes an OS system call. The OS checks its own local DNS resolver cache and the local hosts file (e.g., `/etc/hosts` on macOS/Linux or `C:\Windows\System32\drivers\etc\hosts` on Windows).
   * **Router Cache:** If still not found, the query is forwarded to the local router, which checks its local cache.
2. **Step 2: Query the Recursive Resolver**
   * If the local checks fail, the client sends a query to a **DNS Recursive Resolver** (usually provided by your ISP, or public resolvers like Google's `8.8.8.8` or Cloudflare's `1.1.1.1`).
   * If the resolver has the IP cached from a recent lookup, it returns it immediately. If not, it begins searching the public internet hierarchy.
3. **Step 3: Query the Root Nameserver (`.`)**
   * The resolver sends a query to one of the 13 logical **Root Nameservers** scattered globally.
   * The Root server does not know the IP for `google.com`. Instead, it reads the Top-Level Domain (TLD) suffix (`.com`) and returns the IP address of the **TLD Nameserver** responsible for `.com` domains.
4. **Step 4: Query the TLD Nameserver (`.com`)**
   * The resolver queries the `.com` TLD Nameserver.
   * The TLD Nameserver does not know the final IP address. Instead, it reads the domain `google.com` and responds with the IP address of Google's **Authoritative Nameservers**.
5. **Step 5: Query the Authoritative Nameserver**
   * The resolver queries the Authoritative Nameserver for `google.com`.
   * Since this server owns the DNS database for `google.com`, it returns the actual IP address (e.g., `142.250.190.46`) along with a **TTL (Time to Live)** value.
6. **Step 6: Return Result and Cache**
   * The Recursive Resolver receives the IP address, saves it in its local cache for the duration of the TTL, and passes the IP address back to the client OS.
   * Your computer's OS and web browser cache the IP address, and the browser proceeds to establish the TCP connection to load `https://google.com`.

#### Key DNS Record Types

* **A Record:** Maps a domain name to an **IPv4** address (e.g., `google.com` → `142.250.190.46`).
* **AAAA Record:** Maps a domain name to an **IPv6** address.
* **CNAME (Canonical Name):** Maps an alias domain to another domain (e.g., `www.google.com` → `google.com`).
* **MX (Mail Exchanger):** Specifies mail servers responsible for receiving email for the domain.
* **NS (Name Server):** Delegates a domain (or subdomain) to a specific set of authoritative nameservers.
* **PTR (Pointer):** Used for reverse DNS lookups — maps an IP address back to a hostname.
* **SOA (Start of Authority):** Stores administrative information about a DNS zone, including the primary nameserver and refresh/retry timers.
* **SRV (Service):** Specifies a hostname and port for specific services (e.g., locating a SIP or XMPP server).
* **TXT (Text):** Stores arbitrary text data. Used heavily for domain validation (e.g., Let's Encrypt SSL verification, SPF, DKIM, and DMARC email security records).

---

### DNS Hijacking: Intercepting the Traffic

**DNS Hijacking** (or DNS redirection) is a cyberattack where an attacker subverts the resolution of DNS queries. Instead of returning the correct IP address for a requested domain, the hijacked DNS settings return the IP address of a rogue server controlled by the attacker.

Users are redirected to spoofed websites (e.g., a fake online banking portal) designed to steal credentials or inject ads.

#### How Attackers Perform DNS Hijacking

Attackers target different points in the DNS resolution flow:

1. **Local Host Hijacking (Malware):**
   * **Mechanism:** An attacker infects a client machine with trojan malware that modifies the local OS hosts file (`/etc/hosts`) or alters the network adapter's DNS settings to point to a rogue resolver.
   * **Result:** Any browser request immediately resolves to fake IPs without ever querying the public DNS network.
2. **Router Hijacking (Firmware Exploitation):**
   * **Mechanism:** Attackers exploit default passwords or firmware vulnerabilities in home/office routers. They log in and rewrite the default DNS servers provided by the ISP to rogue DNS servers.
   * **Result:** Every device on that local Wi-Fi/network is hijacked.
3. **Man-in-the-Middle (MitM) / DNS Spoofing:**
   * **Mechanism:** Attackers intercept unencrypted DNS traffic (which runs over UDP port 53 by default) on a local network (e.g., public Wi-Fi). The attacker sends a fake, fast DNS response before the real resolver can reply.
4. **Domain Registrar Hijacking (Social Engineering / Credential Theft):**
   * **Mechanism:** Attackers compromise the organization's account at the domain registrar (like GoDaddy or Namecheap) using phishing, credential stuffing, or support social engineering. They then modify the NS (Name Server) records to point to rogue authoritative servers.
   * **Result:** Global traffic is diverted because the source-of-truth records are changed at the registry level.

#### How to Protect Against DNS Hijacking

##### For Organizations
* **Deploy DNSSEC (DNS Security Extensions):** DNSSEC cryptographically signs DNS records at the authoritative nameserver. Resolvers verify these signatures, ensuring records have not been forged or tampered with in transit.
* **Enable Registry Lock (Multi-Registry Lock):** A service offered by registrars that prevents domain modifications (like changing NS records) unless a manual, multi-person out-of-band authentication check is passed.
* **Enforce MFA and IP Whitelisting:** Strictly require Multi-Factor Authentication (MFA) and lock down registrar administration panels to specific corporate IP ranges.
* **Monitor DNS Records:** Use automated scripts or external monitoring services to alert on changes to authoritative NS and A records.

##### For Users
* **Use Encrypted DNS Protocols:** Configure browsers and operating systems to use **DoH (DNS over HTTPS)** or **DoT (DNS over TLS)**. This encrypts DNS queries inside a secure TLS tunnel, preventing local networks from snooping or spoofing responses.
* **Secure Local Devices & Routers:** Keep router firmware updated, disable WAN management access, change default router passwords, and run active endpoint security (antivirus).

---

### HTTP Methods and Status Codes

Since HTTP/HTTPS is the dominant Layer 7 protocol for the web, it's worth knowing its two core vocabularies: **methods** (what the client wants to do) and **status codes** (how the server responds).

#### Common HTTP Methods

| Method | Purpose | Idempotent? | Has Body? |
| :--- | :--- | :--- | :--- |
| **GET** | Retrieve a resource | Yes | No |
| **POST** | Create a resource / submit data | No | Yes |
| **PUT** | Replace a resource entirely | Yes | Yes |
| **PATCH** | Partially update a resource | No | Yes |
| **DELETE** | Remove a resource | Yes | No |
| **HEAD** | Like GET, but headers only (no body) | Yes | No |
| **OPTIONS** | Discover allowed methods/CORS preflight | Yes | No |

#### HTTP Status Code Ranges

| Range | Class | Meaning | Common Examples |
| :--- | :--- | :--- | :--- |
| **1xx** | Informational | Request received, continuing process | `101 Switching Protocols` |
| **2xx** | Success | The request was successfully processed | `200 OK`, `201 Created`, `204 No Content` |
| **3xx** | Redirection | Further action needed to complete the request | `301 Moved Permanently`, `304 Not Modified` |
| **4xx** | Client Error | The request has a problem | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests` |
| **5xx** | Server Error | The server failed to fulfill a valid request | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout` |

> [!TIP]
> `502` and `504` almost always point to a problem *behind* your load balancer or reverse proxy (an upstream app server that crashed, is unreachable, or is too slow) — not the proxy itself.

---

## Layer 6 — Presentation Layer

### What It Does
Responsible for formatting, translating, compressing, and encrypting/decrypting data. It ensures that data sent from the application layer of one system can be read by the application layer of another.

### In Our Example
* When accessing `https://google.com`, the connection must be secure. The **TLS (Transport Layer Security)** handshake establishes encryption.
* Data sent to Google is encrypted here, and incoming Google data is decrypted so the browser can render it.
* Data formatting (e.g., converting text to UTF-8 or parsing JSON/HTML) also conceptually aligns here.

> [!NOTE]
> In modern TCP/IP networks, Layer 6 is heavily integrated into the application itself (like Node.js, Nginx, or browser engines handling JSON serialization and TLS negotiation). If a client complains of a handshake error (`SSL_ERROR_SYSCALL` or certificate expiry), this is your Layer 6 checkpoint.

---

## Layer 5 — Session Layer

### What It Does
Manages the start, maintenance, and termination of semi-permanent connections (sessions) between applications on different devices.

### In Our Example
* The session layer handles authentication, authorization, and session restoration.
* It coordinates the communication flow so that your browser session with Google remains distinct from other tabs or active network connections.

> [!NOTE]
> In real-world TCP/IP networks, Layer 5 functions are combined into L7 (e.g., HTTP Cookies, JWT tokens, WebSockets) and L4 (TCP persistent connections). If you are troubleshooting stateful microservices and users are getting logged out prematurely, you are dealing with session-management issues.

---

## Layer 4 — Transport Layer

### What It Does
Handles end-to-end data delivery, flow control, error recovery, and segmentation. It is responsible for making sure data reaches the target *process* on the destination system.

### In Our Example
1. Because HTTPS requires reliability, **TCP** is used.
2. The operating system breaks down the encrypted HTTP data into smaller chunks called **Segments**.
3. It assigns source and destination ports. The destination port is **443** (the standard port for HTTPS), and the source port is a random high port assigned to your browser tab (e.g., `53214`).

```
┌────────────────────────────────────────────────────────┐
│ TCP Header: Source Port: 53214 | Destination Port: 443 │
└────────────────────────────────────────────────────────┘
```

> [!NOTE]
> This is a crucial layer for DevOps engineers. Firewalls, security groups, and load balancers rely heavily on L4 rules. If you cannot reach a server, use `nc -zv <IP> <Port>` to check if the port is listening and reachable. If you get a connection timeout, L4 TCP packets are likely being dropped by a firewall.

### Ports, Sockets, and IP Addresses: The Multiplexing Magic

To understand how data gets from a browser to a specific application on a server, we must distinguish between three core network identifiers:

```
┌──────────────────────────────────────────────────────────────┐
│  IP Address (Identifies Host) : 142.250.190.46               │
├──────────────────────────────────────────────────────────────┤
│  Port Number (Identifies Process) : 443                       │
├──────────────────────────────────────────────────────────────┤
│  Socket (Bound Connection Endpoint) : 142.250.190.46:443      │
└──────────────────────────────────────────────────────────────┘
```

* **IP Address (Layer 3):** Identifies a specific physical or virtual machine on a network. It is like the mailing address of a large apartment building. It gets the packet to the correct building, but not to the individual resident.
* **Port Number (Layer 4):** Identifies a specific application or service running on that machine. It is like an individual mailbox or apartment number in that building. For example, web servers bind to port `80` (HTTP) or `443` (HTTPS), while SSH servers listen on port `22`.
* **Socket:** The logical software endpoint that enables a two-way communication channel between two programs. A socket is created by binding an IP address and a port number together with a protocol (TCP or UDP).
  $$\text{Socket} = \text{IP Address} + \text{Port Number} + \text{Protocol}$$

#### How Multiple Applications Communicate Through the Same Server IP

A common question is: *If a server has only one public IP address and a single web server application listening on Port 443, how can thousands of clients browse Google at the same time?*

The operating system handles this using **socket pairs** (specifically, the **TCP 5-Tuple**). Every unique connection on a network is defined by five parameters:
1. **Source IP Address**
2. **Source Port**
3. **Destination IP Address**
4. **Destination Port**
5. **Protocol** (TCP or UDP)

Because of this, the server does not identify connections solely by its own IP and port. It identifies them by the *combination* of the client's socket and the server's socket.

For example, on Google's web server (`142.250.190.46` listening on Port `443`):

| Active Connection | Client IP (Source IP) | Client Port (Source Port) | Server IP (Dest IP) | Server Port (Dest Port) | Connection Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **User A (Tab 1)** | `198.51.100.22` | `54321` | `142.250.190.46` | `443` | Unique Connection |
| **User A (Tab 2)** | `198.51.100.22` | `54322` | `142.250.190.46` | `443` | Unique Connection |
| **User B** | `203.0.113.88` | `54321` | `142.250.190.46` | `443` | Unique Connection |

Even though the Destination IP and Port are identical for all entries, the operating system's TCP stack easily routes incoming packets to the correct buffer because each client has a unique Source IP and/or Source Port combination.

---

### TCP vs. UDP: Key Differences

At the Transport Layer (Layer 4), two primary protocols carry traffic: **TCP (Transmission Control Protocol)** and **UDP (User Datagram Protocol)**.

#### Comparison Table

| Parameter | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
| :--- | :--- | :--- |
| **Connection** | Connection-oriented (requires 3-way handshake) | Connectionless (no setup or teardown) |
| **Reliability** | Guaranteed delivery (retransmits lost packets) | Best-effort (packets can be lost silently) |
| **Ordering** | Guaranteed in-order delivery of data | No order guarantees (packets can arrive out of order) |
| **Flow Control** | Yes (limits sender speed to match receiver's speed) | No (sends data as fast as the network allows) |
| **Congestion Control** | Yes (throttles transmission during network congestion) | No |
| **Header Size** | 20 to 60 bytes (variable) | 8 bytes (fixed) |
| **Transmission Unit** | Byte Stream (reconstructs continuous blocks) | Datagrams (independent packets with boundaries) |

#### A Note on Congestion Control Algorithms

TCP's congestion control is what keeps the internet from collapsing under load. A few algorithms you'll see referenced in the wild:

* **Slow Start:** A new connection starts with a small congestion window and doubles it each round-trip until it detects loss or hits a threshold.
* **Congestion Avoidance:** After the threshold, the window grows linearly instead of exponentially, probing for available bandwidth more cautiously.
* **CUBIC:** The default algorithm on most Linux systems; grows the window based on a cubic function of time since the last loss event, which performs well on high-bandwidth, high-latency ("long fat") networks.
* **BBR (Bottleneck Bandwidth and RTT):** A newer, model-based algorithm developed by Google that estimates the actual bottleneck bandwidth and round-trip time rather than reacting purely to packet loss — often used by CDNs and large-scale services.

---

#### Practical Examples: When to Choose TCP

Choose **TCP** when **data integrity** and **order** are more important than speed. A single lost bit can corrupt the entire message.

1. **HTTP/HTTPS (Web Browsing):** Loading the HTML, CSS, and Javascript for `https://google.com` requires absolute accuracy. A missing Javascript byte could break the page execution.
2. **SSH / Telnet (Remote Terminal):** Terminal commands and keystrokes must arrive in the exact order they were typed; otherwise, you could run incorrect shell commands.
3. **Database Queries:** Retrieving records from databases (PostgreSQL, MySQL) requires strict integrity. You cannot afford to lose database rows or cell data in transit.
4. **API Requests (REST, gRPC):** Application integration payloads must be complete and formatted correctly to prevent JSON or protobuf parsing errors.

#### Practical Examples: When to Choose UDP

Choose **UDP** when **speed** and **low latency** are critical, and losing a few packets is acceptable or easily managed.

1. **Live Video / Audio Streaming (VoIP, Zoom, FaceTime):** Low latency is vital. If a video frame packet is lost, it is better to display a brief glitch (pixelation) rather than pause the video stream for 500ms waiting for a TCP retransmission.
2. **DNS Queries:** Resolving `google.com` to an IP requires speed. A DNS query is a single request-response. If it gets lost, the client simply times out and retries. Using TCP would double the lookup time due to handshake overhead.
3. **Real-time Online Gaming:** Fast shooter or sports games need coordinate updates immediately. A packet showing where your teammate was 200ms ago is useless; you only care about the latest packet.
4. **DHCP (Dynamic Host Configuration Protocol):** Used to assign IPs. DHCP uses UDP broadcasts to reach out to the network when the client doesn't even have an IP address configured yet.

> [!NOTE]
> **QUIC and HTTP/3:** Modern browsers and CDNs increasingly use **QUIC**, a transport protocol built on top of UDP that reimplements TCP-like reliability, ordering, and congestion control in user space, plus built-in TLS 1.3 encryption. QUIC avoids "head-of-line blocking" (where one lost packet stalls all streams on a connection) and speeds up connection setup by combining the transport and TLS handshakes into fewer round trips. HTTP/3 is HTTP running over QUIC instead of TCP.

---

### The TCP Header Structure

At Layer 4 (Transport Layer), application data is wrapped inside a TCP Segment. The operating system prepends a **TCP Header** containing control fields that enable reliability, flow control, and connection tracking.

A standard TCP header is **20 bytes** (160 bits) long when no options are specified.

#### TCP Header Layout (32-Bit Grid)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├───────────────────────────────┼───────────────────────────────┤
│          Source Port          │       Destination Port        │
├───────────────────────────────┴───────────────────────────────┤
│                        Sequence Number                        │
├───────────────────────────────────────────────────────────────┤
│                     Acknowledgment Number                     │
├───────┬───────┬───────────────┬───────────────────────────────┤
│ Data  │       │  C E U A P R S│                               │
│ Offset│Reserved│  W R G C S S Y│          Window Size          │
│       │       │  R E   K H T N│                               │
├───────┴───────┴───────────────┼───────────────────────────────┤
│           Checksum            │        Urgent Pointer         │
├───────────────────────────────┴───────────────────────────────┤
│                    Options (0 to 40 bytes)                    │
└───────────────────────────────────────────────────────────────┘
```

#### Important TCP Header Fields

1. **Source Port (16 bits) & Destination Port (16 bits)**
   * **What they do:** Identify the sending process (source) and the receiving process (destination).
   * **Details:** The destination port indicates the target service (e.g., `443` for HTTPS, `80` for HTTP, `22` for SSH). The source port is a dynamic high port (often between `32768` and `60999`) assigned by the client OS.
2. **Sequence Number (32 bits)**
   * **What it does:** Tracks the position of the first data byte of this segment in the overall data stream.
   * **Why it matters:** Ensures data arrives intact and allows the receiving host to reassemble out-of-order packets.
3. **Acknowledgment Number (32 bits)**
   * **What it does:** Specifies the next sequence number/byte that the sender expects to receive.
   * **Details:** This field is only valid if the `ACK` flag is set. It tells the other side: "I have successfully received all bytes up to `Ack - 1`. Send byte `Ack` next."
4. **Data Offset / Header Length (4 bits)**
   * **What it does:** Specifies the size of the TCP header in 32-bit words.
   * **Why it matters:** Since the Options field is variable-length, the receiver needs to know where the TCP header ends and the actual payload data begins.
5. **Control Flags (9 bits total)** — these flags control connection states and how data is handled:
   * **SYN (Synchronize):** Requests connection establishment (used during handshake).
   * **ACK (Acknowledgment):** Indicates that the Acknowledgment Number field is valid.
   * **FIN (Finish):** Requests graceful connection termination.
   * **RST (Reset):** Forcefully terminates/rejects a connection (usually due to a closed port or crash).
   * **PSH (Push):** Instructs the receiver to pass data immediately to the application rather than buffering it.
   * **URG (Urgent):** Indicates that the data in the segment is urgent (should be processed out of order).
6. **Window Size (16 bits)**
   * **What it does:** Used for **Flow Control** (via the Sliding Window algorithm).
   * **Details:** Specifies the number of bytes the sender of this segment is willing to accept from the receiver without receiving another acknowledgment. This prevents a fast sender from overwhelming a slow receiver.
7. **Checksum (16 bits)**
   * **What it does:** Used for error checking.
   * **Details:** The sender calculates a checksum hash of the TCP header, the payload data, and a pseudo-IP header. If the receiver computes a different checksum value upon arrival, the segment is discarded as corrupted.

---

### TCP Three-Way Handshake: Establishing a Connection

Before Layer 4 (Transport Layer) can transmit any application data (such as the HTTP request for `https://google.com`), TCP must establish a reliable, stateful connection between the client (your laptop) and the server (Google). This is achieved through the **TCP Three-Way Handshake**.

The main purpose of the handshake is to synchronize sequence numbers and establish initial buffer sizes for flow control on both sides.

#### The Handshake Diagram

```
       CLIENT                                                     SERVER
   (192.168.1.10)                                             (Google Server)
   [ State: CLOSED ]                                         [ State: LISTEN ]
          │                                                          │
          │                   1. SYN (Seq = X)                       │
          │─────────────────────────────────────────────────────────>│
   [ State: SYN-SENT ]                                       [ State: SYN-RECEIVED ]
          │                                                          │
          │               2. SYN-ACK (Seq = Y, Ack = X+1)            │
          │<─────────────────────────────────────────────────────────│
          │                                                          │
          │                   3. ACK (Ack = Y+1)                     │
          │─────────────────────────────────────────────────────────>│
   [ State: ESTABLISHED ]                                    [ State: ESTABLISHED ]
          │                                                          │
          │ ════════════════════════════════════════════════════════ │
          │             Connection Ready for Data Transfer           │
```

#### Step-by-Step Breakdown

1. **SYN (Synchronize)**
   * **Who sends it:** The Client (your browser/OS).
   * **What it contains:** A packet with the `SYN` (Synchronize) control flag set to `1`. It includes a randomly generated **Initial Sequence Number (ISN)**, let's call it `X` (`Seq=X`).
   * **Meaning:** "Hello! I want to open a connection with you on port 443. My sequence numbering will start at `X`."
   * **State Change:** Client transitions from `CLOSED` to `SYN-SENT`.
2. **SYN-ACK (Synchronize-Acknowledgment)**
   * **Who sends it:** The Server (Google).
   * **What it contains:** A packet with both `SYN` and `ACK` flags set to `1`. It contains:
     1. An acknowledgment number `Ack = X + 1` (confirming it received the client's SYN).
     2. Its own randomly generated Initial Sequence Number, let's call it `Y` (`Seq=Y`).
   * **Meaning:** "I received your request and acknowledge your starting sequence number `X` (I'm ready for byte `X+1`). I also want to establish connection on my side; here is my starting sequence number `Y`."
   * **State Change:** Server transitions from `LISTEN` to `SYN-RECEIVED`.
3. **ACK (Acknowledgment)**
   * **Who sends it:** The Client.
   * **What it contains:** A packet with the `ACK` flag set to `1`. It contains an acknowledgment number `Ack = Y + 1` (confirming it received the server's SYN-ACK).
   * **Meaning:** "Received! I acknowledge your starting sequence number `Y` (I am ready for byte `Y+1`). The connection is now established."
   * **State Change:** Client transitions to `ESTABLISHED`. Upon receiving this packet, the server also transitions to `ESTABLISHED`.

---

### TCP Connection Termination: The Four-Way Handshake

Unlike connection establishment (which is a 3-way process), connection termination is a **4-way process**. Because TCP connections are full-duplex (data can flow in both directions independently), each direction must be shut down individually.

The party that initiates the termination sends a **FIN** (Finish) packet, and the other side acknowledges it. Once both sides have finished sending data and terminated their directions, the connection is closed.

#### The Termination Diagram

```
       CLIENT                                                     SERVER
   (192.168.1.10)                                             (Google Server)
 [ State: ESTABLISHED ]                                    [ State: ESTABLISHED ]
          │                                                          │
          │             1. FIN (Seq = A, Ack = B)                    │
          │─────────────────────────────────────────────────────────>│
   [ State: FIN-WAIT-1 ]                                     [ State: CLOSE-WAIT ]
          │                                                          │
          │             2. ACK (Seq = B, Ack = A+1)                  │
          │<─────────────────────────────────────────────────────────│
   [ State: FIN-WAIT-2 ]                                             │
          │                                                    (Server sends
          │                                                    remaining data)
          │                                                          │
          │             3. FIN (Seq = B, Ack = A+1)                  │
          │<─────────────────────────────────────────────────────────│
   [ State: TIME-WAIT ]                                      [ State: LAST-ACK ]
          │                                                          │
          │             4. ACK (Seq = A+1, Ack = B+1)                │
          │─────────────────────────────────────────────────────────>│
          │                                                  [ State: CLOSED ]
   [ Wait 2MSL (2 mins) ]                                            │
   [ State: CLOSED ]                                                 │
```

#### Step-by-Step Breakdown

1. **FIN (from Active Initiator)**
   * **Who sends it:** The host closing the connection (usually the Client, but can be the Server).
   * **What it contains:** A packet with the `FIN` (Finish) flag set to `1` and sequence number `Seq = A`.
   * **Meaning:** "I have finished sending data. I want to close my side of the connection."
   * **State Change:** Initiator transitions to `FIN-WAIT-1`.
2. **ACK (from Receiver)**
   * **Who sends it:** The receiving host.
   * **What it contains:** A packet with the `ACK` flag set to `1` and acknowledgment number `Ack = A + 1`.
   * **Meaning:** "I received your request to close. I acknowledge your FIN."
   * **State Change:** Receiver transitions to `CLOSE-WAIT`. Initiator transitions to `FIN-WAIT-2`.
   * **Half-Closed State:** The connection is now half-closed. The initiator can no longer send data, but it can still receive data if the receiver has pending payloads to send.
3. **FIN (from Receiver)**
   * **Who sends it:** The receiving host (once it is done sending all remaining data).
   * **What it contains:** A packet with the `FIN` flag set to `1` and sequence number `Seq = B` (and acknowledging the initiator's state via `Ack = A + 1`).
   * **Meaning:** "I have finished sending my remaining data too. I want to close my side of the connection now."
   * **State Change:** Receiver transitions to `LAST-ACK`.
4. **ACK (from Active Initiator)**
   * **Who sends it:** The initiator.
   * **What it contains:** A packet with the `ACK` flag set to `1` and acknowledgment number `Ack = B + 1`.
   * **Meaning:** "Acknowledged. I received your FIN. Goodbye."
   * **State Change:** Initiator transitions to `TIME-WAIT`. The receiver transitions to `CLOSED` immediately upon receiving this final ACK.

#### The Importance of the `TIME-WAIT` State

When a host initiates a graceful TCP shutdown, it does not transition directly to `CLOSED`. Instead, it remains in the `TIME-WAIT` state for a duration of **2MSL** (Maximum Segment Lifetime, typically 1 to 4 minutes total).

There are two primary reasons for this design:
1. **To guarantee delivery of the final ACK:** If the final `ACK` (Step 4) is lost in transit, the receiver will timeout in `LAST-ACK` state and retransmit its `FIN` (Step 3). Since the initiator is still in `TIME-WAIT`, it can receive that `FIN` and re-send the `ACK`. If the initiator had closed immediately, it would respond to the retransmitted `FIN` with a `RST` (Reset), causing the receiver to report a connection error.
2. **To allow duplicate segments to drain:** It prevents delayed or wandering packets from the old connection from being received by a new connection utilizing the same IP address and port numbers.

> [!TIP]
> A server that accumulates thousands of sockets stuck in `TIME-WAIT` under heavy short-lived-connection load is a classic performance issue. Common fixes include enabling `SO_REUSEADDR`/`SO_REUSEPORT`, using HTTP keep-alive/connection pooling to reduce churn, and tuning kernel parameters like `net.ipv4.tcp_tw_reuse` on Linux.

---

## Layer 3 — Network Layer

### What It Does
Handles the routing and forwarding of data packets across different physical networks. It uses logical addresses (IP addresses) to identify hosts on the internet.

### In Our Example
1. The OS takes the TCP Segment and wraps it inside an **IP Packet**.
2. It attaches your computer's local IP as the **Source IP** (e.g., `192.168.1.10`) and Google's server IP as the **Destination IP** (e.g., `142.250.190.46`).
3. Routers along the path read this destination IP and determine the next hop to forward the packet.

```
┌─────────────────────────────────────────────────────────────────┐
│ IP Header: Source IP: 192.168.1.10 | Destination: 142.250.190.46│
├─────────────────────────────────────────────────────────────────┤
│ TCP Header + Encrypted Payload                                  │
└─────────────────────────────────────────────────────────────────┘
```

> [!NOTE]
> Layer 3 is the domain of routing tables, VPC subnets, and internet gateways. When testing L3 connectivity, tools like `ping` (ICMP) and `traceroute` are your best friends. If `ping` fails but you can resolve the DNS, you have an IP routing or firewall policy issue at Layer 3.

### The IPv4 Header Structure

When Layer 3 (Network Layer) receives a TCP Segment from Layer 4, it wraps it inside an **IP Packet** and prepends an **IPv4 Header**. This header contains routing parameters and is typically **20 bytes** long.

#### IPv4 Header Layout (32-Bit Grid)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├───────┬───────┬───────────────┬───────────────────────────────┤
│Version│  IHL  │TypeOf Service │          Total Length         │
├───────┴───────┴───────────────┼───┬───┬───┬───────────────────┤
│        Identification         │ R │ D │ M │  Fragment Offset  │
│                               │   │ F │ F │                   │
├───────────────┬───────────────┼───┴───┴───┴───────────────────┤
│ Time to Live  │   Protocol    │        Header Checksum        │
├───────────────┴───────────────┴───────────────────────────────┤
│                       Source IP Address                       │
├───────────────────────────────────────────────────────────────┤
│                     Destination IP Address                    │
├───────────────────────────────────────────────────────────────┤
│                    Options (0 to 40 bytes)                    │
└───────────────────────────────────────────────────────────────┘
```

#### Important IPv4 Header Fields

* **Version (4 bits):** Identifies the IP version. For IPv4, this is always `0100` (binary for 4).
* **IHL (Internet Header Length) (4 bits):** Specifies the length of the IP header in 32-bit words (minimum is `5`, representing 20 bytes).
* **Total Length (16 bits):** The size of the entire packet (header + payload) in bytes. The maximum possible size is 65,535 bytes.
* **Time to Live (TTL) (8 bits):** An expiration counter that prevents packets from routing in infinite loops. Every router that forwards the packet decrements this value by 1. If it hits 0, the packet is discarded and an ICMP "Time Exceeded" message is sent back to the sender.
* **Protocol (8 bits):** Indicates the next-level protocol in the payload. Common values include `6` for TCP, `17` for UDP, and `1` for ICMP.
* **Header Checksum (16 bits):** Used for error checking of the header. It must be recalculated at every single router hop because the TTL value changes.
* **Source IP (32 bits) & Destination IP (32 bits):** The logical addresses of the sender (client) and receiver (server).

#### Packet Fragmentation Fields
When a packet is larger than the Maximum Transmission Unit (MTU) of a physical link it must traverse (e.g., passing from a 1500-byte Ethernet link to a 1400-byte VPN tunnel), routers use these fields to split the packet:
* **Identification (16 bits):** A unique ID shared by all fragments of the same original packet.
* **Flags (3 bits):**
  * **DF (Don't Fragment):** If set to 1, tells routers *not* to split this packet. If the packet exceeds a link's MTU, the router drops it and sends back an ICMP "Fragmentation Needed" error.
  * **MF (More Fragments):** If set to 1, indicates that this is a fragment and more are coming. If 0, this is the last fragment.
* **Fragment Offset (13 bits):** Specifies where this fragment's payload fits in the original, reassembled IP packet payload (measured in 8-byte blocks).

> [!TIP]
> **Path MTU Discovery (PMTUD)** is the technique modern hosts use to avoid fragmentation altogether: they send packets with the `DF` bit set and shrink the packet size in response to ICMP "Fragmentation Needed" messages until the packet passes every hop unfragmented. This is why blocking all ICMP at a firewall is a common (and sneaky) cause of "connection hangs but ping/handshake works" bugs — the TCP handshake succeeds with small packets, but larger data transfers silently stall because the PMTUD ICMP replies are being dropped.

---

### IPv6: The Next-Generation Protocol

IPv4 provides roughly 4.3 billion unique addresses (2³²), a number the internet exhausted years ago through address reuse, NAT, and address-market trading. **IPv6** was designed to solve this permanently with a vastly larger address space and a simplified header.

#### IPv6 Addressing

* **128-bit addresses** (2¹²⁸ possible addresses — enough to assign trillions of addresses to every person on Earth).
* Written as **eight groups of four hexadecimal digits**, separated by colons:
  `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
* **Shorthand rules:**
  * Leading zeros in each group can be dropped: `2001:db8:85a3:0:0:8a2e:370:7334`
  * One consecutive run of all-zero groups can be collapsed to `::` (only once per address): `2001:db8:85a3::8a2e:370:7334`
* **Loopback:** `::1` (equivalent to IPv4's `127.0.0.1`)
* **Link-local addresses:** `fe80::/10` — auto-assigned to every interface, used for on-link communication and neighbor discovery, never routed off the local network.

#### IPv6 Header Layout (Simplified & Fixed at 40 Bytes)

Unlike IPv4's variable-length header, the IPv6 header is a **fixed 40 bytes**, which simplifies router processing:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌────────┬───────────────┬──────────────────────────────────────┐
│Version │ Traffic Class │              Flow Label              │
├────────┴───────────────┴─────┬───────────────┬────────────────┤
│        Payload Length        │  Next Header  │   Hop Limit    │
├──────────────────────────────┴───────────────┴────────────────┤
│                   Source Address (128 bits)                   │
├───────────────────────────────────────────────────────────────┤
│                 Destination Address (128 bits)                │
└───────────────────────────────────────────────────────────────┘
```

#### Key Differences from IPv4

| Feature | IPv4 | IPv6 |
| :--- | :--- | :--- |
| **Address Length** | 32 bits (~4.3 billion addresses) | 128 bits (~340 undecillion addresses) |
| **Header Size** | Variable (20–60 bytes) | Fixed (40 bytes) |
| **Header Checksum** | Present (recalculated every hop) | **Removed** — reliability delegated to L2/L4 |
| **Fragmentation** | Performed by routers and the sender | Performed **only by the sending host**; routers drop oversized packets |
| **Broadcast** | Supported | **Removed** — replaced by Multicast and Anycast |
| **Address Resolution** | ARP | **NDP** (Neighbor Discovery Protocol, built on ICMPv6) |
| **Auto-configuration** | Requires DHCP | Supports **SLAAC** (Stateless Address Autoconfiguration) in addition to DHCPv6 |
| **Built-in Security** | Optional (IPsec bolted on) | IPsec support designed in from the start |

> [!NOTE]
> Because IPv6 has no broadcast, protocols that relied on it (like ARP) are replaced by **multicast**-based equivalents. **NDP** uses ICMPv6 multicast messages (Neighbor Solicitation / Neighbor Advertisement) to discover a neighbor's Layer 2 address — functionally the same job ARP does for IPv4, just implemented differently.

---

### Subnetting and CIDR Notation

A single flat network of millions of hosts would be unmanageable and would flood every device with broadcast traffic. **Subnetting** divides a larger network into smaller, logical sub-networks, and **CIDR (Classless Inter-Domain Routing)** notation is the modern shorthand for expressing how a network is divided.

#### Reading CIDR Notation

An address like `192.168.1.0/24` means:
* The first **24 bits** are the **network portion** (fixed, identifies the subnet).
* The remaining **8 bits** are the **host portion** (variable, identifies individual devices on that subnet).

This is equivalent to the subnet mask `255.255.255.0`.

```
192.168.1.0    /24
├─────────────┤├──┤
 Network bits   Host bits (32 - 24 = 8 bits → 256 addresses)
```

#### Common CIDR Prefixes

| CIDR | Subnet Mask | Total Addresses | Usable Hosts* |
| :--- | :--- | :--- | :--- |
| `/32` | `255.255.255.255` | 1 | 1 (a single host route) |
| `/30` | `255.255.255.252` | 4 | 2 (common for point-to-point links) |
| `/29` | `255.255.255.248` | 8 | 6 |
| `/28` | `255.255.255.240` | 16 | 14 |
| `/27` | `255.255.255.224` | 32 | 30 |
| `/26` | `255.255.255.192` | 64 | 62 |
| `/25` | `255.255.255.128` | 128 | 126 |
| `/24` | `255.255.255.0` | 256 | 254 |
| `/16` | `255.255.0.0` | 65,536 | 65,534 |
| `/8` | `255.0.0.0` | 16,777,216 | 16,777,214 |

*Usable hosts = Total addresses − 2 (the first address is reserved as the **network address**, the last as the **broadcast address**).

#### Private (RFC 1918) IP Ranges

These ranges are reserved for internal/private networks and are never routed on the public internet — they're the addresses you'll see behind almost every home router or corporate LAN:

| Range | CIDR | Typical Use |
| :--- | :--- | :--- |
| `10.0.0.0 – 10.255.255.255` | `10.0.0.0/8` | Large enterprise/cloud networks (e.g., AWS VPCs) |
| `172.16.0.0 – 172.31.255.255` | `172.16.0.0/12` | Medium networks, Docker default bridge networks |
| `192.168.0.0 – 192.168.255.255` | `192.168.0.0/16` | Home and small office routers |

---

### NAT (Network Address Translation)

**NAT** allows many devices on a private network (using RFC 1918 addresses) to share a single public IP address when communicating with the internet. It's the mechanism that has kept IPv4 viable for decades despite its limited address space, and it also hides internal network topology from the outside world.

#### How NAT Works (PAT / NAT Overload)

The most common form, **PAT (Port Address Translation)**, distinguishes internal devices by rewriting the *source port* along with the source IP:

```
   Internal Device                      Home Router (NAT)                    Internet
 [ 192.168.1.10:54321 ] ───────────>  [ Translates to ]  ───────────>  [ 203.0.113.5:61000 ]
                                       ┌─────────────────────┬─────────────┐
                                       │             NAT Table             │
                                       ├─────────────────────┬─────────────┤
                                       │   Internal Socket   │ Public Port │
                                       ├─────────────────────┼─────────────┤
                                       │  192.168.1.10:54321 │    61000    │
                                       │  192.168.1.11:53210 │    61001    │
                                       └─────────────────────┴─────────────┘
```

1. Your laptop (`192.168.1.10:54321`) sends a packet destined for Google.
2. Your router rewrites the source IP to its own public IP (`203.0.113.5`) and assigns a unique public source port (`61000`), recording the mapping in its NAT table.
3. Google replies to `203.0.113.5:61000`.
4. The router looks up `61000` in its NAT table, rewrites the destination back to `192.168.1.10:54321`, and forwards the reply internally.

#### Types of NAT

* **Static NAT:** A fixed one-to-one mapping between one private IP and one public IP — used when an internal server needs a consistent, dedicated public-facing address.
* **Dynamic NAT:** Maps a private IP to *any* available address from a pool of public IPs, on a first-come basis.
* **PAT / NAT Overload (most common):** Many private IPs share a *single* public IP, differentiated by source port, as shown above. This is what nearly every home router does.

> [!NOTE]
> NAT is not a security feature by design, but it has a side effect that resembles one: because unsolicited inbound connections have no matching entry in the NAT table, they're dropped by default. This is why devices behind a home NAT router are not directly reachable from the internet unless you configure **port forwarding**.

---

## Layer 2 — Data Link Layer

### What It Does
Provides node-to-node data transfer (directly connected devices on the same local network) and handles error detection from the physical layer. It translates logical IP packets into physical frames using hardware **MAC Addresses**.

### In Our Example
1. Your laptop cannot talk directly to Google's public IP over a local link. It must first send the packet to its local gateway (your home router, e.g., `192.168.1.1`).
2. Your laptop uses **ARP** (Address Resolution Protocol) to discover your router's MAC address.
3. The IP Packet is packaged into an **Ethernet Frame** or **Wi-Fi Frame**, stamping your laptop's MAC as the **Source MAC** and your router's MAC as the **Destination MAC**.

```
┌─────────────────────────────────────────────────────────────────┐
│ Frame Header: Source MAC: AA:BB:CC... | Destination MAC: 11:22..│
├─────────────────────────────────────────────────────────────────┤
│ IP Header + TCP Header + Encrypted Payload                      │
└─────────────────────────────────────────────────────────────────┘
```

> [!NOTE]
> Layer 2 issues usually show up as local interface flaps, duplicate IP assignments (ARP conflicts), or VLAN tagging problems inside virtualized hypervisors or cloud container networks (like Kubernetes CNI overlays). If a VM cannot reach its gateway on the same subnet, check the ARP table (`arp -a`).

### ARP (Address Resolution Protocol): Mapping IPs to MACs

While routers use logical IP addresses (Layer 3) to route packets across networks, computers can only transmit data locally using physical **MAC addresses** (Layer 2). **ARP** is the glue protocol that maps a known logical IP address to a physical MAC address on a local area network (LAN).

#### How ARP Resolves Addresses (Step-by-Step)

If a laptop (`192.168.1.10`) needs to send an IP packet to its local router (`192.168.1.1`), it must discover the router's MAC address:

```
       CLIENT (192.168.1.10)                                          ROUTER (192.168.1.1)
         [ MAC: AA:BB:CC ]                                             [ MAC: 11:22:33 ]
                 │                                                             │
                 │ 1. ARP Request: "Who has 192.168.1.1? Tell 192.168.1.10"   │
                 │────────────────────────────────────────────────────────────>│
                 │ (Destination MAC: FF:FF:FF:FF:FF:FF - Broadcast)            │
                 │                                                             │
                 │ 2. ARP Reply: "I have 192.168.1.1. My MAC is 11:22:33"      │
                 │<────────────────────────────────────────────────────────────│
                 │ (Destination MAC: AA:BB:CC - Unicast)                       │
                 │                                                             │
        [ Caches mapping: ]                                                    │
     192.168.1.1 -> 11:22:33                                                   │
```

1. **Step 1: Check Local ARP Cache:**
   * The OS first checks its internal database, called the **ARP Cache**. If the IP-to-MAC mapping is already listed, it grabs the MAC address and skips any network requests.
2. **Step 2: Send an ARP Request (Broadcast):**
   * If the MAC is not cached, the client broadcasts an ARP Request frame to all devices on the local subnet.
   * Because it doesn't know the destination MAC, it sets the destination MAC to `FF:FF:FF:FF:FF:FF` (the layer-2 broadcast address).
   * Every device on the local network segment reads the request: *"Who has IP address `192.168.1.1`? Tell `192.168.1.10`."*
3. **Step 3: Send an ARP Reply (Unicast):**
   * All devices receive the broadcast, but only the host matching the requested IP address (`192.168.1.1`) responds.
   * The router replies directly to the client's MAC address (Unicast): *"I have `192.168.1.1`. My MAC address is `11:22:33:44:55:66`."*
4. **Step 4: Cache and Transmit:**
   * The client receives the unicast reply, saves the mapping in its local ARP cache, and uses the MAC address to package the pending IP packet into an Ethernet frame for physical transmission.

#### Special ARP Types

* **Gratuitous ARP:** A broadcast packet sent by a host to announce its own IP-to-MAC mapping to the local network when its interface turns on or changes. It serves two purposes:
  1. **IP Conflict Detection:** If another host responds to this broadcast, it indicates an IP address conflict.
  2. **Force-Update Caches:** It updates the local ARP caches of all other devices on the segment immediately (crucial when shifting an IP to a backup server during high-availability failovers).

---

### Switching, VLANs, and Broadcast Domains

#### How a Switch Forwards Frames

An Ethernet switch operates at Layer 2 and builds a **MAC address table** (also called a CAM table — Content Addressable Memory) that maps MAC addresses to physical switch ports:

1. **Learning:** When a frame arrives, the switch records the source MAC address and the port it arrived on.
2. **Forwarding/Filtering:** If the destination MAC is already in the table, the switch forwards the frame *only* out that specific port (unlike an old-fashioned hub, which blindly repeats every frame to every port).
3. **Flooding:** If the destination MAC is unknown, the switch floods the frame out every port except the one it arrived on, and learns the reply.

This behavior is what separates a **collision domain** (a segment where frames can collide — historically each switch port is its own collision domain) from a **broadcast domain** (a segment where broadcast frames, like ARP requests, are seen by every device — by default, an entire switched LAN is one broadcast domain).

#### VLANs (Virtual LANs)

A **VLAN** logically segments a single physical switch (or set of switches) into multiple, isolated broadcast domains — without needing separate physical hardware for each segment.

* **Why:** Isolate traffic for security or organizational reasons (e.g., separate `Guest-WiFi`, `Engineering`, and `Finance` VLANs on the same physical switches), reduce the size of each broadcast domain to improve performance, and enforce routing/firewall policy between segments.
* **802.1Q Tagging:** When a frame needs to cross a link that carries multiple VLANs (a **trunk** port), the switch inserts a 4-byte **VLAN tag** into the Ethernet frame header identifying which VLAN the frame belongs to.
* **Access vs. Trunk Ports:**
  * **Access port:** Connects to a single end device (a laptop, a server) and belongs to exactly one VLAN; the device itself is unaware VLANs exist.
  * **Trunk port:** Connects switches to each other (or to a router/firewall) and carries tagged traffic for multiple VLANs simultaneously.
* **Inter-VLAN Routing:** Because VLANs are separate broadcast domains (effectively separate IP subnets), a device in one VLAN needs a **Layer 3 device** (a router, or a "Layer 3 switch") to reach a device in another VLAN — this is also a natural enforcement point for firewall rules between segments.

> [!NOTE]
> **VLAN hopping** is an attack where a host on one VLAN gains unauthorized access to traffic on another VLAN, typically by exploiting misconfigured trunk ports (e.g., "switch spoofing" or "double tagging"). Best practice: never leave unused switch ports in the default/native VLAN, and explicitly disable auto-trunking negotiation on access ports.

---

### DHCP DORA: Obtaining an IP Address

Before a device can communicate at Layer 3 (Network Layer) using IP addresses, it must obtain an IP address. On most networks, this is handled dynamically by a DHCP (Dynamic Host Configuration Protocol) server using the **DORA** process:

* **D**iscover
* **O**ffer
* **R**equest
* **A**cknowledge

#### The DORA Process Diagram

Because a new client does not yet have an IP address, it communicates using MAC addresses and IP **broadcasts** (`255.255.255.255`).

```
       CLIENT                                                     SERVER
   (MAC: AA:BB:CC)                                            (DHCP Server)
  [ IP: 0.0.0.0 ]                                            [ IP: 192.168.1.1 ]
          │                                                          │
          │             1. DHCP DISCOVER (Broadcast)                 │
          │─────────────────────────────────────────────────────────>│
          │      Src IP: 0.0.0.0        | Dest IP: 255.255.255.255   │
          │      Src Port: 68           | Dest Port: 67              │
          │                                                          │
          │             2. DHCP OFFER (Unicast/Broadcast)            │
          │<─────────────────────────────────────────────────────────│
          │      Offered IP: 192.168.1.10                            │
          │      Gateway: 192.168.1.1   | Subnet: 255.255.255.0      │
          │                                                          │
          │             3. DHCP REQUEST (Broadcast)                  │
          │─────────────────────────────────────────────────────────>│
          │      Requesting: 192.168.1.10                            │
          │                                                          │
          │             4. DHCP ACK (Unicast/Broadcast)              │
          │<─────────────────────────────────────────────────────────│
          │      Lease Confirmed (e.g., 24 Hours)                    │
          │                                                          │
  [ IP: 192.168.1.10 ]                                               │
```

#### Step-by-Step Breakdown

1. **Discover (Client → Server):**
   * **What happens:** The client boots up and sends a broadcast packet on the local subnet searching for a DHCP server.
   * **Networking details:**
     * Source IP: `0.0.0.0` | Destination IP: `255.255.255.255` (Broadcast)
     * Source Port: `68` | Destination Port: `67` (UDP)
2. **Offer (Server → Client):**
   * **What happens:** Any DHCP servers on the subnet receive the request and respond with a lease offer.
   * **Networking details:** The server proposes an IP address (e.g., `192.168.1.10`), subnet mask (`255.255.255.0`), default gateway (`192.168.1.1`), lease time (e.g., 24 hours), and DNS server addresses.
3. **Request (Client → Server):**
   * **What happens:** The client chooses one offer (if there are multiple servers) and broadcasts a request packet to finalize the lease.
   * **Why it is broadcast:** The packet is broadcast so *all* DHCP servers on the subnet see it. The chosen server knows to lock down the lease, while other servers know their offers were declined and can release those reserved IPs back to their pool.
4. **Acknowledge (Server → Client):**
   * **What happens:** The selected DHCP server acknowledges the request, committing the IP-to-MAC binding to its database. The client is now configured and can begin standard network communication.

---

## Layer 1 — Physical Layer

### What It Does
Transmits raw, unstructured bit streams over a physical medium (copper cables, fiber optics, or radio waves).

### In Our Example
* The frame is converted into raw electrical voltages, light pulses, or radio signals representing **0s and 1s** (`1010100110...`).
* These signals travel through your Ethernet cable or Wi-Fi antenna, out through your ISP's physical fiber infrastructure, and eventually reach Google's servers.

> [!NOTE]
> "Always check Layer 1 first." A loose cable, a dirty fiber connector, a faulty SFP module, or heavy radio interference can bring down the most sophisticated cloud architectures. If your interface link light is off (`no link` in `ethtool` or `ip link`), do not waste time debugging routing tables.

---

## Summary of the Communication Flow

### Encapsulation (On Your Laptop)
As your request travels down the OSI stack, headers and trailers are wrapped around your data:

```
[ L7: Data ]           --> "Get Google homepage"
     ↓
[ L6: Encrypted ]      --> Encrypted using TLS
     ↓
[ L5: Session ]        --> Session tracked
     ↓
[ L4: Segment ]        --> [ TCP Header (Port 443) ] [ Data ]
     ↓
[ L3: Packet ]         --> [ IP Header (Source/Dest IP) ] [ Segment ]
     ↓
[ L2: Frame ]          --> [ Frame Header (Source/Dest MAC) ] [ Packet ] [ FCS ]
     ↓
[ L1: Bits ]           --> 01001000 01000101 01000001 01010010 01010100
```

### Decapsulation (On Google's Server)
When the bits arrive at Google, the process is reversed:
1. **L1**: Receives physical signals and reconstructs bits.
2. **L2**: Strips off the frame header, checks for errors, verifies destination MAC, and passes the packet up.
3. **L3**: Strips off the IP header, verifies destination IP, and routes the packet internally.
4. **L4**: Strips off the TCP header, verifies port 443, reassembles segment fragments, and hands it to the web server process.
5. **L5/L6**: Handles TLS decryption and session context.
6. **L7**: The web server receives the HTTPS request, processes it, and generates an HTML response to send back down the stack.

---

## Load Balancing: Layer 4 vs. Layer 7

A **Load Balancer (LB)** acts as a traffic cop sitting in front of your servers, distributing client requests across all servers capable of fulfilling those requests. In cloud architecture (like AWS), load balancing is divided into two primary types operating at different layers of the OSI model:

```
        CLIENT TRAFFIC
              │
              ▼
    ┌───────────────────┐
    │   LOAD BALANCER   │
    └─────────┬─────────┘
              ├──────────────────────────────┐
              ▼ (Layer 4)                    ▼ (Layer 7)
    ┌───────────────────┐          ┌───────────────────┐
    │     TCP / UDP     │          │   HTTP / HTTPS    │
    │  IPs & Ports Only │          │   Path / Header   │
    └───────────────────┘          └───────────────────┘
```

### Layer 4 Load Balancing (Transport Layer)

Layer 4 load balancing operates at the **Transport Layer** (TCP/UDP). It makes routing decisions purely based on network variables without inspecting the contents of the packet.

* **How it works:** The load balancer intercepts a packet and reads the **Source IP, Source Port, Destination IP, Destination Port, and Protocol**. It then applies a load-balancing algorithm (like Round Robin) and forwards the packet to a backend server.
* **SSL/TLS Handling:** It does not decrypt the packet. It simply forwards the encrypted byte stream directly to the target server, where SSL termination occurs.
* **Pros:**
  * **Ultra-high Performance:** Since it doesn't open or inspect the packet payload, it requires very little CPU. It can handle millions of requests per second with microsecond latency.
  * **Protocol Agnostic:** Can load balance any TCP/UDP traffic (e.g., database pools, SMTP, RTMP, gaming protocols).
* **Cons:**
  * **No Smart Routing:** Cannot route based on HTTP headers, cookies, or URL paths (e.g., cannot route `/api` to one pool and `/static` to another).
  * **No Cookie Sticky Sessions:** Cannot read HTTP cookies to pin a client to a specific server.

> [!TIP]
> **AWS Example — Network Load Balancer (NLB):**
> Use an AWS NLB when you need to handle sudden, volatile traffic spikes (millions of RPS), route non-HTTP protocols (such as SSH, database traffic, or MQTT), or preserve the client's source IP address all the way to the backend instances.

### Layer 7 Load Balancing (Application Layer)

Layer 7 load balancing operates at the **Application Layer** (HTTP/HTTPS, gRPC). It opens the packet and makes routing decisions based on the **actual content** of the request.

* **How it works:** The load balancer terminates the incoming TCP connection, decrypts the SSL/TLS session, and parses the HTTP request headers, cookie states, URL paths, and query parameters. It then makes a smart routing decision before establishing a new TCP connection to forward the request to the target server.
* **SSL/TLS Handling:** It performs **SSL Termination**—decrypting the incoming HTTPS request at the load balancer level so it can read the headers, then optionally re-encrypting it or forwarding it as plaintext HTTP to the backend server in a private VPC subnet.
* **Pros:**
  * **Smart / Content-Based Routing:** Can route traffic based on URL path (e.g., `/images/*` → S3 Target Group, `/api/*` → ECS Cluster), or Host headers (e.g., `app1.domain.com` → Group A, `app2.domain.com` → Group B).
  * **Sticky Sessions:** Can read or inject cookies to ensure a client stays connected to the same backend server (vital for stateful legacy apps).
  * **Security Integration:** Can integrate with Web Application Firewalls (WAF) to inspect payloads for SQL injection or cross-site scripting (XSS) before forwarding traffic.
* **Cons:**
  * **Higher Latency:** Decrypting, parsing, and re-encrypting packets requires significant CPU power, introducing millisecond-level latency.
  * **HTTP Centric:** Only supports application-layer protocols (HTTP, HTTPS, gRPC, WebSockets).

> [!TIP]
> **AWS Example — Application Load Balancer (ALB):**
> Use an AWS ALB for modern microservice architectures, containers (ECS/EKS), serverless functions (Lambda targets), and standard web applications where path-based routing, SSL termination, and WAF integration are required.

#### Summary Comparison

| Feature | Layer 4 Load Balancer (NLB) | Layer 7 Load Balancer (ALB) |
| :--- | :--- | :--- |
| **OSI Layer** | Layer 4 (Transport) | Layer 7 (Application) |
| **Routing Input** | IPs, Ports, Protocol | URL Path, Host Header, Cookies, Query Params |
| **Inspection** | None (acts as a packet forwarder) | Full (decrypts and parses HTTP request) |
| **Latency** | Microsecond range (extremely low) | Millisecond range (higher CPU utilization) |
| **AWS Product** | **Network Load Balancer (NLB)** | **Application Load Balancer (ALB)** |

---

## Firewalls and Security Devices by Layer

Different firewall generations inspect traffic at different layers, trading off performance against visibility:

| Firewall Type | OSI Layer(s) | What It Inspects | Notes |
| :--- | :--- | :--- | :--- |
| **Packet-Filtering Firewall** | L3 / L4 | Source/destination IP, port, protocol | Stateless — evaluates each packet independently against static rules (ACLs) |
| **Stateful Inspection Firewall** | L3 / L4 | Same as above, plus **connection state** | Tracks active connections (e.g., only allows inbound traffic that is a reply to an outbound request) |
| **Proxy / Circuit-Level Gateway** | L5 | Session-level handshake validity | Relays traffic on behalf of clients, hiding internal hosts entirely |
| **Application-Layer / Next-Gen Firewall (NGFW)** | L7 | Full payload — application identity, user identity, malware signatures | Can distinguish "this is Zoom traffic" from "this is a tunnel disguised as HTTPS on port 443" |
| **Web Application Firewall (WAF)** | L7 (HTTP specifically) | HTTP request/response bodies, headers, query strings | Purpose-built to catch SQL injection, XSS, and other web-specific attack payloads |

> [!NOTE]
> Cloud "Security Groups" (AWS) and "Network Security Groups" (Azure) are effectively stateful, virtual L3/L4 firewalls attached directly to instances or subnets — they are the cloud-native equivalent of a stateful inspection firewall.

---

## VPNs and Tunneling

A **VPN (Virtual Private Network)** creates an encrypted tunnel between two points across an untrusted network (like the public internet), making remote traffic behave as if it were on a private, local network.

### IPsec (Layer 3 VPN)

**IPsec** operates at the Network Layer and is the traditional standard for site-to-site VPNs (e.g., connecting an office network to a cloud VPC).

* **Two modes:**
  * **Transport Mode:** Encrypts only the payload of the original IP packet; the original IP header stays intact. Typically used for host-to-host communication.
  * **Tunnel Mode:** Encrypts the *entire* original IP packet (header included) and wraps it inside a new IP packet with new source/destination addresses. This is the standard mode for site-to-site VPNs.
* **Two core protocols:**
  * **AH (Authentication Header):** Provides integrity and authentication, but **no encryption** (rarely used alone today).
  * **ESP (Encapsulating Security Payload):** Provides encryption *and* authentication — the protocol actually used in almost all modern IPsec deployments.

### TLS-Based VPNs

Modern remote-access VPNs increasingly build their tunnel on top of TLS or a custom UDP-based protocol rather than IPsec, since it traverses firewalls and NAT more reliably:

* **OpenVPN:** Runs over TCP or UDP, uses the OpenSSL library for encryption; highly configurable but with more overhead.
* **WireGuard:** A newer, much simpler VPN protocol built into the Linux kernel, using modern cryptography (Curve25519, ChaCha20) and a minimal codebase — generally faster and easier to audit than IPsec or OpenVPN.

### Site-to-Site vs. Remote Access

| Type | Use Case | Typical Protocol |
| :--- | :--- | :--- |
| **Site-to-Site VPN** | Connects two networks permanently (e.g., branch office ↔ HQ, on-prem ↔ cloud VPC) | IPsec |
| **Remote-Access VPN** | Connects a single roaming device (laptop, phone) to a network | OpenVPN, WireGuard, IPsec/IKEv2 |

---

## Common Network Attacks by Layer

Building on the DNS Hijacking deep-dive above, here's how common attacks map onto the OSI stack:

| Layer | Attack | Description |
| :--- | :--- | :--- |
| **L7** | SQL Injection / XSS | Malicious input embedded in application data to manipulate a database or run scripts in a victim's browser |
| **L7** | HTTP Flood | Overwhelms a web server with a high volume of seemingly legitimate HTTP requests |
| **L7** | DNS Hijacking | Redirects DNS resolution to attacker-controlled servers (see full breakdown above) |
| **L6** | SSL Stripping / Downgrade Attack | Forces a connection to fall back to unencrypted HTTP or an outdated, weak TLS version |
| **L5** | Session Hijacking | Steals or predicts a valid session token/cookie to impersonate an authenticated user |
| **L4** | SYN Flood | Sends a flood of SYN packets without completing the handshake, exhausting the server's connection table |
| **L4** | Port Scanning | Probes a range of ports to discover which services are running and potentially vulnerable |
| **L3** | IP Spoofing | Forges the source IP address of a packet to impersonate a trusted host or hide the attacker's identity |
| **L3** | ICMP Flood / Ping of Death | Overwhelms a target with ICMP traffic, or sends malformed oversized ICMP packets to crash older systems |
| **L3** | BGP Hijacking | Announces false routing information to redirect internet traffic through an attacker-controlled path |
| **L2** | ARP Spoofing / Poisoning | Sends forged ARP replies to associate the attacker's MAC with a victim's IP, enabling man-in-the-middle attacks |
| **L2** | MAC Flooding | Floods a switch's MAC address table until it overflows and starts broadcasting all traffic (defeating switch isolation) |
| **L2** | VLAN Hopping | Exploits trunk port misconfiguration to access traffic on a VLAN the attacker shouldn't reach |
| **L1** | Wiretapping / Cable Tapping | Physically intercepts signals from a cable or fiber line |
| **L1** | Jamming | Floods a wireless frequency with noise to deny legitimate radio communication |

---

## Common Port Numbers Reference

| Port | Protocol | Service |
| :--- | :--- | :--- |
| **20 / 21** | TCP | FTP (data / control) |
| **22** | TCP | SSH (Secure Shell) |
| **23** | TCP | Telnet (unencrypted — avoid) |
| **25** | TCP | SMTP (outbound mail) |
| **53** | UDP / TCP | DNS |
| **67 / 68** | UDP | DHCP (server / client) |
| **80** | TCP | HTTP |
| **110** | TCP | POP3 |
| **123** | UDP | NTP (time sync) |
| **143** | TCP | IMAP |
| **161 / 162** | UDP | SNMP (monitoring) |
| **389** | TCP | LDAP |
| **443** | TCP | HTTPS |
| **445** | TCP | SMB (Windows file sharing) |
| **465 / 587** | TCP | SMTPS / SMTP Submission (encrypted mail) |
| **993** | TCP | IMAPS |
| **995** | TCP | POP3S |
| **3306** | TCP | MySQL |
| **3389** | TCP | RDP (Remote Desktop) |
| **5432** | TCP | PostgreSQL |
| **6379** | TCP | Redis |
| **8080** | TCP | HTTP alternate (common for dev servers / proxies) |
| **27017** | TCP | MongoDB |

---

## DevOps Reference: Troubleshooting Tools by Layer

Having a structured mental model allows you to isolate issues systematically.

| Layer | Focus | Command-Line Tools | Common Issues |
| :--- | :--- | :--- | :--- |
| **L7** | Application | `curl`, `wget`, `dig`, `nslookup` | HTTP errors, DNS resolution failures |
| **L6** | Presentation | `openssl s_client`, `curl -v` | SSL/TLS handshake failures, certificate expiry |
| **L5** | Session | Web consoles, Cookie inspectors | Session expiry, unauthorized access |
| **L4** | Transport | `nc`, `telnet`, `netstat`, `ss`, `nmap` | Closed ports, blocked firewall rules |
| **L3** | Network | `ping`, `traceroute`, `ip route`, `mtr` | Routing loops, wrong gateway configurations |
| **L2** | Data Link | `arp`, `ip link`, `bridge` | ARP mismatch, VLAN misconfigurations |
| **L1** | Physical | `ethtool`, `dmesg`, `ifconfig` | Bad cables, disconnected links, hardware errors |

> [!TIP]
> A good debugging habit is to work **bottom-up**: confirm the link light (L1) → confirm ARP resolves the gateway (L2) → confirm you can `ping` the gateway and destination (L3) → confirm the port is reachable with `nc -zv` (L4) → confirm `curl -v` gets a clean TLS handshake (L6) → confirm the HTTP response itself makes sense (L7).

---

## Glossary of Key Terms

| Term | Definition |
| :--- | :--- |
| **ARP** | Address Resolution Protocol — maps an IP address to a MAC address on a local network |
| **CIDR** | Classless Inter-Domain Routing — notation (`/24`) expressing how many bits of an IP address are the network portion |
| **DHCP** | Dynamic Host Configuration Protocol — automatically assigns IP addresses and network config to devices |
| **DNS** | Domain Name System — translates human-readable domain names into IP addresses |
| **FQDN** | Fully Qualified Domain Name — a complete domain name specifying its exact location in the DNS hierarchy (e.g., `www.google.com`) |
| **ICMP** | Internet Control Message Protocol — used for diagnostic and error messages (e.g., `ping`, `traceroute`) |
| **MAC Address** | Media Access Control address — a hardware identifier burned into a network interface card |
| **MTU** | Maximum Transmission Unit — the largest packet size a network link can carry without fragmentation |
| **NAT** | Network Address Translation — allows multiple private IPs to share one public IP address |
| **OSI Model** | Open Systems Interconnection model — the 7-layer conceptual framework for network communication |
| **QUIC** | A UDP-based transport protocol with built-in encryption, used as the foundation for HTTP/3 |
| **Socket** | The combination of an IP address, port number, and protocol that uniquely identifies a connection endpoint |
| **TCP** | Transmission Control Protocol — a reliable, connection-oriented transport protocol |
| **TLS** | Transport Layer Security — the cryptographic protocol that secures HTTPS and other traffic (successor to SSL) |
| **TTL** | Time to Live — a counter that limits how many hops a packet can traverse before being discarded |
| **UDP** | User Datagram Protocol — a fast, connectionless transport protocol with no delivery guarantees |
| **VLAN** | Virtual LAN — a logically isolated broadcast domain within one or more physical switches |
| **VPN** | Virtual Private Network — an encrypted tunnel connecting two points over an untrusted network |
| **WAF** | Web Application Firewall — an L7 firewall purpose-built to catch web-specific attacks like SQL injection and XSS |