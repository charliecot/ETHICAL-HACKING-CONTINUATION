# Python `ipaddress` Module — Complete Learning Guide

## 1. Purpose

Python's built-in `ipaddress` module validates and manipulates IPv4/IPv6 addresses, interfaces, networks, and subnets. It is part of the standard library; no `pip install` is normally needed.

```python
import ipaddress

ip = ipaddress.ip_address("192.168.1.10")
print(ip, ip.version)  # 192.168.1.10 4
```

## 2. Address constructors

```python
ipaddress.ip_address("192.168.1.10")       # Automatically detects IPv4/IPv6
ipaddress.IPv4Address("192.168.1.10")      # Explicit IPv4
ipaddress.IPv6Address("2001:db8::10")      # Explicit IPv6
```

IPv4 is 32 bits (4 octets); IPv6 is 128 bits (8 groups of 16 bits).

```python
v4 = ipaddress.IPv4Address("192.168.1.10")
v6 = ipaddress.IPv6Address("2001:0db8:0000:0000:0000:0000:0000:0001")

print(v4.version)          # 4
print(v4.max_prefixlen)    # 32
print(v4.packed)           # Four bytes
print(int(v4))             # Integer representation
print(v6.compressed)       # 2001:db8::1
print(v6.exploded)         # Full expanded IPv6 form
print(v6.packed)           # Sixteen bytes
```

## 3. Important address properties

```python
address = ipaddress.ip_address("127.0.0.1")

address.version
address.is_private
address.is_global
address.is_loopback
address.is_link_local
address.is_multicast
address.is_reserved
address.is_unspecified
address.packed
int(address)
str(address)
```

| Property | Meaning |
|---|---|
| `version` | 4 or 6 |
| `is_private` | Classified as private-use/non-global address space |
| `is_global` | Classified as globally reachable |
| `is_loopback` | Loopback, such as `127.0.0.1` or `::1` |
| `is_link_local` | Link-local address |
| `is_multicast` | Multicast address |
| `is_reserved` | Reserved address space |
| `is_unspecified` | `0.0.0.0` or `::` |
| `packed` | Address represented as bytes |

These properties classify address ranges; they do not prove that a host is safe or authorized.

## 4. Network objects and CIDR

CIDR notation appends a prefix length: `192.168.1.0/24`.

The prefix tells how many leading bits identify the network. IPv4 `/24` leaves 8 host bits; `/16` leaves 16; `/32` represents one address.

```python
network = ipaddress.ip_network("192.168.1.0/24")

print(network.network_address)    # 192.168.1.0
print(network.broadcast_address)  # 192.168.1.255
print(network.netmask)            # 255.255.255.0
print(network.hostmask)           # 0.0.0.255
print(network.prefixlen)          # 24
print(network.num_addresses)      # 256
```

Constructors:

```python
ipaddress.ip_network("192.168.1.0/24")
ipaddress.IPv4Network("192.168.1.0/24")
ipaddress.IPv6Network("2001:db8::/32")
```

### `strict=True` vs `strict=False`

```python
# Default strict=True rejects host bits set in the supplied network address:
# ipaddress.ip_network("192.168.1.10/24")  # ValueError

network = ipaddress.ip_network("192.168.1.10/24", strict=False)
print(network)  # 192.168.1.0/24
```

`strict=False` normalizes the supplied address/prefix to its containing network. Do not use it unknowingly when validating configuration.

## 5. Iterate addresses and hosts

```python
network = ipaddress.ip_network("192.168.1.0/29")

for address in network:
    print(address)        # Includes network and broadcast addresses

for host in network.hosts():
    print(host)           # Conventional usable IPv4 hosts
```

A `/29` has 8 total addresses; in an ordinary IPv4 subnet, 6 are conventional host addresses. `/31` and `/32` have special rules, so do not blindly apply the two-unusable-address rule to every prefix.

## 6. Check membership

```python
network = ipaddress.ip_network("10.10.0.0/16")
host = ipaddress.ip_address("10.10.20.30")

print(host in network)  # True
```

Useful for validating IPs in configuration files, logs, allowlists, and network policies.

## 7. Interface objects

An interface combines one host address with a prefix:

```python
interface = ipaddress.ip_interface("192.168.1.10/24")

print(interface.ip)              # 192.168.1.10
print(interface.network)         # 192.168.1.0/24
print(interface.with_prefixlen)
print(interface.with_netmask)
print(interface.with_hostmask)
```

Constructors: `ip_interface()`, `IPv4Interface()`, `IPv6Interface()`.

## 8. Subnetting and supernetting

Split `/24` into `/26` networks:

```python
network = ipaddress.ip_network("192.168.1.0/24")

for subnet in network.subnets(new_prefix=26):
    print(subnet)
```

There are four `/26` subnets. Alternatively, `network.subnets(prefixlen_diff=2)` increases the prefix by two bits.

Create a larger network:

```python
small = ipaddress.ip_network("192.168.1.0/24")
large = small.supernet(new_prefix=23)
print(large)  # 192.168.0.0/23
```

## 9. Compare, overlap, and summarize

```python
a = ipaddress.ip_network("192.168.1.0/25")
b = ipaddress.ip_network("192.168.1.128/25")
parent = ipaddress.ip_network("192.168.1.0/24")

print(a.subnet_of(parent))
print(parent.supernet_of(a))
print(a.overlaps(b))
```

Additional functions:

```python
ipaddress.collapse_addresses([a, b])
ipaddress.summarize_address_range(
    ipaddress.IPv4Address("192.168.1.0"),
    ipaddress.IPv4Address("192.168.1.255")
)
```

- `collapse_addresses()` combines adjacent/contained networks where possible.
- `summarize_address_range()` represents an address range as CIDR networks.

## 10. Validate user input

```python
import ipaddress

value = input("IP address: ")

try:
    address = ipaddress.ip_address(value)
except ValueError as error:
    print("Invalid IP:", error)
else:
    print("Valid:", address)
    print("Version:", address.version)
```

`ValueError` is raised for invalid address/network strings. Catch it when parsing untrusted input.

## 11. Practical log classifier

```python
import ipaddress

entries = ["127.0.0.1", "192.168.1.20", "8.8.8.8", "bad-value"]

for value in entries:
    try:
        address = ipaddress.ip_address(value)
    except ValueError:
        print(value, "is invalid")
        continue

    print(
        value,
        "IPv", address.version,
        "private:", address.is_private,
        "loopback:", address.is_loopback,
        "global:", address.is_global
    )
```

## 12. Common mistakes

1. `IPv4Address` represents one address; `IPv4Network` represents a range.
2. `192.168.1.10/24` is an interface-like address/prefix, not necessarily the canonical network address.
3. `strict=False` silently normalizes host bits.
4. Not every IPv4 prefix follows the ordinary network/broadcast/usable-host rule (`/31`, `/32`).
5. `is_private` is classification, not a security decision.
6. Do not manually split IP strings on dots to perform subnet calculations; use `ipaddress`.

## 13. Exercises

1. Create `10.20.30.40`; print version, packed bytes, and integer value.
2. Create `10.20.30.0/24`; print netmask, broadcast, and address count.
3. Test whether `10.20.30.99` belongs to that network.
4. Divide the network into `/26` subnets.
5. Print compressed and expanded forms of `2001:0db8::1`.
6. Write a function that returns whether a string is a valid IPv4 or IPv6 address.
7. Read addresses from a file and classify them.

Reference: https://docs.python.org/3/library/ipaddress.html
