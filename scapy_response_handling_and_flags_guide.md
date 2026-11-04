# Scapy: Complete Guide to Packet Responses, Flags, and Response Handling

**Purpose:** Learn how to receive, inspect, interpret, and handle different packet responses with Scapy.

This guide covers TCP flags, IP flags, ICMP types and codes, UDP, ARP, DNS, payloads, answered/unanswered packets, and reusable response-inspection patterns.

> Use packet crafting and scanning only on systems and networks you own or have explicit permission to test.

## 1. What Scapy returns

| Function | Purpose | Return value |
|---|---|---|
| `send()` | Send a Layer 3 packet | Does not return a received response |
| `sendp()` | Send a Layer 2 frame | Does not return a received response |
| `sr()` | Send Layer 3 packets and receive replies | `(answered, unanswered)` |
| `sr1()` | Send a Layer 3 packet and receive one reply | One packet or `None` |
| `srp()` | Send Layer 2 frames and receive replies | `(answered, unanswered)` |
| `srp1()` | Send a Layer 2 frame and receive one reply | One packet or `None` |

```python
import scapy.all as scapy

packet = scapy.IP(dst="192.168.1.1") / scapy.ICMP()

response = scapy.sr1(packet, timeout=2, verbose=0)

if response is not None:
    response.show()
else:
    print("No response received before timeout")
```

`sr1()` returns one response packet or `None`, not a `(sent, received)` pair. `sr()` returns answered and unanswered collections.

```python
answered, unanswered = scapy.sr(packet, timeout=2, verbose=0)

for sent_packet, received_packet in answered:
    print("Sent:")
    sent_packet.show()
    print("Received:")
    received_packet.show()

for packet_without_reply in unanswered:
    print("No matching response:")
    packet_without_reply.show()
```

Each item in `answered` is a pair: `(sent_packet, received_packet)`. `unanswered` contains sent packets for which Scapy did not receive a matching response.

## 2. A response is a stack of protocol layers

A response may contain layers such as:

```text
Ethernet
    IP
        TCP
```

or:

```text
Ethernet
    IP
        UDP
            DNS
```

Inspect a packet:

```python
response.show()       # Detailed decoded packet
response.summary()    # Short summary
response.layers()     # Layer classes present
bytes(response)       # Raw packet bytes
```

### Accessing a layer safely

```python
if response is not None and response.haslayer(scapy.TCP):
    tcp = response.getlayer(scapy.TCP)
    print(tcp.flags)
```

- `haslayer(Layer)` checks whether a layer exists.
- `getlayer(Layer)` returns the layer or `None`.
- `response[Layer]` accesses a layer directly, but can raise an error if it is absent.

Do not assume every response is TCP. It may be ICMP, UDP, ARP, or another protocol.

---

## 3. TCP flags

TCP flags are control bits in the TCP header. They communicate how a TCP endpoint is handling a connection or data.

| Flag | Hex value | Meaning |
|---|---:|---|
| FIN | `0x01` | Sender has finished sending data |
| SYN | `0x02` | Synchronize sequence numbers; commonly starts a connection |
| RST | `0x04` | Reset or reject a connection |
| PSH | `0x08` | Push data toward the receiving application |
| ACK | `0x10` | Acknowledgment field is valid |
| URG | `0x20` | Urgent pointer is significant |
| ECE | `0x40` | ECN Echo |
| CWR | `0x80` | Congestion Window Reduced |
| NS | `0x100` | ECN Nonce Sum bit |

The classic TCP flags occupy eight bits; the NS bit is an additional flag in the TCP header's reserved/NS area.

### 3.1 Hexadecimal combinations

A packet can set multiple flags at once. The hexadecimal value is the bitwise combination of the individual flags.

| Hex | Set flags | Common interpretation |
|---|---|---|
| `0x02` | SYN | Connection request |
| `0x12` | SYN + ACK | Typical response accepting a TCP SYN |
| `0x04` | RST | Reset |
| `0x14` | RST + ACK | Typical response rejecting a TCP SYN |
| `0x10` | ACK | ACK only |
| `0x11` | FIN + ACK | Connection-closing packet |
| `0x18` | PSH + ACK | Data-bearing packet commonly carrying pushed data |
| `0x19` | FIN + PSH + ACK | Combination of three flags |

These values tell you which bits are set. Interpret them in context: direction, packet sequence, ports, sequence numbers, and the request that caused the response all matter.

### Why `0x14` means RST + ACK

```python
RST = 0x04
ACK = 0x10

combined = RST | ACK

print(hex(combined))  # 0x14
print(int(combined))  # 20
```

Binary:

```text
RST  00000100
ACK  00010000
     --------
OR   00010100  = 0x14
```

Similarly:

```python
SYN = 0x02
ACK = 0x10

print(hex(SYN | ACK))  # 0x12
```

### 3.2 Exact comparison versus bitwise testing

Use `==` to test an exact combination:

```python
if int(tcp.flags) == 0x14:
    print("Exactly RST + ACK")
```

Use bitwise AND (`&`) to test whether a flag is present, even if other flags are also set:

```python
flags = int(tcp.flags)

if flags & 0x04:
    print("RST is set")

if flags & 0x10:
    print("ACK is set")

if flags & 0x02:
    print("SYN is set")
```

`flags & 0x04` keeps only the RST bit. A nonzero result means that bit was set.

```python
flags = 0x14

print(bool(flags & 0x04))  # True: RST is present
print(bool(flags & 0x10))  # True: ACK is present
print(bool(flags & 0x02))  # False: SYN is absent
```

To test a required combination, use parentheses:

```python
if (flags & 0x12) == 0x12:
    print("Both SYN and ACK are set")
```

This checks for SYN and ACK, but does not by itself exclude additional flags such as RST. Add a separate test if that matters:

```python
if (flags & 0x12) == 0x12 and not (flags & 0x04):
    print("SYN and ACK are set, RST is not set")
```

### 3.3 Reading and setting flags in Scapy

```python
tcp = response.getlayer(scapy.TCP)

print(tcp.flags)           # Symbolic representation, e.g. SA or RA
print(int(tcp.flags))      # Decimal integer
print(hex(int(tcp.flags))) # Hexadecimal representation
```

Creating packets:

```python
scapy.TCP(flags="S")   # SYN
scapy.TCP(flags="SA")  # SYN + ACK
scapy.TCP(flags="R")   # RST
scapy.TCP(flags="RA")  # RST + ACK
scapy.TCP(flags="PA")  # PSH + ACK
scapy.TCP(flags="FA")  # FIN + ACK
```

---

## 4. TCP response scenarios

A TCP SYN scan sends a SYN packet and interprets the response.

```python
syn_packet = (
    scapy.IP(dst="192.168.1.10") /
    scapy.TCP(dport=80, flags="S")
)

response = scapy.sr1(syn_packet, timeout=2, verbose=0)
```

| Response | Typical SYN-scan interpretation |
|---|---|
| SYN + ACK (`0x12`) | Port is open |
| RST (`0x04`) or RST + ACK (`0x14`) | Port is closed |
| No response | Filtered, dropped, lost, or otherwise unknown |
| ICMP unreachable | Network/host/firewall error; inspect ICMP type and code |
| Other TCP flags | Unexpected; inspect packet and context |

### 4.1 SYN-ACK: likely open

```python
if response is not None and response.haslayer(scapy.TCP):
    tcp = response[scapy.TCP]
    flags = int(tcp.flags)

    if (flags & 0x12) == 0x12 and not (flags & 0x04):
        print("SYN and ACK received: port is likely open")
```

### 4.2 RST: likely closed

```python
if response is not None and response.haslayer(scapy.TCP):
    tcp = response[scapy.TCP]
    flags = int(tcp.flags)

    if flags & 0x04:
        print("TCP reset received: port is likely closed")
```

For a normal SYN scan, a RST response is the usual indication that the destination port is closed.

### 4.3 No response

```python
if response is None:
    print("No response: possibly filtered or packet dropped")
```

A timeout does not prove that a port is closed or open. Firewalls may silently drop probes, a response may be lost, or the host may not respond.

### 4.4 Unexpected TCP response

```python
if response is not None and response.haslayer(scapy.TCP):
    tcp = response[scapy.TCP]
    flags = int(tcp.flags)

    if (flags & 0x12) == 0x12 and not (flags & 0x04):
        print("Likely open")
    elif flags & 0x04:
        print("Reset received; likely closed for a SYN scan")
    else:
        print("Unexpected TCP flags:", tcp.flags, hex(flags))
```

Do not label every RST as a closed port in every possible context. That interpretation is specific to the probe and response context.

---

## 5. TCP header fields beyond flags

```python
if response is not None and response.haslayer(scapy.TCP):
    tcp = response[scapy.TCP]

    print("Source port:", tcp.sport)
    print("Destination port:", tcp.dport)
    print("Sequence number:", tcp.seq)
    print("Acknowledgment number:", tcp.ack)
    print("Flags:", tcp.flags)
    print("Window size:", tcp.window)
    print("Header length:", tcp.dataofs)
    print("Urgent pointer:", tcp.urgptr)
    print("Options:", tcp.options)
```

| Field | Meaning |
|---|---|
| `sport` | Source TCP port |
| `dport` | Destination TCP port |
| `seq` | Sequence number |
| `ack` | Acknowledgment number |
| `flags` | TCP control flags |
| `window` | Advertised receive window |
| `dataofs` | TCP header length in 32-bit words |
| `urgptr` | Urgent pointer |
| `options` | TCP options |

TCP sequence numbers track bytes in the stream. The acknowledgment number indicates the next byte expected. A SYN consumes one sequence number. If a client sends a SYN with sequence number 1000, a typical SYN-ACK acknowledges 1001.

```python
tcp = response[scapy.TCP]

print("Server sequence:", tcp.seq)
print("Server acknowledgment:", tcp.ack)
```

When constructing a reset in response to a SYN-ACK, the response's acknowledgment number is commonly used as the reset's sequence number. The exact reset construction depends on the TCP state and packet context.

---

## 6. IPv4 response fields and IP flags

Do not confuse `IP.flags` with `TCP.flags`.

- `ip.flags` relates to IPv4 fragmentation.
- `tcp.flags` relates to TCP connection and data control.

```python
if response is not None and response.haslayer(scapy.IP):
    ip = response[scapy.IP]

    print("Source IP:", ip.src)
    print("Destination IP:", ip.dst)
    print("TTL:", ip.ttl)
    print("Protocol number:", ip.proto)
    print("Identification:", ip.id)
    print("Fragment offset:", ip.frag)
    print("IP flags:", ip.flags)
    print("Header length:", ip.ihl)
    print("Total length:", ip.len)
```

| Field | Meaning |
|---|---|
| `src` | Source IP address |
| `dst` | Destination IP address |
| `ttl` | Time To Live |
| `proto` | Encapsulated IP protocol number |
| `id` | IPv4 identification |
| `frag` | Fragment offset |
| `flags` | IPv4 fragmentation flags |
| `ihl` | IPv4 header length in 32-bit words |
| `len` | Total IPv4 packet length |

IPv4 fragmentation flags:

- Reserved: reserved bit.
- DF: Don't Fragment.
- MF: More Fragments.

```python
ip = response[scapy.IP]

print("IP flags:", ip.flags)

if ip.flags.DF:
    print("Don't Fragment is set")

if ip.flags.MF:
    print("More Fragments is set")
```

TTL is decremented by routers as they forward IPv4 packets. The TTL in a response is the remaining TTL, not necessarily the original TTL chosen by the sender.

---

## 7. ICMP responses: types and codes

ICMP reports network conditions and supports diagnostics such as ping and traceroute.

An ICMP Echo Request:

```python
packet = (
    scapy.IP(dst="192.168.1.1") /
    scapy.ICMP(type=8, code=0)
)
```

A typical Echo Reply has Type 0, Code 0.

### 7.1 Common ICMP types and codes

| Type | Name | Code | Meaning |
|---:|---|---:|---|
| 0 | Echo Reply | 0 | Ping reply |
| 3 | Destination Unreachable | 0 | Network unreachable |
| 3 | Destination Unreachable | 1 | Host unreachable |
| 3 | Destination Unreachable | 2 | Protocol unreachable |
| 3 | Destination Unreachable | 3 | Port unreachable |
| 3 | Destination Unreachable | 4 | Fragmentation needed |
| 3 | Destination Unreachable | 5 | Source route failed |
| 3 | Destination Unreachable | 9 | Network administratively prohibited |
| 3 | Destination Unreachable | 10 | Host administratively prohibited |
| 3 | Destination Unreachable | 13 | Communication administratively prohibited |
| 5 | Redirect | 0–3 | Redirect message |
| 8 | Echo Request | 0 | Ping request |
| 9 | Router Advertisement | 0 | Router advertisement |
| 10 | Router Solicitation | 0 | Router solicitation |
| 11 | Time Exceeded | 0 | TTL expired in transit |
| 11 | Time Exceeded | 1 | Fragment reassembly timeout |
| 12 | Parameter Problem | 0 | Problem with IP header |

A code only has meaning in the context of its type. For example, Type 3 Code 3 means port unreachable.

### 7.2 Reading ICMP fields

```python
if response is not None and response.haslayer(scapy.ICMP):
    icmp = response[scapy.ICMP]

    print("Type:", icmp.type)
    print("Code:", icmp.code)
    print("Checksum:", icmp.chksum)
    print("Identifier:", icmp.id)
    print("Sequence:", icmp.seq)
```

Some fields are meaningful only for particular ICMP messages. For example, `id` and `seq` are associated with Echo messages.

### 7.3 Interpreting ICMP responses

```python
if response is not None and response.haslayer(scapy.ICMP):
    icmp = response[scapy.ICMP]

    if icmp.type == 0 and icmp.code == 0:
        print("Echo Reply: host responded")
    elif icmp.type == 3 and icmp.code == 3:
        print("Destination port unreachable")
    elif icmp.type == 3 and icmp.code == 1:
        print("Destination host unreachable")
    elif icmp.type == 11 and icmp.code == 0:
        print("TTL expired in transit")
    else:
        print("Other ICMP response:", icmp.type, icmp.code)
```

For traceroute, Type 11 Code 0 is especially important: it indicates that the probe's TTL expired in transit.

---

## 8. UDP responses

UDP is connectionless. It has no TCP-style SYN, ACK, FIN, or RST flags.

A UDP probe may receive:

1. A UDP response from an application.
2. ICMP Destination Unreachable, commonly Port Unreachable.
3. Another ICMP error.
4. No response.

```python
packet = (
    scapy.IP(dst="192.168.1.1") /
    scapy.UDP(dport=53)
)

response = scapy.sr1(packet, timeout=2, verbose=0)
```

Inspect:

```python
if response is None:
    print("No UDP response")
elif response.haslayer(scapy.UDP):
    udp = response[scapy.UDP]

    print("UDP response received")
    print("Source port:", udp.sport)
    print("Destination port:", udp.dport)
    print("Length:", udp.len)
    print("Checksum:", udp.chksum)
elif response.haslayer(scapy.ICMP):
    icmp = response[scapy.ICMP]

    print("ICMP Type:", icmp.type)
    print("ICMP Code:", icmp.code)

    if icmp.type == 3 and icmp.code == 3:
        print("Destination UDP port is unreachable")
```

For UDP scanning, ICMP Type 3 Code 3 usually indicates a closed UDP port. No response is inconclusive; it does not prove that a UDP port is open.

---

## 9. ARP responses

ARP maps an IPv4 address to a MAC address on a local network. ARP operates at Layer 2, so use `srp()` or `srp1()` for ARP Ethernet frames.

```python
packet = (
    scapy.Ether(dst="ff:ff:ff:ff:ff:ff") /
    scapy.ARP(pdst="192.168.1.1", op=1)
)

response = scapy.srp1(packet, timeout=2, verbose=0)
```

Inspect:

```python
if response is not None and response.haslayer(scapy.ARP):
    arp = response[scapy.ARP]

    print("Operation:", arp.op)
    print("Sender MAC:", arp.hwsrc)
    print("Sender IP:", arp.psrc)
    print("Target MAC:", arp.hwdst)
    print("Target IP:", arp.pdst)
else:
    print("No ARP response")
```

| ARP field | Meaning |
|---|---|
| `op` | Operation: 1=request, 2=reply |
| `hwsrc` | Sender hardware/MAC address |
| `psrc` | Sender protocol/IP address |
| `hwdst` | Target hardware/MAC address |
| `pdst` | Target protocol/IP address |

---

## 10. DNS responses

DNS commonly uses UDP port 53, although TCP is also used. A DNS response may contain A, AAAA, CNAME, MX, NS, TXT, and other record types.

```python
packet = (
    scapy.IP(dst="8.8.8.8") /
    scapy.UDP(dport=53) /
    scapy.DNS(
        rd=1,
        qd=scapy.DNSQR(qname="example.com")
    )
)

response = scapy.sr1(packet, timeout=3, verbose=0)
```

Inspect:

```python
if response is not None and response.haslayer(scapy.DNS):
    dns = response[scapy.DNS]

    print("Transaction ID:", dns.id)
    print("QR:", dns.qr)
    print("Response code:", dns.rcode)
    print("Answer count:", dns.ancount)

    if dns.qr == 1:
        print("DNS response received")

    if dns.an:
        dns.an.show()
else:
    print("No DNS response")
```

DNS `qr`:

- `0`: query
- `1`: response

Common DNS response codes (`rcode`):

| Code | Name | Meaning |
|---:|---|---|
| 0 | NOERROR | No DNS error |
| 1 | FORMERR | Format error |
| 2 | SERVFAIL | Server failure |
| 3 | NXDOMAIN | Domain name does not exist |
| 4 | NOTIMP | Operation not implemented |
| 5 | REFUSED | Server refused the query |

Do not assume every DNS answer is an IPv4 address. Check the DNS record type before interpreting its data.

---

## 11. Ethernet responses

The Ethernet layer contains local-link addressing information.

```python
if response is not None and response.haslayer(scapy.Ether):
    eth = response[scapy.Ether]

    print("Source MAC:", eth.src)
    print("Destination MAC:", eth.dst)
    print("EtherType:", hex(eth.type))
```

Common EtherTypes:

| EtherType | Protocol |
|---|---|
| `0x0800` | IPv4 |
| `0x0806` | ARP |
| `0x86DD` | IPv6 |

Across routers, Ethernet MAC addresses normally identify devices on the local link. They do not necessarily identify the ultimate IP source and destination.

---

## 12. Raw payloads and data representation

The payload may contain application data that is not represented by the header fields. Scapy uses `Raw` for undecoded raw payload bytes.

```python
if response is not None and response.haslayer(scapy.Raw):
    payload = response[scapy.Raw].load

    print("Bytes:", payload)
    print("Hexadecimal:", payload.hex())
    print("Text:", payload.decode("utf-8", errors="replace"))
```

Example:

```python
data = b"Hello"

print(data)       # b'Hello'
print(data.hex()) # 48656c6c6f
print(data.decode("utf-8"))  # Hello
```

A payload is not necessarily text. It can contain binary data, compressed content, or encrypted data. For unknown payloads, inspect bytes and hexadecimal before attempting text decoding.

TLS application data is encrypted. Converting it to text does not decrypt it.

---

## 13. HTTP and TLS response data

HTTP is an application-layer protocol. A plaintext HTTP response may contain a status line such as:

```text
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 125
```

Common HTTP status codes:

| Code | Meaning |
|---:|---|
| 200 | OK |
| 201 | Created |
| 301 | Moved Permanently |
| 302 | Found / temporary redirect |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |

These are HTTP status codes. They are not TCP flags or ICMP codes.

For an undecoded plaintext payload:

```python
if response is not None and response.haslayer(scapy.Raw):
    print(response[scapy.Raw].load)
```

Scapy does not automatically make every TCP response an HTTP response. You need an actual HTTP exchange and appropriate dissection. HTTPS application data is encrypted; without suitable decryption context, you generally cannot read the application content.

---

## 14. Answered and unanswered collections

Use `sr()` when you send multiple probes and want to associate responses with the probes.

```python
packets = []

for port in [22, 80, 443]:
    packet = (
        scapy.IP(dst="192.168.1.1") /
        scapy.TCP(dport=port, flags="S")
    )
    packets.append(packet)

answered, unanswered = scapy.sr(
    packets,
    timeout=2,
    verbose=0
)
```

Inspect answered pairs:

```python
for sent_packet, received_packet in answered:

    print("Requested destination port:",
          sent_packet[scapy.TCP].dport)

    if received_packet.haslayer(scapy.TCP):
        tcp = received_packet[scapy.TCP]
        print("Response flags:", tcp.flags)
```

Inspect unanswered packets:

```python
for packet_without_reply in unanswered:
    print(
        "No matching response for port:",
        packet_without_reply[scapy.TCP].dport
    )
```

A packet being unanswered does not by itself tell you why no response arrived.

---

## 15. A reusable response-inspection function

This function does not send probes. It accepts a packet and displays information from the layers it contains.

```python
import scapy.all as scapy


def inspect_response(response):

    if response is None:
        print("No response received")
        return

    print("\\n[ PACKET SUMMARY ]")
    print(response.summary())

    if response.haslayer(scapy.Ether):
        eth = response[scapy.Ether]

        print("\\n[ ETHERNET ]")
        print("Source MAC:", eth.src)
        print("Destination MAC:", eth.dst)
        print("EtherType:", hex(eth.type))

    if response.haslayer(scapy.IP):
        ip = response[scapy.IP]

        print("\\n[ IPv4 ]")
        print("Source IP:", ip.src)
        print("Destination IP:", ip.dst)
        print("TTL:", ip.ttl)
        print("Protocol:", ip.proto)
        print("IP flags:", ip.flags)

    if response.haslayer(scapy.TCP):
        tcp = response[scapy.TCP]

        print("\\n[ TCP ]")
        print("Source port:", tcp.sport)
        print("Destination port:", tcp.dport)
        print("Sequence:", tcp.seq)
        print("Acknowledgment:", tcp.ack)
        print("Flags:", tcp.flags)
        print("Flags in hex:", hex(int(tcp.flags)))
        print("Window:", tcp.window)

    if response.haslayer(scapy.UDP):
        udp = response[scapy.UDP]

        print("\\n[ UDP ]")
        print("Source port:", udp.sport)
        print("Destination port:", udp.dport)
        print("Length:", udp.len)

    if response.haslayer(scapy.ICMP):
        icmp = response[scapy.ICMP]

        print("\\n[ ICMP ]")
        print("Type:", icmp.type)
        print("Code:", icmp.code)

    if response.haslayer(scapy.ARP):
        arp = response[scapy.ARP]

        print("\\n[ ARP ]")
        print("Operation:", arp.op)
        print("Sender MAC:", arp.hwsrc)
        print("Sender IP:", arp.psrc)
        print("Target MAC:", arp.hwdst)
        print("Target IP:", arp.pdst)

    if response.haslayer(scapy.DNS):
        dns = response[scapy.DNS]

        print("\\n[ DNS ]")
        print("Transaction ID:", dns.id)
        print("QR:", dns.qr)
        print("Response code:", dns.rcode)
        print("Answer count:", dns.ancount)

    if response.haslayer(scapy.Raw):
        payload = response[scapy.Raw].load

        print("\\n[ RAW PAYLOAD ]")
        print("Bytes:", payload)
        print("Hex:", payload.hex())
```

Usage:

```python
packet = scapy.IP(dst="192.168.1.1") / scapy.ICMP()

response = scapy.sr1(
    packet,
    timeout=2,
    verbose=0
)

inspect_response(response)
```

---

## 16. General response-handling workflow

Use this order when building a network tool:

1. **Check whether a response exists.**

   ```python
   if response is None:
       print("No response")
   ```

2. **Inspect the packet.**

   ```python
   response.show()
   print(response.summary())
   ```

3. **Check which layers exist.**

   ```python
   response.haslayer(scapy.IP)
   response.haslayer(scapy.TCP)
   response.haslayer(scapy.ICMP)
   ```

4. **Retrieve the layer safely.**

   ```python
   tcp = response.getlayer(scapy.TCP)
   ```

5. **Extract the fields needed for the task.**

   ```python
   print(tcp.sport)
   print(tcp.dport)
   print(tcp.flags)
   ```

6. **Interpret the fields in context.**

   Ask: What did I send? Which host replied? Which protocol responded? What do the flags or codes mean?

7. **Choose an appropriate action.**

   Report the result, record it, or send a follow-up packet if the protocol operation requires one.

---

## 17. Common mistakes

| Mistake | Why it is a problem | Better approach |
|---|---|---|
| Assuming every response is TCP | Responses may be ICMP, UDP, ARP, etc. | Check `haslayer()` |
| Treating no response as a closed port | A timeout is inconclusive | Report no response / possible filtering |
| Confusing `IP.flags` and `TCP.flags` | They represent different protocol fields | Access the correct layer |
| Using `==` for every flag check | Extra flags cause exact comparisons to fail | Use bitwise checks where appropriate |
| Assuming `sr1()` returns a pair | It returns one packet or `None` | Use `sr()` for answered/unanswered pairs |
| Using `send()` when you need a reply | `send()` does not wait for a response | Use `sr1()` or `sr()` |
| Assuming raw payload is text | Payload can be binary or encrypted | Inspect bytes and hex first |
| Treating all ICMP Type 3 messages as closed ports | Type 3 has several different codes | Check both `type` and `code` |
| Failing to match replies to probes | A reply may be associated with a different request | Compare addresses, ports, and protocol fields |

### Correlating TCP replies

When sending multiple probes, check that a response belongs to the expected exchange.

For a typical TCP response:

- The response source port should normally equal the target service port.
- The response destination port should normally equal the source port used by your probe.
- The IP addresses should match the expected endpoints.
- Sequence and acknowledgment numbers can help validate the exchange.

```python
if response is not None and response.haslayer(scapy.TCP):
    tcp = response[scapy.TCP]

    print("Response source port:", tcp.sport)
    print("Response destination port:", tcp.dport)
```

---

## 18. Useful Scapy inspection methods and attributes

| Method / attribute | Purpose |
|---|---|
| `response.show()` | Display decoded packet fields |
| `response.summary()` | Short packet description |
| `response.layers()` | Show packet layer classes |
| `response.haslayer(Layer)` | Check whether a layer exists |
| `response.getlayer(Layer)` | Retrieve a layer or `None` |
| `response[Layer]` | Access a specific layer |
| `bytes(response)` | Convert packet to bytes |
| `response.time` | Packet timestamp, when available |
| `response.sniffed_on` | Capture interface, when available |
| `layer.payload` | Next payload layer |
| `layer.fields` | Decoded field dictionary |
| `Raw.load` | Raw payload bytes |

```python
response.show()
print(response.summary())
print(response.layers())

if response.haslayer(scapy.TCP):
    tcp = response.getlayer(scapy.TCP)
    print(tcp.fields)

raw_bytes = bytes(response)
print(raw_bytes.hex())
```

---

## 19. Quick reference cheat sheet

### TCP

```python
tcp = response.getlayer(scapy.TCP)

tcp.sport
tcp.dport
tcp.seq
tcp.ack
tcp.flags
tcp.window
tcp.options
```

### IP

```python
ip = response.getlayer(scapy.IP)

ip.src
ip.dst
ip.ttl
ip.proto
ip.id
ip.flags
ip.frag
ip.len
```

### ICMP

```python
icmp = response.getlayer(scapy.ICMP)

icmp.type
icmp.code
icmp.chksum
icmp.id
icmp.seq
```

### UDP

```python
udp = response.getlayer(scapy.UDP)

udp.sport
udp.dport
udp.len
udp.chksum
```

### ARP

```python
arp = response.getlayer(scapy.ARP)

arp.op
arp.hwsrc
arp.psrc
arp.hwdst
arp.pdst
```

### DNS

```python
dns = response.getlayer(scapy.DNS)

dns.id
dns.qr
dns.opcode
dns.rcode
dns.qd
dns.an
dns.ancount
```

### Raw payload

```python
payload = response.getlayer(scapy.Raw).load

payload
payload.hex()
payload.decode("utf-8", errors="replace")
```

Always check `haslayer()` before accessing a layer, and check that the response is not `None`.

---

## 20. Recommended learning progression

1. **Packet construction:** `Ether()`, `IP()`, `TCP()`, `UDP()`, `ICMP()`, `ARP()`.
2. **Sending and receiving:** `send()`, `sendp()`, `sr()`, `sr1()`, `srp()`, `srp1()`.
3. **Response inspection:** `show()`, `summary()`, `haslayer()`, `getlayer()`, `fields`.
4. **Protocol interpretation:** TCP flags, ICMP types/codes, IP fields, ARP fields, DNS responses.
5. **Response correlation:** Match requests and replies using addresses, ports, sequence numbers, and protocol fields.
6. **Building tools:** Ping utility, traceroute, ARP discovery, TCP SYN scanner, UDP scanner, packet analyzer.

## Official references

- [Scapy documentation](https://scapy.readthedocs.io/en/latest/)
- [Scapy usage guide](https://scapy.readthedocs.io/en/latest/usage.html)
- [RFC 9293 – Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293)
- [RFC 792 – Internet Control Message Protocol](https://www.rfc-editor.org/rfc/rfc792)
- [RFC 768 – User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768)
- [RFC 826 – Address Resolution Protocol](https://www.rfc-editor.org/rfc/rfc826)

**Final principle:** Do not just receive a packet and print it. Identify its layers, extract the fields relevant to your task, interpret them in the context of the request, and only then decide what the result means.
