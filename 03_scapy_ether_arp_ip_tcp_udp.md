# Scapy Packet Construction: `Ether()/ARP()`, `Ether()/IP()/TCP()`, `Ether()/IP()/UDP()`

> Study by constructing and inspecting packets first. Send crafted traffic only on your own device/lab or with explicit authorization.

## 1. What is Scapy?

Scapy is a Python library for constructing, dissecting, sending, receiving, and inspecting network packets.

Install:

```bash
python -m pip install scapy
```

On Linux, raw packet operations commonly require root privileges or capabilities. On Windows, packet capture/transmission typically needs Npcap and an elevated terminal.

```python
from scapy.all import Ether, ARP, IP, TCP, UDP, ICMP, Raw
```

## 2. What does `/` mean?

In Scapy, `/` stacks one protocol layer on top of another. It does not mean filesystem division and does not itself send anything.

```python
packet = Ether() / IP() / TCP()
```

Read left to right:

```text
Ethernet frame
└── IPv4 packet
    └── TCP segment
```

With payload:

```python
packet = Ether() / IP() / UDP() / Raw(load=b"hello")
```

This represents `[Ethernet header][IPv4 header][UDP header][payload bytes]`.

## 3. `Ether()` — Ethernet layer

```python
frame = Ether()
frame.show()
```

Important fields:

| Field | Meaning |
|---|---|
| `dst` | Destination MAC address |
| `src` | Source MAC address |
| `type` | EtherType of the payload |

Common EtherTypes:

- `0x0800`: IPv4
- `0x0806`: ARP
- `0x86DD`: IPv6

Construct an Ethernet header:

```python
frame = Ether(
    src="02:00:00:00:00:01",
    dst="ff:ff:ff:ff:ff:ff",
    type=0x0806
)
frame.show()
```

`ff:ff:ff:ff:ff:ff` is the Ethernet broadcast MAC address. It reaches all interfaces in the local Layer-2 broadcast domain.

Scapy often infers EtherType from the next layer. Explicit values are useful for learning, but must match the payload.

## 4. `ARP()` — Address Resolution Protocol

ARP resolves an IPv4 address to a MAC address on the local link. ARP is **not inside IPv4**; it is carried directly in Ethernet.

### ARP fields

| Field | Meaning |
|---|---|
| `hwtype` | Hardware type; Ethernet is normally 1 |
| `ptype` | Protocol type; IPv4 is `0x0800` |
| `hwlen` | Hardware address length; Ethernet is 6 |
| `plen` | Protocol address length; IPv4 is 4 |
| `op` | 1=request, 2=reply |
| `hwsrc` | Sender MAC |
| `psrc` | Sender IPv4 address |
| `hwdst` | Target MAC |
| `pdst` | Target IPv4 address |

### Construct an ARP request (inspection only)

```python
from scapy.all import Ether, ARP

request = (
    Ether(dst="ff:ff:ff:ff:ff:ff") /
    ARP(
        op=1,
        pdst="192.168.1.1"
    )
)

request.show()
```

Explanation:

- `Ether(...)`: create the local-link frame.
- `dst=...`: broadcast it.
- `/ ARP(...)`: place the ARP message inside Ethernet.
- `op=1`: request.
- `pdst`: ask for the MAC associated with this IPv4 address.

Scapy may fill source address fields from the chosen interface during sending. Do not assume a source field has a particular value until you inspect it.

### Construct an ARP reply (do not forge/send)

```python
reply = (
    Ether(
        src="02:00:00:00:00:02",
        dst="02:00:00:00:00:01"
    ) /
    ARP(
        op=2,
        hwsrc="02:00:00:00:00:02",
        psrc="192.168.1.1",
        hwdst="02:00:00:00:00:01",
        pdst="192.168.1.10"
    )
)
reply.show()
```

This is a made-up lab illustration. Forged ARP replies can redirect or disrupt network traffic; never claim an address you do not own.

### One-address local ARP lookup

For a permitted local lab, `srp()` sends/receives at Layer 2:

```python
from scapy.all import Ether, ARP, srp

target = "192.168.1.1"
packet = Ether(dst="ff:ff:ff:ff:ff:ff") / ARP(pdst=target)

answered, unanswered = srp(
    packet,
    timeout=2,
    retry=1,
    verbose=False
)

for sent, received in answered:
    print("IP:", received.psrc)
    print("MAC:", received.hwsrc)
```

This checks one known local address, not an entire network range.

## 5. `Ether()/IP()/TCP()` — Ethernet + IPv4 + TCP

```text
Ethernet frame
└── IPv4 packet
    └── TCP segment
        └── optional payload
```

Construct, but do not send:

```python
from scapy.all import Ether, IP, TCP

packet = (
    Ether() /
    IP(dst="192.0.2.10") /
    TCP(
        sport=49152,
        dport=80,
        flags="S",
        seq=1000
    )
)

packet.show()
```

`192.0.2.10` is in a documentation-only IPv4 range, suitable for examples.

### IP layer fields

| Field | Meaning |
|---|---|
| `src` | Source IPv4 address |
| `dst` | Destination IPv4 address |
| `ttl` | IPv4 hop limit |
| `id` | Identification |
| `flags` | Fragmentation flags |
| `frag` | Fragment offset |
| `proto` | Next protocol number |
| `tos` | DS field / legacy Type of Service representation |
| `len` | Total IP packet length |
| `chksum` | IPv4 header checksum |

Scapy can infer `proto` from TCP/UDP/ICMP and calculate `len` and checksums during serialization.

### TCP layer fields

| Field | Meaning |
|---|---|
| `sport` | Source port |
| `dport` | Destination port |
| `seq` | Sequence number |
| `ack` | Acknowledgment number |
| `flags` | TCP control flags |
| `window` | Advertised receive window |
| `chksum` | TCP checksum |
| `options` | TCP options |

Common flags:

| Flag | Meaning |
|---|---|
| `S` | SYN: start/synchronize |
| `A` | ACK: acknowledgment field is valid |
| `F` | FIN: orderly close |
| `R` | RST: reset |
| `P` | PSH: push |
| `U` | URG: urgent pointer |

A SYN packet is only a connection-opening step; it does not mean a TCP connection is established.

Inspect values:

```python
print(packet[Ether].src)
print(packet[Ether].dst)
print(packet[IP].src)
print(packet[IP].dst)
print(packet[TCP].sport)
print(packet[TCP].dport)
print(packet[TCP].flags)
```

## 6. `Ether()/IP()/UDP()` — Ethernet + IPv4 + UDP

```text
Ethernet frame
└── IPv4 packet
    └── UDP datagram
        └── optional payload
```

Construct without sending:

```python
from scapy.all import Ether, IP, UDP

packet = (
    Ether() /
    IP(dst="192.0.2.20", ttl=64) /
    UDP(sport=53000, dport=53)
)
packet.show()
```

UDP fields:

| Field | Meaning |
|---|---|
| `sport` | Source port |
| `dport` | Destination port |
| `len` | UDP header plus data length |
| `chksum` | UDP checksum |

UDP header is 8 bytes:

```text
+----------------------+----------------------+
| Source Port (2B)     | Destination Port (2B)|
+----------------------+----------------------+
| Length (2B)          | Checksum (2B)        |
+----------------------+----------------------+
```

UDP has no TCP-style handshake, sequence numbers, retransmission, or built-in flow control.

### Add payload bytes

```python
from scapy.all import Ether, IP, UDP, Raw

packet = (
    Ether() /
    IP(dst="192.0.2.20") /
    UDP(sport=53000, dport=9999) /
    Raw(load=b"hello from my lab")
)
packet.show()
```

`Raw(load=...)` holds application payload bytes. The application protocol determines what those bytes mean.

## 7. Compare the three expressions

| Scapy expression | Layer stack | Meaning |
|---|---|---|
| `Ether()/ARP()` | Ethernet → ARP | Resolve local IPv4 address to MAC |
| `Ether()/IP()/TCP()` | Ethernet → IPv4 → TCP | TCP segment carried in an IP packet and frame |
| `Ether()/IP()/UDP()` | Ethernet → IPv4 → UDP | UDP datagram carried in an IP packet and frame |

Remember: ARP is directly inside Ethernet, not inside IP. TCP and UDP are inside IP.

## 8. Inspect packets

```python
from scapy.all import Ether, IP, TCP, UDP, Raw, hexdump, ls

packet = Ether() / IP(dst="192.0.2.10") / TCP(dport=80, flags="S")

packet.show()       # Display current fields
packet.show2()      # Serialize first; calculated fields are filled
print(packet.summary())
print(packet.layers())
hexdump(packet)     # Hexadecimal + ASCII view
ls(IP)              # List IP layer fields
ls(TCP)             # List TCP layer fields
```

Access a layer with `packet[IP]`; check it with `packet.haslayer(TCP)`.

## 9. Bytes and serialization

```python
raw_bytes = bytes(packet)

print(raw_bytes)
print(raw_bytes.hex())
print(len(raw_bytes))
```

`bytes(packet)` serializes the packet. Calculated fields such as lengths/checksums can be populated during serialization.

Do not confuse:
- Python text: `"hello"`
- bytes: `b"hello"`
- hexadecimal text: `"68656c6c6f"`

Hex is a printable representation of bytes, not a separate network protocol.

## 10. Scapy send/receive functions

| Function | Layer | Purpose |
|---|---|---|
| `send()` | Layer 3 | Send network-layer packets |
| `sendp()` | Layer 2 | Send link-layer frames |
| `sr()` | Layer 3 | Send and receive |
| `sr1()` | Layer 3 | Send and return first response |
| `srp()` | Layer 2 | Send and receive Layer-2 packets |
| `srp1()` | Layer 2 | Send and return first Layer-2 response |

Build and inspect first. A local loopback example:

```python
from scapy.all import IP, ICMP, sr1

packet = IP(dst="127.0.0.1") / ICMP()
response = sr1(packet, timeout=2, verbose=False)

if response:
    response.show()
else:
    print("No response")
```

Raw packet permissions may be required.

## 11. Why Ethernet destination MAC may be the gateway

Suppose Computer A sends to a server on a different network:

- IP destination = remote server IP.
- Ethernet destination = default gateway's MAC address.

A does not normally use the remote server's MAC because MAC delivery is local-link delivery. A uses ARP to learn the gateway MAC, then sends the frame to the router. The router removes that frame, routes the IP packet, and creates a new frame for the next link.

## 12. Common mistakes

1. `Ether()/ARP()` is correct for ordinary Ethernet ARP; ARP is not inside IPv4.
2. `/` stacks layers; it does not transmit.
3. `send()` is Layer 3; `sendp()` is Layer 2.
4. `show()` may not show calculated values; use `show2()` or `bytes(packet)`.
5. Ethernet `dst` expects a MAC; IP `dst` expects an IP address.
6. UDP is not simply TCP without a handshake; their services differ.
7. A constructed SYN is not an established TCP connection.
8. Scapy can infer fields, but explicit values are useful for understanding.

## 13. Practice exercises

1. Construct `Ether()/ARP()` and inspect it.
2. Change ARP `op` from 1 to 2; compare the fields without sending a forged reply.
3. Construct `Ether()/IP()/TCP()` with destination `192.0.2.10`, port 80, SYN flag.
4. Change destination port to 443; explain what changes.
5. Construct `Ether()/IP()/UDP()` with destination port 53.
6. Add `Raw(load=b"hello")` and inspect the payload.
7. Compare `show()`, `show2()`, `summary()`, and `hexdump()`.
8. Explain why the Ethernet destination may be the gateway MAC for a remote IP destination.

## 14. References

- Scapy docs: https://scapy.readthedocs.io/
- Scapy usage: https://scapy.readthedocs.io/en/latest/usage.html
- ARP RFC 826: https://www.rfc-editor.org/rfc/rfc826
- IPv4 RFC 791: https://www.rfc-editor.org/rfc/rfc791
- TCP RFC 9293: https://www.rfc-editor.org/rfc/rfc9293
- UDP RFC 768: https://www.rfc-editor.org/rfc/rfc768
