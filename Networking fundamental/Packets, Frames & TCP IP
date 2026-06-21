# Packets, Frames & TCP/IP 

---

## Part 1 — Packets and Frames

### 1. Definition

**Analogy — The Letter System:**

Think of sending a letter internationally.
- You write a **letter** (your data/payload)
- You put it in an **envelope** with a destination country and city (IP address) → this is a **packet**
- The post office then puts it in a **mail bag** labeled for the specific local van/route (MAC address) → this is a **frame**

Every time data moves across a network, it gets **wrapped in layers** — each layer adds its own header with routing info.

---

### 2. Packet — Layer 3 (Network Layer)

- Lives at **OSI Layer 3**
- Handles **end-to-end delivery** across networks (city to city, country to country)
- Contains **IP addresses** to route traffic globally

**What is inside a packet?**

| Field | What it is |
|---|---|
| Source IP | Where the packet came from |
| Destination IP | Where the packet is going |
| TTL | How long the packet can live before being discarded |
| Checksum | Error-checking value |
| Payload | The actual data being carried |

**Analogy:**
A packet is like an **international shipping label** on a box.
It says: *"From: Mumbai → To: New York"*
It does not care which truck or van carries it — just the final destination.

---

### 3. Frame — Layer 2 (Data Link Layer)

- Lives at **OSI Layer 2**
- Handles **local delivery** within the same network (street to street)
- Contains **MAC addresses**, NOT IP addresses
- A frame is **destroyed and rebuilt** at every router hop

**What is inside a frame?**

| Field | What it is |
|---|---|
| Source MAC | Physical address of the sending device |
| Destination MAC | Physical address of the next receiving device |
| Payload | The packet it is carrying |
| FCS (Frame Check Sequence) | Error detection at Layer 2 |

**Analogy:**
A frame is like the **local delivery van label**.
The van does not care about New York — it only knows *"pick up from Warehouse A, drop at Warehouse B (next router)"*.
At every router, the old frame is stripped off and a new one is created for the next hop.

---

### Notable Header Fields Explained

#### Time to Live (TTL)

- A **countdown counter** on every packet
- Every router the packet passes through **decrements TTL by 1**
- When TTL hits **0**, the packet is **dropped** and discarded
- Prevents packets from bouncing around the internet forever (infinite loops)

**Analogy:**
TTL is like an **expiry date on food**.
If the food takes too long to reach you, it gets thrown away at the last checkpoint rather than delivered rotten.

**Example:**
- Packet leaves your PC with TTL = 64
- Passes through 3 routers → TTL becomes 61
- If it passes through 64 routers with no destination found → dropped

---

#### Checksum

- A **math value** calculated from the packet's contents
- The receiver recalculates the checksum and compares it
- If the values **do not match** → packet is corrupted → dropped and re-requested
- Catches errors introduced during transmission (interference, hardware faults)

**Analogy:**
Checksum is like a **seal on a medicine bottle**.
If the seal is broken when you receive it, you know something went wrong in transit.

---

#### Source Address

- The **IP address of the device that sent the packet**
- Used so the destination knows where to send the reply

**Example:** Your PC at `192.168.1.5` sends a request → source address = `192.168.1.5`

---

#### Destination Address

- The **IP address of the device that should receive the packet**
- Routers read this to decide where to forward the packet next

**Example:** You open `google.com` → destination address = Google's server IP (`142.250.x.x`)

---

---

## Part 2 — TCP/IP

### Definition

TCP/IP is the **standard communication rulebook** used across the internet and most networks.

- **TCP** = Transmission Control Protocol → ensures reliable, ordered delivery
- **IP** = Internet Protocol → handles addressing and routing

**Analogy:**
- **IP** is the **postal system** — it routes your letter to the right address
- **TCP** is the **registered mail service** — it makes sure the letter arrives, confirms delivery, and re-sends if lost

---

### TCP/IP Model — 5 Layers

> Note: TCP/IP uses a 5-layer model (some textbooks say 4 by merging the bottom two)

| Layer | Name | What it does | Example Protocols |
|---|---|---|---|
| 5 | Application | Interface for user apps and data | HTTP, DNS, FTP, SMTP |
| 4 | Transport | End-to-end delivery, ports, reliability | TCP, UDP |
| 3 | Network | IP addressing, routing between networks | IP, ICMP, ARP |
| 2 | Data Link | MAC addressing, local delivery | Ethernet, Wi-Fi (802.11) |
| 1 | Physical | Raw bits over cables/wireless | Cables, Radio waves, Fiber |

**How data flows (sending):**

```
Application (your browser request)
       ↓
Transport (add port numbers)
       ↓
Network (add IP addresses)
       ↓
Data Link (add MAC addresses)
       ↓
Physical (convert to bits and transmit)
```

**How data flows (receiving):** Exact reverse — each layer strips its header and passes up.

---

### TCP Packet Structure

A TCP packet (also called a **segment**) contains these key fields:

---

#### Source Port and Source IP

- **Source Port** — the port number on the sender's machine used for this connection
- **Source IP** — the IP address of the sender

**Example:** Your browser at `192.168.1.5:54321` is the source (port 54321 is randomly assigned by your OS)

---

#### Destination Port and Destination IP

- **Destination Port** — the port on the target server (e.g., 80 for HTTP, 443 for HTTPS)
- **Destination IP** — the IP address of the target server

**Example:** Google's web server at `142.250.x.x:443`

---

#### Sequence Number

- A **unique number assigned to each byte of data** in the stream
- Used to **reassemble packets in the correct order** at the destination
- Even if packets arrive out of order, the sequence number fixes the order

**Analogy:**
Sequence numbers are like **page numbers in a book**.
Even if pages 3, 1, 2 arrive in that order — you reassemble them as 1, 2, 3 because of the numbering.

---

#### Acknowledgement Number

- The receiver sends back this number to say *"I received up to byte X, now send me byte X+1"*
- Confirms successful receipt
- If the sender does not get an acknowledgement in time → it **retransmits**

**Analogy:**
Acknowledgement is like a **delivery confirmation text**.
*"Your package was delivered."* If you never get the text, the courier sends it again.

---

#### Checksum

- Same concept as at Layer 3
- Verifies the TCP segment was not corrupted in transit
- Covers the header and data together

---

#### Data (Payload)

- The **actual content** being sent
- Could be an HTTP request, file contents, login credentials, etc.
- Everything above is just "wrapping" — the data is what the application actually cares about

---

#### Flags

TCP flags are **1-bit switches** in the header that control the state of the connection.

| Flag | Full Name | What it does |
|---|---|---|
| SYN | Synchronize | Initiates a connection |
| ACK | Acknowledgement | Confirms receipt of data |
| FIN | Finish | Gracefully closes a connection |
| RST | Reset | Forcefully terminates a connection |
| PSH | Push | Tells receiver to pass data to the app immediately |
| URG | Urgent | Marks data as high priority |

---

### 3-Way Handshake

Before any data is sent, TCP establishes a connection using a **3-step process**.

**Analogy:**
It is like calling someone on the phone:
1. You call → their phone rings (SYN)
2. They pick up and say "Hello?" (SYN-ACK)
3. You say "Hello, can you hear me?" (ACK)
Now you are connected and start talking.

---

**Step-by-step:**

```
Client                            Server
  |                                  |
  |  -------- SYN (seq=100) -------> |   "I want to connect, my sequence starts at 100"
  |                                  |
  |  <-- SYN-ACK (seq=200, ack=101)--|   "OK, my sequence starts at 200, I got your 100"
  |                                  |
  |  -------- ACK (ack=201) -------> |   "Got it, your sequence starts at 200"
  |                                  |
  |         [Connection Open]        |
```

| Step | Who sends | Flag | Meaning |
|---|---|---|---|
| 1 | Client | SYN | "I want to connect" |
| 2 | Server | SYN-ACK | "OK, I am ready too, confirm?" |
| 3 | Client | ACK | "Confirmed, let us communicate" |

After Step 3 — the connection is established and data transfer begins.

---

**Why does this matter for hacking?**

- **SYN Flood Attack** — attacker sends thousands of SYN packets but never completes Step 3 → server waits for ACKs that never come → server resources get exhausted → DoS
- **Port Scanning (nmap -sS)** — sends SYN, if server replies SYN-ACK → port is open, attacker then sends RST to avoid completing the handshake → stealthy scan

---

