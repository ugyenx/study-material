
# Technical Study Notes

> A structured collection of concepts learned from the original study conversation.

## Table of Contents

* [1. Why Networking Needs Models](#1-why-networking-needs-models)
* [2. The OSI Model — Overview](#2-the-osi-model--overview)
* [3. The 7 OSI Layers in Detail](#3-the-7-osi-layers-in-detail)
* [4. Encapsulation and Decapsulation](#4-encapsulation-and-decapsulation)
* [5. A Complete Webpage Request (Full Walkthrough)](#5-a-complete-webpage-request-full-walkthrough)
* [6. OSI Model vs TCP/IP Model](#6-osi-model-vs-tcpip-model)
* [7. Networking Devices by Layer](#7-networking-devices-by-layer)
* [8. Troubleshooting Using OSI Layers](#8-troubleshooting-using-osi-layers)
* [9. MAC Address vs IP Address](#9-mac-address-vs-ip-address)
* [10. DNS (Domain Name System)](#10-dns-domain-name-system)
* [11. DHCP (Dynamic Host Configuration Protocol)](#11-dhcp-dynamic-host-configuration-protocol)
* [12. ARP (Address Resolution Protocol)](#12-arp-address-resolution-protocol)
* [13. TCP (Transmission Control Protocol)](#13-tcp-transmission-control-protocol)
* [14. UDP (User Datagram Protocol)](#14-udp-user-datagram-protocol)
* [15. HTTP and HTTPS](#15-http-and-https)
* [16. TLS (Transport Layer Security)](#16-tls-transport-layer-security)
* [Final Revision](#final-revision)
* [Glossary](#glossary)

---

# 1. Why Networking Needs Models

## 1.1 Core Concept

Computers only understand electrical, light, or radio signals representing 1s and 0s. Networking exists to solve one core problem: how do you take something meaningful (like "show me example.com") and turn it into signals that can travel across wires, fiber, or air — then turn it back into something meaningful on the other end?

## 1.2 Why It Matters

Without agreed-upon rules (**protocols**), data could physically arrive but nobody would know where it came from, where it's going, what's inside it, whether it arrived correctly, or what to do with it. Protocols are agreements: "if you want to talk to me, do it this way."

## 1.3 How It Works — The Layering Idea

Networking is broken into independent **layers**, each with one specific job. A layer doesn't need to know how the layers above or below it do their jobs — only how to hand data to them. This is called **separation of concerns**.

**Analogy — mailing a letter:**
1. You write a letter (the message/content)
2. You put it in an envelope addressed to your friend
3. You put that envelope in a mailbox
4. The postal service routes it through trucks/planes based on the address
5. It arrives at the local post office, then the mailbox
6. Your friend opens the envelope and reads the letter

The postal worker never reads the letter's content — they only look at the address. Similarly, each networking layer only looks at its own header information to do its job, regardless of what's inside the layers above it.

**Analogy — company departments:** Sales, Accounting, Shipping, and IT don't need to know how each other works internally — they just need a consistent handoff between them. This means, e.g., WiFi can be swapped for Ethernet, or IPv4 upgraded to IPv6, without redesigning how browsers work.

> **Note — corrected misconception:** An "envelope" in this analogy is NOT encryption. It represents **addressing/header information** (who it's from, who it's to, how to handle it) — not content-hiding. Encryption is a separate, specific function (handled at Layer 6 / via TLS), not something that happens at every layer.

> **Note — corrected misconception:** The reason each layer doesn't need to know what's inside other layers isn't about "security" — it's about **separation of concerns / modularity**. A router doesn't need to know if data is a video or webpage; it just needs the address to move it forward.

## 1.4 Key Takeaways

* Networking's core problem: turning meaningful information into transmittable signals and back again.
* Protocols = agreed-upon rules for communication.
* Layers exist for separation of concerns: each layer does one job, independent of the others.
* The postal/envelope analogy: envelope = addressing/header info, letter = actual data — not encryption.

---

# 2. The OSI Model — Overview

## 2.1 Core Concept

**OSI (Open Systems Interconnection Model)**, created by ISO in 1984, is a **conceptual blueprint** — not software or hardware — describing 7 stages ("layers") that networking systems should follow so that any two systems, regardless of manufacturer, can communicate.

## 2.2 Why It Matters (Historical Context)

Before OSI, different companies (IBM, DEC, Xerox, etc.) built incompatible networking systems that couldn't communicate with each other — similar to different countries having incompatible postal requirements. OSI standardized the "shape of the plug," analogous to standardizing electrical outlets so any manufacturer's device can connect.

## 2.3 What OSI Is NOT

* Not a physical thing you install.
* Not exactly how the modern internet works internally (the internet runs on the **TCP/IP model** — see Section 6).
* It is a **teaching and design/reference tool**, used heavily in troubleshooting and certifications even though real implementations don't follow it with strict 1:1 separation.

## 2.4 The 7 Layers (Map)

```
Layer 7 → Application     (closest to the user)
Layer 6 → Presentation
Layer 5 → Session
Layer 4 → Transport
Layer 3 → Network
Layer 2 → Data Link
Layer 1 → Physical        (closest to the actual wire/signal)
```

**Mnemonic (bottom to top):** *"Please Do Not Throw Sausage Pizza Away"* → Physical, Data Link, Network, Transport, Session, Presentation, Application.

## 2.5 Key Takeaways

* OSI = 7-layer **reference/teaching model**, created to solve vendor incompatibility.
* Each layer only talks to the layer directly above/below it — never skips around.
* Real-world engineers still say "Layer 3 issue" or "Layer 7 firewall" using OSI terminology even though the actual internet runs on TCP/IP.

---

# 3. The 7 OSI Layers in Detail

For each layer: job, why it exists, what happens, data unit name, protocols, devices, analogy, example, and what breaks without it.

## 3.1 Layer 1 — Physical Layer

| Aspect | Detail |
|---|---|
| Main job | Move raw 1s and 0s as physical signals (electrical, light, or radio) |
| Why it exists | Data must physically travel through a medium; something must define how a "1" or "0" is physically represented |
| What happens | Digital data converted to physical signals (voltage/light/radio) and back |
| Data unit | **Bits** |
| Protocols/standards | Ethernet cabling standards (Cat5e, Cat6), USB, fiber standards, WiFi (802.11) |
| Devices | Hubs (mostly obsolete), cables, NICs, wireless radios/antennas |
| Analogy | The road itself — doesn't care what's inside the truck |
| Example | Ethernet cable carrying electrical signals; WiFi antenna sending radio waves |
| Without it | No way to physically transmit anything — no cables, no radio waves, nothing to run the layers above on |

## 3.2 Layer 2 — Data Link Layer

| Aspect | Detail |
|---|---|
| Main job | Organize raw bits into **frames**; enable communication between devices directly connected on the same local network |
| Why it exists | Layer 1 has no concept of "where does one piece of data start/end" or "whose data is this" |
| What happens | Data wrapped into a frame with sender/recipient **MAC addresses** + basic error-checking; handles only the "next hop," not the final destination |
| Data unit | **Frame** |
| Protocols | Ethernet (wired), WiFi/802.11 (wireless), ARP |
| Devices | Switches (use MAC addresses to forward frames), NICs (each has a unique MAC address) |
| Analogy | Street addresses within a single neighborhood — knows the house on the block, not how to reach a different city |
| Example | Laptop sends data to its WiFi router labeled with laptop's MAC (sender) and router's MAC (recipient) |
| Without it | Devices on the same local network couldn't identify which specific device raw bits are meant for |

> **Important:** MAC addresses only work for devices on the *same local network*. Once data must leave the local network, IP addresses (Layer 3) are needed.

## 3.3 Layer 3 — Network Layer

| Aspect | Detail |
|---|---|
| Main job | Get data from one device to a device on a **completely different network** — addressing and routing across the internet |
| Why it exists | MAC addresses only work locally; the internet is a network of networks requiring a globally meaningful addressing scheme |
| What happens | Data wrapped into a **packet** with source/destination **IP addresses**; routers examine the destination IP and forward toward it |
| Data unit | **Packet** |
| Protocols | IP (Internet Protocol), ICMP (used by `ping`) |
| Devices | Routers (examine destination IP, decide forwarding direction using a routing table) |
| Analogy | Full postal address system across countries/cities — routed hop by hop, each router only deciding "which direction next" |
| Example | Visiting https://example.com — packet labeled with your IP (source) and example.com's server IP (destination) |
| Without it | Communication would be limited to your own local network only |

> **Correction noted during learning:** A router does not calculate the entire shortest path in advance — it makes a **local decision** per packet: "given this destination IP, which direction/next router should this go to?" based on its own routing table. Routing is a series of independent, local decisions, not one grand pre-calculated plan.

## 3.4 Layer 4 — Transport Layer

| Aspect | Detail |
|---|---|
| Main job | Deliver data reliably (or quickly) to the correct **application** on the destination device — app-to-app, not just device-to-device |
| Why it exists | A device can run many applications simultaneously; Layer 3 has no way to know which application data is meant for |
| What happens | Data wrapped into a **segment** (TCP) or **datagram** (UDP) with source/destination **port numbers**; TCP also handles ordering, acknowledgment, retransmission |
| Data unit | **Segment** (TCP) / **Datagram** (UDP) |
| Protocols | TCP (reliable, ordered, connection-based), UDP (fast, no delivery/order guarantee, connectionless) |
| Devices | Mostly software (OS networking stack); firewalls sometimes filter here based on ports |
| Analogy | Apartment/suite number within a building — Layer 3 gets you to the building, Layer 4 gets you to the right resident. TCP = registered mail with tracking; UDP = dropping something in the mail with no confirmation |
| Example | Browser uses TCP to port 443 (HTTPS) for a webpage; a video call often uses UDP |
| Without it | Data would arrive at the device with no way to sort it into the correct application; no guarantee of complete/undamaged delivery |

> **Correction noted during learning:** TCP reliability isn't just "not corrupted" — it also guarantees **ordering** (data arrives in the sequence sent) and **retransmission** (lost pieces are re-sent). UDP has none of these guarantees.

## 3.5 Layer 5 — Session Layer

| Aspect | Detail |
|---|---|
| Main job | Open, manage, and close a "conversation" (**session**) between two devices, keeping both sides in sync |
| Why it exists | Without session tracking, any interruption would force restarting a conversation from scratch |
| What happens | Establishes a session, tracks state (e.g., login state, progress in a multi-step exchange), coordinates orderly close |
| Data unit | No distinct standardized name — generally just "data" |
| Protocols/concepts | NetBIOS, RPC (Remote Procedure Call); in modern practice, session-like behavior (e.g. staying logged in) is often handled at Layer 7 via cookies/tokens |
| Devices | None — purely software/protocol concept |
| Analogy | A phone call: dialing/connecting = establishing session; talking = maintaining session; hanging up properly = closing session |
| Example | Logging into online banking — session tracks that your browser is authenticated across multiple page views |
| Without it | Every exchange would be treated as a fresh, disconnected interaction with no memory of what happened before; multi-step processes (banking, gaming, video calls) would be very difficult |

> **Note:** This is one of the layers where OSI's theoretical model doesn't map cleanly onto real implementations — much session-like behavior today (staying logged in) is implemented at the Application layer (Layer 7) via cookies/session tokens rather than a strictly separate Layer 5 protocol.

## 3.6 Layer 6 — Presentation Layer

| Aspect | Detail |
|---|---|
| Main job | Ensure data is in a format both sender and receiver understand — **translation, encryption, compression** |
| Why it exists | Different systems may represent data differently internally; data may need securing or shrinking before transmission |
| What happens | Translation (character encoding), encryption/decryption, compression/decompression |
| Data unit | No distinct standardized name — generally "data" |
| Protocols/concepts | SSL/TLS encryption (conceptually maps here, though TLS spans Session/Presentation concepts), ASCII/UTF-8, JPEG/MPEG formats |
| Devices | None — software/protocol level only |
| Analogy | Writing a letter in English but translating it to French for a friend; writing in code (encryption) so only the friend with the key can read it; abbreviating (compression) to save paper |
| Example | HTTPS's "S" means data is encrypted via TLS before sending — conceptually a Presentation-layer function |
| Without it | Data could arrive intact but be uninterpretable by the receiving system; could be sent insecurely or inefficiently |

## 3.7 Layer 7 — Application Layer

| Aspect | Detail |
|---|---|
| Main job | Provide the actual interface/services letting applications (browser, email client, etc.) communicate over the network — the user-facing layer |
| Why it exists | Lower layers move data around but don't know what the data *means*; Layer 7 defines rules for specific communication types |
| What happens | Constructs the actual request/content (e.g., an HTTP request) before handing it down to Layer 6 |
| Data unit | Just "data" (e.g., the HTTP request itself) |
| Protocols | HTTP/HTTPS, SMTP/IMAP/POP3 (email), FTP, DNS, DHCP |
| Devices | None — entirely software (browsers, email clients, apps) |
| Analogy | The actual content/purpose of a letter — what you're asking for, not how it's delivered |
| Example | Typing https://example.com builds an HTTP GET request: `GET / HTTP/1.1, Host: example.com` |
| Without it | Even with perfect lower-layer delivery, there'd be no standard "language" for applications to request things from each other |

## 3.8 Layer Summary Table

| Layer | Name | Main Job | Data Unit | Example Protocol | Example Device |
|---|---|---|---|---|---|
| 7 | Application | User-facing interface/services | Data | HTTP, DNS, SMTP | None (software) |
| 6 | Presentation | Translation, encryption, compression | Data | TLS/SSL, JPEG | None (software) |
| 5 | Session | Manage session state | Data | NetBIOS, RPC | None (software) |
| 4 | Transport | App-to-app delivery, reliability | Segment/Datagram | TCP, UDP | Firewalls (some) |
| 3 | Network | Cross-network addressing/routing | Packet | IP | Router |
| 2 | Data Link | Local addressing/framing | Frame | Ethernet, WiFi, ARP | Switch |
| 1 | Physical | Raw signal transmission | Bits | Ethernet cabling, 802.11 | Hub, cables, NICs |

## 3.9 Mental Models (One-Word Essence per Layer)

* **L1 (Physical):** *Transmit* — representing 0s/1s as physical signals
* **L2 (Data Link):** *Frame* — framing bits, adding MAC addresses, error-checking
* **L3 (Network):** *Route* — packeting, IP addressing, routing
* **L4 (Transport):** *Deliver* — app-to-app delivery using port numbers
* **L5 (Session):** *Remember* — maintaining session/login state between nodes
* **L6 (Presentation):** *Format* — translation, encryption, compression
* **L7 (Application):** *Interface* — provides interface/services for apps to communicate

---

# 4. Encapsulation and Decapsulation

## 4.1 Core Concept

**Encapsulation** = wrapping data with a new header (and sometimes trailer) as it moves **down** through each layer on the sending device. **Decapsulation** = the reverse — removing each header/trailer as data moves **up** through each layer on the receiving device.

## 4.2 The "Package Inside Package" Analogy

Shipping a valuable item internationally:
1. Item in a small box (actual data/message)
2. Small box inside a padded envelope noting "Room 402" (≈ Layer 4 — port/app addressing)
3. Padded envelope inside a shipping box with full destination address (≈ Layer 3 — IP address)
4. Shipping box handed to a courier truck with a local routing tag for "next depot" (≈ Layer 2 — MAC address, next hop only)
5. Truck physically drives on roads (≈ Layer 1 — physical transmission)

At the destination, the process reverses exactly. Critically, **no intermediate handler opens the small box itself** — each only reads/acts on its own layer's label (separation of concerns).

## 4.3 The Encapsulation Chain (Going DOWN) — HTTPS Example

```
LAYER 7 (Application)
  GET / HTTP/1.1
  Host: example.com                <- Application Data

        ↓

LAYER 6 (Presentation) — encrypts/formats
  [ENCRYPTED: xK9$mP...]           <- still "Application Data", now encrypted

        ↓

LAYER 5 (Session) — session info tracked

        ↓

LAYER 4 (Transport) — adds TCP header
  TCP HEADER (Src Port:51000, Dst Port:443) + [ENCRYPTED DATA]   <- SEGMENT

        ↓

LAYER 3 (Network) — adds IP header
  IP HEADER (Src IP:192.168.1.5, Dst IP:93.184.216.34)
    + TCP HEADER + [ENCRYPTED DATA]                              <- PACKET

        ↓

LAYER 2 (Data Link) — adds Frame header + trailer (FCS)
  FRAME HEADER (Src MAC, Dst MAC) + IP HEADER + TCP HEADER
    + [DATA] + FCS                                               <- FRAME

        ↓

LAYER 1 (Physical) — converts entire frame into raw signal
  1010110100101101011010010110101101...                         <- BITS
```

Each layer only adds its own header, wrapping the entire previous package, without touching or looking inside what's already wrapped — like nested Russian dolls.

## 4.4 The Decapsulation Chain (Going UP, at the destination)

```
LAYER 1 (Physical): raw bits received → reconstructed into a frame (per L2 rules)
        ↑
LAYER 2 (Data Link): reads frame header → checks "is Dst MAC me?" → yes →
                      strips frame header/trailer → passes packet up
        ↑
LAYER 3 (Network): reads IP header → checks "is Dst IP me?" → yes →
                    strips IP header → passes segment up
        ↑
LAYER 4 (Transport): reads TCP header → checks port → confirms
                      complete/ordered delivery → strips → passes data up
        ↑
LAYER 5 (Session): confirms this belongs to an active/new session
        ↑
LAYER 6 (Presentation): decrypts data → now readable
        ↑
LAYER 7 (Application): reads "GET / HTTP/1.1, Host: example.com"
```

> **Important clarification (from Q&A):** Layer 1 does not understand frames — it just converts signals into a raw stream of bits with zero concept of frame boundaries. Layer 2 **interprets/reconstructs** that raw bitstream into a frame using known frame-format rules (locating header fields, running the error check), then strips the header/trailer and passes the payload up. The general rule: **going down, each layer wraps (adds a header). Going up, each layer doesn't just "unwrap" — it interprets the raw data according to its own format rules, extracts its header info, and passes the remainder up.**

## 4.5 PDU (Protocol Data Unit) Reference

Each layer's packaged data has a formal name:

```
Layer 7/6/5 →  Data
Layer 4      →  Segment (TCP) or Datagram (UDP)
Layer 3      →  Packet
Layer 2      →  Frame
Layer 1      →  Bits
```

**Mental model:** Russian nesting dolls — going down = dolls nested inside bigger dolls (headers added); going up = dolls unstacked one at a time (headers removed) until reaching the original content.

## 4.6 Key Insight: Frame Changes at Every Hop, Packet Doesn't

```
[Laptop] --frame1(MAC: laptop→router1)--> [Router1]
[Router1] strips frame1, reads packet, decides next hop, builds NEW frame2
[Router1] --frame2(MAC: router1→router2)--> [Router2]
[Router2] strips frame2, reads packet, builds frame3...
[Router2] --frame3(MAC: router2→server)--> [Server]
```

The **IP packet** (Layer 3) — with its original source and destination IP — stays untouched and identical the entire journey. Only the **frame** (Layer 2) is stripped and rebuilt fresh at every hop, because MAC addresses are only meaningful for the immediate next device.

A router reading only Layer 3 (IP) info to forward a packet does **not** need to decrypt Layer 6 (encrypted) content — separation of concerns applies here too.

## 4.7 Key Takeaways

* Encapsulation = wrapping headers on the way down; decapsulation = interpreting/removing them on the way up.
* PDU names: Data → Segment/Datagram → Packet → Frame → Bits.
* Frame (Layer 2) is rebuilt at every hop; Packet (Layer 3) remains unchanged end-to-end.
* Each layer only processes its own header — never needs to inspect content above its own scope.

---

# 5. A Complete Webpage Request (Full Walkthrough)

## 5.1 Step 0 — DNS Lookup

Before anything else, the device needs the IP address for the domain, since routing uses IP addresses, not names. This is its own full round-trip through the layers, typically via UDP.

```
[Your Laptop] --"what's the IP for example.com?"--> [DNS Server]
[Your Laptop] <--"93.184.216.34"------------------- [DNS Server]
```

## 5.2 Step 1 — TCP Three-Way Handshake

Establishes a reliable connection on port 443 before any actual request is sent:

```
[Laptop] ---------- SYN ----------------> [Server]   "Can we start a connection?"
[Laptop] <------- SYN-ACK --------------- [Server]   "Yes, ready. Are you?"
[Laptop] ---------- ACK ----------------> [Server]   "Confirmed, let's begin."
```

## 5.3 Step 2 — TLS Handshake (HTTPS only)

Happens on top of the TCP connection:
* Browser sends supported encryption methods ("ClientHello")
* Server responds with its certificate + chosen cipher suite
* Browser verifies the certificate
* Both sides agree on shared encryption keys

Only after DNS + TCP handshake + TLS handshake is the browser ready to send the real request.

## 5.4 Step 3 — The Actual HTTP Request (Down the Stack)

```
LAYER 7: Browser builds: "GET / HTTP/1.1, Host: example.com"
LAYER 6: Encrypts using TLS keys from Step 2
LAYER 5: Session context maintained
LAYER 4: TCP segment (Src port 51000, Dst port 443)
LAYER 3: IP packet (Src IP 192.168.1.5, Dst IP 93.184.216.34)
LAYER 2: Frame (Src MAC laptop, Dst MAC router — next hop only)
LAYER 1: Converted to radio signal, transmitted
```

## 5.5 Step 4 — Server Processes the Request (Up the Stack)

```
LAYER 1: Signal received, converted to bits
LAYER 2: Frame reconstructed, Dst MAC confirmed as server's own → strip
LAYER 3: Packet read, Dst IP confirmed as server's own → strip
LAYER 4: TCP segment read, port 443 → routed to web server software → strip
LAYER 5: Session recognized
LAYER 6: Data decrypted using shared TLS keys → readable
LAYER 7: Server reads: "GET / HTTP/1.1, Host: example.com" → prepares response
```

## 5.6 Step 5 — Response Returns (Reversed Roles)

Server becomes the "sender": builds HTTP response → encrypts → TCP segment (Src port 443, Dst port 51000) → IP packet (Src IP server, Dst IP laptop) → frame (next hop from server side) → transmitted as signal. Travels hop by hop back to the laptop, which decapsulates up through Layers 1→7 to obtain the raw HTML.

> **Why source/destination ports swap:** In the response, the server is now sending — so it uses its own port (443) as source and the browser's original ephemeral port (51000) as destination, mirroring the reversed sender/receiver roles.

## 5.7 Step 6 — Rendering and Additional Resources

Browser parses HTML, discovers additional resources (CSS, JS, images), and repeats the process for each. Same-domain resources reuse the already-open TCP/TLS connection (**persistent/keep-alive connections**) rather than repeating the full DNS→TCP→TLS setup — only the *first* request to a domain pays that full setup cost. Resources from a *different* domain (e.g., a CDN) require their own fresh DNS/TCP/TLS setup, which is part of why loading resources from many domains can feel slower.

## 5.8 Full Picture Diagram

```
DNS lookup → TCP handshake → TLS handshake → HTTP request (down 7→1)
                                                        |
                                                        v
                                            [travels across internet]
                                                        |
                                                        v
                              Server receives (up 1→7) → processes
                                                        |
                                                        v
                              Server sends response (down 7→1)
                                                        |
                                                        v
                                            [travels across internet]
                                                        |
                                                        v
              Your laptop receives (up 1→7) → browser renders page
```

## 5.9 Key Takeaways

* Order of operations: DNS → TCP handshake → TLS handshake → actual HTTP request/response.
* Persistent connections avoid repeating full setup for every resource on the same domain.
* Each direction (request, response) fully repeats the down-then-up 7-layer journey.

---

# 6. OSI Model vs TCP/IP Model

## 6.1 Core Concept

**OSI (7 layers)** is a theoretical, universal reference model designed for teaching/precision. **TCP/IP (4, sometimes described as 5 layers)** is the model actually implemented in the real internet, developed practically alongside ARPANET.

## 6.2 Why It Matters

TCP/IP compresses some of OSI's layers together because, in practice, that level of separation wasn't necessary for real implementations. OSI optimizes for **conceptual clarity**; TCP/IP optimizes for **practical implementation** — neither is "wrong," they serve different purposes.

## 6.3 Side-by-Side Mapping

```
        OSI MODEL                      TCP/IP MODEL
   ---------------------          ---------------------
   Layer 7: Application     ┐
   Layer 6: Presentation    ├───→   Application Layer
   Layer 5: Session         ┘
   ---------------------          ---------------------
   Layer 4: Transport       ───→   Transport Layer
   ---------------------          ---------------------
   Layer 3: Network         ───→   Internet Layer
   ---------------------          ---------------------
   Layer 2: Data Link       ┐
   Layer 1: Physical        ├───→   Network Access Layer
                             ┘      (also called Link Layer)
```

TCP/IP merges OSI Layers 5, 6, 7 into one Application Layer (session management and encryption handled within applications themselves, e.g. browsers handling TLS/cookies internally). TCP/IP also merges OSI Layers 1 and 2 into one Network Access Layer.

## 6.4 Comparison Table

| Aspect | OSI Model | TCP/IP Model |
|---|---|---|
| Number of layers | 7 | 4 (sometimes 5) |
| Origin | Theoretical standard by ISO (1984) | Practical model from ARPANET/internet development |
| Used for | Teaching, troubleshooting, reference | What the real internet actually runs on |
| Session/Presentation | Separate layers (5, 6) | Folded into Application layer |
| Physical/Data Link | Separate layers (1, 2) | Folded into Network Access layer |

## 6.5 Why This Matters in Practice

When people say "Layer 3 device" or "Layer 7 firewall," they're using **OSI terminology**, even though the internet runs on TCP/IP — OSI's precise separation makes it a better teaching/troubleshooting/reference tool, so it's still used constantly in real jobs and certifications.

## 6.6 Common Misconceptions

* ❌ TCP/IP is theoretical and OSI is what the internet actually uses.
* ✅ It's the opposite: **TCP/IP is what's actually implemented; OSI is the theoretical/reference model.**

---

# 7. Networking Devices by Layer

## 7.1 Layer 1 Devices — Physical

**Hub** (mostly obsolete): repeats/broadcasts every electrical signal to **all** connected devices with zero intelligence — no understanding of MAC/IP addresses. Analogy: shouting a message into a room hoping the right person hears it.

**Cables, connectors, NICs, radios**: also Layer 1, carrying raw physical signal.

## 7.2 Layer 2 Devices — Data Link

**Switch**: reads the MAC address in each incoming frame and forwards only to the intended device, using a **MAC address table** mapping MAC addresses to physical ports. Analogy: a smart receptionist routing your call to the exact right office.

**Wireless Access Point** (partially): handles frames/MAC addresses (Layer 2); consumer WiFi routers are often multi-layer devices (also doing Layer 3 routing).

## 7.3 Layer 3 Devices — Network

**Router**: reads the destination IP address and decides which direction to forward — potentially across different networks — using a **routing table**. Connects home networks to the wider internet. Analogy: a highway interchange reading your destination city and directing you to the correct highway.

**Layer 3 switch**: an advanced switch that also performs some routing — common in larger business networks.

## 7.4 Layer 4 Devices — Transport

Mostly software-based:

**Firewalls**: often filter traffic based on port numbers (e.g., blocking port 23/Telnet, allowing port 443/HTTPS). "Next-gen"/"Layer 7 firewalls" also inspect application content.

**Load balancers**: distribute traffic across servers based on IP/port info (Layer 4) or actual request content like URL path (Layer 7).

## 7.5 Layer 7 Devices — Application

Mostly software:

**Layer 7 Firewalls / Web Application Firewalls (WAFs)**: inspect actual application content (e.g., HTTP requests) to block specific attacks, not just IP/port-based filtering.

**Proxy servers**: operate at the application layer, handling/modifying actual application-level requests on behalf of a client.

## 7.6 Device Summary Table

| Device | Primary OSI Layer | What It Reads/Uses |
|---|---|---|
| Hub | Layer 1 | Nothing — just repeats signals |
| Switch | Layer 2 | MAC addresses |
| Router | Layer 3 | IP addresses |
| Layer 4 Firewall | Layer 4 | Port numbers |
| Layer 7 Firewall / WAF | Layer 7 | Actual application content |
| Proxy Server | Layer 7 | Application-level requests |

## 7.7 Key Realization

A "Layer 7 device" doesn't skip lower layers — it still processes Layers 1 through 6 first to reach the Layer 7 content; its *primary decision-making logic* just happens at that point. By contrast, a Layer 3 device (router) never needs to decapsulate up to Layer 6/7 — it only needs its own layer's header info to do its job.

> **Refinement on router vs switch scope:** A router's job is to get traffic to the correct **destination network**; once on that local network, it's the switch (Layer 2) that delivers to the exact device via MAC address. Router = right network; Switch = right device within that network.

---

# 8. Troubleshooting Using OSI Layers

## 8.1 General Approach

Use a **bottom-up approach**: check Layer 1 first (is anything physically connected/working?), then move upward. This avoids wasting time debugging application-layer issues when the real problem is a loose cable or disabled radio.

## 8.2 Problem: WiFi Isn't Connecting

* **Layer(s):** 1 (Physical), 2 (Data Link)
* **Why:** Can't even connect to the network — before getting an IP or sending real data.
* **Checks:** WiFi enabled/airplane mode off (L1); signal strength (L1); correct WiFi password (L2, WPA2/WPA3 handshake).
* **Tools:** `netsh wlan show interfaces` (Windows), `iwconfig` (Linux).
* **Interpretation:** "No networks found"/"can't connect" = stuck at Layer 1/2.

## 8.3 Problem: I Have WiFi But No Internet

* **Layer(s):** 3 (Network)
* **Why:** Layer 1/2 succeeded (connected to WiFi), but no valid IP or the router can't route traffic externally.
* **Checks:**
  * `ipconfig` (Windows) / `ifconfig` or `ip addr` (Linux/Mac) — a `169.254.x.x` address means **DHCP failed** and the device self-assigned via **APIPA (Automatic Private IP Addressing)**, pointing to a local-network Layer 3 configuration failure.
  * `ping 192.168.1.1` (router) — success means local network is fine.
  * `ping 8.8.8.8` (external) — failure here (with local ping succeeding) points to an ISP-side issue.
* **Interpretation:** Local ping succeeds + external ping fails = routing issue beyond the router (likely ISP). Local ping fails = own Layer 3 config issue.

## 8.4 Problem: I Can Ping an IP But Can't Open a Website

* **Layer(s):** 7 (Application) — specifically likely a **DNS** issue.
* **Why:** Successful ping to an IP proves Layers 1–3 work fine; failure to open a site by name points above Layer 3.
* **Checks:** `ping example.com` (does the name resolve?); `nslookup example.com` or `dig example.com` (directly query DNS); try accessing via raw IP directly.
* **Interpretation:** Domain-based access fails but raw-IP access works = confirmed DNS-specific issue, not deeper connectivity.

## 8.5 Problem: A Website Works by IP But Not by Domain Name

Same underlying cause/diagnosis as 8.4 — a **DNS resolution issue**, confirmed the same way (`nslookup`/`dig`, comparing IP-based vs domain-based access).

## 8.6 Problem: Two Computers on the Same Network Can't Communicate

* **Layer(s):** 2 (Data Link), possibly 3 (Network)
* **Why:** Devices on the same local network should reach each other via Layer 2 (MAC) without a router; failure suggests a Layer 2 issue (not actually on the same segment, switch/VLAN misconfiguration) or Layer 3 issue (mismatched subnet/IP configuration).
* **Checks:** Compare subnets/IP+subnet mask; `ping` between local IPs directly; check firewall settings on either device; `arp -a` to see if MAC addresses resolve (failure points to Layer 2/VLAN/switch issue).
* **Interpretation:** Mismatched subnets = Layer 3 issue. Same subnet but ARP not resolving = Layer 2 issue.

## 8.7 Problem: The Network Is Extremely Slow

* **Layer(s):** Potentially any — requires elimination.
* **Checks:** WiFi signal strength (L1); `ping` for latency/packet loss (L1–3); `tracert`/`traceroute` to find exactly which hop introduces delay; check if slowness is site-specific (likely Layer 7/server-side, not your network) vs. everywhere (Layer 1–3 issue).
* **Interpretation:** Slow everywhere + high latency to own router = local Layer 1/2 issue. Slow everywhere but fine to router = Layer 3 issue further out (ISP). Slow on one site only = likely server-side (Layer 7), not a networking problem at all.
* **Example finding:** If `traceroute` shows fine times for the first 3 hops but huge delays starting at hop 4, the problem likely lies specifically at/after hop 4.

## 8.8 Key Takeaways

* Always start troubleshooting from Layer 1 and move upward.
* `169.254.x.x` = DHCP failure / APIPA self-assignment.
* Successful IP ping + failed domain access = DNS issue, confirm with `nslookup`/`dig`.
* `arp -a` reveals whether Layer 2 resolution is working on a local network.
* `traceroute` pinpoints which hop in the path is introducing delay.

---

# 9. MAC Address vs IP Address

## 9.1 Comparison Table

| | MAC Address | IP Address |
|---|---|---|
| Layer | 2 (Data Link) | 3 (Network) |
| Scope | Local network only (hop-to-hop) | Entire journey (end-to-end) |
| Assigned by | Manufacturer (burned into hardware) | Network (via DHCP, or manually) |
| Changes during a journey? | Yes — rebuilt at every hop | No — same source/destination the whole way (aside from NAT) |
| Format example | AA:BB:CC:11:22:33 | 192.168.1.5 |
| Analogy | Local street address, useful within the neighborhood | Full postal address, works across the whole country |

## 9.2 Why Two Separate Addressing Systems Exist

Layer 2 and Layer 3 solve different problems: Layer 2 answers "which physically-connected device should get this next?" (narrow, local); Layer 3 answers "which network, anywhere in the world, is the ultimate destination?" (global). This lets each layer solve its own scoped problem independently.

## 9.3 Practical Walkthrough: End-to-End vs Hop-by-Hop

Scenario: `[Laptop] → [Home Router] → [ISP Router] → [Core Internet Router] → [example.com Server]`

```
Your Laptop:          IP: 192.168.1.5      MAC: AA:AA:AA:AA:AA:AA
Home Router:          IP: 192.168.1.1 (local) / 203.0.113.5 (public)
                       MAC: BB:BB:BB:BB:BB:BB (local side)
ISP Router:            MAC: CC:CC:CC:CC:CC:CC
Core Internet Router:  MAC: DD:DD:DD:DD:DD:DD
example.com Server:    IP: 93.184.216.34    MAC: EE:EE:EE:EE:EE:EE
```

**Hop 1 (Laptop → Home Router):**
```
FRAME: Src MAC AA:AA... (laptop), Dst MAC BB:BB... (home router)
PACKET (untouched): Src IP 192.168.1.5, Dst IP 93.184.216.34
```

**Hop 2 (Home Router → ISP Router):** Home router strips frame, reads packet's destination IP, builds a brand-new frame:
```
FRAME (new): Src MAC BB:BB... (router outward), Dst MAC CC:CC... (ISP router)
PACKET: Dst IP still 93.184.216.34 (unchanged)
```
> **NAT note:** In reality, the home router usually performs **NAT (Network Address Translation)** here, replacing the private source IP (192.168.1.5) with its own public IP (203.0.113.5), since private IPs aren't valid on the wider internet. This is a deliberate, one-time Layer 3 modification — distinct from the hop-by-hop rebuilding that happens to the frame. The destination IP itself never changes.

**Hop 3 (ISP Router → Core Router):** Same pattern — strip, read destination IP, build a new frame.

**Hop 4 (Core Router → Server):** Final hop — frame's destination MAC finally matches the ultimate destination device.

```
              Frame 1        Frame 2         Frame 3         Frame 4
              (rebuilt)      (rebuilt)       (rebuilt)       (rebuilt)
Laptop -----> Home Router --> ISP Router --> Core Router --> Server
  |________________ Same packet, same Dst IP, the whole way ______|
```

**Analogy:** The frame is a single-use shipping label meaningful only for one hop, rewritten fresh each time. The packet is the sealed shipping box carried unopened the whole journey — only the outer hop label (frame) keeps getting swapped.

## 9.4 MAC Addresses Aren't Known in Advance — ARP

A device does not automatically know the MAC address of its next hop — it must actively discover it using **ARP** (see Section 12).

## 9.5 Key Takeaways

* MAC = hop-to-hop, local scope. IP = end-to-end, global scope.
* Frame rebuilt at every router hop; packet (with original source/destination IP) unchanged throughout, aside from NAT.
* MAC addresses must be actively resolved via ARP before a frame can be built.

---

# 10. DNS (Domain Name System)

## 10.1 Core Concept

DNS translates human-friendly **domain names** (example.com) into machine-usable **IP addresses** (93.184.216.34) — a distributed, hierarchical "phone book" for the internet.

## 10.2 Why It's Distributed and Hierarchical

No single server holds every domain-to-IP mapping (would be a single point of failure/bottleneck). Instead, DNS is organized like an upside-down tree:

```
                    "." (root)
                   /    |    \
                .com  .org  .net  ... (Top-Level Domains, TLDs)
               /
          example.com  (Authoritative Domain)
               /
      www.example.com  (specific hostname)
```

## 10.3 Full Step-by-Step Resolution Process (Worst Case — Nothing Cached)

1. **Check local caches** — browser cache, then OS DNS cache.
2. **Ask the Recursive Resolver** — often ISP's DNS server or a public one (Google `8.8.8.8`, Cloudflare `1.1.1.1`); does the hard work of tracking down the answer.
3. **Resolver asks a Root Server** — "Who handles `.com`?" Root server replies with a TLD server address.
4. **Resolver asks the TLD Server** — "Who handles example.com?" TLD server replies with the authoritative name server address.
5. **Resolver asks the Authoritative Name Server** — holds the real answer, replies with the IP.
6. **Resolver replies to device and caches the result** for the record's TTL duration.

```
[Your Device] → [Recursive Resolver] → [Root Server] → "ask .com TLD"
                                     → [.com TLD Server] → "ask example.com's server"
                                     → [example.com Authoritative Server] → "93.184.216.34"
[Recursive Resolver] → [Your Device]: "93.184.216.34"
```

## 10.4 DNS Record Types

| Record Type | Purpose |
|---|---|
| **A** | Maps a domain name to an IPv4 address |
| **AAAA** | Maps a domain name to an IPv6 address |
| **CNAME** | Maps a domain name to another domain name (alias) |
| **MX** | Specifies mail servers for the domain |
| **NS** | Specifies authoritative name servers for the domain |
| **TXT** | Arbitrary text, often used for verification/security policies (e.g. SPF) |

## 10.5 Protocol/Port

DNS queries typically use **UDP port 53** (small queries, speed favored over guaranteed reliability — lost queries are simply retried). Falls back to **TCP port 53** when a response is too large for one UDP packet, or for zone transfers between DNS servers.

## 10.6 Caching and TTL

Every DNS record has a **TTL (Time To Live)** — how long resolvers may cache the answer before re-querying. Short TTL = faster propagation of changes, more query load. Long TTL = less load, slower propagation.

## 10.7 DNS Propagation

When records change, different resolvers worldwide have cached the old answer for varying durations (per the old TTL), so changes can take minutes up to 48 hours to be visible everywhere.

## 10.8 Key Takeaways

* DNS = hierarchical, distributed system: root → TLD → authoritative server.
* Typically UDP/53; falls back to TCP/53 for large responses or zone transfers.
* TTL governs caching duration and propagation speed.
* Tools: `nslookup`, `dig`, `ping <domain>` (to check name resolution specifically).

---

# 11. DHCP (Dynamic Host Configuration Protocol)

## 11.1 Core Concept

DHCP automates assigning IP configuration to devices joining a network, avoiding manual setup on every device.

## 11.2 What DHCP Hands Out

* IP address (from a defined pool/scope)
* Subnet mask (defines what counts as "local network")
* Default gateway (router's IP, for traffic destined outside the local network)
* DNS server addresses
* Lease time (how long the device may keep the IP)

## 11.3 The DORA Process

```
[Device]                                    [DHCP Server]
   |------ 1. DISCOVER (broadcast) ------------->|   "Is there a DHCP server? I need an IP."
   |<----- 2. OFFER ------------------------------|   "I can offer 192.168.1.50 + config."
   |------ 3. REQUEST (broadcast) --------------->|   "I accept that offer — assign it to me."
   |<----- 4. ACKNOWLEDGE (ACK) ------------------|   "Confirmed — here's your full config."
```

* **Discover is broadcast** because the device has no IP address yet and can't address a specific server.
* **Request is also broadcast** because multiple DHCP servers might have made offers — broadcasting the acceptance lets other servers know their offer was declined, freeing that address back up.

## 11.4 Lease Time and Renewal

The assigned IP is not permanent — it has a **lease time** (hours to days). Devices automatically try to renew before expiration (typically getting the same IP again). If not renewed and the lease fully expires, the IP becomes available for reassignment.

## 11.5 Layer/Protocol

DHCP uses **UDP**, port **67** (server) and port **68** (client) — lightweight, since this happens at network setup time before more complex connections are possible.

## 11.6 DHCP Failure — APIPA

If DORA doesn't complete successfully, most OSes fall back to **APIPA (Automatic Private IP Addressing)**, self-assigning a `169.254.x.x` address — allows communication with other devices on the same broken network but no valid gateway/DNS, hence no internet access.

## 11.7 Static IP vs DHCP-Assigned IP

* **Static IP** — manually configured, fixed, never changes (used for servers, printers, network equipment — devices you want to reliably reach at a known address).
* **DHCP-assigned IP** — automatic, ideal for regular client devices where the specific IP doesn't matter.
* **DHCP Reservation** — middle ground: device still goes through DHCP, but the server always assigns the same IP to that device (identified by MAC address) every time.

## 11.8 Key Takeaways

* DORA: Discover → Offer → Request → Acknowledge.
* Discover and Request are both broadcast (device has no IP yet / to inform other DHCP servers).
* DHCP failure → APIPA (`169.254.x.x`), no internet access.
* DHCP reservations combine automatic assignment with a predictable, fixed IP.

---

# 12. ARP (Address Resolution Protocol)

## 12.1 Core Concept

ARP resolves a known **IP address** into its corresponding **MAC address** on the local network — necessary because a device knows the IP of its next hop (e.g., default gateway) but not, initially, its MAC address.

## 12.2 The ARP Exchange

```
Your Laptop broadcasts (to everyone on the local network):
  "Who has IP 192.168.1.1? Tell 192.168.1.5" (ARP Request)

Home Router replies directly:
  "192.168.1.1 is at MAC BB:BB:BB:BB:BB:BB" (ARP Reply)
```

## 12.3 Actual ARP Packet Structure

```
ARP REQUEST:
  Sender MAC: AA:AA:AA:AA:AA:AA   (my own MAC)
  Sender IP:  192.168.1.5          (my own IP)
  Target MAC: 00:00:00:00:00:00    (unknown — what we're asking for)
  Target IP:  192.168.1.1          (IP we want the MAC for)

ARP REPLY:
  Sender MAC: BB:BB:BB:BB:BB:BB   (router's MAC — now filled in)
  Sender IP:  192.168.1.1
  Target MAC: AA:AA:AA:AA:AA:AA   (sent back to whoever asked)
  Target IP:  192.168.1.5
```

## 12.4 Where ARP Sits in the Layers

ARP doesn't cleanly fit into a single OSI layer — often described as sitting at the boundary between Layer 2 and Layer 3 (sometimes informally called "Layer 2.5"), since its job is to translate between IP (Layer 3) and MAC (Layer 2) addressing.

## 12.5 ARP Caching

Devices maintain a local **ARP table/cache** — a temporary IP-to-MAC mapping for recently contacted devices. View with:
```bash
arp -a          # Windows / Linux / Mac
ip neigh        # modern Linux alternative
```
Entries expire after a timeout (commonly a couple of minutes); the device re-runs ARP once expired.

## 12.6 Gratuitous ARP

An ARP sent **without being asked**, to announce "this IP now belongs to this MAC." Used when:
* A device first joins the network (proactively updates others' ARP tables; checks for duplicate IP)
* A failover event occurs (a backup server takes over an IP from a failed primary, telling the network to redirect traffic)

## 12.7 ARP Spoofing (Security Concern)

Because ARP has **no built-in authentication**, any device can falsely claim ownership of an IP address, and others will generally believe it. Attackers exploit this (**ARP spoofing/poisoning**) — e.g., impersonating the default gateway to intercept traffic. Mitigations include **Dynamic ARP Inspection (DAI)** on switches.

## 12.8 ARP and IPv6

IPv6 does not use ARP at all — it uses **NDP (Neighbor Discovery Protocol)**, built into ICMPv6, to perform a similar IP-to-MAC resolution function.

## 12.9 Key Takeaways

* ARP resolves known IP → unknown MAC, via broadcast request + direct reply.
* Results are cached (ARP table) to avoid repeating the lookup for every packet.
* No authentication built in → vulnerable to ARP spoofing.
* IPv6 replaces ARP with NDP.

---

# 13. TCP (Transmission Control Protocol)

## 13.1 Core Concept

TCP is a reliable, ordered, connection-oriented Layer 4 protocol. Its reliability is built from several concrete mechanisms detailed below.

## 13.2 TCP Segment Header (Key Fields)

```
TCP HEADER:
  Source Port
  Destination Port
  Sequence Number         (tracks order of bytes sent)
  Acknowledgment Number   (confirms what's been received so far)
  Flags                   (SYN, ACK, FIN, RST, PSH, URG)
  Window Size             (how much data the receiver can accept right now)
  Checksum                (error-detection)
```

## 13.3 Sequence Numbers — Guaranteeing Order

Every byte sent is assigned a sequence number, letting the receiver reconstruct data in the correct order even if segments arrive out of order (they can travel different physical paths).

```
Segment 1: bytes 1-500     (Seq: 1)
Segment 2: bytes 501-1000  (Seq: 501)
Segment 3: bytes 1001-1500 (Seq: 1001)
```
Even if Segment 2 arrives first, sequence numbers let the receiver reassemble correctly.

## 13.4 Acknowledgment Numbers — Guaranteeing Delivery

The receiver sends back an ACK confirming "I've received everything up through byte X, send what's next." If the sender doesn't get an ACK within an expected window, it retransmits.

```
[Sender] --- Segment (Seq:1, bytes 1-500) -----> [Receiver]
[Sender] <--- ACK (Ack:501, "send 501+") -------- [Receiver]
```

## 13.5 Three-Way Handshake (with Flags)

```
[Client]                                          [Server]
   |--- SYN (Seq: x) --------------------------->|   "I want to connect. My seq starts at x."
   |<-- SYN-ACK (Seq: y, Ack: x+1) --------------|   "Ack your x. My seq starts at y."
   |--- ACK (Ack: y+1) --------------------------->|  "Ack your y. Connection established."
```
Both sides pick their own randomized initial sequence number (partly for security — makes it harder to guess/inject fake data into a connection).

## 13.6 Closing a Connection — Four-Way Handshake (FIN)

```
[Client]                                          [Server]
   |--- FIN ------------------------------------->|   "I'm done sending data."
   |<-- ACK ----------------------------------------|  "Acknowledged."
   |<-- FIN ------------------------------------------|"I'm also done sending data."
   |--- ACK --------------------------------------->|  "Acknowledged. Connection closed."
```
Both sides independently signal "done" and get acknowledged, since either side could still have data left to send after the other finishes (a "half-closed" state).

## 13.7 Flow Control

Governed by the **Window Size** field — tells the sender how much data the receiver can currently handle before an acknowledgment is needed, preventing a fast sender from overwhelming a slower receiver's buffer. Can dynamically shrink/grow during a connection.

## 13.8 Congestion Control

Protects the **network itself** (distinct from flow control, which protects the receiver). TCP starts sending cautiously (**"slow start"**), gradually increasing rate as delivery succeeds, but backs off immediately upon detecting packet loss (interpreted as a sign of congestion).

## 13.9 Retransmission Triggers

1. **Timeout** — sender waits a calculated time for an ACK; if absent, assumes loss and resends.
2. **Duplicate ACKs** — repeated ACKs for the same sequence number (receiver still waiting on a specific missing piece) trigger **"fast retransmit"** without waiting for a full timeout.

## 13.10 Key Takeaways

* Reliability = sequence numbers (order) + acknowledgments (confirmed delivery) + retransmission (recovery from loss).
* Opening a connection: 3-way handshake (SYN, SYN-ACK, ACK). Closing: 4-way handshake (FIN/ACK each direction).
* Flow control protects the receiver; congestion control protects the network.
* Retransmission triggers: timeout or duplicate ACKs (fast retransmit).

---

# 14. UDP (User Datagram Protocol)

## 14.1 Core Concept

UDP is a deliberately minimal, connectionless Layer 4 protocol — optimized for speed and simplicity rather than guaranteed delivery/order. Not "worse" than TCP — a different set of trade-offs.

## 14.2 UDP Header (Only 4 Fields, 8 Bytes)

```
UDP HEADER:
  Source Port
  Destination Port
  Length
  Checksum   (optional in IPv4, mandatory in IPv6)
```
No sequence numbers, no acknowledgment numbers, no window size, no handshake flags.

## 14.3 No Connection = No Handshake

```
[Sender] ------- UDP Datagram (data) -----------> [Receiver]
```
The first packet sent *is* the actual data — no setup phase required.

## 14.4 No Guarantee = No Retransmission, No Ordering

If a datagram is lost, UDP itself never notices or fixes it — no acknowledgment mechanism exists. Out-of-order datagrams are handed to the application exactly as they arrive. If reliability/ordering is needed, **the application itself must implement it** — some UDP-based applications do build lightweight reliability on top of UDP when partial reliability is needed without TCP's full overhead.

## 14.5 Real-World Use Cases

| Use Case | Why UDP |
|---|---|
| DNS queries | Small, quick request/response; simpler to just retry than pay TCP's handshake cost |
| Video calls / VoIP | Losing a fraction of a second is less disruptive than pausing for retransmission of stale data |
| Live streaming | A dropped frame is better than buffering/freezing for a frame that's already "in the past" |
| Online multiplayer gaming | Frequent real-time updates make a lost update irrelevant by the time a resend would arrive |
| DHCP | Lightweight setup exchange doesn't need connection overhead |

## 14.6 QUIC (Modern Development)

**QUIC** (used by HTTP/3) is built on top of UDP but implements its own custom reliability/ordering — more efficient than TCP for some modern use cases, notably avoiding **head-of-line blocking** (where one lost TCP segment delays everything else behind it, even unrelated data). Shows "TCP vs UDP" isn't always strictly either/or — some systems selectively borrow UDP's lightweight transport while rebuilding just the reliability features they need.

## 14.7 TCP vs UDP Feature Comparison

| Feature | TCP | UDP |
|---|---|---|
| Handshake | Yes (3-way) | No |
| Sequence numbers | Yes | No |
| Acknowledgments | Yes | No |
| Retransmission | Yes | No |
| Ordering guarantee | Yes | No |
| Flow control | Yes | No |
| Congestion control | Yes | No |
| Header size | 20+ bytes | 8 bytes |

## 14.8 Common Misconceptions

* ❌ "UDP is just a worse version of TCP."
* ✅ UDP is a deliberate trade-off: for real-time data, an old retransmitted packet is often already useless by the time it arrives (the "moment" has passed), so TCP's reliability can actively work against real-time use cases.
* ❌ "UDP applications can never have any reliability."
* ✅ False — applications can build their own reliability mechanisms on top of UDP (e.g., QUIC) when needed.

## 14.9 Key Takeaways

* UDP header: 4 fields, 8 bytes — no reliability machinery.
* No handshake, no retransmission, no ordering by default.
* Chosen when speed/low overhead matters more than guaranteed delivery.
* QUIC (HTTP/3) = custom reliability built on top of UDP.

---

# 15. HTTP and HTTPS

## 15.1 Core Concept

**HTTP (HyperText Transfer Protocol)** is the Application-layer, text-based, request-response protocol defining how a browser requests web content and how a server responds.

## 15.2 Anatomy of an HTTP Request

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml
Connection: keep-alive

(optional body, mainly for POST requests)
```
* **Method** — the requested action
* **Path** — which resource
* **Version** — protocol version
* **Headers** — metadata
* **Body** (optional) — data sent, common with POST

## 15.3 Common HTTP Methods

| Method | Purpose |
|---|---|
| **GET** | Retrieve a resource — should not change anything server-side |
| **POST** | Submit data (form submission, creating a resource) |
| **PUT** | Update/replace an existing resource entirely |
| **DELETE** | Remove a resource |
| **HEAD** | Like GET, but only headers returned, not the body |

## 15.4 Anatomy of an HTTP Response

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256
Server: nginx

<html>...(actual webpage content)...</html>
```
* **Status line** — protocol version, status code, short description
* **Headers** — metadata
* **Body** — actual requested content

## 15.5 HTTP Status Code Categories

| Range | Category | Common Examples |
|---|---|---|
| 1xx | Informational | 100 Continue |
| 2xx | Success | 200 OK, 201 Created |
| 3xx | Redirection | 301 Moved Permanently, 302 Found |
| 4xx | Client Error | 404 Not Found, 403 Forbidden, 401 Unauthorized |
| 5xx | Server Error | 500 Internal Server Error, 503 Service Unavailable |

**Mental model:** 2xx = "it worked," 3xx = "go look elsewhere," 4xx = "you did something wrong," 5xx = "the server messed up."

## 15.6 Statelessness

HTTP is fundamentally **stateless** — each request is independent, with no memory of previous requests by default. "Staying logged in" is implemented via **cookies**: the server sends `Set-Cookie`, the browser resends it with every future request, often containing a **session ID** the server uses to look up the logged-in user. This is the real-world implementation of OSI's Layer 5 "session" concept, handled at Layer 7 rather than a dedicated OSI layer.

## 15.7 HTTPS = HTTP + TLS

HTTPS is not a separate protocol — it's HTTP passed through a **TLS encryption layer** before being handed to TCP.

**What HTTPS protects:**
* **Confidentiality** — content unreadable to interceptors
* **Integrity** — tampering is detectable
* **Authentication** — verifies you're talking to the real server (via certificates)

**What HTTPS does NOT hide:** An observer can typically still see *which domain* you're connecting to (via DNS queries and the unencrypted SNI field in the TLS handshake) and roughly how much data is transferred. HTTPS hides content, not the fact/destination of a connection.

## 15.8 Ports

* HTTP: **port 80**
* HTTPS: **port 443**

## 15.9 HTTP/1.1 vs HTTP/2 vs HTTP/3

| Version | Key Feature |
|---|---|
| HTTP/1.1 | Persistent/keep-alive connections; requests processed somewhat sequentially per connection |
| HTTP/2 | **Multiplexing** — many requests/responses interleaved over a single TCP connection simultaneously, speeding up pages with many resources |
| HTTP/3 | Built on **QUIC** (UDP-based) instead of TCP, avoiding head-of-line blocking where one lost TCP segment stalls all multiplexed requests sharing that connection |

## 15.10 Key Takeaways

* HTTP = stateless, text-based request/response protocol; HTTPS = HTTP + TLS.
* Statefulness (login persistence) implemented via cookies/session IDs at the application layer.
* HTTPS hides content but not destination/domain (visible via DNS + SNI).
* HTTP/2 multiplexing and HTTP/3's QUIC foundation both address performance/head-of-line-blocking issues.

---

# 16. TLS (Transport Layer Security)

## 16.1 Core Concept

TLS provides **encryption, authentication, and integrity** for data sent over a network. It's the modern successor to **SSL (Secure Sockets Layer)**, which is now obsolete/insecure, though "SSL" is still used colloquially (e.g., "SSL certificate").

## 16.2 The Three Guarantees

1. **Confidentiality** — data encrypted, unreadable to interceptors
2. **Integrity** — tampering in transit is detectable
3. **Authentication** — verifies identity of the server via certificates

## 16.3 The TLS Handshake (TLS 1.3, Streamlined)

```
[Client]                                          [Server]
   |----- ClientHello -------------------------------->|
   |   Supported cipher suites + a random number
   |
   |<---- ServerHello + Certificate + Key Share --------|
   |   Chosen cipher suite, certificate (identity),
   |   server's part of the key exchange
   |
   |----- (Client verifies certificate) --------------->|
   |   Checks: signed by trusted CA? domain matches?
   |   not expired/revoked?
   |
   |----- Finished (encrypted) ------------------------>|
   |<---- Finished (encrypted) -------------------------|
   |   Both sides now share an encryption key; all
   |   further communication is encrypted.
```
TLS 1.3 reduced this to essentially one round trip (vs. TLS 1.2's two round trips) — a meaningful performance improvement.

## 16.4 Certificates and Certificate Authorities (CAs)

A digital certificate says "I (a trusted authority) verify this public key belongs to example.com":

```
Root CA (globally trusted, built into OS/browser)
   | signs/vouches for
   v
Intermediate CA
   | signs/vouches for
   v
example.com's certificate
```

Browsers come pre-loaded with trusted **Root CAs** (e.g., DigiCert, Let's Encrypt). If the certificate's chain of trust traces back to a trusted root, it's accepted; if the chain breaks, is expired, or the domain doesn't match, the browser shows a warning (e.g., "Your connection is not private").

## 16.5 Asymmetric vs Symmetric Encryption

| | Asymmetric Encryption | Symmetric Encryption |
|---|---|---|
| Keys | Public/private key pair | One shared secret key |
| Speed | Computationally expensive | Fast |
| Use in TLS | Handshake — identity verification, safely exchanging initial secrets | Bulk of actual application data transfer, using the now-shared key |

TLS uses **both strategically**: asymmetric encryption during the handshake to safely establish a shared secret, then fast symmetric encryption for the actual data — combining asymmetric's identity verification with symmetric's speed.

## 16.6 SNI (Server Name Indication)

During the `ClientHello`, the browser must tell the server which domain it wants (since one server IP can host many sites). This is sent via the **SNI** field — **not yet encrypted**, since encryption keys aren't established at that point. This means an observer can typically still see which domain you're connecting to even over HTTPS, simply by watching the initial handshake. Newer extensions like **Encrypted Client Hello (ECH)** aim to close this gap but aren't universally deployed yet.

## 16.7 Where TLS Fits in the Layer Model

TLS conceptually maps to OSI's **Presentation layer (Layer 6)** functions (encryption, certificate-based authentication), but in real TCP/IP implementation it sits as its own distinct layer **between TCP and HTTP** — sometimes informally called "Layer 4.5" or "Layer 6.5," since it doesn't cleanly fit the traditional model's boundaries. A concrete example of OSI's theoretical separation not perfectly matching real protocol implementation.

## 16.8 Key Takeaways

* TLS = successor to SSL; provides confidentiality, integrity, authentication.
* TLS 1.3 handshake ≈ one round trip.
* Certificates form a chain of trust back to a pre-trusted Root CA.
* Handshake uses asymmetric encryption; bulk data transfer uses symmetric encryption (for speed).
* SNI is sent unencrypted — domain is visible to network observers even over HTTPS.
* TLS doesn't map cleanly to a single OSI layer — sits between Transport and Application in practice.

---

# Final Revision

## Important Concepts

* Networking layers exist for separation of concerns — each layer only handles its own header/job.
* Encapsulation (down) adds headers; decapsulation (up) interprets and strips them.
* PDU names: Data → Segment/Datagram → Packet → Frame → Bits.
* MAC = hop-to-hop/local; IP = end-to-end/global; frame rebuilt every hop, packet unchanged.
* OSI = 7-layer theoretical reference model; TCP/IP = 4-layer practical implementation actually used on the internet.
* A complete webpage request: DNS → TCP handshake → TLS handshake → HTTP request/response, repeated per new domain, reused via persistent connections for same-domain resources.
* ARP resolves IP → MAC on the local network; not known in advance, must be discovered and cached.
* TCP guarantees reliability via sequence numbers, acknowledgments, and retransmission; UDP provides none of this by default, trading reliability for speed.
* DNS is hierarchical/distributed (root → TLD → authoritative server); DHCP auto-configures IP settings via DORA.
* HTTPS = HTTP + TLS; TLS provides confidentiality, integrity, and authentication but does not hide the destination domain (visible via DNS/SNI).

## Must-Know Definitions

* **Protocol** — an agreed-upon set of rules for how two systems communicate.
* **Encapsulation/Decapsulation** — wrapping data with headers going down the layers; interpreting/removing headers going up.
* **PDU (Protocol Data Unit)** — the formal name for a layer's packaged data (segment, packet, frame, etc.).
* **MAC Address** — a hardware address, unique per NIC, used for local-network (Layer 2) delivery.
* **IP Address** — a network-layer address used for end-to-end (Layer 3) delivery across networks.
* **Port Number** — identifies which application on a device should receive data (Layer 4).
* **TCP** — reliable, ordered, connection-oriented transport protocol.
* **UDP** — fast, connectionless transport protocol with no delivery/order guarantees.
* **DNS** — system translating domain names into IP addresses.
* **DHCP** — protocol that automatically assigns IP configuration to devices.
* **ARP** — protocol resolving a known IP address to its MAC address on the local network.
* **TLS** — protocol providing encryption, integrity, and authentication for data in transit.
* **NAT** — process of translating a private IP address to a public one (typically at a router).
* **APIPA** — self-assigned `169.254.x.x` address used when DHCP fails.

## Important Comparisons

* **TCP vs UDP** — see Section 14.7
* **Hub vs Switch** — Hub (Layer 1, no intelligence, broadcasts to all) vs Switch (Layer 2, MAC-address-based targeted delivery)
* **Switch vs Router** — Switch (Layer 2, MAC addresses, same local network) vs Router (Layer 3, IP addresses, connects different networks)
* **MAC vs IP** — see Section 9.1
* **HTTP vs HTTPS** — HTTP is plaintext; HTTPS = HTTP + TLS encryption, authentication, integrity
* **OSI vs TCP/IP** — see Section 6.4

## Common Mistakes

* Thinking the "envelope" in the mail analogy represents encryption — it actually represents addressing/header information, not content-hiding.
* Thinking layers avoid inspecting each other's data for "security" reasons — it's actually about separation of concerns/modularity.
* Thinking a router calculates the entire path to a destination in advance — it only makes a local, per-hop forwarding decision based on its routing table.
* Thinking TCP reliability just means "not corrupted" — it also guarantees ordering and retransmission of lost data.
* Thinking a session gets established "instantly" on the first try — it actually requires negotiation (e.g., TCP three-way handshake, then a TLS handshake with multiple round trips) before real data flows.
* Thinking the physical layer directly "hands over" a pre-formed frame — Layer 1 only provides raw bits; Layer 2 interprets/reconstructs the frame from those bits according to its own rules.
* Thinking OSI's 7-layer separation is "unnecessary" — it's not wrong, just more granular than TCP/IP's practical implementation needs; OSI optimizes for conceptual clarity, TCP/IP for practical implementation.
* Thinking TCP/IP is the theoretical model and OSI is what the internet actually runs on — it's the reverse: OSI is theoretical/reference, TCP/IP is the practical implementation.
* Thinking UDP is simply "a worse version of TCP" — it's a deliberate trade-off suited to real-time use cases where stale retransmitted data is often useless.
* Thinking a `169.254.x.x` address just means "no internet" generically — it specifically indicates DHCP failure and APIPA self-assignment, pointing to a local network issue.

## Quick Revision Cheat Sheet

* **7 Layers (top to bottom):** Application, Presentation, Session, Transport, Network, Data Link, Physical
* **Mnemonic:** Please Do Not Throw Sausage Pizza Away
* **PDU per layer:** Data (7/6/5) → Segment/Datagram (4) → Packet (3) → Frame (2) → Bits (1)
* **Devices:** Hub = L1, Switch = L2, Router = L3, Firewall = L4/L7
* **Addressing:** MAC = local/hop-to-hop (L2), IP = global/end-to-end (L3)
* **TCP/IP Model:** Application (App+Pres+Session), Transport, Internet, Network Access (Data Link+Physical)
* **TCP handshake:** SYN → SYN-ACK → ACK (open) / FIN → ACK → FIN → ACK (close)
* **TLS guarantees:** Confidentiality, Integrity, Authentication
* **Common ports:** DNS 53 (UDP, TCP fallback), DHCP 67/68 (UDP), HTTP 80, HTTPS 443
* **Full request order:** DNS lookup → TCP handshake → TLS handshake → HTTP request/response
* **DORA (DHCP):** Discover → Offer → Request → Acknowledge
* **DNS resolution order:** Local cache → Recursive Resolver → Root Server → TLD Server → Authoritative Server
* **Troubleshooting tools:** `ipconfig`/`ifconfig`, `ping`, `nslookup`/`dig`, `arp -a`, `tracert`/`traceroute`

## Questions I Should Be Able to Answer

**Basic:**
1. What is the main job of each of the 7 OSI layers?
2. What is the PDU name at each layer?
3. What port numbers do HTTP, HTTPS, DNS, and DHCP typically use?

**Conceptual:**
4. Why does networking need layers instead of one monolithic system?
5. Why do MAC addresses change at every hop while IP addresses remain constant end-to-end?
6. Why does TCP need a four-step process to close a connection but only three to open one?
7. Why is ARP necessary even though a device already knows the IP address it wants to reach?

**"Why" Questions:**
8. Why does encryption happen before the data is wrapped in TCP/IP/frame headers, rather than after?
9. Why does DNS typically use UDP instead of TCP?
10. Why does TLS use both asymmetric and symmetric encryption instead of just one?
11. Why doesn't OSI map perfectly onto how the real internet is implemented?

**Scenario-Based:**
12. Trace exactly what happens, layer by layer, from typing `https://example.com` to the page rendering.
13. A device shows a `169.254.x.x` IP address — what does this tell you, and what should you check next?
14. If a `traceroute` shows a sudden delay starting at hop 4, what does that suggest?

**Troubleshooting:**
15. You can ping an IP address but can't open the corresponding website by domain name — walk through your full diagnostic process.
16. Two computers on the same local network can't communicate — what layers would you check, in what order, and why?
17. The network feels slow — what's your step-by-step approach to isolating where the slowness originates?

---

# Glossary

* **APIPA (Automatic Private IP Addressing)** — a self-assigned `169.254.x.x` IP address used by a device when DHCP fails.
* **ARP (Address Resolution Protocol)** — resolves a known IP address to its corresponding MAC address on the local network.
* **ARP Spoofing** — an attack exploiting ARP's lack of authentication to impersonate another device's IP address.
* **Certificate Authority (CA)** — a trusted organization that issues and vouches for digital certificates.
* **Congestion Control** — a TCP mechanism that throttles sending rate to avoid overwhelming the network.
* **DHCP (Dynamic Host Configuration Protocol)** — automatically assigns IP configuration to devices joining a network.
* **DNS (Domain Name System)** — translates domain names into IP addresses via a hierarchical, distributed lookup system.
* **DORA** — the four-step DHCP process: Discover, Offer, Request, Acknowledge.
* **Encapsulation** — wrapping data with a new header at each layer going down the stack.
* **Decapsulation** — removing/interpreting headers at each layer going up the stack.
* **Flow Control** — a TCP mechanism preventing a fast sender from overwhelming a slower receiver.
* **Frame** — the Layer 2 PDU, containing MAC addresses and an error-check trailer.
* **Gratuitous ARP** — an unsolicited ARP announcement, e.g. to update others' ARP tables or handle failover.
* **HTTP** — Application-layer protocol defining rules for requesting/delivering web content.
* **HTTPS** — HTTP passed through TLS encryption.
* **IP Address** — a Layer 3 address identifying a device's location on a network, meaningful end-to-end.
* **MAC Address** — a Layer 2 hardware address identifying a device on its local network, meaningful hop-to-hop.
* **NAT (Network Address Translation)** — translating a private IP address to a public one, typically at a router.
* **OSI Model** — a 7-layer theoretical reference model for networking, created by ISO in 1984.
* **Packet** — the Layer 3 PDU, containing source/destination IP addresses.
* **Port Number** — identifies which application on a device should handle incoming/outgoing data.
* **Protocol** — an agreed-upon set of rules governing communication.
* **QUIC** — a modern transport protocol built on UDP with custom reliability, used by HTTP/3.
* **Segment/Datagram** — the Layer 4 PDU (segment for TCP, datagram for UDP).
* **SNI (Server Name Indication)** — the unencrypted field in a TLS handshake indicating which domain is being requested.
* **Symmetric/Asymmetric Encryption** — symmetric uses one shared key (fast); asymmetric uses a public/private key pair (used for identity verification/key exchange).
* **TCP (Transmission Control Protocol)** — reliable, ordered, connection-oriented Layer 4 protocol.
* **TCP/IP Model** — the practical 4-layer model actually implemented on the real internet.
* **TLS (Transport Layer Security)** — protocol providing confidentiality, integrity, and authentication; successor to SSL.
* **TTL (Time To Live)** — how long a DNS record may be cached before re-querying.
* **UDP (User Datagram Protocol)** — fast, connectionless Layer 4 protocol with no delivery/order guarantees.
