# SCAPY — COMPLETE ICMP GUIDE
## Understanding, Creating, Sending, Receiving and Analysing ICMP Packets

# TABLE OF CONTENTS

1. Introduction to ICMP
2. Understanding the ICMP Header
3. Understanding ICMP Type and Code
4. Important ICMP Types and Codes
5. Understanding ICMP with Scapy
6. Creating ICMP Packets
7. Sending ICMP Packets
8. Receiving ICMP Responses
9. Understanding `sr()`, `sr1()`, `srp()` and `srp1()`
10. Reading ICMP Responses
11. Checking ICMP Type and Code
12. Working with ICMP Echo Request and Echo Reply
13. Understanding ICMP Destination Unreachable
14. Understanding ICMP Time Exceeded
15. Using ICMP in Traceroute
16. Handling Missing Responses
17. Complete Scapy ICMP Examples
18. Important Scapy Best Practices
19. Quick Reference Cheat Sheet

---

# 1. INTRODUCTION TO ICMP

ICMP stands for:

**Internet Control Message Protocol.**

It is used by network devices to communicate diagnostic information and report network errors.

For example, ICMP can tell you:

- A destination host cannot be reached.
- A destination network cannot be reached.
- A packet's TTL has expired.
- A destination port is unreachable.
- A device has received an Echo Request.

ICMP is commonly used by:

- `ping`
- `traceroute`
- `tracert`
- Network diagnostic tools
- Network monitoring applications

ICMP operates alongside IPv4 at the network layer.

Unlike TCP and UDP, ICMP does not use port numbers.

## Example

When you execute:

```bash
ping 8.8.8.8
```

Your computer sends an ICMP Echo Request to `8.8.8.8`.

If the destination allows and receives the request, it normally responds with an ICMP Echo Reply.

---

# 2. UNDERSTANDING THE ICMP HEADER

An ICMP message contains several fields.

The important fields include:

| Field | Meaning |
|---|---|
| Type | Identifies the ICMP message category. |
| Code | Provides more specific information about the message. |
| Checksum | Detects errors in the ICMP message. |
| Identifier | Identifies an Echo Request/Reply exchange. |
| Sequence | Identifies the sequence number of an Echo Request/Reply. |

The Identifier and Sequence fields are used in Echo messages.

The ICMP header structure depends on the message type. Not every ICMP message uses all these fields in the same way.

## Type and Code

These are the two most important fields when analysing ICMP responses.

For example:

```python
icmp_type = 11
icmp_code = 0
```

This means:

- Type 11 = Time Exceeded.
- Code 0 = TTL exceeded in transit.

Another example:

```python
icmp_type = 3
icmp_code = 3
```

This means:

- Type 3 = Destination Unreachable.
- Code 3 = Port Unreachable.

**Remember:**

The meaning of a Code depends on its Type.

Code 0 does not have one universal meaning.

---

# 3. IMPORTANT ICMP TYPES AND CODES

The following are commonly encountered IPv4 ICMP messages.

## TYPE 0 — ECHO REPLY

| Type | Code | Meaning |
|---|---|---|
| 0 | 0 | Echo Reply |

An Echo Reply is sent in response to an Echo Request.

Example:

```python
if icmp_type == 0 and icmp_code == 0:
    print("Echo Reply received")
```

This normally indicates that the destination responded to your ping request.

## TYPE 3 — DESTINATION UNREACHABLE

This indicates that a packet could not be delivered to its destination.

| Type | Code | Meaning |
|---|---|---|
| 3 | 0 | Network unreachable |
| 3 | 1 | Host unreachable |
| 3 | 2 | Protocol unreachable |
| 3 | 3 | Port unreachable |
| 3 | 4 | Fragmentation needed, but DF flag is set |
| 3 | 5 | Source route failed |
| 3 | 6 | Destination network unknown |
| 3 | 7 | Destination host unknown |
| 3 | 8 | Source host isolated (obsolete) |
| 3 | 9 | Communication with destination network administratively prohibited |
| 3 | 10 | Communication with destination host administratively prohibited |
| 3 | 11 | Network unreachable for specified type of service |
| 3 | 12 | Host unreachable for specified type of service |
| 3 | 13 | Communication administratively prohibited |
| 3 | 14 | Host precedence violation (obsolete) |
| 3 | 15 | Precedence cutoff in effect (obsolete) |

### Understanding the important codes

**Code 0 — Network Unreachable**

The network cannot be reached.

**Code 1 — Host Unreachable**

The destination host cannot be reached.

**Code 2 — Protocol Unreachable**

The destination does not support the requested protocol.

**Code 3 — Port Unreachable**

The destination port cannot be reached.

**Code 4 — Fragmentation Needed**

The packet needs fragmentation, but the Don't Fragment (DF) flag prevents it.

**Code 13 — Administratively Prohibited**

Communication is blocked by an administrative rule, such as a firewall policy.

Scapy example:

```python
if icmp_type == 3:

    if icmp_code == 0:
        print("Network unreachable")

    elif icmp_code == 1:
        print("Host unreachable")

    elif icmp_code == 2:
        print("Protocol unreachable")

    elif icmp_code == 3:
        print("Port unreachable")

    elif icmp_code == 4:
        print("Fragmentation needed")

    elif icmp_code == 13:
        print("Communication administratively prohibited")
```

## TYPE 4 — SOURCE QUENCH

| Type | Code | Meaning |
|---|---|---|
| 4 | 0 | Source Quench |

This message was historically used to request that a sender reduce its transmission rate.

It is deprecated and should not be used for modern congestion control.

## TYPE 5 — REDIRECT

A router uses this message to inform a host that a better route may be available.

| Type | Code | Meaning |
|---|---|---|
| 5 | 0 | Redirect for network |
| 5 | 1 | Redirect for host |
| 5 | 2 | Redirect for network based on type of service |
| 5 | 3 | Redirect for host based on type of service |

## TYPE 8 — ECHO REQUEST

| Type | Code | Meaning |
|---|---|---|
| 8 | 0 | Echo Request |

This is the message sent when you use `ping`.

Example:

```python
if icmp_type == 8 and icmp_code == 0:
    print("Echo Request")
```

### How Ping Works

```text
SOURCE COMPUTER                 DESTINATION
      |                              |
      | ------ Type 8, Code 0 ------> |
      |                              |
      | <----- Type 0, Code 0 ------ |
      |                              |
```

1. Your computer sends Type 8, Code 0.
2. The destination receives the request.
3. The destination responds with Type 0, Code 0.
4. Your computer receives the reply.

## TYPE 9 — ROUTER ADVERTISEMENT

| Type | Code | Meaning |
|---|---|---|
| 9 | 0 | Router Advertisement |

Routers use this message to advertise their presence.

## TYPE 10 — ROUTER SOLICITATION

| Type | Code | Meaning |
|---|---|---|
| 10 | 0 | Router Solicitation |

A host can send this message to request information from routers.

## TYPE 11 — TIME EXCEEDED

This is one of the most important ICMP types when building a traceroute tool.

| Type | Code | Meaning |
|---|---|---|
| 11 | 0 | TTL exceeded in transit |
| 11 | 1 | Fragment reassembly time exceeded |

### Type 11, Code 0 — TTL Exceeded in Transit

When a router forwards an IPv4 packet, it decreases its TTL by at least one.

If the TTL reaches zero, the router discards the packet and normally sends an ICMP Type 11, Code 0 response.

Traceroute uses this behaviour to discover intermediate routers.

Example:

```text
YOUR COMPUTER
     |
     | TTL = 1
     |
     v
ROUTER 1
     |
     | TTL becomes 0
     |
     X Packet discarded
     |
     <---- ICMP Type 11, Code 0
```

The source IP of the ICMP response identifies the router that sent the response.

## TYPE 12 — PARAMETER PROBLEM

| Type | Code | Meaning |
|---|---|---|
| 12 | 0 | Pointer indicates the error |
| 12 | 1 | Missing required option |
| 12 | 2 | Bad length |

This indicates a problem with a field in the IPv4 header.

## TYPE 13 — TIMESTAMP REQUEST

| Type | Code | Meaning |
|---|---|---|
| 13 | 0 | Timestamp Request |

## TYPE 14 — TIMESTAMP REPLY

| Type | Code | Meaning |
|---|---|---|
| 14 | 0 | Timestamp Reply |

## TYPE 15 — INFORMATION REQUEST (OBSOLETE)

| Type | Code | Meaning |
|---|---|---|
| 15 | 0 | Information Request |

## TYPE 16 — INFORMATION REPLY (OBSOLETE)

| Type | Code | Meaning |
|---|---|---|
| 16 | 0 | Information Reply |

## TYPE 17 — ADDRESS MASK REQUEST (OBSOLETE)

| Type | Code | Meaning |
|---|---|---|
| 17 | 0 | Address Mask Request |

## TYPE 18 — ADDRESS MASK REPLY (OBSOLETE)

| Type | Code | Meaning |
|---|---|---|
| 18 | 0 | Address Mask Reply |

---

# 4. INTRODUCTION TO SCAPY ICMP

Scapy is a Python library for creating, sending, receiving and analysing network packets.

You can use it to work with:

- Ethernet
- ARP
- IPv4
- IPv6
- ICMP
- TCP
- UDP
- DNS

Install Scapy:

```bash
py -m pip install scapy
```

Import Scapy:

```python
import scapy.all as scapy
```

You can also import individual components:

```python
from scapy.all import IP, ICMP, sr1
```

For the examples in this document, we will mainly use:

```python
import scapy.all as scapy
```

---

# 5. CREATING ICMP PACKETS

Scapy allows you to construct packets by combining protocol layers.

## Creating an IP Packet

```python
IP_PKT = scapy.IP(dst="8.8.8.8")
```

This creates an IPv4 packet.

The `dst` argument specifies the destination IP address.

## Creating an ICMP Packet

```python
ICMP_PKT = scapy.ICMP()
```

This creates an ICMP packet.

By default, Scapy creates an Echo Request.

Its default values are:

```python
type = 8
code = 0
```

## Combining IP and ICMP

```python
ICMP_PKT = scapy.IP(dst="8.8.8.8") / scapy.ICMP()
```

The `/` operator combines protocol layers.

The resulting packet is:

```text
IP Layer
   |
   v
ICMP Layer
```

You can also write:

```python
IP_PKT = scapy.IP(dst="8.8.8.8")

ICMP_LAYER = scapy.ICMP()

ICMP_PKT = IP_PKT / ICMP_LAYER
```

Both approaches create the same layered packet.

## Displaying the Packet

```python
ICMP_PKT.show()
```

To display a summary:

```python
ICMP_PKT.summary()
```

To display a detailed packet representation:

```python
ICMP_PKT.show2()
```

`show2()` builds the packet before displaying it, so automatically calculated fields may be populated.

---

# 6. CHANGING ICMP TYPE AND CODE

You can specify the ICMP Type and Code manually.

Example:

```python
ICMP_PKT = scapy.IP(dst="8.8.8.8") / scapy.ICMP(
    type=8,
    code=0
)
```

This creates an Echo Request.

Another example:

```python
ICMP_PKT = scapy.IP(dst="8.8.8.8") / scapy.ICMP(
    type=0,
    code=0
)
```

This constructs an Echo Reply message.

Important: Constructing a particular ICMP message does not mean the destination will accept it or respond to it.

Some ICMP messages are normally generated by routers or destination hosts under specific network conditions.

## Accessing Type and Code

You can read the values after constructing the packet:

```python
print(ICMP_PKT[scapy.ICMP].type)

print(ICMP_PKT[scapy.ICMP].code)
```

You can also change them:

```python
ICMP_PKT[scapy.ICMP].type = 8

ICMP_PKT[scapy.ICMP].code = 0
```

---

# 7. SENDING ICMP PACKETS

Scapy provides different functions for sending and receiving packets.

The important functions are:

| Function | Purpose |
|---|---|
| `send()` | Sends packets at Layer 3. |
| `sendp()` | Sends packets at Layer 2. |
| `sr()` | Sends at Layer 3 and receives matching responses. |
| `sr1()` | Sends at Layer 3 and returns one response. |
| `srp()` | Sends at Layer 2 and receives matching responses. |
| `srp1()` | Sends at Layer 2 and returns one response. |

## 7.1 send()

Sends a packet without waiting for a response.

```python
scapy.send(ICMP_PKT)
```

## 7.2 sendp()

Sends a packet at Layer 2.

It normally requires an Ethernet frame or another appropriate Layer 2 packet.

```python
ETH_PKT = scapy.Ether() / scapy.IP(dst="8.8.8.8") / scapy.ICMP()

scapy.sendp(ETH_PKT)
```

## 7.3 sr()

Sends packets and collects answered and unanswered packets.

```python
ans, unans = scapy.sr(ICMP_PKT, timeout=3, verbose=False)
```

It returns two collections:

- `ans`: answered packets.
- `unans`: packets that did not receive a matching response.

## 7.4 sr1()

Sends a packet and returns one response packet.

```python
ans = scapy.sr1(ICMP_PKT, timeout=3, verbose=False)
```

If no response arrives before the timeout, it returns:

```python
None
```

**Important:** `sr1()` returns one response packet, not the `(sent, received)` pair used in each answered entry from `sr()`.

## 7.5 srp()

Sends and receives packets at Layer 2.

```python
ans, unans = scapy.srp(
    ETH_PKT,
    timeout=3,
    verbose=False
)
```

## 7.6 srp1()

Sends a Layer 2 packet and returns one response.

```python
ans = scapy.srp1(
    ETH_PKT,
    timeout=3,
    verbose=False
)
```

---

# 8. UNDERSTANDING ANSWERED AND UNANSWERED PACKETS

Consider:

```python
ans, unans = scapy.sr(
    ICMP_PKT,
    timeout=3,
    verbose=False
)
```

### ans

Contains answered packet pairs.

Each answered entry contains:

1. The original packet sent.
2. The response packet received.

Example:

```python
for sent_packet, received_packet in ans:

    print("Sent packet:")
    sent_packet.show()

    print("Received packet:")
    received_packet.show()
```

You can access the response ICMP fields:

```python
for sent_packet, received_packet in ans:

    print(received_packet[scapy.ICMP].type)

    print(received_packet[scapy.ICMP].code)
```

### unans

Contains packets that did not receive a matching response.

```python
for packet in unans:

    print("No matching response received")

    packet.show()
```

### Important Difference

With `sr()`:

```python
for sent_packet, received_packet in ans:
    print(received_packet[scapy.ICMP].type)
```

With `sr1()`:

```python
ans = scapy.sr1(ICMP_PKT, timeout=3, verbose=False)

if ans is not None:
    print(ans[scapy.ICMP].type)
```

Do not confuse these two return formats.

---

# 9. READING ICMP RESPONSES

After sending a packet with `sr1()`, you can inspect the response.

```python
ans = scapy.sr1(
    ICMP_PKT,
    timeout=3,
    verbose=False
)
```

First, check whether a response was received.

```python
if ans is not None:

    print("Response received")

else:

    print("No response received")
```

A response packet can contain different protocol layers.

For example:

```text
IP Layer
   |
   v
ICMP Layer
```

Access the IP layer:

```python
ans[scapy.IP]
```

Access the ICMP layer:

```python
ans[scapy.ICMP]
```

Read the source IP address:

```python
ans[scapy.IP].src
```

Read the ICMP Type:

```python
ans[scapy.ICMP].type
```

Read the ICMP Code:

```python
ans[scapy.ICMP].code
```

## Check Whether a Layer Exists

Use:

```python
ans.haslayer(scapy.ICMP)
```

Example:

```python
if ans is not None and ans.haslayer(scapy.ICMP):

    print(ans[scapy.ICMP].type)

    print(ans[scapy.ICMP].code)
```

This prevents attempts to access an ICMP layer that is not present.

---

# 10. UNDERSTANDING NONE AND is not None

When `sr1()` does not receive a response before its timeout, it returns `None`.

Example:

```python
ans = scapy.sr1(
    ICMP_PKT,
    timeout=3,
    verbose=False
)
```

Check the result:

```python
if ans is None:

    print("No response received")

else:

    print("Response received")
```

You may also write:

```python
if ans is not None:
    print("Response received")
```

Another valid but less readable approach is:

```python
if isinstance(ans, type(None)) == False:
    print("Response received")
```

The recommended Python style is:

```python
if ans is not None:
```

Remember that `None` is different from:

- `0`
- `False`
- `""`
- `[]`

These are different Python values.

---

# 11. WORKING WITH ECHO REQUEST AND ECHO REPLY

Let's build a basic ping program.

```python
import scapy.all as scapy

target_ip = "8.8.8.8"

ICMP_PKT = scapy.IP(dst=target_ip) / scapy.ICMP()

ans = scapy.sr1(
    ICMP_PKT,
    timeout=3,
    verbose=False
)

if ans is not None and ans.haslayer(scapy.ICMP):

    icmp_type = ans[scapy.ICMP].type

    icmp_code = ans[scapy.ICMP].code

    if icmp_type == 0 and icmp_code == 0:

        print("Destination responded")

        print("Source IP:", ans[scapy.IP].src)

    else:

        print("Different ICMP response")

        print("Type:", icmp_type)

        print("Code:", icmp_code)

else:

    print("No ICMP response received")
```

This program:

1. Creates an IP packet.
2. Adds an ICMP Echo Request.
3. Sends the packet.
4. Waits for a response.
5. Checks whether a response exists.
6. Reads its ICMP Type and Code.
7. Determines whether an Echo Reply was received.

A timeout does not prove that the destination is offline. It may simply be filtering ICMP.

---

# 12. UNDERSTANDING DESTINATION UNREACHABLE

ICMP Type 3 indicates that a packet could not be delivered.

You can inspect the different codes.

```python
if ans is not None and ans.haslayer(scapy.ICMP):

    icmp_type = ans[scapy.ICMP].type

    icmp_code = ans[scapy.ICMP].code

    if icmp_type == 3:

        if icmp_code == 0:
            print("Network unreachable")

        elif icmp_code == 1:
            print("Host unreachable")

        elif icmp_code == 2:
            print("Protocol unreachable")

        elif icmp_code == 3:
            print("Port unreachable")

        elif icmp_code == 4:
            print("Fragmentation needed")

        elif icmp_code == 13:
            print("Communication administratively prohibited")

        else:
            print("Other Destination Unreachable code:", icmp_code)
```

Notice that we first check:

```python
if icmp_type == 3:
```

Then we inspect the code.

This is important because Code 3 has different meanings under different ICMP Types.

---

# 13. UNDERSTANDING TIME EXCEEDED

ICMP Type 11 is particularly important for traceroute.

The two important values are:

| Type | Code | Meaning |
|---|---|---|
| 11 | 0 | TTL exceeded in transit |
| 11 | 1 | Fragment reassembly time exceeded |

When a router decreases the TTL to zero, it discards the packet and normally sends Type 11, Code 0.

Example:

```python
if ans is not None and ans.haslayer(scapy.ICMP):

    icmp_type = ans[scapy.ICMP].type

    icmp_code = ans[scapy.ICMP].code

    if icmp_type == 11 and icmp_code == 0:

        print("TTL exceeded in transit")

        print("Router IP:", ans[scapy.IP].src)
```

The router's IP address is obtained from:

```python
ans[scapy.IP].src
```

It is the source address of the response packet.

---

# 14. USING ICMP IN TRACEROUTE

Traceroute discovers the routers between your computer and a destination.

It works by sending packets with progressively increasing TTL values.

For example:

```text
TTL = 1
TTL = 2
TTL = 3
TTL = 4
TTL = 5
```

Each router decreases the TTL.

When TTL becomes zero, the router normally sends an ICMP Type 11, Code 0 response.

When the destination responds, the program stops tracing.

## Your Traceroute Logic

Your original approach is:

1. Start TTL at 1.
2. Create an IP packet with the destination and TTL.
3. Add an ICMP packet.
4. Send the packet using `sr1()`.
5. Check whether a response exists.
6. If Type 11 and Code 0, print the router IP.
7. If Type 0, print the destination.
8. If no response exists, print an unknown router.
9. Increase TTL.
10. Repeat until the destination is reached or the maximum hop limit is exceeded.

## Corrected Version of Your Original Logic

```python
import time
import scapy.all as scapy

TTL = 1

print("IP Tracerouting has started...")

T1 = time.time()

while TTL <= max_hop_limit:

    ICMP_PKT = scapy.IP(
        dst=target_ip_address,
        ttl=TTL
    ) / scapy.ICMP()

    ans = scapy.sr1(
        ICMP_PKT,
        timeout=3,
        retry=1,
        verbose=False
    )

    # Valid response is received
    if isinstance(ans, type(None)) == False:

        if ans.haslayer(scapy.ICMP):

            icmp_type = ans[scapy.ICMP].type

            icmp_code = ans[scapy.ICMP].code

            # ICMP TTL expired
            if icmp_type == 11 and icmp_code == 0:

                print(
                    "router",
                    ans[scapy.IP].src,
                    "| TTL:",
                    TTL
                )

            # ICMP Echo Reply
            elif icmp_type == 0 and icmp_code == 0:

                print(
                    "router",
                    target_ip_address,
                    "| TTL:",
                    TTL
                )

                break

            # Destination Unreachable
            elif icmp_type == 3:

                print(
                    "Destination unreachable",
                    "| Router:",
                    ans[scapy.IP].src,
                    "| Code:",
                    icmp_code,
                    "| TTL:",
                    TTL
                )

                break

            else:

                print(
                    "Other ICMP response",
                    "| Type:",
                    icmp_type,
                    "| Code:",
                    icmp_code,
                    "| TTL:",
                    TTL
                )

    else:

        # No response received
        print("unknown router", "| TTL:", TTL)

    TTL += 1

T2 = time.time()

print(
    "IP Tracerouting done in",
    round(T2 - T1, 2),
    "seconds"
)
```

Before running the code, define:

```python
target_ip_address = "8.8.8.8"

max_hop_limit = 20
```

### Important Explanation

**Why does TTL increase outside the conditions?**

Because every iteration must move to the next TTL value, whether:

- A router responds.
- No router responds.
- Another ICMP response is received.

If you only increase TTL inside one condition, your loop can become stuck.

**Why do we use `break`?**

When the destination responds with Echo Reply, there is no reason to continue tracing.

**Why do we use `ans[scapy.IP].src`?**

Because the source IP address of the received ICMP response identifies the device that sent the response.

Note: This is a simplified IPv4 traceroute. Some networks filter ICMP, and destination responses can vary. A timeout does not necessarily mean that a router is absent.

---

# 15. HANDLING MISSING RESPONSES

Sometimes `sr1()` returns `None`.

For example:

```python
ans = scapy.sr1(
    ICMP_PKT,
    timeout=3,
    verbose=False
)
```

This can happen because:

- The destination is offline.
- A router is not responding to ICMP.
- A firewall blocks ICMP.
- A packet is lost.
- The timeout is too short.
- The network does not allow the packet to pass.

You can handle it using:

```python
if ans is None:

    print("No response received")

else:

    print("Response received")
```

In traceroute, a missing response is commonly represented by `*` or an unknown hop.

However, remember that a router may forward traffic normally while refusing to send ICMP diagnostic responses.

---

# 16. COMPLETE ICMP RESPONSE ANALYSER

The following example demonstrates how to classify common ICMP responses.

```python
import scapy.all as scapy

def analyse_icmp_response(packet):

    if packet is None:

        print("No response received")

        return

    if not packet.haslayer(scapy.ICMP):

        print("Response does not contain an ICMP layer")

        return

    icmp_type = packet[scapy.ICMP].type

    icmp_code = packet[scapy.ICMP].code

    if packet.haslayer(scapy.IP):

        source_ip = packet[scapy.IP].src

    else:

        source_ip = "Unknown"

    print("Source IP:", source_ip)

    print("ICMP Type:", icmp_type)

    print("ICMP Code:", icmp_code)

    if icmp_type == 0 and icmp_code == 0:

        print("Message: Echo Reply")

    elif icmp_type == 3:

        print("Message: Destination Unreachable")

        if icmp_code == 0:
            print("Reason: Network unreachable")

        elif icmp_code == 1:
            print("Reason: Host unreachable")

        elif icmp_code == 2:
            print("Reason: Protocol unreachable")

        elif icmp_code == 3:
            print("Reason: Port unreachable")

        elif icmp_code == 4:
            print("Reason: Fragmentation needed")

        elif icmp_code == 13:
            print("Reason: Administratively prohibited")

        else:
            print("Other unreachable code:", icmp_code)

    elif icmp_type == 8 and icmp_code == 0:

        print("Message: Echo Request")

    elif icmp_type == 11 and icmp_code == 0:

        print("Message: TTL exceeded in transit")

    elif icmp_type == 11 and icmp_code == 1:

        print("Message: Fragment reassembly time exceeded")

    else:

        print("Other ICMP message")


target_ip = "8.8.8.8"

packet = scapy.IP(dst=target_ip) / scapy.ICMP()

answer = scapy.sr1(
    packet,
    timeout=3,
    verbose=False
)

analyse_icmp_response(answer)
```

---

# 17. IMPORTANT SCAPY BEST PRACTICES

## 1. Always Check for None

Before accessing the response packet:

```python
if ans is not None:
```

This prevents errors when no response arrives.

## 2. Check Whether the Layer Exists

```python
if ans.haslayer(scapy.ICMP):
```

This ensures the packet contains an ICMP layer.

## 3. Use the Correct Layer

For the source IP:

```python
ans[scapy.IP].src
```

For ICMP Type:

```python
ans[scapy.ICMP].type
```

For ICMP Code:

```python
ans[scapy.ICMP].code
```

## 4. Understand the Return Value

Remember:

```python
ans = scapy.sr1(...)
```

Returns one response packet or `None`.

But:

```python
ans, unans = scapy.sr(...)
```

Returns answered and unanswered collections.

## 5. Use Meaningful Variable Names

Instead of:

```python
T1 = time.time()

T2 = time.time()
```

You can use:

```python
start_time = time.time()

end_time = time.time()
```

Your original names are valid, but descriptive names make larger programs easier to maintain.

## 6. Use Timeouts

Always consider a timeout when waiting for network responses:

```python
ans = scapy.sr1(
    ICMP_PKT,
    timeout=3,
    verbose=False
)
```

Without an appropriate timeout, your program may wait longer than expected.

## 7. Use Appropriate Permissions on Windows

On Windows, Scapy packet operations may require Npcap and elevated permissions.

If you encounter an error involving Windows native Layer 3 raw sockets:

- Confirm Npcap is installed and running.
- Check that your terminal has the required permissions.
- Consult Scapy's Windows-specific packet-capture configuration.
- Do not assume that installing Npcap automatically fixes every raw-socket configuration.

Only perform packet testing on systems and networks you own or are authorized to test.

---

# 18. QUICK SCAPY ICMP CHEAT SHEET

## Creating Packets

```python
scapy.IP(dst="8.8.8.8")

scapy.ICMP()

scapy.IP(dst="8.8.8.8") / scapy.ICMP()
```

## Sending and Receiving

```python
scapy.send(packet)

scapy.sendp(packet)

ans, unans = scapy.sr(packet)

ans = scapy.sr1(packet)

ans, unans = scapy.srp(packet)

ans = scapy.srp1(packet)
```

## Displaying Packets

```python
packet.show()

packet.show2()

packet.summary()
```

## Checking Responses

```python
if ans is None:
    print("No response")

if ans is not None:
    print("Response received")
```

## Checking Layers

```python
ans.haslayer(scapy.IP)

ans.haslayer(scapy.ICMP)
```

## Reading IP Information

```python
ans[scapy.IP].src

ans[scapy.IP].dst

ans[scapy.IP].ttl
```

## Reading ICMP Information

```python
ans[scapy.ICMP].type

ans[scapy.ICMP].code

ans[scapy.ICMP].id

ans[scapy.ICMP].seq
```

## Important ICMP Values

| Type | Code | Meaning |
|---:|---:|---|
| 0 | 0 | Echo Reply |
| 3 | 0 | Network unreachable |
| 3 | 1 | Host unreachable |
| 3 | 2 | Protocol unreachable |
| 3 | 3 | Port unreachable |
| 3 | 4 | Fragmentation needed |
| 3 | 13 | Administratively prohibited |
| 5 | 0–3 | Redirect |
| 8 | 0 | Echo Request |
| 11 | 0 | TTL exceeded in transit |
| 11 | 1 | Fragment reassembly time exceeded |
| 12 | 0–2 | Parameter Problem |

---

# 19. FINAL SUMMARY

The most important concepts to understand when working with ICMP in Scapy are:

1. `IP()` creates an IPv4 layer.
2. `ICMP()` creates an ICMP layer.
3. `/` combines packet layers.
4. `send()` sends at Layer 3.
5. `sendp()` sends at Layer 2.
6. `sr()` sends and receives multiple packets and returns answered and unanswered collections.
7. `sr1()` sends a packet and returns one response or `None`.
8. `ans[scapy.ICMP].type` reads the ICMP Type.
9. `ans[scapy.ICMP].code` reads the ICMP Code.
10. `ans[scapy.IP].src` reads the response source IP address.
11. Type 8, Code 0 represents Echo Request.
12. Type 0, Code 0 represents Echo Reply.
13. Type 3 represents Destination Unreachable.
14. Type 11, Code 0 represents TTL exceeded in transit.

**For your traceroute project, concentrate on these three ICMP Types:**

- **Type 11:** Identify intermediate routers when TTL expires.
- **Type 0:** Identify when the destination responds.
- **Type 3:** Identify destination-unreachable errors.

Once you understand these three, you will have a strong foundation for building your own IPv4 traceroute tool with