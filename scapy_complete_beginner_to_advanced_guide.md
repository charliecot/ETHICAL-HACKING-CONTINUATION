# Scapy: Complete Beginner-to-Advanced Practical Guide

**Purpose:** Learn Scapy from the ground up: understand packets, build them layer by layer, inspect fields, send and receive traffic, interpret responses, troubleshoot Windows/Npcap, and write maintainable network tools.

> **Authorization and safety:** Use packet crafting, discovery, sniffing, and scanning only on your own devices, lab networks, or systems for which you have explicit permission. Start with loopback, a local virtual lab, or a device you control. Do not treat a public IP address as permission to scan it.

---

# Part 1 — Foundations

## 1. What is Scapy?

Scapy is a Python library for manipulating network packets. It can:

- Create packets from protocol layers.
- Read and inspect packet fields.
- Send packets and, where supported, receive replies.
- Capture packets from a network interface.
- Decode many common protocols.
- Save packets to and read packets from capture files.
- Build custom diagnostic and authorized testing tools.

Scapy is not a replacement for every networking tool. It gives you low-level control, so you are responsible for choosing the correct interface, packet layers, fields, timeouts, and interpretation.

## 2. Install Scapy

In PowerShell or your activated virtual environment:

```powershell
py -m pip install scapy
```

Check the installation:

```powershell
py -m pip show scapy
```

Start Python:

```powershell
py
```

Import Scapy:

```python
import scapy.all as scapy
```

This import style means `scapy` is now the alias for the `scapy.all` module. Therefore write:

```python
scapy.IP()
scapy.TCP()
scapy.sr1()
```

Do not redundantly write `scapy.all.IP()` after using `import scapy.all as scapy`.

You may also see:

```python
from scapy.all import IP, TCP, sr1
```

Then call `IP()`, `TCP()`, and `sr1()` directly. Pick one import style and use it consistently.

## 3. Networking concepts you need first

### 3.1 Packet, frame, and payload

- **Frame:** a Layer 2 unit, commonly Ethernet, used on a local link.
- **Packet:** commonly refers to a Layer 3 IP unit.
- **Segment:** commonly refers to a TCP transport unit.
- **Datagram:** commonly refers to UDP or IP data units.
- **Payload:** the data carried inside a protocol layer.

The exact terminology depends on the protocol and networking context.

### 3.2 Encapsulation

A packet is often built as a stack of layers. For example:

```text
Ethernet frame
└── IPv4 packet
    └── TCP segment
        └── Application data
```

In Scapy, `/` stacks layers:

```python
packet = scapy.Ether() / scapy.IP() / scapy.TCP()
```

For an IP-level packet:

```python
packet = scapy.IP() / scapy.ICMP()
```

The `/` operator means “put this layer after the previous layer.” It is not division.

### 3.3 Common protocol layers

| Layer / protocol | Purpose | Scapy class |
|---|---|---|
| Ethernet | Local-link framing and MAC addresses | `Ether` |
| ARP | IPv4 address to link-layer address resolution | `ARP` |
| IPv4 | Addressing and routing | `IP` |
| IPv6 | IPv6 addressing and routing | `IPv6` |
| ICMP | Network diagnostics and error messages | `ICMP` |
| TCP | Reliable, connection-oriented transport | `TCP` |
| UDP | Connectionless transport | `UDP` |
| DNS | Domain name queries and responses | `DNS`, `DNSQR` |
| Raw | Uninterpreted payload bytes | `Raw` |

Scapy supports many more protocols. Discover them in the installed version with Python help and the official API documentation.

---

# Part 2 — Discovering interfaces and routes

## 4. What is a network interface?

An interface is a network adapter or virtual connection, such as Wi-Fi, Ethernet, loopback, VPN, or a virtual-machine adapter.

An interface may have:
- A name and description.
- An interface index.
- One or more IPv4/IPv6 addresses.
- A MAC address.
- A status and MTU.

The interface name, IP address, MAC address, and default gateway are different pieces of information.

## 5. List interfaces

```python
import scapy.all as scapy

scapy.show_interfaces()
```

This prints interfaces Scapy can see. The exact columns and identifiers vary by operating system, capture backend, and Scapy version.

Get the interface identifiers:

```python
for name in scapy.get_if_list():
    print(name)
```

Inspect Scapy's interface collection:

```python
scapy.conf.ifaces.show()
```

Inspect the default interface:

```python
print(scapy.conf.iface)
```

### 5.1 Inspect interface objects

```python
for iface in scapy.conf.ifaces.values():
    print("Object:", iface)
    print("Name:", iface.name)
    print("Index:", iface.index)
    print("MAC:", iface.mac)
    print("IPv4:", iface.ip)
    print("Description:", iface.description)
    print("-" * 50)
```

Not every backend or platform exposes every attribute in exactly the same way. If an attribute is missing, inspect the object:

```python
print(vars(iface))
```

Use the output from `show_interfaces()` as the practical reference for your installed system.

### 5.2 Select an interface

Use the exact identifier shown by Scapy:

```python
selected_iface = "YOUR EXACT SCAPY INTERFACE IDENTIFIER"
```

Pass it with the parameter `iface`:

```python
answer = scapy.sr1(packet, iface=selected_iface, timeout=2, verbose=False)
```

**Important:** Scapy's common send/receive functions use `iface=`, not `interface=`. This is the cause of the error:

```text
TypeError: L3pcapSocket.send() got an unexpected keyword argument 'interface'
```

Correct:

```python
scapy.sr1(packet, iface=selected_iface)
```

Incorrect:

```python
scapy.sr1(packet, interface=selected_iface)
```

### 5.3 Interface object or interface name?

Depending on the function and Scapy version, `iface` can accept an interface identifier or interface object. The simplest beginner approach is to use the exact name/identifier shown by Scapy. Do not assume the Windows adapter display name and Scapy's capture identifier are always identical.

## 6. Find the route Scapy would use

For a destination, inspect Scapy's route decision:

```python
scapy.conf.route.route("192.0.2.10")
```

The returned tuple contains route information such as the selected interface, source address, and gateway, depending on the platform and Scapy version.

Inspect the route table:

```python
scapy.conf.route
```

For IPv6, inspect:

```python
scapy.conf.route6
```

A route lookup helps answer: “Which interface and source address would be used to reach this destination?” It does not prove that the destination is reachable.

---

# Part 3 — Creating and inspecting packets

## 7. Build your first packet

Build an IP + ICMP packet:

```python
packet = scapy.IP(dst="127.0.0.1") / scapy.ICMP()
```

This is a packet object. Creating it does not send anything.

Inspect it in three ways:

```python
packet.show()       # Detailed decoded fields
packet.summary()    # Short one-line description
packet.show2()      # Show after Scapy has built/calculated fields
```

Compare:
- `show()` displays the current packet fields.
- `summary()` gives a compact description.
- `show2()` builds the packet first, which can calculate fields such as lengths and checksums.

Convert to bytes:

```python
raw_bytes = bytes(packet)
print(raw_bytes)
print(raw_bytes.hex())
```

`bytes(packet)` serializes the packet. It does not transmit it.

## 8. Important packet inspection methods

| Method / expression | What it does |
|---|---|
| `packet.show()` | Displays decoded fields |
| `packet.summary()` | Short summary |
| `packet.show2()` | Builds then displays packet |
| `packet.command()` | Produces a Scapy expression representing the packet |
| `packet.haslayer(scapy.TCP)` | Checks whether a TCP layer exists |
| `packet.getlayer(scapy.TCP)` | Gets a layer, or `None` if absent |
| `packet[scapy.TCP]` | Gets a layer; raises an error if it is absent |
| `packet.getlayer(1)` | Gets a layer by index in the packet stack |
| `bytes(packet)` | Serializes packet to bytes |
| `len(packet)` | Packet length in bytes after serialization/building as needed |
| `packet.copy()` | Makes a packet copy |
| `packet.summary()` | Returns a short textual summary |

Examples:

```python
if packet.haslayer(scapy.ICMP):
    icmp_layer = packet.getlayer(scapy.ICMP)
    print(icmp_layer.type)
```

Safer access when a layer may be absent:

```python
tcp_layer = packet.getlayer(scapy.TCP)

if tcp_layer is not None:
    print("TCP destination port:", tcp_layer.dport)
else:
    print("No TCP layer")
```

## 9. Accessing packet fields

A layer is an object containing fields.

```python
ip = scapy.IP(dst="192.0.2.10")

print(ip.src)
print(ip.dst)
print(ip.ttl)
print(ip.proto)
```

For a TCP packet:

```python
tcp = scapy.TCP(sport=50000, dport=80, flags="S")

print(tcp.sport)
print(tcp.dport)
print(tcp.flags)
print(tcp.seq)
print(tcp.ack)
print(tcp.window)
```

The values shown are the values currently stored in the Scapy object. Some fields are automatically calculated when the packet is built or sent.

## 10. Change a field

```python
packet = scapy.IP(dst="127.0.0.1") / scapy.ICMP()

packet[scapy.IP].ttl = 32
packet[scapy.ICMP].type = 8

packet.show()
```

The packet is modified in memory. It is not sent until you call a sending function.

## 11. Field defaults and automatic calculations

Scapy protocol classes have many fields with defaults. For example, IPv4 fields include source, destination, TTL, protocol, length, and checksum.

Some fields may be `None` before Scapy builds the packet. Scapy can calculate values such as:
- IP total length.
- IP header checksum.
- TCP/UDP checksum.
- Some protocol-dependent fields.

Use `show2()` or `bytes(packet)` to trigger packet building and see calculated values. Do not assume every field is automatically inferred in every unusual or custom packet.

---

# Part 4 — Common protocol fields

## 12. Ethernet: `Ether`

Example:

```python
frame = scapy.Ether()
frame.show()
```

Important Ethernet fields:

| Field | Meaning |
|---|---|
| `src` | Source MAC address |
| `dst` | Destination MAC address |
| `type` | EtherType identifying the next protocol |

Example:

```python
ether = scapy.Ether(
    src="00:11:22:33:44:55",
    dst="ff:ff:ff:ff:ff:ff",
    type=0x0800
)
```

`0x0800` identifies IPv4. `0x0806` identifies ARP. Scapy often infers the type when stacking known layers.

## 13. IPv4: `IP`

Example:

```python
ip = scapy.IP(dst="192.0.2.10")
ip.show()
```

Important IPv4 fields:

| Field | Meaning |
|---|---|
| `src` | Source IPv4 address |
| `dst` | Destination IPv4 address |
| `version` | IP version; IPv4 uses 4 |
| `ihl` | Header length |
| `tos` | Type of Service / DS field |
| `len` | Total IP packet length |
| `id` | Identification field |
| `flags` | Fragmentation flags |
| `frag` | Fragment offset |
| `ttl` | Time To Live |
| `proto` | Next protocol number |
| `chksum` | Header checksum |
| `options` | IPv4 options |

Example:

```python
ip = scapy.IP(
    src="192.0.2.20",
    dst="192.0.2.10",
    ttl=64,
    id=1234
)
```

### 13.1 IP fragmentation flags

IPv4 fragmentation flags are not TCP flags.

- `DF`: Don't Fragment.
- `MF`: More Fragments.
- Reserved: reserved bit.

Example:

```python
ip = scapy.IP(dst="192.0.2.10", flags="DF")
print(ip.flags)
```

## 14. IPv6: `IPv6`

Important fields include:

| Field | Meaning |
|---|---|
| `src` | Source IPv6 address |
| `dst` | Destination IPv6 address |
| `version` | IPv6 version (6) |
| `tc` | Traffic Class |
| `fl` | Flow Label |
| `plen` | Payload length |
| `nh` | Next Header |
| `hlim` | Hop Limit |

Example:

```python
packet = scapy.IPv6(dst="2001:db8::10") / scapy.ICMPv6EchoRequest()
packet.show()
```

`2001:db8::/32` is a documentation prefix; replace it with an appropriate lab address when testing.

## 15. ICMP: `ICMP`

ICMP is used for diagnostics and network-layer error reporting.

Example:

```python
icmp = scapy.ICMP()
icmp.show()
```

Important fields:

| Field | Meaning |
|---|---|
| `type` | ICMP message type |
| `code` | More specific meaning within the type |
| `chksum` | ICMP checksum |
| `id` | Identifier for messages that use it |
| `seq` | Sequence number for messages that use it |

Common IPv4 ICMP types:

| Type | Meaning |
|---:|---|
| 0 | Echo Reply |
| 3 | Destination Unreachable |
| 5 | Redirect |
| 8 | Echo Request |
| 11 | Time Exceeded |
| 12 | Parameter Problem |

For Type 3 (Destination Unreachable), common codes include:
- 0: Network unreachable.
- 1: Host unreachable.
- 3: Port unreachable.
- 4: Fragmentation needed and DF set.
- 13: Communication administratively prohibited.

Interpret the type and code together. The same code number under another type means something different.

Example inspection:

```python
if response.haslayer(scapy.ICMP):
    icmp = response[scapy.ICMP]
    print("ICMP type:", icmp.type)
    print("ICMP code:", icmp.code)
```

## 16. TCP: `TCP`

TCP is connection-oriented. Its header contains ports, sequence and acknowledgment numbers, flags, window information, and other fields.

Example:

```python
tcp = scapy.TCP(sport=50000, dport=80, flags="S")
tcp.show()
```

Important fields:

| Field | Meaning |
|---|---|
| `sport` | Source port |
| `dport` | Destination port |
| `seq` | Sequence number |
| `ack` | Acknowledgment number |
| `dataofs` | TCP data offset / header length |
| `reserved` | Reserved bits |
| `flags` | TCP control flags |
| `window` | Advertised receive window |
| `chksum` | TCP checksum |
| `urgptr` | Urgent pointer |
| `options` | TCP options |

### 16.1 TCP flags

| Flag | Bit value | Meaning |
|---|---:|---|
| FIN | `0x01` | Finish/close direction |
| SYN | `0x02` | Synchronize sequence numbers |
| RST | `0x04` | Reset |
| PSH | `0x08` | Push indication |
| ACK | `0x10` | Acknowledgment field is valid |
| URG | `0x20` | Urgent pointer is valid |
| ECE | `0x40` | ECN Echo |
| CWR | `0x80` | Congestion Window Reduced |
| NS | `0x100` | ECN nonce sum flag |

Common combinations:

| Hex | Flags | Meaning |
|---:|---|---|
| `0x02` | S | SYN |
| `0x10` | A | ACK |
| `0x12` | SA | SYN + ACK |
| `0x14` | RA | RST + ACK |
| `0x18` | PA | PSH + ACK |
| `0x11` | FA | FIN + ACK |
| `0x04` | R | RST |

**Important:** `0x14` in `response[TCP].flags` means RST + ACK. It is not an IPv4 fragmentation flag. `0x16` means RST + SYN + ACK and is not the normal closed-port response pattern.

Use Scapy's flag representation:

```python
tcp = response.getlayer(scapy.TCP)

if tcp is not None:
    print("Flags:", tcp.flags)
    print("Flags as integer:", int(tcp.flags))
    print("Source port:", tcp.sport)
    print("Destination port:", tcp.dport)
```

### 16.2 Test individual TCP flags

Use bitwise AND (`&`) to test whether a flag is present:

```python
if tcp.flags & 0x02:
    print("SYN is set")

if tcp.flags & 0x10:
    print("ACK is set")

if tcp.flags & 0x04:
    print("RST is set")
```

Do not compare only against one exact combination if your goal is to test whether a particular bit is present.

For example:

```python
if (int(tcp.flags) & 0x12) == 0x12:
    print("Both SYN and ACK are set")
```

## 17. UDP: `UDP`

UDP is connectionless and does not establish a TCP-style connection.

Important fields:

| Field | Meaning |
|---|---|
| `sport` | Source port |
| `dport` | Destination port |
| `len` | UDP length |
| `chksum` | UDP checksum |

Example:

```python
packet = scapy.IP(dst="127.0.0.1") / scapy.UDP(sport=53000, dport=53)
packet.show()
```

A UDP application may respond, or an ICMP error may be returned. No response alone does not prove a UDP port is closed.

## 18. DNS: `DNS` and `DNSQR`

DNS questions and answers are represented with DNS layers.

Example DNS query packet:

```python
dns_packet = (
    scapy.IP(dst="127.0.0.1")
    / scapy.UDP(dport=53)
    / scapy.DNS(
        rd=1,
        qd=scapy.DNSQR(qname="example.com")
    )
)

dns_packet.show()
```

Important DNS fields include:
- `id`: transaction identifier.
- `qr`: query (`0`) or response (`1`).
- `opcode`: query operation.
- `aa`: authoritative answer flag.
- `tc`: truncated message flag.
- `rd`: recursion desired.
- `ra`: recursion available.
- `rcode`: response code.
- `qd`: question section.
- `an`: answer section.
- `ns`: authority section.
- `ar`: additional section.

DNS has nested records and fields. Inspect a real response with `show()` before assuming the shape of its answer section.

---

# Part 5 — Sending and receiving packets

## 19. Know the six important send/receive functions

| Function | Layer | Waits for responses? | Return idea |
|---|---|---|---|
| `send()` | Layer 3 | No | Sends packets |
| `sendp()` | Layer 2 | No | Sends frames |
| `sr()` | Layer 3 | Yes | `(answered, unanswered)` |
| `srp()` | Layer 2 | Yes | `(answered, unanswered)` |
| `sr1()` | Layer 3 | Yes | One response or `None` |
| `srp1()` | Layer 2 | Yes | One response or `None` |

### 19.1 `send()`

Sends a Layer 3 packet without waiting for a reply:

```python
scapy.send(
    scapy.IP(dst="127.0.0.1") / scapy.ICMP(),
    verbose=False
)
```

### 19.2 `sendp()`

Sends a Layer 2 frame:

```python
scapy.sendp(
    scapy.Ether() / scapy.ARP(pdst="192.0.2.1"),
    iface=selected_iface,
    verbose=False
)
```

### 19.3 `sr()`

Sends at Layer 3 and collects answered and unanswered packets:

```python
answered, unanswered = scapy.sr(
    scapy.IP(dst="127.0.0.1") / scapy.ICMP(),
    timeout=2,
    verbose=False
)
```

Loop through answered pairs:

```python
for sent_packet, received_packet in answered:
    print("Sent:", sent_packet.summary())
    print("Received:", received_packet.summary())
```

Inspect unanswered packets:

```python
for sent_packet in unanswered:
    print("No matching reply for:", sent_packet.summary())
```

### 19.4 `sr1()`

Sends at Layer 3 and returns one matching response:

```python
response = scapy.sr1(
    scapy.IP(dst="127.0.0.1") / scapy.ICMP(),
    timeout=2,
    verbose=False
)

if response is None:
    print("No response before timeout")
else:
    response.show()
```

`sr1()` does not return the `(sent, received)` pair returned by the answered collection of `sr()`.

### 19.5 `srp()` and `srp1()`

These perform the corresponding send/receive operations at Layer 2. They are commonly used with Ethernet and ARP frames.

```python
frame = scapy.Ether() / scapy.ARP(pdst="192.0.2.1")

answer = scapy.srp1(
    frame,
    iface=selected_iface,
    timeout=2,
    verbose=False
)
```

Use a real local address in a network you control; the documentation address above is only illustrative.

## 20. Common send/receive parameters

Common parameters include the following. Exact availability and defaults can vary by function and Scapy version; inspect the installed function signature and help.

| Parameter | Typical purpose |
|---|---|
| `iface` | Interface used for sending/capture |
| `timeout` | Maximum wait for replies |
| `verbose` | Show or suppress progress output |
| `inter` | Delay between packets in a multi-packet send |
| `count` | Number of packets to send in applicable functions |
| `retry` | Retry unanswered packets in applicable send/receive functions |
| `multi` | Allow multiple answers to a sent packet in applicable functions |
| `filter` | Capture filter in functions that support it |
| `store` | Whether to retain captured packets in memory in applicable functions |
| `promisc` | Request promiscuous capture mode in applicable capture functions |

Do not assume every parameter is accepted by every Scapy function. For example, `iface` is common, but the exact signature differs between `send`, `sr`, `sniff`, and other functions.

## 21. Inspect the exact parameters in your installed Scapy

Use Python's built-in `help()`:

```python
help(scapy.sr1)
help(scapy.sr)
help(scapy.send)
help(scapy.sendp)
help(scapy.sniff)
```

Use `inspect.signature()` when available:

```python
import inspect
import scapy.all as scapy

print(inspect.signature(scapy.sr1))
print(inspect.signature(scapy.sr))
print(inspect.signature(scapy.sniff))
```

This is the safest way to learn the exact arguments supported by the version you installed. The complete Scapy API is very large; there is no short universal list of every parameter for every protocol and function.

---

# Part 6 — Understanding responses correctly

## 22. Always check whether a response exists

Incorrect:

```python
response = scapy.sr1(packet, timeout=2, verbose=False)
print(response[scapy.IP].src)
```

If no reply arrives, `response` is `None` and indexing it fails.

Correct:

```python
response = scapy.sr1(packet, timeout=2, verbose=False)

if response is None:
    print("No response")
else:
    print(response.summary())
```

## 23. Check for layers before reading fields

```python
if response is not None and response.haslayer(scapy.IP):
    ip = response[scapy.IP]
    print("Source IP:", ip.src)
    print("Destination IP:", ip.dst)
```

For TCP:

```python
if response is not None and response.haslayer(scapy.TCP):
    tcp = response[scapy.TCP]
    print("Source port:", tcp.sport)
    print("Destination port:", tcp.dport)
    print("Flags:", tcp.flags)
```

For ICMP:

```python
if response is not None and response.haslayer(scapy.ICMP):
    icmp = response[scapy.ICMP]
    print("Type:", icmp.type)
    print("Code:", icmp.code)
```

## 24. Useful IP response fields

```python
ip = response.getlayer(scapy.IP)

if ip:
    print("Source:", ip.src)
    print("Destination:", ip.dst)
    print("TTL:", ip.ttl)
    print("Protocol:", ip.proto)
    print("IP ID:", ip.id)
    print("Fragment flags:", ip.flags)
    print("Fragment offset:", ip.frag)
    print("Total length:", ip.len)
```

## 25. Useful TCP response fields

```python
tcp = response.getlayer(scapy.TCP)

if tcp:
    print("Source port:", tcp.sport)
    print("Destination port:", tcp.dport)
    print("Sequence:", tcp.seq)
    print("Acknowledgment:", tcp.ack)
    print("Flags:", tcp.flags)
    print("Window:", tcp.window)
    print("Header length:", tcp.dataofs)
    print("Options:", tcp.options)
```

## 26. Useful UDP response fields

```python
udp = response.getlayer(scapy.UDP)

if udp:
    print("Source port:", udp.sport)
    print("Destination port:", udp.dport)
    print("Length:", udp.len)
    print("Checksum:", udp.chksum)
```

## 27. Useful Ethernet response fields

```python
ether = response.getlayer(scapy.Ether)

if ether:
    print("Source MAC:", ether.src)
    print("Destination MAC:", ether.dst)
    print("EtherType:", hex(ether.type))
```

A Layer 3 receive path may not expose the Ethernet header in the same way as a Layer 2 capture. If you need Ethernet fields, use a suitable Layer 2 capture/send path.

## 28. Inspect raw payload

```python
if response.haslayer(scapy.Raw):
    payload = response[scapy.Raw].load

    print("Payload bytes:", payload)
    print("Payload hex:", payload.hex())
```

Payload is bytes, not automatically text. Decode only when you know the encoding:

```python
text_payload = payload.decode("utf-8", errors="replace")
print(text_payload)
```

Do not assume arbitrary network payload is valid UTF-8.

---

# Part 7 — TCP connection and port-state concepts

## 29. Understand the TCP three-way handshake

A normal TCP connection begins with:

1. Client sends SYN.
2. Server responds SYN-ACK.
3. Client sends ACK.

Conceptually:

```text
Client                         Server
  | -------- SYN -------------> |
  | <------ SYN + ACK ---------- |
  | -------- ACK --------------> |
```

A TCP SYN probe is a packet-level test, not a complete application connection. A response must be interpreted carefully.

## 30. Common TCP responses to a SYN probe

For an authorized test, common interpretations are:

| Observed response | Typical interpretation |
|---|---|
| SYN + ACK (`SA`) | Port appears open |
| RST + ACK (`RA`) | Port appears closed |
| No response | Filtered, dropped, unreachable, or otherwise inconclusive |
| ICMP unreachable/admin prohibited | Network or policy is preventing access |

These are common interpretations, not universal guarantees. Firewalls, proxies, middleboxes, unusual stacks, packet loss, and routing can change observed behavior.

### 30.1 Inspect flags with bitwise operations

```python
tcp = response.getlayer(scapy.TCP)

if tcp:
    flags = int(tcp.flags)

    syn_set = bool(flags & 0x02)
    ack_set = bool(flags & 0x10)
    rst_set = bool(flags & 0x04)

    if syn_set and ack_set and not rst_set:
        print("SYN and ACK are set")
    elif rst_set and ack_set:
        print("RST and ACK are set")
    else:
        print("Other TCP flag combination:", tcp.flags)
```

This is more flexible than comparing only one exact integer.

### 30.2 Important limitation

A timeout is not the same as “closed.” A firewall may silently drop probes. Use labels such as `no response` or `filtered/unknown` unless you have sufficient evidence to classify more precisely.

---

# Part 8 — Capturing packets

## 31. What does `sniff()` do?

`sniff()` captures packets visible to the selected capture interface.

A simple, bounded example:

```python
packets = scapy.sniff(
    iface=selected_iface,
    count=5,
    timeout=10,
    store=True
)

packets.summary()
```

This captures up to five packets or stops after ten seconds, whichever happens first.

## 32. Important `sniff()` parameters

| Parameter | Meaning |
|---|---|
| `iface` | Interface or interfaces to capture from |
| `count` | Stop after this many packets; `0` commonly means no count limit |
| `timeout` | Stop after the specified number of seconds |
| `filter` | BPF capture filter, if supported by the capture backend |
| `prn` | Function called for each captured packet |
| `store` | Whether to retain packets in the returned packet list |
| `stop_filter` | Function that can stop capture when it returns true |
| `promisc` | Request promiscuous mode, if supported |

Check your installed version:

```python
help(scapy.sniff)
```

## 33. Print each captured packet

```python
def show_packet(packet):
    print(packet.summary())

scapy.sniff(
    iface=selected_iface,
    count=10,
    timeout=15,
    prn=show_packet,
    store=False
)
```

`prn` receives each captured packet. `store=False` avoids retaining every packet in the returned list, which is useful for long-running capture jobs.

## 34. Filter captured packets

A BPF filter is a capture filter, not a Python expression.

Examples:

```python
# Capture TCP traffic
scapy.sniff(filter="tcp", count=10, timeout=10, iface=selected_iface)

# Capture traffic involving TCP port 80
scapy.sniff(filter="tcp port 80", count=10, timeout=10, iface=selected_iface)

# Capture ICMP
scapy.sniff(filter="icmp", count=10, timeout=10, iface=selected_iface)

# Capture traffic to or from a host
scapy.sniff(filter="host 192.0.2.10", count=10, timeout=10, iface=selected_iface)
```

Filters depend on the capture backend and platform. On Windows, Npcap supplies packet-capture support.

## 35. `filter` versus `lfilter`

- `filter`: capture-layer BPF expression, applied by the capture backend when supported.
- `lfilter`: Python callable that returns `True` for packets you want to keep.

Example:

```python
packets = scapy.sniff(
    iface=selected_iface,
    count=10,
    timeout=10,
    lfilter=lambda packet: packet.haslayer(scapy.TCP)
)
```

BPF filtering can reduce the number of packets delivered to Python. A Python `lfilter` runs after packets have been captured and decoded.

---

# Part 9 — Saving and reading packet captures

## 36. Save packets to a PCAP file

```python
packets = scapy.sniff(
    iface=selected_iface,
    count=10,
    timeout=10,
    store=True
)

scapy.wrpcap("capture.pcap", packets)
```

PCAP is a common packet-capture file format.

## 37. Read a PCAP file

```python
packets = scapy.rdpcap("capture.pcap")

print("Number of packets:", len(packets))

for packet in packets:
    print(packet.summary())
```

Inspect a selected packet:

```python
packets[0].show()
```

Check the number of packets before indexing. An empty capture has no `packets[0]`.

## 38. Read one packet at a time from a capture file

For large files, consider Scapy's `PcapReader`:

```python
with scapy.PcapReader("capture.pcap") as reader:
    for packet in reader:
        print(packet.summary())
```

This avoids loading the entire capture into a list at once.

---

# Part 10 — ARP and local network discovery

## 39. What is ARP?

ARP resolves an IPv4 address to a link-layer address on a local IPv4 network.

An ARP request asks: “Who has this IPv4 address?” An ARP reply can provide the corresponding MAC address.

ARP is a local-link protocol; routers do not forward ordinary ARP broadcasts across networks.

## 40. Build and inspect an ARP request

```python
arp_request = scapy.ARP(pdst="192.0.2.1")
arp_request.show()
```

Build an Ethernet broadcast frame around it:

```python
arp_frame = (
    scapy.Ether(dst="ff:ff:ff:ff:ff:ff")
    / scapy.ARP(pdst="192.0.2.1")
)

arp_frame.show()
```

This only builds a packet. Sending it is a separate action.

## 41. Inspect ARP fields

Important ARP fields:

| Field | Meaning |
|---|---|
| `hwtype` | Hardware type |
| `ptype` | Protocol type |
| `hwlen` | Hardware address length |
| `plen` | Protocol address length |
| `op` | Operation: request or reply |
| `hwsrc` | Sender hardware/MAC address |
| `psrc` | Sender protocol/IPv4 address |
| `hwdst` | Target hardware address |
| `pdst` | Target protocol/IPv4 address |

Example:

```python
arp = response.getlayer(scapy.ARP)

if arp:
    print("Operation:", arp.op)
    print("Sender MAC:", arp.hwsrc)
    print("Sender IP:", arp.psrc)
    print("Target MAC:", arp.hwdst)
    print("Target IP:", arp.pdst)
```

---

# Part 11 — Advanced packet handling

## 42. Use `sr()` to examine answered and unanswered packets

```python
answered, unanswered = scapy.sr(
    packet_list,
    timeout=2,
    verbose=False
)

for sent, received in answered:
    print("Request:", sent.summary())
    print("Reply:", received.summary())

for sent in unanswered:
    print("No matched response:", sent.summary())
```

The collections are different:
- `answered` contains sent/received pairs.
- `unanswered` contains sent packets without a matched response.

The variable names are not special Python keywords; they are names chosen by the programmer.

## 43. Use `srloop()` for repeated diagnostic probes

Scapy provides loop-style functions such as `srloop()` for repeated send/receive operations. Check its exact signature and behavior in your installed version:

```python
help(scapy.srloop)
```

Use bounded tests and avoid generating unnecessary traffic.

## 44. Use `AsyncSniffer` for controlled background capture

`AsyncSniffer` can start and stop a capture without blocking the main flow immediately:

```python
sniffer = scapy.AsyncSniffer(
    iface=selected_iface,
    filter="icmp",
    store=True
)

sniffer.start()

# Perform a short, authorized test here.

sniffer.stop()
captured = sniffer.results

if captured:
    captured.summary()
```

For robust applications, handle exceptions and ensure the sniffer is stopped during cleanup. Check `help(scapy.AsyncSniffer)` for options in your installed version.

## 45. Use `Raw` for application payload bytes

```python
payload = b"Example lab payload"

packet = (
    scapy.IP(dst="127.0.0.1")
    / scapy.UDP(dport=9999)
    / scapy.Raw(load=payload)
)

packet.show()
```

A bytes literal begins with `b`. Network payloads are bytes, not necessarily strings.

## 46. Bindings and protocol dissection

Scapy chooses how to decode payloads based on protocol fields and registered layer bindings. You can inspect a packet's layers:

```python
packet = scapy.IP() / scapy.TCP() / scapy.Raw(load=b"hello")

print(packet.layers())
packet.show()
```

You can also inspect a layer's payload:

```python
print(packet.payload)
```

And its underlayer:

```python
print(packet.underlayer)
```

- `payload` is the next encapsulated layer.
- `underlayer` is the layer containing the current one.

---

# Part 12 — Finding attributes, methods, and documentation

## 47. Scapy is large: discover instead of guessing

There are hundreds of protocol classes and many function parameters. No single short list can accurately enumerate every field and method across all Scapy versions.

Use these tools interactively:

### `help()`

```python
help(scapy.IP)
help(scapy.TCP)
help(scapy.ARP)
help(scapy.sr1)
help(scapy.sniff)
```

### `dir()`

```python
print(dir(scapy.IP))
print(dir(scapy.TCP))
```

`dir()` lists names available on an object/class. It includes internal names, so not every item is intended for everyday use.

### `show()` on a layer

```python
scapy.IP().show()
scapy.TCP().show()
scapy.ICMP().show()
scapy.ARP().show()
```

This is one of the most useful ways to discover protocol fields.

### `fields_desc`

Scapy protocol classes commonly expose field definitions through `fields_desc`:

```python
for field in scapy.IP.fields_desc:
    print("Field:", field.name)
    print("Type:", type(field).__name__)
    print("Default:", field.default)
    print("-" * 30)
```

Some field defaults may be `None` or computed dynamically. A field definition's default is not necessarily the final value in a transmitted packet.

### `getfieldval()`

```python
ip = scapy.IP(dst="127.0.0.1")

print(ip.getfieldval("dst"))
print(ip.getfieldval("ttl"))
```

### `setfieldval()`

```python
ip.setfieldval("ttl", 32)
print(ip.ttl)
```

Direct attribute assignment is often easier:

```python
ip.ttl = 32
```

### `fieldtype()`

```python
print(ip.fieldtype("ttl"))
```

This can help identify the field definition/type.

---

# Part 13 — Windows and Npcap

## 48. Why Windows needs extra attention

On Windows, Scapy commonly uses Npcap to capture and transmit packets. Ordinary Python socket behavior and packet-capture behavior are not identical.

Check interfaces:

```python
scapy.show_interfaces()
```

Check Npcap installation and service status in Windows. Install Npcap from its official website and follow its current installation guidance.

Some operations may require an elevated terminal. Run with administrator privileges only when required and only for authorized testing.

### 48.1 Common Windows errors

**Error:**
```text
L3pcapSocket.send() got an unexpected keyword argument 'interface'
```

**Cause:** The wrong keyword was passed.

**Fix:**
```python
iface=selected_iface
```

**Error:** Windows raw socket or administrator-related error.

Possible causes include:
- Npcap missing or incorrectly installed.
- Npcap service unavailable.
- Insufficient permissions.
- Unsupported packet operation or socket backend.
- Incorrect interface selection.

Do not assume that changing permissions fixes every capture or send problem.

### 48.2 Verify the adapter in Windows

PowerShell:

```powershell
Get-NetAdapter
Get-NetIPConfiguration
ipconfig /all
```

These commands show Windows adapter names and IP configuration. Compare them with Scapy's interface output. The names may differ.

---

# Part 14 — Write reliable Scapy programs

## 49. A basic safe structure

A good Scapy script should:
1. Import only what it needs.
2. Validate input.
3. Select an interface intentionally.
4. Build packets separately from sending them.
5. Use timeouts.
6. Handle `None` responses.
7. Check layers before reading fields.
8. Handle exceptions.
9. Avoid unnecessary packet volume.
10. Clearly distinguish no response from a confirmed protocol response.

Example structure:

```python
import scapy.all as scapy


def inspect_icmp_response(response):
    """Print useful information from an ICMP diagnostic response."""

    if response is None:
        print("No response received.")
        return

    response.show()

    ip = response.getlayer(scapy.IP)
    if ip is not None:
        print("Source IP:", ip.src)
        print("Destination IP:", ip.dst)
        print("TTL:", ip.ttl)

    icmp = response.getlayer(scapy.ICMP)
    if icmp is not None:
        print("ICMP type:", icmp.type)
        print("ICMP code:", icmp.code)


def main():
    target = "127.0.0.1"

    packet = scapy.IP(dst=target) / scapy.ICMP()

    try:
        response = scapy.sr1(
            packet,
            timeout=2,
            verbose=False
        )
        inspect_icmp_response(response)

    except PermissionError as error:
        print("Permission problem:", error)

    except OSError as error:
        print("Network or capture backend problem:", error)


if __name__ == "__main__":
    main()
```

This is a diagnostic example using loopback. Adapt it only to a network you control.

## 50. Separate responsibilities into functions

For a larger program, keep different jobs separate:

```python
def discover_interfaces():
    """Display interfaces available to Scapy."""
    pass


def build_packet(target):
    """Construct a packet without sending it."""
    pass


def send_packet(packet, interface):
    """Send/receive using an explicitly selected interface."""
    pass


def inspect_response(response):
    """Interpret a response safely."""
    pass


def main():
    """Coordinate the program."""
    pass
```

This separation makes the code easier to test and debug.

## 51. Validate user input

If you accept an IP address from a command line, use Python's `ipaddress` module:

```python
import ipaddress

def validate_ipv4(value):
    address = ipaddress.ip_address(value)

    if address.version != 4:
        raise ValueError("An IPv4 address is required.")

    return str(address)
```

For port numbers, valid TCP/UDP port values are from 1 through 65535:

```python
def validate_port(value):
    port = int(value)

    if not 1 <= port <= 65535:
        raise ValueError("Port must be between 1 and 65535.")

    return port
```

Validate lists of ports individually. Do not silently accept malformed values.

## 52. Logging and error handling

Use Python's `logging` module for larger tools:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s"
)

logging.info("Starting diagnostic")
logging.warning("No response received")
logging.exception("Unexpected failure")
```

Use `logging.exception()` inside an `except` block when you want the traceback included.

Do not use a broad `except Exception: pass`. It hides errors and makes network debugging difficult.

---

# Part 15 — Common mistakes and their fixes

## 53. Mistake: using `scapy.all` twice

If you wrote:

```python
import scapy.all as scapy
```

Use:

```python
scapy.IP()
scapy.TCP()
scapy.sr1()
```

Not:

```python
scapy.all.IP()
```

## 54. Mistake: `interface=` instead of `iface=`

Incorrect:

```python
scapy.sr1(packet, interface=selected_iface)
```

Correct:

```python
scapy.sr1(packet, iface=selected_iface)
```

## 55. Mistake: assuming `sr1()` returns a pair

`sr1()` returns one response packet or `None`.

`sr()` returns answered and unanswered collections.

## 56. Mistake: reading a layer without checking

Potentially unsafe:

```python
tcp = response[scapy.TCP]
```

Safer when the layer may not exist:

```python
tcp = response.getlayer(scapy.TCP)

if tcp is not None:
    print(tcp.flags)
```

## 57. Mistake: treating timeout as proof of closed port

No response can mean packet loss, filtering, routing failure, target behavior, or an unsuitable timeout. Report it as inconclusive unless you have more evidence.

## 58. Mistake: confusing TCP and IP flags

- `response[scapy.TCP].flags`: TCP control flags such as SYN, ACK, RST.
- `response[scapy.IP].flags`: IPv4 fragmentation flags such as DF and MF.

They are different fields in different headers.

## 59. Mistake: confusing IP and MAC addresses

- IP address: Layer 3 address.
- MAC address: Layer 2 address.
- Interface: the adapter that has addresses assigned to it.

A remote server's MAC address is generally not visible across a router. On an Ethernet LAN, the destination MAC for remote traffic is usually the next-hop router's MAC.

## 60. Mistake: assuming a port number identifies the application with certainty

Ports are conventions, not proof of the service running there. A service can use a non-standard port, and a conventional port can host something else.

---

# Part 16 — Practice exercises

Work through these in order. Use loopback or a lab network you control.

1. Import Scapy and display the installed version.
2. Run `scapy.show_interfaces()` and identify your Wi-Fi adapter.
3. Print `scapy.conf.iface`.
4. Create `IP()/ICMP()` and call `show()`, `summary()`, and `show2()`.
5. Print the IP layer's `src`, `dst`, `ttl`, `proto`, and `flags`.
6. Create `IP()/TCP()` and inspect `sport`, `dport`, `flags`, `seq`, `ack`, and `window`.
7. Convert a packet to bytes and hexadecimal.
8. Use `sr1()` against loopback and handle a timeout.
9. Use `sr()` and print both answered and unanswered packets.
10. Inspect an ICMP response's `type` and `code`.
11. Inspect a TCP response's flags using bitwise operations.
12. Capture a small number of packets with `sniff(count=5, timeout=10)`.
13. Save a short capture with `wrpcap()`.
14. Read the capture with `rdpcap()` and inspect one packet.
15. Print every field definition in `IP.fields_desc`.
16. Use `help(scapy.sr1)` and `inspect.signature()` to explore parameters.
17. Write a function that safely prints IP, TCP, UDP, ICMP, and Raw information only when each layer exists.

For each exercise, write down:
- What packet/layer you created or received.
- Which function you used.
- What it returned.
- Which fields you inspected.
- What the result does and does not prove.

---

# Part 17 — Quick reference

## 61. Common imports

```python
import scapy.all as scapy
import inspect
```

## 62. Interface discovery

```python
scapy.show_interfaces()
scapy.get_if_list()
scapy.conf.iface
scapy.conf.ifaces.show()
```

## 63. Packet construction

```python
scapy.Ether()
scapy.ARP()
scapy.IP()
scapy.IPv6()
scapy.ICMP()
scapy.TCP()
scapy.UDP()
scapy.DNS()
scapy.Raw(load=b"data")
```

## 64. Packet inspection

```python
packet.show()
packet.summary()
packet.show2()
packet.haslayer(scapy.TCP)
packet.getlayer(scapy.TCP)
packet[scapy.TCP]
bytes(packet)
len(packet)
```

## 65. Send and receive

```python
scapy.send(packet, iface=selected_iface)
scapy.sendp(frame, iface=selected_iface)

answered, unanswered = scapy.sr(packet, iface=selected_iface, timeout=2)
answered, unanswered = scapy.srp(frame, iface=selected_iface, timeout=2)

response = scapy.sr1(packet, iface=selected_iface, timeout=2)
response = scapy.srp1(frame, iface=selected_iface, timeout=2)
```

## 66. Capture and PCAP

```python
packets = scapy.sniff(iface=selected_iface, count=5, timeout=10)
scapy.wrpcap("capture.pcap", packets)
packets = scapy.rdpcap("capture.pcap")
```

## 67. Discover documentation

```python
help(scapy.IP)
help(scapy.TCP)
help(scapy.sr1)
help(scapy.sniff)

dir(scapy.IP)
scapy.IP().show()

for field in scapy.IP.fields_desc:
    print(field.name, field.default)
```

---

# Part 18 — Official references

Use the official documentation for the exact Scapy version you have installed:

- Scapy documentation: https://scapy.readthedocs.io/
- Scapy usage guide: https://scapy.readthedocs.io/en/latest/usage.html
- Scapy API reference: https://scapy.readthedocs.io/en/latest/api/
- Scapy installation documentation: https://scapy.readthedocs.io/en/latest/installation.html
- Npcap official site: https://npcap.com/
- Python `ipaddress` documentation: https://docs.python.org/3/library/ipaddress.html
- Python `inspect` documentation: https://docs.python.org/3/library/inspect.html
- Python `logging` documentation: https://docs.python.org/3/library/logging.html

**Version reminder:** Scapy's API is broad and evolves. Use `help()`, `inspect.signature()`, `fields_desc`, and the documentation matching your installed version to verify a method's exact arguments and a protocol's complete field list.
