# NETWORK SCANNING TECHNIQUES
## A Beginner-to-Advanced Guide to Understanding Network Scanning

**Author:** Nzegge Charles Ngong  
**Field:** Networking and Ethical Hacking  
**Language:** Python, Scapy, TCP/IP

---

# 1. What Is Network Scanning?

Network scanning is the process of sending network probes to a computer, server, or network to discover information about its availability, ports, services, and network configuration.

As an ethical hacker or network administrator, you might want to answer questions such as:

1. Is a machine available on the network?
2. What IP addresses are active?
3. Which ports are open?
4. Which ports are closed?
5. Is a firewall filtering network traffic?
6. Which services are running?
7. Which application or service version is installed?
8. What operating system might the machine be using?
9. Are there security weaknesses that require investigation?

Different scanning techniques answer different questions.

**Important:** Scanning a port does not automatically identify the application running behind it. Port scanning and service identification are separate activities.

Only scan machines and networks you own or have explicit permission to assess.

---

# 2. Understanding Ports Before Learning Scanning Types

A port is a logical communication endpoint used by network applications.

For example:

| Port | Common service | Purpose |
|---|---|---|
| 20 | FTP Data | FTP data transfer |
| 21 | FTP | File transfer control |
| 22 | SSH | Secure remote administration |
| 23 | Telnet | Remote terminal |
| 25 | SMTP | Email transmission |
| 53 | DNS | Domain name resolution |
| 80 | HTTP | Web communication |
| 110 | POP3 | Email retrieval |
| 143 | IMAP | Email retrieval |
| 443 | HTTPS | Secure web communication |
| 3306 | MySQL | Database communication |
| 5432 | PostgreSQL | Database communication |
| 8080 | Alternative HTTP | Web applications |

A server does not automatically have every port open.

A port is generally considered open when a service is listening and accepting connections through the relevant network interface.

However, firewalls and network policies can affect whether you can reach that service.

---

# 3. Understanding TCP Flags

Before learning TCP scanning techniques, understand the TCP flags.

TCP flags are control bits inside the TCP header.

They help TCP devices manage connections and communication.

| Flag | Meaning | Purpose |
|---|---|---|
| SYN | Synchronize | Initiates a TCP connection |
| ACK | Acknowledgment | Acknowledges received TCP data |
| FIN | Finish | Requests termination of a TCP connection |
| RST | Reset | Resets or rejects a TCP connection |
| PSH | Push | Requests delivery of buffered data |
| URG | Urgent | Indicates urgent data |
| ECE | ECN Echo | Congestion notification |
| CWR | Congestion Window Reduced | Indicates congestion response |
| NS | Nonce Sum | Additional ECN-related flag |

## TCP flag hexadecimal values

| Flag | Hexadecimal | Binary |
|---|---|---|
| FIN | 0x01 | 00000001 |
| SYN | 0x02 | 00000010 |
| RST | 0x04 | 00000100 |
| PSH | 0x08 | 00001000 |
| ACK | 0x10 | 00010000 |
| URG | 0x20 | 00100000 |
| ECE | 0x40 | 01000000 |
| CWR | 0x80 | 10000000 |

### Combining TCP flags

TCP flags can be combined.

For example:

| Hexadecimal | Combined flags |
|---|---|
| 0x02 | SYN |
| 0x10 | ACK |
| 0x12 | SYN + ACK |
| 0x14 | RST + ACK |
| 0x18 | PSH + ACK |
| 0x11 | FIN + ACK |

In Scapy:

```python
import scapy.all as scapy

packet = scapy.IP(dst="192.168.1.100") / scapy.TCP(
    dport=80,
    flags="S"
)
```

Here, `flags="S"` means that you are constructing a TCP SYN packet.

---

# 4. Different Types of Network Scanning

We will divide network scanning into several categories.

1. Host discovery scanning.
2. Port scanning.
3. Service and version detection.
4. Operating system detection.
5. Firewall scanning.
6. Vulnerability scanning.

Each category has a different purpose.

---

# 5. Host Discovery Scanning

## What is host discovery?

Host discovery attempts to determine which devices are available on a network.

For example, imagine your local network:

```text
192.168.1.1      Router
192.168.1.10     Laptop
192.168.1.20     Phone
192.168.1.30     Printer
192.168.1.110    Your computer
```

You might want to discover which of these addresses correspond to active devices.

Host discovery helps answer:

- Is the computer online?
- Is the server responding?
- Which devices are present on my local network?

## A. ICMP Ping Scan

ICMP is a network-layer protocol used for diagnostic and error-reporting purposes.

A traditional ping sends an ICMP Echo Request.

The destination may return an ICMP Echo Reply.

### Example using Scapy

```python
import scapy.all as scapy

target = "192.168.1.100"

packet = (
    scapy.IP(dst=target) /
    scapy.ICMP(type=8, code=0)
)

response = scapy.sr1(
    packet,
    timeout=2,
    verbose=False
)

if response and response.haslayer(scapy.ICMP):

    if response[scapy.ICMP].type == 0:
        print("Host responded to ICMP ping")

else:
    print("No ICMP response")
```

### Understanding the ICMP types

| ICMP type | Meaning |
|---|---|
| 0 | Echo Reply |
| 3 | Destination Unreachable |
| 8 | Echo Request |
| 11 | Time Exceeded |

Important: No ping response does not prove that a machine is offline. Firewalls may block ICMP.

## B. ARP Scan

ARP stands for Address Resolution Protocol.

It is used to discover the MAC address associated with an IPv4 address on the local network.

ARP scanning is particularly useful for discovering devices on the same local network.

### Example

```python
import scapy.all as scapy

target = "192.168.1.100"

packet = (
    scapy.Ether(dst="ff:ff:ff:ff:ff:ff") /
    scapy.ARP(pdst=target)
)

answered, unanswered = scapy.srp(
    packet,
    timeout=2,
    verbose=False
)

for sent, received in answered:

    print("IP:", received.psrc)
    print("MAC:", received.hwsrc)
```

### Understanding the layers

```text
Ether()
   |
ARP()
```

This is a Layer 2 operation.

You use `srp()` because you are sending and receiving Ethernet frames.

ARP does not normally discover devices across routers. It operates on the local broadcast domain.

---

# 6. TCP SYN Scan

**Also called a Half-Open Scan.**

This is the scanning technique you have been implementing in your Scapy project.

## Purpose

A SYN scan attempts to determine whether a TCP port is open without completing the normal TCP three-way handshake.

It sends a TCP SYN packet.

### How it works

Suppose you want to test port 80.

```text
Your computer                    Target server

     |                                  |
     | -------- SYN ------------------> |
     |                                  |
     | <------ SYN-ACK ---------------- |
     |                                  |
     | -------- RST ------------------> |
```

The scanner sends SYN.

If the port is open, the server typically replies with SYN-ACK.

The scanner then sends RST to terminate the incomplete connection attempt.

### Scapy example

```python
import scapy.all as scapy

target = "192.168.1.100"
port = 80

packet = (
    scapy.IP(dst=target) /
    scapy.TCP(
        dport=port,
        flags="S"
    )
)

response = scapy.sr1(
    packet,
    timeout=2,
    verbose=False
)

if response and response.haslayer(scapy.TCP):

    tcp = response[scapy.TCP]

    if tcp.flags & 0x12 == 0x12:
        print("Port is open")

    elif tcp.flags & 0x14 == 0x14:
        print("Port is closed")

    else:
        print("Unexpected TCP response")

else:
    print("No TCP response: filtered or inconclusive")
```

### Understanding the responses

| Response | Interpretation |
|---|---|
| SYN-ACK | Open |
| RST-ACK | Closed |
| No response | Filtered or inconclusive |
| Certain ICMP errors | May indicate filtering or network restrictions |

### When should you use SYN scanning?

- Discovering accessible TCP ports.
- Checking whether a service may be listening.
- Performing authorized network assessments.
- Building a TCP port scanner.

---

# 7. TCP Connect Scan

## Purpose

A TCP Connect scan attempts to establish a complete TCP connection with the destination port.

Unlike a SYN scan, it completes the TCP three-way handshake.

### How it works

```text
Your computer                    Target server

     |                                  |
     | -------- SYN ------------------> |
     |                                  |
     | <------ SYN-ACK ---------------- |
     |                                  |
     | -------- ACK ------------------> |
     |                                  |
     |       Connection established     |
```

The three-way handshake:

1. SYN: Client requests a connection.
2. SYN-ACK: Server acknowledges and responds.
3. ACK: Client acknowledges the server.

### Example using Python sockets

```python
import socket

target = "192.168.1.100"
port = 80

client = socket.socket(
    socket.AF_INET,
    socket.SOCK_STREAM
)

client.settimeout(2)

try:

    result = client.connect_ex((target, port))

    if result == 0:
        print("Port is open")

    else:
        print("Connection failed")

except socket.error as error:

    print("Socket error:", error)

finally:

    client.close()
```

### Understanding `connect_ex()`

Unlike `connect()`, which raises an exception for connection failures, `connect_ex()` returns an error code.

A return value of `0` means the connection succeeded.

### SYN Scan vs TCP Connect Scan

| Feature | SYN Scan | TCP Connect Scan |
|---|---|---|
| Sends SYN | Yes | Yes |
| Completes handshake | Normally no | Yes |
| Uses RST termination | Usually | Not as its normal connection process |
| Identifies open TCP ports | Yes | Yes |
| Requires raw packet access | Usually | No, ordinary sockets can perform it |

---

# 8. TCP ACK Scan

## Purpose

An ACK scan is primarily used to investigate firewall filtering behavior.

It does NOT reliably identify whether a port is open or closed.

This is an important distinction.

### How it works

The scanner sends a TCP packet with the ACK flag set.

```python
packet = (
    scapy.IP(dst="192.168.1.100") /
    scapy.TCP(
        dport=80,
        flags="A"
    )
)
```

The destination may respond with RST.

### Typical interpretation

| Response | Interpretation |
|---|---|
| RST | Unfiltered |
| No response | Filtered or inconclusive |
| Certain ICMP errors | Filtered |

### What does unfiltered mean?

Unfiltered means that the probe appears to have reached the target's TCP stack and received a response.

It does NOT mean the port is open.

For example:

```text
Port 80: Unfiltered
```

This does not prove that a web server is running on port 80.

### When should you use ACK scanning?

- Investigating firewall behavior.
- Understanding packet filtering.
- Comparing network security rules.
- Studying how firewalls handle unsolicited TCP packets.

---

# 9. TCP FIN Scan

## Purpose

A FIN scan sends a TCP packet with the FIN flag set.

FIN normally means that a TCP endpoint wants to finish a connection.

However, FIN scanning uses the response behavior of TCP implementations to investigate ports.

### Scapy packet

```python
packet = (
    scapy.IP(dst="192.168.1.100") /
    scapy.TCP(
        dport=80,
        flags="F"
    )
)
```

### Typical responses

| Response | Interpretation |
|---|---|
| RST | Closed |
| No response | Open or filtered |

Notice that the absence of a response cannot distinguish an open port from a filtered port.

### Important limitation

FIN scanning is not equally reliable across all operating systems and network configurations.

Do not assume that every system follows identical response behavior.

---

# 10. TCP NULL Scan

## Purpose

A NULL scan sends a TCP packet with no TCP control flags set.

### Scapy packet

```python
packet = (
    scapy.IP(dst="192.168.1.100") /
    scapy.TCP(
        dport=80,
        flags=0
    )
)
```

Notice:

```python
flags=0
```

No SYN, ACK, FIN, RST, PSH, or URG flags are set.

### Typical responses

| Response | Interpretation |
|---|---|
| RST | Closed |
| No response | Open or filtered |

### Why study NULL scanning?

It helps you understand TCP flag processing and how different TCP implementations respond to unusual packets.

---

# 11. TCP Xmas Scan

## Purpose

An Xmas scan sends a TCP packet with several flags set simultaneously.

Traditionally, it sets:

- FIN
- PSH
- URG

### Scapy packet

```python
packet = (
    scapy.IP(dst="192.168.1.100") /
    scapy.TCP(
        dport=80,
        flags="FPU"
    )
)
```

`FPU` represents FIN, PSH, and URG.

### Typical responses

| Response | Interpretation |
|---|---|
| RST | Closed |
| No response | Open or filtered |

### Why is it called Xmas?

The combination of several illuminated TCP flags gave rise to the name Christmas Tree scan.

### Important limitation

NULL, FIN, and Xmas scans depend on TCP implementation behavior. Some operating systems do not follow the expected response patterns.

---

# 12. UDP Scan

Not all network communication uses TCP.

UDP stands for User Datagram Protocol.

UDP is connectionless. It does not use the TCP three-way handshake.

Common UDP services include:

| Port | Common service |
|---|---|
| 53 | DNS |
| 67 | DHCP Server |
| 68 | DHCP Client |
| 69 | TFTP |
| 123 | NTP |
| 161 | SNMP |

## How UDP scanning works

A UDP scanner sends a UDP packet to a destination port.

### Scapy example

```python
import scapy.all as scapy

target = "192.168.1.100"
port = 53

packet = (
    scapy.IP(dst=target) /
    scapy.UDP(dport=port)
)

response = scapy.sr1(
    packet,
    timeout=3,
    verbose=False
)

if response is None:

    print("Open or filtered: inconclusive")

elif response.haslayer(scapy.UDP):

    print("UDP service responded")

elif response.haslayer(scapy.ICMP):

    icmp = response[scapy.ICMP]

    if icmp.type == 3 and icmp.code == 3:
        print("UDP port is closed")

    else:
        print("ICMP error:", icmp.type, icmp.code)
```

### Understanding UDP responses

| Response | Interpretation |
|---|---|
| UDP response | Port is open |
| ICMP type 3, code 3 | Port unreachable; typically closed |
| ICMP type 3, code 1 | Host unreachable |
| ICMP type 3, code 13 | Administratively prohibited |
| No response | Open or filtered; inconclusive |

### Why is UDP scanning different from TCP scanning?

TCP provides connection-oriented responses.

UDP does not establish a connection before transmitting data.

Therefore, UDP scanning can be harder to interpret.

A service may silently ignore an empty or invalid UDP request even when it is listening.

---

# 13. Service and Version Detection

Finding an open port is not the same as identifying the service.

For example:

```text
Port 22: Open
```

This tells you that a TCP connection may be accepted on port 22.

It does not prove that SSH is running.

Service detection attempts to identify the application behind an accessible port.

### Example: HTTP response

An HTTP server may return headers such as:

```http
HTTP/1.1 200 OK
Server: ExampleServer
Content-Type: text/html
```

The response can provide clues about the application.

### Example using Python

```python
import socket

target = "192.168.1.100"
port = 80

client = socket.socket(
    socket.AF_INET,
    socket.SOCK_STREAM
)

client.settimeout(3)

try:

    client.connect((target, port))

    request = (
        "HEAD / HTTP/1.1\r\n"
        f"Host: {target}\r\n"
        "Connection: close\r\n"
        "\r\n"
    )

    client.sendall(request.encode())

    response = client.recv(4096)

    print(response.decode(errors="replace"))

except OSError as error:

    print(error)

finally:

    client.close()
```

This sends an HTTP HEAD request and examines the response.

Some servers hide or alter their software-identification headers, so service detection is not always conclusive.

---

# 14. Operating System Detection

Operating system detection attempts to estimate which operating system a remote machine is running.

Examples:

- Windows
- Linux
- macOS
- FreeBSD

It is sometimes called OS fingerprinting.

## How does it work?

Different operating systems may implement networking behavior differently.

A scanner can examine characteristics such as:

1. TCP window size.
2. Initial TTL.
3. TCP options.
4. IP ID behavior.
5. Responses to particular probes.

For example, the initial TTL may provide a clue:

| Common initial TTL | Possible system family |
|---|---|
| 64 | Linux and Unix-like systems |
| 128 | Windows |
| 255 | Some network devices |

These are only common defaults, not reliable identification rules.

The TTL observed remotely may have been reduced by routers along the path.

---

# 15. Firewall Scanning and Filtering Detection

A firewall controls network traffic according to configured rules.

It may:

- Allow packets.
- Reject packets.
- Drop packets silently.
- Permit certain services.
- Block particular source addresses.

### Common results

| Scan result | Meaning |
|---|---|
| Open | A TCP connection can be established |
| Closed | The destination actively rejects the connection |
| Filtered | A firewall or network policy may be preventing a clear response |
| Unfiltered | The probe can reach the target's TCP stack |
| Open or filtered | Available evidence cannot distinguish the two |

### Example

Imagine:

```text
Port 80: Open
Port 22: Filtered
Port 23: Closed
```

Possible explanation:

- Port 80 accepts connections.
- Port 22 is blocked or silently dropped.
- Port 23 responds that no service is accepting connections.

Remember: These states describe what your scanner can observe, not necessarily the complete internal configuration of the target.

---

# 16. Vulnerability Scanning

Vulnerability scanning is different from ordinary port scanning.

Port scanning identifies accessible ports.

Vulnerability scanning investigates whether systems, services, applications, or configurations may contain known security weaknesses.

Examples of what vulnerability assessments investigate:

- Outdated software versions.
- Known vulnerable software.
- Weak security configurations.
- Unnecessary exposed services.
- Missing security updates.
- Insecure protocols.
- Misconfigured permissions.

A vulnerability scanner may use:

- Service detection.
- Software version detection.
- Configuration checks.
- Known vulnerability databases.
- Safe validation tests.

An identified version does not automatically prove that a vulnerability is exploitable. Configuration, patches, operating-system backports, and other conditions matter.

---

# 17. Summary: Comparing the Different Scanning Types

| Scan | Probe | Main purpose |
|---|---|---|
| ICMP Ping | ICMP Echo Request | Host discovery |
| ARP Scan | ARP Request | Local network discovery |
| TCP SYN | SYN | Discover open TCP ports |
| TCP Connect | Full TCP handshake | Discover accessible TCP ports |
| TCP ACK | ACK | Investigate firewall filtering |
| TCP FIN | FIN | Study TCP response behavior |
| TCP NULL | No flags | Study TCP response behavior |
| TCP Xmas | FIN + PSH + URG | Study TCP response behavior |
| UDP Scan | UDP | Discover UDP services |
| Service Detection | Application probes | Identify services |
| OS Detection | TCP/IP fingerprinting | Estimate operating system |
| Vulnerability Scan | Various checks | Identify potential security weaknesses |

---

# 18. Important Scapy Methods to Understand

Since you are building your scanner in Python, understand these methods.

| Method | Purpose |
|---|---|
| `send()` | Send Layer 3 packets |
| `sendp()` | Send Layer 2 frames |
| `sr()` | Send Layer 3 packets and receive multiple responses |
| `srp()` | Send Layer 2 frames and receive multiple responses |
| `sr1()` | Send Layer 3 packet and receive one response |
| `srp1()` | Send Layer 2 frame and receive one response |
| `sniff()` | Capture network packets |
| `wrpcap()` | Write packets to a PCAP file |
| `rdpcap()` | Read packets from a PCAP file |
| `IP()` | Construct an IPv4 layer |
| `IPv6()` | Construct an IPv6 layer |
| `Ether()` | Construct an Ethernet layer |
| `TCP()` | Construct a TCP layer |
| `UDP()` | Construct a UDP layer |
| `ICMP()` | Construct an ICMP layer |
| `ARP()` | Construct an ARP layer |

## Understanding packet responses

```python
if response:

    if response.haslayer(scapy.TCP):

        tcp = response.getlayer(scapy.TCP)

        print("TCP flags:", tcp.flags)

    elif response.haslayer(scapy.ICMP):

        icmp = response.getlayer(scapy.ICMP)

        print("ICMP type:", icmp.type)
        print("ICMP code:", icmp.code)

else:

    print("No response received")
```

This is a useful foundation for building a scanner that can interpret different response types.

---

# 19. Recommended Learning Order

Do not try to implement every scanning technique at once.

Build your understanding progressively.

## Stage 1: Networking foundations

- IP addresses.
- MAC addresses.
- Ethernet frames.
- IPv4 headers.
- TCP headers.
- UDP headers.
- ICMP.
- ARP.
- TCP flags.
- TCP three-way handshake.

## Stage 2: Basic scanning

1. ICMP Ping.
2. ARP Scan.
3. TCP SYN Scan.
4. TCP Connect Scan.
5. UDP Scan.

## Stage 3: Advanced TCP scanning

6. ACK Scan.
7. FIN Scan.
8. NULL Scan.
9. Xmas Scan.

## Stage 4: Network intelligence

10. Service Detection.
11. Version Detection.
12. OS Fingerprinting.
13. Firewall Analysis.

## Stage 5: Security assessment

14. Vulnerability Identification.
15. Security Configuration Assessment.
16. Report Generation.

---

# 20. Practice Questions

Use these questions to test your understanding.

1. What is the difference between host discovery and port scanning?
2. Why does a SYN scan send a SYN packet?
3. What does SYN-ACK tell you?
4. What is the difference between a SYN scan and a TCP Connect scan?
5. Why can't an ACK scan reliably identify an open port?
6. What does an RST response mean in a SYN scan?
7. Why can a UDP scan be difficult to interpret?
8. What is the difference between port scanning and service detection?
9. Why is an open port not proof of a particular application?
10. What is the purpose of OS fingerprinting?
11. What does filtered mean?
12. What is the difference between `sr1()` and `srp1()`?

## Final concept to remember

Network scanning is not just about finding open ports.

It is about collecting and interpreting network information.

A useful way to think about the process is:

```text
HOST DISCOVERY
      |
      v
PORT SCANNING
      |
      v
SERVICE IDENTIFICATION
      |
      v
VERSION DETECTION
      |
      v
OS FINGERPRINTING
      |
      v
SECURITY ASSESSMENT
      |
      v
REPORTING
```

Not every assessment needs every stage, and the order can vary depending on the objective.

**Your goal as a network security learner is not merely to send packets. It is to understand why you send each packet, what response you expect, and what conclusions that response actually supports.**
