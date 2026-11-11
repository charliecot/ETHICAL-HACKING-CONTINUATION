# COMPLETE NETWORKING, NETWORK SECURITY & SYSTEM ADMINISTRATION GUIDE

**Author:** Nzegge Charles Ngong  
**Level:** Beginner to Advanced  
**Operating Systems:** Windows 11, Ubuntu Linux and WSL  
**Focus:** Network discovery, monitoring, packet analysis, firewalls, ACLs, SSH and security administration.

---

# TABLE OF CONTENTS

1. Networking Fundamentals
2. Linux Network Configuration Tools
3. Nmap – Network Discovery and Port Scanning
4. Netcat, Socat and Nping
5. Network Diagnostic Tools
6. Wireshark and TShark
7. Tcpdump – Packet Capture
8. Monitoring Network Devices and Listening Ports
9. Firewall Fundamentals
10. UFW Firewall Configuration
11. iptables and nftables
12. Windows Defender Firewall
13. Access Control Lists (ACLs)
14. SSH – Secure Shell
15. SSH Server Configuration
16. SSH Password and Public-Key Authentication
17. SSH Port Forwarding and Tunnelling
18. Network Interfaces and Routing
19. DNS, ARP and Network Troubleshooting
20. Network Security Best Practices
21. Practical Networking Projects
22. Important Ports Reference
23. Useful Commands Cheat Sheet
24. Official Documentation

---

# 1. NETWORKING FUNDAMENTALS

## 1.1 What is a network?

A network is a collection of devices connected together to exchange information.

Examples:

- Computers
- Servers
- Routers
- Switches
- Printers
- Mobile phones
- IoT devices

## 1.2 Important networking terms

| Term | Meaning |
|---|---|
| IP address | Logical address identifying a network interface. |
| MAC address | Link-layer address identifying a network interface. |
| Port | Identifies a communication endpoint for an application. |
| Protocol | Rules governing communication. |
| TCP | Connection-oriented transport protocol. |
| UDP | Connectionless transport protocol. |
| DNS | Converts domain names into IP addresses. |
| ARP | Resolves IPv4 addresses to MAC addresses on a local network. |
| DHCP | Automatically assigns network configuration. |
| Gateway | Router used to reach other networks. |
| Subnet | A logical division of an IP network. |
| Packet | A unit of data at the network layer. |
| Frame | A unit of data at the data-link layer. |

## 1.3 Important network protocols

| Protocol | Default port | Purpose |
|---|---:|---|
| HTTP | 80 | Web communication |
| HTTPS | 443 | Encrypted web communication |
| SSH | 22 | Secure remote administration |
| DNS | 53 | Domain name resolution |
| DHCP | 67/68 | Automatic IP configuration |
| FTP | 20/21 | File transfer |
| SFTP | 22 | Secure file transfer over SSH |
| SMTP | 25 | Email transmission |
| SMB | 445 | Windows file sharing |
| RDP | 3389 | Remote Desktop |
| PostgreSQL | 5432 | Database communication |
| MySQL | 3306 | Database communication |
| Redis | 6379 | In-memory data store |

Remember: ports are not protocols themselves. TCP port 22 and UDP port 22 are different transport endpoints.

## 1.4 How network communication works

Example: You visit:

`https://example.com`

Simplified process:

1. Your browser needs the IP address of `example.com`.
2. DNS resolves the domain name.
3. Your computer determines whether the destination is local or remote.
4. If remote, it sends traffic through its default gateway.
5. TCP establishes a connection.
6. TLS negotiates encryption.
7. HTTP communicates over the encrypted connection.
8. The server sends its response.

---

# 2. LINUX NETWORK CONFIGURATION TOOLS

## 2.1 `ip` – Manage network interfaces and routes

The `ip` command is part of the iproute2 networking utilities.

### Display network interfaces

```bash
ip link
```

Displays network interfaces and their link states.

### Display IP addresses

```bash
ip addr
```

Short form:

```bash
ip a
```

Show only IPv4 addresses:

```bash
ip -4 addr
```

Show IPv6 addresses:

```bash
ip -6 addr
```

Show a specific interface:

```bash
ip addr show dev eth0
```

### Enable and disable interfaces

```bash
sudo ip link set eth0 up
```

Enable the interface.

```bash
sudo ip link set eth0 down
```

Disable the interface.

### Display routing information

```bash
ip route
```

Display the route to a destination:

```bash
ip route get 8.8.8.8
```

### Display ARP/neighbour information

```bash
ip neigh
```

Important states:

- REACHABLE
- STALE
- DELAY
- PROBE
- FAILED
- INCOMPLETE

## 2.2 `nmtui` – Terminal Network Configuration Interface

This may be the interactive network configuration tool you were thinking about.

Unlike ordinary commands, `nmtui` opens a text-based interface inside your terminal.

```bash
sudo nmtui
```

You can:

- Edit network connections.
- Activate connections.
- Deactivate connections.
- Configure IPv4.
- Configure IPv6.
- Set automatic or manual addressing.
- Configure DNS servers.
- Configure gateways.

Install it on Ubuntu if missing:

```bash
sudo apt update
sudo apt install network-manager
```

Launch:

```bash
nmtui
```

**Note:** WSL networking is managed differently from a normal Ubuntu installation. Some NetworkManager operations may not work in WSL.

## 2.3 `nmcli` – NetworkManager Command-Line Interface

Display connections:

```bash
nmcli connection show
```

Display devices:

```bash
nmcli device status
```

Display active connections:

```bash
nmcli connection show --active
```

Display Wi-Fi networks:

```bash
nmcli device wifi list
```

Connect to a Wi-Fi network:

```bash
nmcli device wifi connect "NETWORK_NAME" password "PASSWORD"
```

Activate a connection:

```bash
sudo nmcli connection up "Wired connection 1"
```

Deactivate:

```bash
sudo nmcli connection down "Wired connection 1"
```

---

# 3. NMAP – NETWORK DISCOVERY AND PORT SCANNING

Nmap means Network Mapper.

It is used for:

- Discovering devices.
- Finding open ports.
- Identifying services.
- Detecting software versions.
- Estimating operating systems.
- Auditing network exposure.
- Performing authorized security assessments.

## 3.1 Installation

Ubuntu:

```bash
sudo apt update
sudo apt install nmap
```

Windows:

Download from:

https://nmap.org/download.html

Verify:

```bash
nmap --version
```

Display help:

```bash
nmap -h
```

Full manual:

```bash
man nmap
```

## 3.2 Basic syntax

```bash
nmap [OPTIONS] TARGET
```

Examples:

```bash
nmap 192.168.1.1
nmap example.com
nmap 192.168.1.0/24
```

Only scan systems you own or are authorized to assess.

## 3.3 Target specification

| Option | Meaning |
|---|---|
| `-iL file.txt` | Read targets from a file. |
| `-iR 10` | Select 10 random targets. |
| `--exclude IP` | Exclude a target. |
| `--excludefile file.txt` | Exclude targets listed in a file. |
| `-6` | Enable IPv6 scanning. |

Examples:

```bash
nmap -iL targets.txt
```

```bash
nmap 192.168.1.1 192.168.1.10
```

```bash
nmap 192.168.1.1-20
```

```bash
nmap 192.168.1.0/24
```

## 3.4 Host discovery

Host discovery determines which devices appear reachable.

Ping scan:

```bash
nmap -sn 192.168.1.0/24
```

`-sn` disables port scanning and performs host discovery.

Skip host discovery:

```bash
nmap -Pn 192.168.1.10
```

Disable DNS resolution:

```bash
nmap -n 192.168.1.0/24
```

Traceroute:

```bash
nmap --traceroute 192.168.1.1
```

## 3.5 Port scanning

| State | Meaning |
|---|---|
| Open | An application is accepting connections. |
| Closed | The port is reachable but no application is listening. |
| Filtered | Filtering prevents Nmap from determining the state. |
| Unfiltered | The port responds, but open/closed status is unknown. |
| Open/Filtered | Nmap cannot distinguish between the two. |
| Closed/Filtered | Nmap cannot distinguish between the two. |

Scan common ports:

```bash
nmap 192.168.1.10
```

Scan a specific port:

```bash
nmap -p 22 192.168.1.10
```

Scan multiple ports:

```bash
nmap -p 22,80,443 192.168.1.10
```

Scan a port range:

```bash
nmap -p 1-1000 192.168.1.10
```

Scan every TCP port:

```bash
nmap -p- 192.168.1.10
```

Scan the most common ports:

```bash
nmap --top-ports 100 192.168.1.10
```

Fast scan:

```bash
nmap -F 192.168.1.10
```

## 3.6 Scan techniques

| Option | Technique |
|---|---|
| `-sS` | TCP SYN scan. |
| `-sT` | TCP connect scan. |
| `-sU` | UDP scan. |
| `-sA` | TCP ACK scan. |
| `-sN` | TCP NULL scan. |
| `-sF` | TCP FIN scan. |
| `-sX` | TCP Xmas scan. |

TCP SYN scan:

```bash
sudo nmap -sS 192.168.1.10
```

TCP connect scan:

```bash
nmap -sT 192.168.1.10
```

UDP scan:

```bash
sudo nmap -sU -p 53,123,161 192.168.1.10
```

UDP scanning can be slower because many UDP services do not respond to unexpected probes.

## 3.7 Service and version detection

```bash
nmap -sV 192.168.1.10
```

Attempts to determine:

- Service name.
- Software version.
- Protocol information.

Control version detection intensity:

```bash
nmap -sV --version-intensity 5 192.168.1.10
```

The intensity ranges from 0 to 9.

## 3.8 Operating system detection

```bash
sudo nmap -O 192.168.1.10
```

Attempts to identify the operating system.

More aggressive guessing:

```bash
sudo nmap -O --osscan-guess 192.168.1.10
```

OS detection is an estimate, not a guarantee.

## 3.9 Nmap Scripting Engine

```bash
nmap -sC 192.168.1.10
```

Runs Nmap's default NSE scripts.

Display script help:

```bash
nmap --script-help default
```

Use a specific safe script:

```bash
nmap --script http-title -p 80 192.168.1.10
```

Run default and safe scripts:

```bash
nmap --script "default,safe" 192.168.1.10
```

Always review scripts before executing them. Some NSE categories and scripts can be intrusive or affect the target.

## 3.10 Aggressive scan

```bash
sudo nmap -A 192.168.1.10
```

`-A` enables:

- OS detection.
- Version detection.
- Default script scanning.
- Traceroute.

It is more intrusive than a basic scan.

## 3.11 Timing options

| Option | Meaning |
|---|---|
| `-T0` | Paranoid timing. |
| `-T1` | Sneaky timing. |
| `-T2` | Polite timing. |
| `-T3` | Normal timing. |
| `-T4` | Aggressive timing. |
| `-T5` | Insane timing. |

Example:

```bash
nmap -T3 192.168.1.10
```

Timing controls scanning speed and probe behaviour. Faster is not always better.

## 3.12 Nmap output options

Normal output:

```bash
nmap -oN scan.txt 192.168.1.10
```

XML output:

```bash
nmap -oX scan.xml 192.168.1.10
```

Grepable output:

```bash
nmap -oG scan.gnmap 192.168.1.10
```

Save multiple formats:

```bash
nmap -oA network_scan 192.168.1.10
```

Increase verbosity:

```bash
nmap -v 192.168.1.10
```

More verbosity:

```bash
nmap -vv 192.168.1.10
```

Show why Nmap classified a port:

```bash
nmap --reason 192.168.1.10
```

Show only open ports:

```bash
nmap --open 192.168.1.10
```

### Useful complete examples

Discover devices:

```bash
nmap -sn 192.168.1.0/24
```

Inspect common TCP ports:

```bash
nmap -sS -p 1-1000 192.168.1.10
```

Identify services:

```bash
nmap -sV -p 22,80,443 192.168.1.10
```

Save an authorized assessment:

```bash
sudo nmap -sV -O -oA assessment 192.168.1.10
```

---

# 4. NETCAT, SOCAT AND NPING

## 4.1 Netcat (`nc`)

Netcat is a networking utility used to read and write data over network connections.

It is useful for:

- Testing TCP connectivity.
- Testing UDP connectivity.
- Connecting to a service.
- Sending simple data.
- Creating basic TCP listeners in a lab.
- Troubleshooting ports.

Install:

```bash
sudo apt install netcat-openbsd
```

Basic syntax:

```bash
nc [OPTIONS] HOST PORT
```

Test whether a TCP port is reachable:

```bash
nc -vz 192.168.1.10 22
```

Options:

- `-v`: Verbose output.
- `-z`: Scan mode; do not send application data.

Test multiple ports:

```bash
nc -vz 192.168.1.10 22 80 443
```

Set a connection timeout:

```bash
nc -vz -w 3 192.168.1.10 80
```

Start a basic TCP listener on your own lab machine:

```bash
nc -lv 9000
```

Options:

- `-l`: Listen.
- `-v`: Verbose.
- `9000`: Listening port.

From another terminal:

```bash
nc 127.0.0.1 9000
```

Type a message and observe how it appears in the listening terminal.

UDP listener:

```bash
nc -u -l 9000
```

Connect:

```bash
nc -u 127.0.0.1 9000
```

`-u` selects UDP.

## 4.2 Socat

Socat is a more advanced data-transfer utility.

It can connect:

- TCP sockets.
- UDP sockets.
- Files.
- Standard input/output.
- UNIX domain sockets.
- Serial devices.

Install:

```bash
sudo apt install socat
```

Display help:

```bash
socat -h
```

Create a TCP listener:

```bash
socat TCP-LISTEN:9000,reuseaddr,fork STDOUT
```

Important components:

- `TCP-LISTEN:9000`: Listen on TCP port 9000.
- `reuseaddr`: Allow address reuse where supported.
- `fork`: Create a child process for each connection.
- `STDOUT`: Write received data to standard output.

Test:

```bash
socat - TCP:127.0.0.1:9000
```

## 4.3 Nping

Nping comes with Nmap.

It is useful for generating and inspecting network probes.

Check installation:

```bash
nping --version
```

Display help:

```bash
nping --help
```

Send ICMP echo probes to a lab host:

```bash
nping --icmp 127.0.0.1
```

Send TCP SYN probes to your own test service:

```bash
sudo nping --tcp -p 80 127.0.0.1
```

Nping is useful when learning packet behaviour and firewall responses.

---

# 5. NETWORK DIAGNOSTIC TOOLS

## 5.1 Ping

Tests basic IP reachability using ICMP echo messages.

```bash
ping 8.8.8.8
```

Send a limited number:

```bash
ping -c 4 8.8.8.8
```

Set the interval:

```bash
ping -i 2 8.8.8.8
```

Send packets with a specified size:

```bash
ping -s 100 8.8.8.8
```

Stop continuous ping:

```text
CTRL + C
```

A failed ping does not prove that a machine is offline. ICMP may be blocked.

## 5.2 Traceroute

Shows the network hops used to reach a destination.

Install:

```bash
sudo apt install traceroute
```

Use:

```bash
traceroute example.com
```

TCP traceroute:

```bash
traceroute -T -p 443 example.com
```

TCP mode is useful when ordinary UDP or ICMP probes are filtered.

## 5.3 MTR

MTR combines traceroute and continuous ping-style measurements.

Install:

```bash
sudo apt install mtr
```

Run:

```bash
mtr example.com
```

Report mode:

```bash
mtr -r -c 10 example.com
```

Options:

- `-r`: Report mode.
- `-c`: Number of cycles.
- `-n`: Do not resolve hostnames.
- `-w`: Wide report.

Useful for investigating latency and packet loss along a route.

## 5.4 Curl

Curl transfers data using network protocols.

```bash
curl https://example.com
```

Display response headers:

```bash
curl -I https://example.com
```

Verbose connection information:

```bash
curl -v https://example.com
```

Display only response status:

```bash
curl -o /dev/null -s -w "%{http_code}\n" https://example.com
```

Follow redirects:

```bash
curl -L https://example.com
```

Specify a request method:

```bash
curl -X GET https://example.com
```

POST JSON:

```bash
curl -X POST https://example.com/api \
  -H "Content-Type: application/json" \
  -d '{"username":"charles"}'
```

Use a proxy:

```bash
curl -x http://127.0.0.1:8080 http://example.com
```

This is useful when testing your own Python HTTP proxy.

## 5.5 DNS tools

Install:

```bash
sudo apt install dnsutils
```

Query DNS:

```bash
dig example.com
```

Query A records:

```bash
dig example.com A
```

Query MX records:

```bash
dig example.com MX
```

Query NS records:

```bash
dig example.com NS
```

Specify a DNS server:

```bash
dig @8.8.8.8 example.com
```

Short output:

```bash
dig +short example.com
```

Alternative:

```bash
nslookup example.com
```

---

# 6. WIRESHARK AND TSHARK

## 6.1 What is Wireshark?

Wireshark is a graphical network protocol analyser.

It allows you to capture packets and inspect their contents.

It can help you understand:

- Ethernet frames.
- IP headers.
- TCP connections.
- UDP datagrams.
- DNS requests.
- HTTP communication.
- TLS handshakes.
- ARP requests and replies.
- DHCP communication.
- Retransmissions.
- TCP connection resets.

Install on Ubuntu:

```bash
sudo apt update
sudo apt install wireshark
```

On Windows, download from:

https://www.wireshark.org/download.html

Wireshark uses packet capture facilities such as Npcap on Windows.

## 6.2 How Wireshark works

```text
Network Interface
       |
       v
Packet Capture Library
       |
       v
Wireshark
       |
       v
Protocol Dissection
       |
       v
Packet Analysis
```

Wireshark does not automatically decrypt every encrypted connection.

For example, HTTPS traffic is normally protected by TLS. You can inspect metadata and handshake information, but application content remains encrypted unless you have an authorized decryption method.

## 6.3 Capture process

1. Open Wireshark.
2. Select your active network interface.
3. Double-click the interface.
4. Generate some network traffic.
5. Observe packets.
6. Apply display filters.
7. Select a packet.
8. Expand its protocol layers.

Example packet structure:

```text
Frame
 └── Ethernet II
      └── Internet Protocol Version 4
           └── Transmission Control Protocol
                └── Hypertext Transfer Protocol
```

## 6.4 Capture filters versus display filters

**Capture filters** determine what packets are collected.

**Display filters** determine which already captured packets are displayed.

| Capture filter | Display filter |
|---|---|
| `host 192.168.1.10` | `ip.addr == 192.168.1.10` |
| `port 80` | `tcp.port == 80` |
| `tcp` | `tcp` |
| `udp` | `udp` |
| `icmp` | `icmp` |
| `port 53` | `dns` |
| `host 192.168.1.10 and tcp` | `ip.addr == 192.168.1.10 && tcp` |

## 6.5 Important Wireshark display filters

### IP filters

Show IPv4 traffic:

```text
ip
```

Show IPv6 traffic:

```text
ipv6
```

Show a specific IP:

```text
ip.addr == 192.168.1.10
```

Source IP:

```text
ip.src == 192.168.1.10
```

Destination IP:

```text
ip.dst == 192.168.1.10
```

### TCP filters

```text
tcp
```

Specific TCP port:

```text
tcp.port == 443
```

TCP SYN packets:

```text
tcp.flags.syn == 1
```

TCP SYN without ACK:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

TCP reset:

```text
tcp.flags.reset == 1
```

TCP retransmissions:

```text
tcp.analysis.retransmission
```

### UDP filters

```text
udp
```

```text
udp.port == 53
```

### HTTP filters

```text
http
```

HTTP POST:

```text
http.request.method == "POST"
```

HTTP GET:

```text
http.request.method == "GET"
```

HTTP host:

```text
http.host
```

### DNS filters

```text
dns
```

DNS queries:

```text
dns.flags.response == 0
```

DNS responses:

```text
dns.flags.response == 1
```

### ARP filters

```text
arp
```

ARP requests:

```text
arp.opcode == 1
```

ARP replies:

```text
arp.opcode == 2
```

### TLS filters

```text
tls
```

TLS handshake:

```text
tls.handshake
```

## 6.6 Wireshark statistics

Explore the Statistics menu.

Important features:

- Conversations.
- Endpoints.
- Protocol Hierarchy.
- IO Graphs.
- Packet Lengths.
- Flow Graph.

### Conversations

Shows communication between endpoints.

Example:

```text
192.168.1.110:52000
        |
        | TCP
        |
142.250.x.x:443
```

You can inspect the amount of data transferred between endpoints.

### Endpoints

Shows participating IP and MAC addresses.

### Protocol Hierarchy

Shows which protocols appear in the capture and their relative traffic volumes.

## 6.7 TShark

TShark is Wireshark's command-line packet analyser.

Install:

```bash
sudo apt install tshark
```

List interfaces:

```bash
tshark -D
```

Capture packets:

```bash
sudo tshark -i eth0
```

Capture only 10 packets:

```bash
sudo tshark -i eth0 -c 10
```

Capture TCP traffic:

```bash
sudo tshark -i eth0 -f "tcp"
```

Use a display filter:

```bash
tshark -i eth0 -Y "dns"
```

Read a capture file:

```bash
tshark -r capture.pcapng
```

Extract selected fields:

```bash
tshark -r capture.pcapng \
  -T fields \
  -e ip.src \
  -e ip.dst \
  -e tcp.dstport
```

---

# 7. TCPDUMP – PACKET CAPTURE FROM THE TERMINAL

Tcpdump is a command-line packet analyser.

It is especially useful on servers without graphical interfaces.

Install:

```bash
sudo apt install tcpdump
```

List interfaces:

```bash
sudo tcpdump -D
```

Capture packets:

```bash
sudo tcpdump -i eth0
```

Capture TCP:

```bash
sudo tcpdump -i eth0 tcp
```

Capture UDP:

```bash
sudo tcpdump -i eth0 udp
```

Capture DNS:

```bash
sudo tcpdump -i eth0 port 53
```

Capture traffic involving a host:

```bash
sudo tcpdump -i eth0 host 192.168.1.10
```

Capture traffic on port 443:

```bash
sudo tcpdump -i eth0 port 443
```

Show packet contents in ASCII:

```bash
sudo tcpdump -A -i eth0
```

Show hexadecimal and ASCII:

```bash
sudo tcpdump -XX -i eth0
```

Save packets:

```bash
sudo tcpdump -i eth0 -w capture.pcap
```

Read a capture:

```bash
tcpdump -r capture.pcap
```

Limit the number of packets:

```bash
sudo tcpdump -i eth0 -c 100
```

Disable hostname resolution:

```bash
sudo tcpdump -n -i eth0
```

Important options:

| Option | Meaning |
|---|---|
| `-i` | Network interface. |
| `-D` | List interfaces. |
| `-w` | Write to a capture file. |
| `-r` | Read a capture file. |
| `-c` | Stop after a specified packet count. |
| `-n` | Disable address resolution. |
| `-nn` | Disable address and port resolution. |
| `-A` | ASCII output. |
| `-X` | Hexadecimal and ASCII output. |
| `-v` | Verbose output. |
| `-vv` | More verbose output. |
| `-s` | Snapshot length. |

Example:

```bash
sudo tcpdump -i eth0 -nn -s 0 -w network.pcap
```

This captures packets without resolving addresses or ports and requests the full available packet contents.

---

# 8. MONITORING NETWORK DEVICES AND LISTENING PORTS

There are two different questions:

1. Which applications are listening on my computer?
2. Which devices are communicating with my computer or network?

You need different tools for each.

## 8.1 What is a listening port?

A listening port is a network endpoint waiting for incoming connections.

Example:

```text
0.0.0.0:22
```

This means a service is listening on TCP port 22 on all IPv4 interfaces.

Compare:

```text
127.0.0.1:8080
```

This service listens only on the local loopback interface.

It is not directly reachable from other network devices through that binding.

## 8.2 `ss` – Socket Statistics

`ss` is one of the most useful Linux networking commands.

Show listening TCP sockets:

```bash
ss -ltn
```

Show listening UDP sockets:

```bash
ss -lun
```

Show all listening sockets:

```bash
ss -lntup
```

Show established connections:

```bash
ss -tn state established
```

Show all TCP connections:

```bash
ss -ant
```

Show process information:

```bash
sudo ss -lntup
```

Important options:

| Option | Meaning |
|---|---|
| `-l` | Listening sockets. |
| `-t` | TCP sockets. |
| `-u` | UDP sockets. |
| `-n` | Numeric addresses and ports. |
| `-p` | Process information. |
| `-a` | All sockets. |
| `-e` | Extended socket information. |
| `-4` | IPv4 only. |
| `-6` | IPv6 only. |

Find a specific port:

```bash
sudo ss -lntup '( sport = :8080 )'
```

## 8.3 `lsof` – List Open Files

Linux represents many resources, including sockets, as file descriptors.

`lsof` can display processes using network sockets.

Install:

```bash
sudo apt install lsof
```

Show network connections:

```bash
sudo lsof -i
```

Show listening sockets:

```bash
sudo lsof -i -P -n
```

Find a specific port:

```bash
sudo lsof -i :8080
```

Show TCP connections:

```bash
sudo lsof -iTCP
```

Show listening TCP connections:

```bash
sudo lsof -iTCP -sTCP:LISTEN
```

Options:

- `-i`: Network files.
- `-P`: Numeric ports.
- `-n`: Numeric IP addresses.
- `-p`: Select a process ID.
- `-u`: Select a user.
- `-sTCP:LISTEN`: Restrict to listening TCP sockets.

## 8.4 `fuser`

Find processes using a network port:

```bash
sudo fuser -n tcp 8080
```

Display process information:

```bash
sudo fuser -v -n tcp 8080
```

This is useful when a Python server reports:

```text
Address already in use
```

## 8.5 `nethogs`

Nethogs shows network bandwidth usage per process.

Install:

```bash
sudo apt install nethogs
```

Run:

```bash
sudo nethogs
```

Select an interface:

```bash
sudo nethogs eth0
```

It can help answer:

- Which process is consuming bandwidth?
- Which application is sending data?
- Which process is receiving data?

## 8.6 `iftop`

Displays bandwidth usage by network connections.

Install:

```bash
sudo apt install iftop
```

Run:

```bash
sudo iftop -i eth0
```

Important options:

| Option | Meaning |
|---|---|
| `-i` | Select interface. |
| `-n` | Disable hostname resolution. |
| `-N` | Disable service-name resolution. |
| `-P` | Show ports. |
| `-B` | Display bandwidth in bytes. |

## 8.7 `iptraf-ng`

Provides interactive terminal-based network monitoring.

Install:

```bash
sudo apt install iptraf-ng
```

Run:

```bash
sudo iptraf-ng
```

It provides information about:

- Interface statistics.
- IP traffic.
- TCP connections.
- UDP traffic.
- Packet statistics.

## 8.8 `vnstat`

Tracks network interface traffic over time.

Install:

```bash
sudo apt install vnstat
```

Display usage:

```bash
vnstat
```

Daily usage:

```bash
vnstat -d
```

Monthly usage:

```bash
vnstat -m
```

Hourly usage:

```bash
vnstat -h
```

## 8.9 Windows network monitoring

Open PowerShell.

Show listening ports:

```powershell
Get-NetTCPConnection -State Listen
```

Show established connections:

```powershell
Get-NetTCPConnection -State Established
```

Show all TCP connections:

```powershell
Get-NetTCPConnection
```

Show listening UDP endpoints:

```powershell
Get-NetUDPEndpoint
```

Show process IDs:

```powershell
Get-NetTCPConnection |
    Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
```

Find the process using a PID:

```powershell
Get-Process -Id 1234
```

Alternative:

```cmd
netstat -ano
```

Options:

| Option | Meaning |
|---|---|
| `-a` | All connections and listening ports. |
| `-n` | Numeric addresses and ports. |
| `-o` | Show process IDs. |
| `-b` | Show executable involved; requires elevated privileges. |
| `-p TCP` | Filter by TCP. |
| `-r` | Display routing table. |
| `-e` | Ethernet statistics. |

Find port 8080:

```cmd
netstat -ano | findstr :8080
```

Then:

```cmd
tasklist /FI "PID eq 1234"
```

Replace `1234` with the actual PID.

## 8.10 Can I monitor another device?

Yes, but the method depends on your access and network topology.

| Method | What you can observe |
|---|---|
| Nmap | Reachable hosts, ports and services. |
| Wireshark | Packets visible to your capture interface. |
| Tcpdump | Packets visible to the selected interface. |
| Router administration | Connected devices and traffic statistics, depending on router features. |
| Endpoint monitoring agent | Process and connection activity on an enrolled device. |
| NetFlow/IPFIX | Network flow metadata collected by compatible infrastructure. |
| SNMP | Device and interface statistics from authorized network devices. |

On a switched Ethernet network, your computer normally does not receive every other device's unicast traffic.

To observe traffic from other devices, you need an authorized observation point, such as:

- Router or firewall telemetry.
- A managed switch's SPAN/mirror port.
- A network TAP.
- An endpoint agent installed on the device.
- An authorized Wi-Fi monitoring setup.

Do not assume that running Wireshark on your laptop allows you to see every packet exchanged by every device on the LAN.

---

# 9. FIREWALL FUNDAMENTALS

## 9.1 What is a firewall?

A firewall is a security mechanism that controls network traffic according to predefined rules.

It can allow, reject or drop traffic.

```text
Incoming Connection
        |
        v
     Firewall
        |
        |--- Port 22  -> ALLOW
        |
        |--- Port 80  -> ALLOW
        |
        |--- Port 23  -> DENY
        |
        v
   Destination System
```

A firewall can protect:

- Individual computers.
- Servers.
- Entire networks.
- Cloud infrastructure.
- Network gateways.

## 9.2 Types of firewalls

### 1. Packet-filtering firewall

Examines packet information such as:

- Source IP.
- Destination IP.
- Source port.
- Destination port.
- Protocol.

### 2. Stateful firewall

Tracks connection state.

It understands whether a packet belongs to an established connection or is initiating a new connection.

### 3. Stateless firewall

Evaluates packets independently without maintaining connection state.

### 4. Application-layer firewall

Can inspect application-level communication.

Examples include web application firewalls (WAFs).

### 5. Network firewall

Protects a network boundary or separates network segments.

### 6. Host-based firewall

Runs on an individual computer.

Examples:

- UFW.
- Windows Defender Firewall.
- Linux nftables.

## 9.3 Firewall chains and directions

Three important traffic directions:

| Direction | Meaning |
|---|---|
| INPUT | Traffic coming into the local machine. |
| OUTPUT | Traffic leaving the local machine. |
| FORWARD | Traffic passing through the machine to another destination. |

Example:

Your Ubuntu machine runs an SSH server.

A remote computer wants to connect to port 22.

The incoming packet is evaluated by the INPUT path.

If Ubuntu is acting as a router, traffic passing through it may be evaluated by FORWARD.

## 9.4 Default firewall policies

Three common decisions:

- ACCEPT: Allow traffic.
- DROP: Silently discard traffic.
- REJECT: Refuse traffic, usually with a response.

**DROP:** The sender may wait for a timeout because no response is sent.

**REJECT:** The sender receives an explicit rejection.

## 9.5 Inbound and outbound rules

Inbound controls traffic entering your computer.

Outbound controls traffic leaving your computer.

Example:

Allow incoming SSH connections only from a trusted administrator network.

Allow outbound HTTPS but restrict unnecessary outbound communication.

## 9.6 Stateful versus stateless

Consider a web browser connecting to a web server.

```text
Client: 192.168.1.10:50000
Server: 93.184.216.34:443
```

The client initiates the connection.

The server sends responses back.

A stateful firewall tracks the connection so the return packets can be accepted without creating a separate unrestricted inbound rule for every possible client port.

## 9.7 Firewall rule order

Rules can be evaluated in an order.

Example:

```text
1. Allow trusted administrator IP.
2. Allow established connections.
3. Deny unwanted traffic.
```

Rule ordering matters, particularly when using ordered ACLs and firewall chains.

## 9.8 Principle of least privilege

Only allow the traffic that is actually needed.

Bad approach:

```text
Allow every incoming port.
```

Better approach:

```text
Allow TCP 22 from the administrator network.
Allow TCP 443 for public HTTPS.
Deny other unnecessary incoming traffic.
```

---

# 10. UFW FIREWALL CONFIGURATION

UFW means Uncomplicated Firewall.

It provides a simpler interface for managing Linux firewall rules.

Install:

```bash
sudo apt update
sudo apt install ufw
```

Check status:

```bash
sudo ufw status
```

Detailed status:

```bash
sudo ufw status verbose
```

## 10.1 Enable and disable

Enable:

```bash
sudo ufw enable
```

Disable:

```bash
sudo ufw disable
```

Reload:

```bash
sudo ufw reload
```

Reset:

```bash
sudo ufw reset
```

Be careful with reset: it removes configured rules.

## 10.2 Default policies

Deny incoming:

```bash
sudo ufw default deny incoming
```

Allow outgoing:

```bash
sudo ufw default allow outgoing
```

Deny outgoing:

```bash
sudo ufw default deny outgoing
```

A common server starting policy is:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Before enabling this on a remote server, make sure you have allowed the management connection.

## 10.3 Allow ports

Allow SSH:

```bash
sudo ufw allow 22/tcp
```

Allow HTTP:

```bash
sudo ufw allow 80/tcp
```

Allow HTTPS:

```bash
sudo ufw allow 443/tcp
```

Allow a custom application port:

```bash
sudo ufw allow 8080/tcp
```

Allow UDP:

```bash
sudo ufw allow 53/udp
```

## 10.4 Deny ports

```bash
sudo ufw deny 23/tcp
```

Deny a specific port.

## 10.5 Restrict access by IP

Allow SSH only from a trusted IP:

```bash
sudo ufw allow from 192.168.1.50 to any port 22 proto tcp
```

Allow access from an entire subnet:

```bash
sudo ufw allow from 192.168.1.0/24
```

Restrict access to an application:

```bash
sudo ufw allow from 192.168.1.50 to any port 8080 proto tcp
```

## 10.6 Delete rules

Display numbered rules:

```bash
sudo ufw status numbered
```

Delete a rule by number:

```bash
sudo ufw delete 2
```

Or delete by expression:

```bash
sudo ufw delete allow 22/tcp
```

## 10.7 Logging

Enable logging:

```bash
sudo ufw logging on
```

Set logging level:

```bash
sudo ufw logging medium
```

Available levels commonly include:

- off
- low
- medium
- high
- full

Inspect system logs:

```bash
sudo journalctl -k
```

On systems where UFW logging is configured to use this file:

```bash
sudo grep UFW /var/log/ufw.log
```

## 10.8 Complete example

For a server hosting an HTTPS application and SSH administration:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow from 192.168.1.50 to any port 22 proto tcp

sudo ufw allow 443/tcp

sudo ufw enable

sudo ufw status verbose
```

This example assumes the administrator IP is trusted and HTTPS is intended to be public.

Do not blindly copy the example to a remote server without checking your actual management IP and firewall environment.

---

# 11. IPTABLES AND NFTABLES

These tools operate at a lower level than UFW.

## 11.1 iptables

iptables is a userspace utility for configuring Linux Netfilter rules.

Display current rules:

```bash
sudo iptables -L
```

Numeric output:

```bash
sudo iptables -L -n
```

Show line numbers:

```bash
sudo iptables -L -n --line-numbers
```

Show rules in command format:

```bash
sudo iptables -S
```

Show NAT rules:

```bash
sudo iptables -t nat -L -n
```

### Allow incoming SSH

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

Meaning:

- `-A INPUT`: Append to INPUT chain.
- `-p tcp`: TCP protocol.
- `--dport 22`: Destination port 22.
- `-j ACCEPT`: Accept matching packets.

### Allow established connections

```bash
sudo iptables -A INPUT -m conntrack \
  --ctstate ESTABLISHED,RELATED -j ACCEPT
```

### Allow loopback

```bash
sudo iptables -A INPUT -i lo -j ACCEPT
```

### Drop unwanted incoming traffic

```bash
sudo iptables -A INPUT -j DROP
```

Important: Rule order is significant. Place necessary allow rules before the final DROP rule.

### Delete a rule

```bash
sudo iptables -D INPUT 1
```

Deletes rule number 1 from INPUT.

### Flush rules

```bash
sudo iptables -F
```

Removes rules from chains in the selected table.

This can remove security protections. Understand the existing rules before flushing.

## 11.2 nftables

nftables is the modern Linux Netfilter framework.

It is generally preferred for new direct Netfilter configurations.

Install:

```bash
sudo apt install nftables
```

Display rules:

```bash
sudo nft list ruleset
```

Create a table:

```bash
sudo nft add table inet filter
```

Create an INPUT chain:

```bash
sudo nft add chain inet filter input \
'{ type filter hook input priority 0; policy drop; }'
```

Allow loopback:

```bash
sudo nft add rule inet filter input iifname "lo" accept
```

Allow established connections:

```bash
sudo nft add rule inet filter input \
ct state established,related accept
```

Allow SSH:

```bash
sudo nft add rule inet filter input \
tcp dport 22 accept
```

Allow HTTPS:

```bash
sudo nft add rule inet filter input \
tcp dport 443 accept
```

Show rules with handles:

```bash
sudo nft -a list ruleset
```

Delete a specific rule using its handle:

```bash
sudo nft delete rule inet filter input handle 5
```

The handle must be obtained from your actual ruleset.

Save configuration in:

```text
/etc/nftables.conf
```

Enable service:

```bash
sudo systemctl enable nftables
sudo systemctl start nftables
```

Validate configuration:

```bash
sudo nft -c -f /etc/nftables.conf
```

Avoid managing the same firewall simultaneously through UFW and unrelated direct nftables configurations unless you understand how the rules interact.

---

# 12. WINDOWS DEFENDER FIREWALL

Windows includes a built-in host firewall.

You can manage it using:

- Windows Defender Firewall with Advanced Security.
- PowerShell.
- `netsh`.

## 12.1 Open the graphical interface

Press:

```text
Windows + R
```

Enter:

```text
wf.msc
```

This opens Windows Defender Firewall with Advanced Security.

You can manage:

- Inbound Rules.
- Outbound Rules.
- Connection Security Rules.
- Monitoring.

## 12.2 PowerShell commands

Check firewall profiles:

```powershell
Get-NetFirewallProfile
```

Profiles include:

- Domain
- Private
- Public

Enable firewall:

```powershell
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True
```

Enable only the private profile:

```powershell
Set-NetFirewallProfile -Profile Private -Enabled True
```

Check rules:

```powershell
Get-NetFirewallRule
```

Display enabled inbound rules:

```powershell
Get-NetFirewallRule -Enabled True -Direction Inbound
```

## 12.3 Create an inbound rule

Allow TCP port 8080:

```powershell
New-NetFirewallRule `
  -DisplayName "Python Proxy" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 8080 `
  -Action Allow `
  -Profile Private
```

This creates a rule allowing inbound TCP traffic to port 8080 on the Private profile.

## 12.4 Restrict source IP

```powershell
New-NetFirewallRule `
  -DisplayName "Allow SSH from Admin" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 22 `
  -RemoteAddress 192.168.1.50 `
  -Action Allow
```

## 12.5 Block a port

```powershell
New-NetFirewallRule `
  -DisplayName "Block TCP 23" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 23 `
  -Action Block
```

## 12.6 Remove a rule

```powershell
Remove-NetFirewallRule -DisplayName "Python Proxy"
```

## 12.7 netsh

Show firewall configuration:

```cmd
netsh advfirewall show allprofiles
```

Enable firewall:

```cmd
netsh advfirewall set allprofiles state on
```

Disable firewall:

```cmd
netsh advfirewall set allprofiles state off
```

Avoid disabling the firewall as a routine troubleshooting technique.

---

# 13. ACCESS CONTROL LISTS (ACLs)

## 13.1 What is an ACL?

ACL means Access Control List.

An ACL defines which users, devices or traffic are allowed or denied access to a resource.

There are two important contexts:

1. Network ACLs.
2. Filesystem ACLs.

They are related through access control, but they control different things.

## 13.2 Network ACLs

Network ACLs filter traffic according to criteria such as:

- Source IP.
- Destination IP.
- Protocol.
- Source port.
- Destination port.
- Direction.

Example:

```text
Rule 1: Allow 192.168.1.0/24 to TCP 443.
Rule 2: Deny 192.168.1.50 to TCP 22.
Rule 3: Deny other incoming traffic.
```

Network ACLs are common in:

- Routers.
- Switches.
- Firewalls.
- Cloud networks.

### Standard and extended ACLs

Traditional network device terminology:

**Standard ACL**

Usually filters based on source IP address.

**Extended ACL**

Can filter based on:

- Source IP.
- Destination IP.
- Protocol.
- Ports.

### Example Cisco-style ACL

```text
access-list 10 permit 192.168.1.0 0.0.0.255
access-list 10 deny any
```

This is a standard IPv4 ACL example.

Extended example:

```text
access-list 101 permit tcp 192.168.1.0 0.0.0.255 any eq 443
access-list 101 deny ip any any
```

These are Cisco IOS-style commands, not Linux commands.

The wildcard mask `0.0.0.255` corresponds to a `/24` subnet.

## 13.3 Linux filesystem ACLs

Linux ACLs control access to files and directories.

Ordinary Linux permissions provide:

- Read (`r`)
- Write (`w`)
- Execute (`x`)

ACLs allow more granular permissions for additional users and groups.

Install:

```bash
sudo apt install acl
```

View permissions:

```bash
getfacl file.txt
```

Set an ACL:

```bash
setfacl -m u:charles:rwx file.txt
```

Meaning:

- `-m`: Modify ACL.
- `u`: User.
- `charles`: Username.
- `rwx`: Read, write and execute.

Give a group read access:

```bash
setfacl -m g:developers:r file.txt
```

Remove a user ACL:

```bash
setfacl -x u:charles file.txt
```

Remove all extended ACL entries:

```bash
setfacl -b file.txt
```

Set a default ACL on a directory:

```bash
setfacl -d -m u:charles:rwx project/
```

Default ACLs affect newly created objects inside the directory, subject to normal permission and application behaviour.

View ACLs recursively:

```bash
getfacl -R project/
```

## 13.4 Windows filesystem ACLs

Display permissions:

```cmd
icacls file.txt
```

Grant permissions:

```cmd
icacls file.txt /grant "Charles:(R)"
```

Grant read and write:

```cmd
icacls file.txt /grant "Charles:(M)"
```

Permission abbreviations:

| Permission | Meaning |
|---|---|
| F | Full control |
| M | Modify |
| RX | Read and execute |
| R | Read |
| W | Write |

Remove permissions:

```cmd
icacls file.txt /remove Charles
```

Save ACLs:

```cmd
icacls project /save permissions.txt /t
```

Restore ACLs:

```cmd
icacls project /restore permissions.txt
```

---

# 14. SSH – SECURE SHELL

## 14.1 What is SSH?

SSH stands for Secure Shell.

It is a cryptographic network protocol used to securely access and administer remote computers.

Example:

```text
Your Computer
192.168.1.110
       |
       | SSH
       |
Remote Server
192.168.1.120
```

You can administer the remote server from your computer without physically accessing it.

SSH normally uses TCP port 22.

## 14.2 SSH components

**SSH Client**

The application that initiates a connection.

Command:

```bash
ssh
```

**SSH Server**

The service that accepts incoming SSH connections.

Daemon:

```bash
sshd
```

Configuration:

```text
/etc/ssh/sshd_config
```

## 14.3 Install SSH server

Ubuntu:

```bash
sudo apt update
sudo apt install openssh-server
```

Check service:

```bash
sudo systemctl status ssh
```

Start:

```bash
sudo systemctl start ssh
```

Enable at boot:

```bash
sudo systemctl enable ssh
```

Restart:

```bash
sudo systemctl restart ssh
```

Reload configuration:

```bash
sudo systemctl reload ssh
```

Check whether SSH is listening:

```bash
sudo ss -lntp | grep ssh
```

## 14.4 Connect to a remote machine

Basic syntax:

```bash
ssh username@IP_ADDRESS
```

Example:

```bash
ssh charles@192.168.1.120
```

Connect using a different port:

```bash
ssh -p 2222 charles@192.168.1.120
```

Enable verbose debugging:

```bash
ssh -v charles@192.168.1.120
```

More detailed debugging:

```bash
ssh -vvv charles@192.168.1.120
```

Execute a remote command:

```bash
ssh charles@192.168.1.120 "ip addr"
```

## 14.5 Important SSH client options

| Option | Meaning |
|---|---|
| `-p` | Specify remote port. |
| `-i` | Specify private key file. |
| `-v` | Verbose debugging. |
| `-vv` | More debugging. |
| `-vvv` | Maximum normal verbosity. |
| `-L` | Local port forwarding. |
| `-R` | Remote port forwarding. |
| `-D` | Dynamic SOCKS forwarding. |
| `-N` | Do not execute a remote command. |
| `-T` | Disable pseudo-terminal allocation. |
| `-X` | Enable X11 forwarding. |
| `-o` | Set an SSH configuration option. |
| `-J` | Connect through a jump host. |

---

# 15. SSH SERVER CONFIGURATION – `/etc/ssh/sshd_config`

This is one of the most important files for SSH server administration.

Open the configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

You can also use:

```bash
sudo vim /etc/ssh/sshd_config
```

If you use VS Code with a remote Linux environment, you can edit the file there as well.

Important distinction:

- `ssh_config`: SSH client configuration.
- `sshd_config`: SSH server configuration.

## 15.1 Important server configuration directives

### Port

```text
Port 22
```

Specifies the listening TCP port.

You can change it:

```text
Port 2222
```

### ListenAddress

```text
ListenAddress 0.0.0.0
```

Controls the addresses on which SSH listens.

For IPv4:

```text
ListenAddress 192.168.1.120
```

For IPv6:

```text
ListenAddress ::
```

Do not bind to an address that is not assigned to the server.

### PermitRootLogin

```text
PermitRootLogin no
```

Prevents direct root login.

This is a recommended security setting for many servers.

### PasswordAuthentication

```text
PasswordAuthentication yes
```

Allows password authentication.

Disable it:

```text
PasswordAuthentication no
```

Only disable it after public-key authentication has been tested successfully.

### PubkeyAuthentication

```text
PubkeyAuthentication yes
```

Allows public-key authentication.

### PermitEmptyPasswords

```text
PermitEmptyPasswords no
```

Prevents authentication with empty passwords.

### MaxAuthTries

```text
MaxAuthTries 3
```

Limits authentication attempts per connection.

### LoginGraceTime

```text
LoginGraceTime 30
```

Controls how long an unauthenticated connection can remain open.

### AllowUsers

```text
AllowUsers charles admin
```

Only specified users are allowed to log in.

### AllowGroups

```text
AllowGroups sshusers
```

Restricts SSH access to members of specified groups.

### X11Forwarding

```text
X11Forwarding no
```

Disables X11 forwarding when not needed.

### AllowTcpForwarding

```text
AllowTcpForwarding no
```

Disables TCP forwarding when not required.

Be careful: this would also prevent SSH-based tunnelling.

### ClientAliveInterval

```text
ClientAliveInterval 300
```

Specifies an interval for server-side connection checks.

### ClientAliveCountMax

```text
ClientAliveCountMax 2
```

Controls how many unanswered checks are tolerated.

## 15.2 How to change the SSH port

Suppose you want to change port 22 to port 2222.

### Step 1: Back up the configuration

```bash
sudo cp /etc/ssh/sshd_config \
/etc/ssh/sshd_config.backup
```

### Step 2: Open the configuration

```bash
sudo nano /etc/ssh/sshd_config
```

Find:

```text
#Port 22
```

Change to:

```text
Port 2222
```

Remove the `#` because commented directives are not active settings.

### Step 3: Allow the new port through the firewall

With UFW:

```bash
sudo ufw allow 2222/tcp
```

Do this before closing your existing SSH session.

### Step 4: Validate configuration

```bash
sudo sshd -t
```

If there is no output, the configuration passed syntax validation.

### Step 5: Reload SSH

```bash
sudo systemctl reload ssh
```

If reload is unsupported in your environment, use the appropriate service restart after validating the configuration.

### Step 6: Check listening ports

```bash
sudo ss -lntp | grep ssh
```

### Step 7: Test the new port

From another terminal:

```bash
ssh -p 2222 charles@192.168.1.120
```

### Step 8: Remove the old firewall rule

Only after confirming that the new connection works:

```bash
sudo ufw delete allow 22/tcp
```

Changing the port reduces exposure to some automated scans, but it is not a replacement for strong authentication, patching and firewall restrictions.

**Important:** Some Linux distributions use socket activation or additional SSH configuration files. If the service continues listening on the old port, inspect the service and socket configuration as well.

## 15.3 Inspect the effective SSH configuration

```bash
sudo sshd -T
```

This prints the effective configuration.

Inspect a particular directive:

```bash
sudo sshd -T | grep passwordauthentication
```

You can also inspect:

```bash
sudo sshd -T | grep permitrootlogin
```

This is useful because configuration defaults and included files can affect the final result.

## 15.4 SSH logs

Ubuntu:

```bash
sudo journalctl -u ssh
```

Follow logs:

```bash
sudo journalctl -u ssh -f
```

Recent logs:

```bash
sudo journalctl -u ssh -n 50
```

On systems using authentication logs:

```bash
sudo tail -f /var/log/auth.log
```

These logs help identify:

- Successful logins.
- Failed logins.
- Authentication methods.
- Connection attempts.
- Configuration problems.

---

# 16. SSH PASSWORD AND PUBLIC-KEY AUTHENTICATION

There are two important authentication methods you should understand.

## 16.1 Password authentication

Example:

```bash
ssh charles@192.168.1.120
```

The SSH client requests a password.

The SSH server verifies the supplied credentials.

The authentication exchange is protected by the SSH encrypted connection.

Advantages:

- Easy to configure.
- Familiar.
- Does not require creating SSH key pairs.

Disadvantages:

- Weak passwords can be guessed.
- Password reuse is dangerous.
- Automated password attacks are common.

## 16.2 Public-key authentication

Public-key authentication uses a cryptographic key pair.

You create:

1. Private key.
2. Public key.

The private key remains with the client.

The public key is placed on the server.

```text
Your Computer
    |
    | Private Key
    |
    | SSH Authentication
    |
Remote Server
    |
    | Authorized Public Key
    |
    v
Authentication Verification
```

The private key should never be shared.

## 16.3 Generate an SSH key pair

```bash
ssh-keygen -t ed25519
```

Options:

| Option | Meaning |
|---|---|
| `-t` | Key type. |
| `-b` | Key size where applicable. |
| `-C` | Comment identifying the key. |
| `-f` | Specify output file. |
| `-N` | Specify a passphrase. |

Generate with a comment:

```bash
ssh-keygen -t ed25519 -C "charles@laptop"
```

Default files:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

- `id_ed25519`: Private key.
- `id_ed25519.pub`: Public key.

## 16.4 Copy the public key to the server

```bash
ssh-copy-id charles@192.168.1.120
```

Specify the SSH port:

```bash
ssh-copy-id -p 2222 charles@192.168.1.120
```

If `ssh-copy-id` is unavailable, the public key can be added manually to:

```text
~/.ssh/authorized_keys
```

On the server, ensure appropriate permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 16.5 Connect using a private key

```bash
ssh -i ~/.ssh/id_ed25519 charles@192.168.1.120
```

Using a custom port:

```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 charles@192.168.1.120
```

## 16.6 Disable password authentication

First, test key authentication in a separate terminal.

After confirming key-based login works, edit:

```bash
sudo nano /etc/ssh/sshd_config
```

Set:

```text
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
```

Validate:

```bash
sudo sshd -t
```

Reload:

```bash
sudo systemctl reload ssh
```

Test another new SSH connection.

Keep the original administrative session open until the new connection has been verified.

## 16.7 SSH agent

An SSH agent manages private keys so you do not need to repeatedly enter their passphrases.

Start:

```bash
eval "$(ssh-agent -s)"
```

Add your private key:

```bash
ssh-add ~/.ssh/id_ed25519
```

List keys:

```bash
ssh-add -l
```

Remove all identities:

```bash
ssh-add -D
```

## 16.8 SSH client configuration

Edit:

```bash
nano ~/.ssh/config
```

Example:

```text
Host myserver
    HostName 192.168.1.120
    User charles
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

Now connect using:

```bash
ssh myserver
```

This avoids repeatedly typing the full command.

---

# 17. SSH PORT FORWARDING AND TUNNELLING

SSH can securely transport traffic through encrypted connections.

This is especially useful when learning proxies and networking.

## 17.1 Local port forwarding

Syntax:

```bash
ssh -L LOCAL_PORT:DESTINATION_HOST:DESTINATION_PORT user@SSH_SERVER
```

Example:

```bash
ssh -L 5433:127.0.0.1:5432 charles@192.168.1.120
```

This allows your local machine to connect to the remote server's PostgreSQL service through local port 5433.

Your local application connects to:

```text
127.0.0.1:5433
```

The SSH connection forwards the traffic to:

```text
127.0.0.1:5432
```

as seen from the SSH server.

## 17.2 Remote port forwarding

Syntax:

```bash
ssh -R REMOTE_PORT:DESTINATION_HOST:DESTINATION_PORT user@SSH_SERVER
```

Example:

```bash
ssh -R 9000:127.0.0.1:8000 charles@192.168.1.120
```

This allows the remote SSH server to forward connections from its port 9000 to your local port 8000, subject to SSH server forwarding policies.

## 17.3 Dynamic port forwarding

```bash
ssh -D 1080 charles@192.168.1.120
```

Creates a local SOCKS proxy.

Configure an application to use:

```text
SOCKS5
127.0.0.1
1080
```

This is useful for understanding proxy networking.

## 17.4 Keep the tunnel open without a remote shell

```bash
ssh -N -L 5433:127.0.0.1:5432 charles@192.168.1.120
```

`-N` prevents execution of a remote command.

---

# 18. NETWORK INTERFACES AND ROUTING

## 18.1 Windows commands

Display IP configuration:

```cmd
ipconfig
```

Detailed information:

```cmd
ipconfig /all
```

Release DHCP address:

```cmd
ipconfig /release
```

Request DHCP address:

```cmd
ipconfig /renew
```

Flush DNS cache:

```cmd
ipconfig /flushdns
```

Display routes:

```cmd
route print
```

Display ARP cache:

```cmd
arp -a
```

Display Ethernet statistics:

```cmd
netstat -e
```

## 18.2 Linux commands

Display addresses:

```bash
ip addr
```

Display routing:

```bash
ip route
```

Display neighbours:

```bash
ip neigh
```

Display interfaces:

```bash
ip link
```

Show interface statistics:

```bash
ip -s link
```

## 18.3 `ethtool`

`ethtool` displays and configures Ethernet device settings.

Install:

```bash
sudo apt install ethtool
```

Display interface information:

```bash
sudo ethtool eth0
```

Display driver information:

```bash
sudo ethtool -i eth0
```

Display statistics:

```bash
sudo ethtool -S eth0
```

This can help investigate link speed, duplex, drivers and interface errors.

---

# 19. DNS, ARP AND NETWORK TROUBLESHOOTING

## 19.1 DNS troubleshooting

Test whether a domain resolves:

```bash
nslookup example.com
```

Use dig:

```bash
dig example.com
```

Test a specific DNS server:

```bash
dig @8.8.8.8 example.com
```

Inspect DNS configuration:

```bash
cat /etc/resolv.conf
```

On Windows:

```cmd
ipconfig /displaydns
```

## 19.2 ARP troubleshooting

Display ARP table:

```bash
arp -a
```

Linux:

```bash
ip neigh
```

Capture ARP packets:

```bash
sudo tcpdump -i eth0 arp
```

Wireshark filter:

```text
arp
```

An ARP request asks:

"Who has this IPv4 address? Tell me your MAC address."

An ARP reply provides the corresponding MAC address.

## 19.3 Basic troubleshooting methodology

When a website cannot be reached, do not immediately blame DNS.

Investigate layer by layer.

### Step 1: Check your interface

```bash
ip addr
```

### Step 2: Check your gateway

```bash
ip route
```

### Step 3: Test gateway reachability

```bash
ping -c 4 192.168.1.1
```

### Step 4: Test external IP reachability

```bash
ping -c 4 8.8.8.8
```

### Step 5: Test DNS

```bash
dig example.com
```

### Step 6: Test HTTPS

```bash
curl -I https://example.com
```

### Step 7: Inspect connections

```bash
ss -ant
```

### Step 8: Capture packets

```bash
sudo tcpdump -i eth0 -nn
```

This sequence helps isolate the problem.

---

# 20. NETWORK SECURITY BEST PRACTICES

## 20.1 Secure listening services

Regularly inspect listening ports:

```bash
sudo ss -lntup
```

Investigate services you do not recognize.

For each service, ask:

1. What application created it?
2. Why is it listening?
3. Does it need network access?
4. Should it listen on all interfaces?
5. Can access be restricted by firewall rules?

## 20.2 Restrict exposed services

Avoid unnecessarily exposing:

- Database servers.
- Redis.
- Development servers.
- Debugging services.
- Administrative interfaces.
- Unprotected file-sharing services.

For example, a development server should normally listen on:

```text
127.0.0.1
```

if only local access is required.

## 20.3 SSH security

Recommended practices:

- Prefer public-key authentication.
- Use strong passphrases for private keys.
- Disable root login where appropriate.
- Restrict SSH access by IP where practical.
- Keep OpenSSH updated.
- Monitor authentication logs.
- Use MFA where supported.
- Avoid exposing SSH unnecessarily.

## 20.4 Firewall security

Recommended practices:

- Default-deny incoming traffic where appropriate.
- Allow only required services.
- Restrict administrative ports.
- Review rules regularly.
- Monitor firewall logs.
- Test IPv4 and IPv6 rules.
- Keep firewall configurations backed up.

## 20.5 Network monitoring

Monitor:

- New listening services.
- Unexpected outbound connections.
- Unusual bandwidth consumption.
- Failed login attempts.
- Repeated connection attempts.
- DNS anomalies.
- Unexpected devices.
- Unusual traffic patterns.

Monitoring should be authorized, documented and proportionate.

---

# 21. PRACTICAL NETWORKING PROJECTS

These projects will help you turn the commands into actual skills.

## Project 1: Network Discovery Tool

**Objective:** Discover devices on your local lab network.

Modules:

- `subprocess`
- `ipaddress`
- `socket`
- `threading`
- `json`

Features:

- Accept a subnet.
- Discover reachable devices.
- Record IP addresses.
- Save results to JSON.
- Display a summary.

Suggested command:

```bash
python network_discovery.py 192.168.1.0/24
```

Use authorized lab addresses.

## Project 2: Python Port Monitor

**Objective:** Identify listening ports and processes on your own machine.

Useful commands:

```bash
ss -lntup
```

```bash
lsof -i -P -n
```

Python modules:

- `subprocess`
- `socket`
- `json`
- `datetime`

Features:

- Record open listening ports.
- Identify process information.
- Save results.
- Compare results between monitoring sessions.

## Project 3: Python HTTP Proxy

**Objective:** Build your own HTTP proxy.

Modules:

- `socket`
- `threading`
- `select`
- `ssl`
- `logging`

Features:

- Accept browser connections.
- Parse HTTP requests.
- Forward requests.
- Receive responses.
- Return responses to the browser.
- Log request methods and destination hosts.

Start with HTTP, then add HTTPS CONNECT tunnelling.

Do not assume that CONNECT tunnelling provides decrypted HTTPS inspection.

## Project 4: Network Packet Analyzer

**Objective:** Understand packet structure using Scapy.

Modules:

- `scapy`
- `socket`
- `struct`

Inspect:

- Ethernet.
- IPv4.
- IPv6.
- TCP.
- UDP.
- ICMP.
- ARP.

Example:

```python
from scapy.all import sniff

def process_packet(packet):
    print(packet.summary())

sniff(
    iface="eth0",
    prn=process_packet,
    count=10,
    store=False
)
```

Run only on an interface and network you are authorized to monitor.

## Project 5: Firewall Audit Tool

**Objective:** Inspect firewall configuration and report possible weaknesses.

Linux commands:

```bash
sudo ufw status verbose
```

```bash
sudo nft list ruleset
```

```bash
sudo iptables -L -n
```

Windows:

```powershell
Get-NetFirewallProfile
```

Features:

- Read firewall configuration.
- Identify enabled rules.
- Identify unnecessary exposed ports.
- Generate a report.
- Recommend restrictions.

## Project 6: SSH Security Auditor

**Objective:** Inspect SSH server configuration.

Configuration:

```text
/etc/ssh/sshd_config
```

Inspect:

```bash
sudo sshd -T
```

Check:

- Root login.
- Password authentication.
- Public-key authentication.
- Listening port.
- Allowed users.
- Authentication limits.

Generate a security report.

---

# 22. IMPORTANT PORTS REFERENCE

| Port | Protocol/service | Common purpose |
|---:|---|---|
| 20 | TCP/FTP | FTP data |
| 21 | TCP/FTP | FTP control |
| 22 | TCP/SSH | Secure Shell |
| 23 | TCP/Telnet | Remote terminal |
| 25 | TCP/SMTP | Mail transmission |
| 53 | TCP/UDP/DNS | Name resolution |
| 67 | UDP/DHCP | DHCP server |
| 68 | UDP/DHCP | DHCP client |
| 69 | UDP/TFTP | Trivial FTP |
| 80 | TCP/HTTP | Web traffic |
| 110 | TCP/POP3 | Email retrieval |
| 123 | UDP/NTP | Time synchronization |
| 143 | TCP/IMAP | Email retrieval |
| 161 | UDP/SNMP | Network management |
| 162 | UDP/SNMP | Traps and notifications |
| 389 | TCP/UDP/LDAP | Directory services |
| 443 | TCP/HTTPS | Secure web traffic |
| 445 | TCP/SMB | Windows file sharing |
| 465 | TCP/SMTPS | SMTP over implicit TLS |
| 514 | UDP/Syslog | Logging |
| 587 | TCP/SMTP | Mail submission |
| 636 | TCP/LDAPS | LDAP over TLS |
| 993 | TCP/IMAPS | IMAP over TLS |
| 995 | TCP/POP3S | POP3 over TLS |
| 1433 | TCP/MSSQL | Microsoft SQL Server |
| 3306 | TCP/MySQL | MySQL |
| 3389 | TCP/UDP/RDP | Remote Desktop |
| 5432 | TCP/PostgreSQL | PostgreSQL |
| 5900 | TCP/VNC | Remote desktop |
| 6379 | TCP/Redis | Redis |
| 8080 | TCP/HTTP | Alternative HTTP |
| 8443 | TCP/HTTPS | Alternative HTTPS |

A port number does not guarantee which application is running there. Services can use custom ports.

---

# 23. USEFUL COMMANDS CHEAT SHEET

## Network discovery

```bash
nmap -sn 192.168.1.0/24
nmap -p 22,80,443 192.168.1.10
nmap -sV 192.168.1.10
nmap -O 192.168.1.10
```

## Network configuration

```bash
ip addr
ip link
ip route
ip neigh
```

## Connection monitoring

```bash
ss -lntup
ss -ant
lsof -i -P -n
fuser -n tcp 8080
```

## Traffic monitoring

```bash
sudo tcpdump -i eth0
sudo tcpdump -i eth0 port 443
sudo tcpdump -i eth0 -w capture.pcap
```

## Connectivity

```bash
ping 8.8.8.8
traceroute example.com
mtr example.com
curl -I https://example.com
```

## DNS

```bash
dig example.com
dig +short example.com
nslookup example.com
```

## Firewall

```bash
sudo ufw status verbose
sudo ufw allow 22/tcp
sudo ufw deny 23/tcp
sudo nft list ruleset
sudo iptables -L -n
```

## SSH

```bash
sudo systemctl status ssh
sudo sshd -t
sudo sshd -T
sudo ss -lntp
ssh username@192.168.1.120
ssh -p 2222 username@192.168.1.120
ssh-keygen -t ed25519
ssh-add -l
```

## Windows

```powershell
ipconfig /all
netstat -ano
arp -a
route print
Get-NetTCPConnection
Get-NetUDPEndpoint
Get-NetFirewallProfile
```

---

# 24. OFFICIAL DOCUMENTATION AND FURTHER LEARNING

Use official documentation when you need the complete list of supported options and their exact behaviour.

1. Nmap Reference Guide:  
   https://nmap.org/book/man.html

2. Nmap Options Summary:  
   https://nmap.org/book/man-briefoptions.html

3. Wireshark Documentation:  
   https://www.wireshark.org/docs/

4. Tcpdump Manual:  
   https://www.tcpdump.org/manpages/tcpdump.1.html

5. OpenSSH Server Configuration:  
   https://man.openbsd.org/sshd_config

6. OpenSSH Client Manual:  
   https://man.openbsd.org/ssh

7. Ubuntu Firewall Documentation:  
   https://documentation.ubuntu.com/security/security-features/network/firewall/

8. nftables Documentation:  
   https://wiki.nftables.org/

9. Linux iproute2 Manual:  
   https://man7.org/linux/man-pages/man8/ip.8.html

10. Linux ss Manual:  
    https://man7.org/linux/man-pages/man8/ss.8.html

---

# FINAL LEARNING ADVICE

Do not try to memorize all these commands at once.

Understand what each tool is responsible for:

| Tool | Main responsibility |
|---|---|
| Nmap | Discover hosts, ports and services. |
| Netcat | Test and communicate over network connections. |
| Socat | Connect and transfer data between different endpoints. |
| Nping | Generate network probes. |
| Wireshark | Graphically inspect captured packets. |
| TShark | Analyse packets from the terminal. |
| Tcpdump | Capture and filter packets. |
| ss | Inspect sockets and network connections. |
| lsof | Identify processes using network sockets. |
| Nethogs | Monitor bandwidth by process. |
| iftop | Monitor bandwidth by connection. |
| MTR | Investigate network paths, latency and loss. |
| UFW | Manage Linux firewall rules simply. |
| iptables | Configure Netfilter using traditional rule syntax. |
| nftables | Configure modern Linux packet filtering. |
| SSH | Securely administer remote machines. |
| ACL | Control access to network or filesystem resources. |
| nmtui | Configure NetworkManager interactively. |
| nmcli | Configure NetworkManager through commands. |

**The most important principle:** Learn to identify a network event, understand which layer it belongs to, identify the responsible process or device, inspect the relevant traffic, and then decide whether the behaviour is legitimate or needs to be restricted.

That is the foundation of practical network administration and network security.

**END OF DOCUMENT**
