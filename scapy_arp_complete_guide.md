# Scapy and ARP: Complete Practical Guide

> Learn how to build, inspect, send, and receive Ethernet/ARP packets with Scapy. Practise only on your own devices, lab, or networks you are authorized to test.

## 1. What is Scapy?

[Scapy](https://scapy.net/) is a Python library for constructing, inspecting, sending, receiving, and dissecting network packets. It is useful for learning protocols, troubleshooting, packet analysis, and authorized security testing.

Install it:

```bash
python -m pip install scapy
python -m pip show scapy
```

Import only what you need:

```python
from scapy.all import Ether, ARP, srp
```

Or use a module alias:

```python
import scapy.all as scapy

packet = scapy.Ether()
```

Both approaches are valid. Explicit imports are often easier to maintain in larger programs than `from scapy.all import *`.

## 2. Understand the protocol layers

For this lesson, remember:

| Layer / concept | Purpose | Scapy class |
|---|---|---|
| Ethernet | Local network frame; uses MAC addresses | `Ether()` |
| ARP | Resolves a local IPv4 address to a MAC address | `ARP()` |
| IPv4 | Carries data between IPv4 addresses | `IP()` |
| TCP | Reliable, connection-oriented transport | `TCP()` |
| UDP | Connectionless transport | `UDP()` |

An Ethernet frame can carry an ARP message:

```text
Ethernet frame
└── ARP message
```

An Ethernet frame can also carry an IPv4 packet, which can carry TCP:

```text
Ethernet frame
└── IPv4 packet
    └── TCP segment
```

Scapy represents this stacking using `/`:

```python
Ether() / ARP()
Ether() / IP() / TCP()
Ether() / IP() / UDP()
```

The `/` operator here is Scapy's packet-layer stacking operator, not ordinary division.

## 3. What is an Ethernet frame?

Ethernet is commonly used for communication on a local network. Its frame includes source and destination MAC addresses and a field identifying the payload protocol.

```python
from scapy.all import Ether

ethernet = Ether()
ethernet.show()
```

Important fields:

| Field | Meaning |
|---|---|
| `dst` | Destination MAC address |
| `src` | Source MAC address |
| `type` | Type of payload carried by the frame |

Example:

```python
ethernet = Ether(dst="ff:ff:ff:ff:ff:ff")
ethernet.show()
```

`ff:ff:ff:ff:ff:ff` is the Ethernet broadcast address. A broadcast frame is delivered to devices in the same Layer 2 broadcast domain.

Inspect fields:

```python
print("Destination:", ethernet.dst)
print("Source:", ethernet.src)
print("Type:", ethernet.type)
```

Some fields may remain unset until Scapy builds or sends the packet. Use `.show()` to inspect the layer.

## 4. What is ARP?

ARP means **Address Resolution Protocol**. On a local IPv4 Ethernet network, a computer may know the destination's IPv4 address but not its MAC address. ARP helps discover the MAC address associated with that local IPv4 address.

For example, your computer wants to communicate with `192.168.1.1`, but does not know its MAC address. It broadcasts a request:

> Who has 192.168.1.1? Tell me your MAC address.

The device using that address can reply with its MAC address.

```text
Your computer                         Device on local network
     |                                           |
     |---- ARP request: Who has 192.168.1.1? --->|
     |                                           |
     |<--- ARP reply: IP is at this MAC ---------|
```

ARP is used for IPv4 address resolution on local networks. IPv6 uses Neighbor Discovery Protocol (NDP), not ARP.

## 5. Create an ARP layer with `ARP()`

```python
from scapy.all import ARP

arp = ARP()
arp.show()
```

Useful ARP fields:

| Field | Meaning |
|---|---|
| `hwtype` | Hardware type; Ethernet is commonly `1` |
| `ptype` | Protocol type; IPv4 is commonly `0x0800` |
| `hwlen` | Hardware address length; Ethernet MAC is 6 bytes |
| `plen` | Protocol address length; IPv4 is 4 bytes |
| `op` | Operation: `1` request, `2` reply |
| `hwsrc` | Sender MAC address |
| `psrc` | Sender IPv4 address |
| `hwdst` | Target MAC address |
| `pdst` | Target IPv4 address |

Create an ARP request for one address in your lab:

```python
arp = ARP(
    op=1,
    pdst="192.168.1.1"
)

arp.show()
```

- `op=1` means ARP request.
- `pdst` is the IPv4 address whose MAC address you want to discover.
- `192.168.1.1` is only an example. Use an address in your own lab network.

## 6. Combine Ethernet and ARP

This is the central expression:

```python
from scapy.all import Ether, ARP

arp_request = Ether() / ARP()
```

It creates an Ethernet layer with an ARP layer stacked inside it.

A more explicit Ethernet broadcast request:

```python
arp_request = (
    Ether(dst="ff:ff:ff:ff:ff:ff")
    / ARP(
        op=1,
        pdst="192.168.1.1"
    )
)

arp_request.show()
```

Read it from left to right:

1. `Ether(...)` creates the Ethernet frame.
2. `dst=...` sets its destination MAC to broadcast.
3. `/` stacks another protocol layer.
4. `ARP(...)` creates the ARP message.
5. `pdst` identifies the IPv4 address being queried.

Conceptually:

```text
Ethernet
├── Destination: ff:ff:ff:ff:ff:ff
└── ARP
    ├── Operation: request
    └── Target IPv4: 192.168.1.1
```

## 7. Inspect packets

### `.show()`

Displays the packet's layers and fields:

```python
arp_request.show()
```

### `.summary()`

Displays a short one-line description:

```python
print(arp_request.summary())
```

### `.show2()`

Builds the packet and displays the built fields:

```python
arp_request.show2()
```

### `ls()`

Lists fields available for a Scapy layer:

```python
from scapy.all import ARP, Ether, ls

ls(ARP)
ls(Ether)
```

Use `ls(ARP)` when you want to discover which fields can be configured.

## 8. Send an ARP request and receive a reply with `srp()`

`srp()` sends and receives packets at Layer 2. It is suitable for an Ethernet ARP request.

```python
from scapy.all import Ether, ARP, srp

target_ip = "192.168.1.1"

arp_request = (
    Ether(dst="ff:ff:ff:ff:ff:ff")
    / ARP(op=1, pdst=target_ip)
)

answered, unanswered = srp(
    arp_request,
    timeout=2,
    verbose=False
)

for sent_packet, received_packet in answered:
    print("IP address:", received_packet.psrc)
    print("MAC address:", received_packet.hwsrc)
```

Understand the arguments:

| Argument | Meaning |
|---|---|
| `arp_request` | Packet to send |
| `timeout=2` | Wait up to two seconds for responses |
| `verbose=False` | Suppress Scapy progress messages |

`srp()` returns two collections:

- `answered`: sent packets that received a matching response.
- `unanswered`: sent packets that did not receive a matching response before timeout.

In the loop, each pair is unpacked:

- `sent_packet` is the packet Scapy sent.
- `received_packet` is the response Scapy received.

For an ARP reply:

- `received_packet.psrc` is the responder's IPv4 address.
- `received_packet.hwsrc` is the responder's MAC address.

Always check for an empty result before accessing a response. A device may not reply because it is offline, the IP is incorrect, the request is blocked, or the selected interface/network does not support the operation.

## 9. Discover and select a network interface

A computer can have several interfaces. Common names include `lo` (loopback), `eth0` (often Ethernet on Linux), `wlan0` (often Wi-Fi), `ens33`, and `enp0s3`. Names vary by operating system and machine.

List interfaces:

```python
from scapy.all import get_if_list

for interface in get_if_list():
    print(interface)
```

Get an interface's MAC address:

```python
from scapy.all import get_if_hwaddr

interface = "eth0"
print(get_if_hwaddr(interface))
```

Get an interface's IPv4 address:

```python
from scapy.all import get_if_addr

interface = "eth0"
print(get_if_addr(interface))
```

Specify the interface in `srp()`:

```python
answered, unanswered = srp(
    arp_request,
    iface="eth0",
    timeout=2,
    verbose=False
)
```

Replace `eth0` with an interface that exists on your machine. Linux terminal commands can help:

```bash
ip link show
ip addr show
ip route
```

## 10. A reusable function for one ARP lookup

This function queries one IPv4 address. It is not a subnet scanner.

```python
from scapy.all import Ether, ARP, srp


def resolve_arp(target_ip: str, interface: str):
    """Return the MAC address responding for a local IPv4 address."""

    request = (
        Ether(dst="ff:ff:ff:ff:ff:ff")
        / ARP(op=1, pdst=target_ip)
    )

    answered, unanswered = srp(
        request,
        iface=interface,
        timeout=2,
        verbose=False
    )

    if not answered:
        return None

    # Each answer is a (sent_packet, received_packet) pair.
    return answered[0][1].hwsrc


interface = "eth0"
target_ip = "192.168.1.1"

mac_address = resolve_arp(target_ip, interface)

if mac_address is None:
    print("No ARP response received.")
else:
    print(f"{target_ip} is associated with {mac_address}")
```

Why return `None`? It represents the absence of a result. The caller can test it explicitly instead of inventing a MAC address or returning misleading data.

## 11. `send()`, `sendp()`, `sr()`, and `srp()`

These functions are easy to confuse:

| Function | Main purpose | Layer |
|---|---|---|
| `send()` | Send packets without waiting for replies | Layer 3 |
| `sendp()` | Send packets without waiting for replies | Layer 2 |
| `sr()` | Send packets and receive replies | Layer 3 |
| `srp()` | Send packets and receive replies | Layer 2 |

For an Ethernet ARP request, use `srp()` because ARP is carried directly inside an Ethernet frame.

## 12. Permissions and platform notes

Sending or capturing raw packets often requires elevated privileges.

On Linux, you may need:

```bash
sudo python your_script.py
```

Use the interpreter from the environment where Scapy is installed. Activate your virtual environment first if you use one.

Notes:

- Windows commonly requires Npcap for Scapy packet capture and transmission.
- WSL networking is virtualized; raw Layer 2 access and interface behavior may differ from a normal Linux installation.
- A virtual machine may expose a virtual adapter rather than the physical network adapter.
- `eth0` is only an example; check the actual interface names.
- A timeout does not prove that a device is offline.

## 13. Common mistakes

### Mistake 1: Creating only an Ethernet layer

```python
arp_request = Ether()
```

This creates an Ethernet layer, not a complete ARP request.

Use:

```python
arp_request = Ether() / ARP(pdst="192.168.1.1")
```

### Mistake 2: Using `sr()` for an Ethernet ARP frame

Use `srp()`:

```python
answered, unanswered = srp(packet, timeout=2, verbose=False)
```

### Mistake 3: Forgetting the broadcast destination

A typical ARP request uses:

```python
Ether(dst="ff:ff:ff:ff:ff:ff")
```

### Mistake 4: Assuming the interface is `eth0`

Check first:

```python
from scapy.all import get_if_list
print(get_if_list())
```

### Mistake 5: Assuming a response is guaranteed

```python
if answered:
    print(answered[0][1].hwsrc)
else:
    print("No response received.")
```

### Mistake 6: Missing required permissions

Raw packet operations may fail with a permission error. Use an authorized environment with appropriate privileges.

## 14. Troubleshooting checklist

If the request does not work, check:

1. Is Scapy installed in the Python interpreter running the script?
2. Does the selected interface exist?
3. Is the target IPv4 address on the same local Layer 2 network?
4. Is the interface up?
5. Are you using `srp()` for the Ethernet ARP frame?
6. Does the operating system permit raw packet operations?
7. Are the required privileges available?
8. Does the environment support Layer 2 access?
9. Is the target device active and configured to respond?

Useful Linux commands:

```bash
ip link show
ip addr show
ip route
```

## 15. Best practices

- Import only the Scapy classes and functions you need.
- Use descriptive names such as `arp_request`, `target_ip`, and `interface`.
- Set a reasonable timeout.
- Check for empty responses before reading packet fields.
- Specify the interface when multiple interfaces are available.
- Separate packet construction from packet transmission.
- Handle exceptions where appropriate.
- Practise only on your own lab or authorized network.
- Do not treat an ARP result as proof of device identity; ARP can be spoofed.

## 16. Quick reference

| Code | Meaning |
|---|---|
| `Ether()` | Create an Ethernet layer |
| `ARP()` | Create an ARP layer |
| `Ether() / ARP()` | Stack ARP inside Ethernet |
| `Ether(dst="ff:ff:ff:ff:ff:ff")` | Ethernet broadcast destination |
| `ARP(op=1)` | ARP request |
| `ARP(op=2)` | ARP reply |
| `ARP(pdst="192.168.1.1")` | Set target IPv4 address |
| `packet.show()` | Display packet fields |
| `packet.summary()` | Display a short summary |
| `ls(ARP)` | List ARP fields |
| `get_if_list()` | List network interfaces |
| `get_if_hwaddr("eth0")` | Get interface MAC address |
| `get_if_addr("eth0")` | Get interface IPv4 address |
| `srp(packet)` | Send and receive at Layer 2 |
| `sendp(packet)` | Send at Layer 2 |

## 17. Practice exercises

1. **Inspect Ethernet:** Create an `Ether()` object and display it with `.show()`.
2. **Inspect ARP:** Create an `ARP()` object and use `ls(ARP)` to discover its fields.
3. **Build a packet:** Combine `Ether(dst="ff:ff:ff:ff:ff:ff")` and `ARP(pdst="YOUR_LAB_IP")`. Display it with `.show()`.
4. **List interfaces:** Use `get_if_list()` and identify the interface connected to your lab network.
5. **Perform one authorized lookup:** Use `srp()` to query one device on your own local network. Print its responding IPv4 and MAC addresses.
6. **Handle no response:** Modify the program to display a helpful message when no ARP response is received.

## 18. Recommended learning order

1. Ethernet frames and MAC addresses.
2. IPv4 addresses and local networks.
3. ARP request versus ARP reply.
4. Scapy layers: `Ether()` and `ARP()`.
5. Layer stacking with `/`.
6. Packet inspection with `.show()`, `.summary()`, and `ls()`.
7. Interface discovery with `get_if_list()`.
8. Sending and receiving with `srp()`.
9. Response validation and exception handling.
10. Packet capture with `sniff()`.

**Central idea:** `Ether()` builds the Ethernet layer, `ARP()` builds the ARP message, `/` combines protocol layers, and `srp()` sends a Layer 2 packet and collects responses.
