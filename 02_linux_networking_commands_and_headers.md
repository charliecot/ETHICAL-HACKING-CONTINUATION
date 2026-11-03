# Linux Networking Commands, Ethernet Frame Header, and IPv4 Header

## 1. Understand the packet journey

```text
Application data
   ↓
TCP segment OR UDP datagram
   ↓
IP packet
   ↓
Ethernet frame
   ↓
Physical medium
```

On reception, the stack is processed in reverse. A router removes the incoming link-layer frame, examines/routes the IP packet, and builds a new link-layer frame for the next link. Therefore MAC addresses normally change at each routed hop; IP addresses generally remain end-to-end unless NAT/tunnelling or another transformation is used.

## 2. `ip` — the main Linux network command

`ip` is part of `iproute2`.

```bash
ip help
ip --help
ip -V
```

### Interfaces

```bash
ip link
ip link show
ip -br link
ip -s link
```

- `link`: Layer-2 interface properties.
- `show`: display current state.
- `-br`: brief/compact output.
- `-s`: interface statistics.

Example output concepts:

- `eth0`: interface name.
- `UP`: administratively enabled.
- `LOWER_UP`: link carrier is detected.
- `link/ether`: Ethernet MAC address.

### IP addresses

```bash
ip address
ip addr show
ip -br address
ip -4 address
ip -6 address
```

`-4` filters IPv4; `-6` filters IPv6; `-br` makes output compact.

### Routes

```bash
ip route
ip route show
ip -4 route
ip -6 route
ip route get 1.1.1.1
```

Typical default route:

```text
default via 192.168.1.1 dev wlan0
```

- `default`: route used if no more-specific route matches.
- `via`: next-hop gateway.
- `dev`: outgoing interface.

`ip route get DESTINATION` asks the kernel which route it would use.

### Neighbour cache

```bash
ip neigh
ip neigh show
ip -s neigh
```

IPv4 neighbour entries are commonly learned through ARP (IP-to-MAC mapping). IPv6 uses Neighbor Discovery Protocol (NDP), not ARP.

States include `REACHABLE`, `STALE`, `DELAY`, `PROBE`, `FAILED`, and `INCOMPLETE`.

### Policy routing

```bash
ip rule
ip rule show
ip route show table all
```

Rules can select routing tables based on source address, marks, or other selectors. This is useful for VPNs and multi-interface systems.

## 3. Older commands and replacements

| Older tool | Modern replacement |
|---|---|
| `ifconfig` | `ip addr`, `ip link` |
| `route -n` | `ip route` |
| `arp -a` | `ip neigh` |
| `netstat -tulnp` | `ss -tulnp` |
| `netstat -rn` | `ip route` |

Some older commands may need separate packages and may not be installed.

## 4. `ping` — ICMP reachability test

```bash
ping 192.168.1.1
ping -c 4 192.168.1.1
ping -i 1 192.168.1.1
ping -W 2 192.168.1.1
ping -w 10 192.168.1.1
ping -s 1200 192.168.1.1
ping -4 example.com
ping -6 example.com
ping -t 1 1.1.1.1
```

Common Linux `ping` options:

- `-c COUNT`: send COUNT requests, then stop.
- `-i SECONDS`: interval between requests.
- `-W SECONDS`: wait for a reply (semantics vary by implementation).
- `-w DEADLINE`: total deadline.
- `-s SIZE`: ICMP data payload size.
- `-4` / `-6`: force IPv4 / IPv6.
- `-t TTL`: set outgoing IPv4 TTL.

Ping uses ICMP Echo Request and Echo Reply. A failed ping does not prove a host is offline; ICMP may be filtered or rate-limited.

## 5. `traceroute` and `tracepath`

```bash
traceroute example.com
traceroute -n example.com
traceroute -I example.com
traceroute -T -p 443 example.com
traceroute -m 20 example.com
traceroute -q 1 example.com
traceroute -w 2 example.com
tracepath example.com
tracepath -n example.com
```

Common `traceroute` options (implementation can vary):

- `-n`: do not resolve hop addresses into names.
- `-I`: use ICMP Echo probes.
- `-T`: use TCP SYN probes.
- `-p PORT`: destination port for relevant probe types.
- `-m HOPS`: maximum hop count / TTL.
- `-q N`: probes per hop.
- `-w SECONDS`: response wait time.

Traditional Linux traceroute often uses UDP probes by default; Windows `tracert` uses ICMP Echo probes. A `*` means no response arrived before timeout, not necessarily that the hop is offline. `tracepath` can also report Path MTU information.

## 6. DNS tools

```bash
getent hosts example.com
getent ahosts example.com

dig example.com
dig A example.com
dig AAAA example.com
dig MX example.com
dig NS example.com
dig +short example.com
dig @1.1.1.1 example.com
dig -x 8.8.8.8

host example.com
host -t AAAA example.com
nslookup example.com
```

- A: IPv4 record.
- AAAA: IPv6 record.
- MX: mail exchanger.
- NS: name server.
- `+short`: concise output.
- `@1.1.1.1`: query a chosen DNS resolver.
- `-x`: reverse lookup.

`getent` uses the host's configured name-service mechanisms, not just DNS.

## 7. `ss` — sockets and ports

```bash
ss
ss -t
ss -u
ss -l
ss -n
ss -p
ss -a
ss -s

ss -tuln
ss -tulnp
ss -tan
ss -uan
ss -ltn
```

| Option | Meaning |
|---|---|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | Listening sockets |
| `-n` | Numeric addresses/ports; no name resolution |
| `-p` | Process information, when permitted |
| `-a` | All sockets |
| `-s` | Summary |

`ss -tulnp` is a useful local troubleshooting command. Elevated privileges may be needed to see all process details.

### `lsof`

```bash
sudo lsof -i
sudo lsof -i :8000
sudo lsof -iTCP -sTCP:LISTEN
```

- `-i`: network sockets.
- `:8000`: filter port.
- `-iTCP`: TCP only.
- `-sTCP:LISTEN`: listening TCP sockets.

## 8. `tcpdump` — packet capture

```bash
sudo tcpdump -i eth0
sudo tcpdump -i any
sudo tcpdump -i eth0 -nn
sudo tcpdump -i eth0 -c 20
sudo tcpdump -i eth0 -w lab.pcap
sudo tcpdump -r lab.pcap
```

Options:

- `-i INTERFACE`: capture on interface (`any` is supported on Linux).
- `-n`: do not resolve addresses.
- `-nn`: do not resolve addresses or ports.
- `-c COUNT`: stop after COUNT packets.
- `-w FILE`: write capture.
- `-r FILE`: read capture.
- `-v`, `-vv`, `-vvv`: increase verbosity.
- `-e`: include link-layer header.
- `-X`: hex and ASCII payload.
- `-A`: ASCII payload.
- `-s LENGTH`: snapshot length.

Useful filters:

```bash
sudo tcpdump -i eth0 -nn icmp
sudo tcpdump -i eth0 -e -nn arp
sudo tcpdump -i eth0 -nn 'tcp port 443'
sudo tcpdump -i eth0 -nn 'udp port 53'
sudo tcpdump -i eth0 -nn 'host 192.168.1.10'
```

Packet captures can contain private data, tokens, cookies, and credentials. Capture only authorized traffic and protect the files.

## 9. Interface and Wi-Fi tools

```bash
sudo ethtool eth0
sudo ethtool -i eth0
sudo ethtool -S eth0

iw dev
iw dev wlan0 link
iw list

nmcli device status
nmcli connection show
nmcli device show
```

- `ethtool`: Ethernet link details.
- `ethtool -i`: driver information.
- `ethtool -S`: driver-specific counters.
- `iw dev`: wireless interfaces.
- `iw dev wlan0 link`: current Wi-Fi link details.
- `nmcli`: NetworkManager CLI.

## 10. Changing network configuration — caution

These can disconnect you, especially over SSH. Use a local console or disposable lab VM.

```bash
sudo ip link set dev eth0 up
sudo ip link set dev eth0 down
sudo ip address add 192.168.1.50/24 dev eth0
sudo ip address del 192.168.1.50/24 dev eth0
sudo ip route add default via 192.168.1.1
sudo ip route del default
```

These changes may be temporary and can be overwritten by DHCP, NetworkManager, systemd-networkd, or reboot. Persistent setup differs by distribution.

## 11. Ethernet II frame header

Simplified Ethernet II frame:

```text
+--------------------------+
| Destination MAC: 6 bytes |
+--------------------------+
| Source MAC:      6 bytes |
+--------------------------+
| EtherType:       2 bytes |
+--------------------------+
| Payload: usually 46–1500 |
+--------------------------+
| FCS:             4 bytes |
+--------------------------+
```

The Ethernet II header is 14 bytes:

| Field | Size | Meaning |
|---|---:|---|
| Destination MAC | 6 bytes | Receiver's local-link hardware address |
| Source MAC | 6 bytes | Sender's local-link hardware address |
| EtherType | 2 bytes | Identifies payload protocol |

The payload commonly contains an ARP message, IPv4 packet, or IPv6 packet. The FCS is a 4-byte CRC error-detection value, often stripped by NIC hardware and therefore absent from packet captures. Preamble and Start Frame Delimiter exist on the medium but normally are not shown as part of a captured frame.

Common EtherTypes:

| Value | Protocol |
|---|---|
| `0x0800` | IPv4 |
| `0x0806` | ARP |
| `0x86DD` | IPv6 |
| `0x8100` | 802.1Q VLAN tag TPID |

An 802.1Q VLAN tag adds 4 bytes between the source MAC and EtherType. Classic Ethernet payload MTU is commonly 1500 bytes; jumbo frames are a network-specific extension.

## 12. IPv4 header in detail

Minimum IPv4 header: 20 bytes. Options can make it longer.

```text
+--------+--------+----------------+----------------+
|Version | IHL    | DSCP / ECN     | Total Length   |
+-----------------+----------------+----------------+
| Identification  | Flags          | Fragment Offset|
+-----------------+----------------+----------------+
| TTL             | Protocol       | Header Checksum|
+---------------------------------------------------+
| Source IPv4 Address                               |
+---------------------------------------------------+
| Destination IPv4 Address                          |
+---------------------------------------------------+
| Options and padding (optional)                    |
+---------------------------------------------------+
```

| Field | Size | Meaning |
|---|---:|---|
| Version | 4 bits | IPv4 value is 4 |
| IHL | 4 bits | Header length in 32-bit words |
| DSCP | 6 bits | Traffic classification |
| ECN | 2 bits | Congestion notification |
| Total Length | 16 bits | Entire IPv4 packet: header + payload |
| Identification | 16 bits | Helps associate fragments |
| Flags | 3 bits | Includes DF (Don't Fragment), MF (More Fragments) |
| Fragment Offset | 13 bits | Fragment position in units of 8 bytes |
| TTL | 8 bits | Hop limit; routers decrement it |
| Protocol | 8 bits | Next protocol: ICMP=1, TCP=6, UDP=17 |
| Header Checksum | 16 bits | Checks IPv4 header only |
| Source | 32 bits | Sender IPv4 address |
| Destination | 32 bits | Destination IPv4 address |
| Options | Variable | Optional control fields |

### IHL calculation

IHL counts 32-bit words, not bytes:

- IHL 5 = 5 × 4 = 20 bytes (minimum).
- IHL 6 = 6 × 4 = 24 bytes.

### Total Length calculation

If Total Length is 1500 and header length is 20 bytes, payload length is 1480 bytes.

### Fragmentation

- DF: Don't Fragment.
- MF: More Fragments follow.
- Fragment Offset: where this fragment belongs in the original payload, measured in 8-byte units.

### TTL

TTL is not normally a timer in seconds. Each router forwarding an IPv4 packet decrements TTL. When it expires, the router discards the packet and may return ICMP Time Exceeded (Type 11). This prevents indefinite routing loops.

### Protocol values

| Number | Protocol |
|---:|---|
| 1 | ICMP |
| 6 | TCP |
| 17 | UDP |
| 41 | IPv6 encapsulation |
| 47 | GRE |
| 50 | ESP |
| 51 | AH |
| 89 | OSPF |

## 13. TCP and UDP headers at a glance

TCP minimum header is 20 bytes. It contains source/destination ports, sequence and acknowledgment numbers, data offset, flags, window, checksum, urgent pointer, and optional fields.

Common TCP flags: SYN (start), ACK (acknowledgment), FIN (orderly close), RST (reset), PSH (push), URG (urgent pointer).

UDP header is 8 bytes:

```text
+-------------------+-------------------+
| Source Port (2B)  | Destination (2B)  |
+-------------------+-------------------+
| Length (2B)       | Checksum (2B)     |
+-------------------+-------------------+
```

UDP does not provide TCP-style connection setup, retransmission, sequencing, or flow control.

## 14. Practice order

```bash
ip -br link
ip -br address
ip route
ip neigh
ping -c 4 127.0.0.1
ss -tuln
sudo tcpdump -i lo -nn icmp
```

Then capture ARP on your own interface:

```bash
sudo tcpdump -i <your-interface> -e -nn arp
```

Use `ip -br link` to discover the correct interface name. Do not assume it is always `eth0`.

## 15. Troubleshooting facts

- Failed ping does not prove a host is offline.
- A `*` in traceroute means no response arrived in time.
- A listening port is not automatically reachable from outside; firewall and routing also matter.
- `ip link set ... down` can disconnect you.
- Protect packet captures because they may contain sensitive information.
