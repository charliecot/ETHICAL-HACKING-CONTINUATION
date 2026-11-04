# Scapy Network Interfaces — Complete Guide

## 1. What is a network interface?

A **network interface** is the connection point that allows your computer to communicate over a network.

Examples include:

- Wi-Fi adapter.
- Ethernet adapter.
- Loopback interface.
- VPN adapter.
- Virtual machine adapter.
- Virtual or host-only network adapter.

A computer can have several interfaces at the same time. Each may have its own:

- Interface name.
- Description.
- MAC address.
- IPv4 address.
- IPv6 address.
- Network status and routing role.

When using Scapy, you may need to select the correct interface to send or capture packets.

## 2. What is Scapy?

Scapy is a Python packet-manipulation library. It can create, send, receive, decode, and inspect network packets.

Import it:

```python
import scapy.all as scapy
```

Scapy is not part of Python's standard library. Install it in your active environment if it is not already installed:

```powershell
py -m pip install scapy
```

On Windows, packet capture and transmission commonly require **Npcap**. Install Npcap from its official website if needed, and follow its installation instructions.

## 3. Display all interfaces known to Scapy

The simplest way is:

```python
import scapy.all as scapy

scapy.show_interfaces()
```

This prints the interfaces Scapy can see, usually including information such as an index, interface name, MAC address, IPv4 address, and description.

The exact display depends on your operating system and Scapy version.

You can also inspect the interface collection:

```python
scapy.conf.ifaces.show()
```

Use these when you are unsure which interface name Scapy expects.

## 4. Get interface names

```python
import scapy.all as scapy

interfaces = scapy.get_if_list()

for interface in interfaces:
    print(interface)
```

`get_if_list()` returns a list of interface identifiers known to Scapy.

Example identifiers may resemble:

```text
lo
eth0
wlan0
Wi-Fi
Ethernet
```

On Windows, Scapy may display Npcap interface identifiers that look like GUIDs. Do not assume the name will always be simply `Wi-Fi` or `Ethernet`.

## 5. Get the default interface

```python
import scapy.all as scapy

default_interface = scapy.conf.iface

print("Default interface:", default_interface)
```

`scapy.conf.iface` is Scapy's configured default interface. It may be suitable for ordinary traffic, but it is not guaranteed to be the interface you want for every target or network.

Check the interface list before choosing one.

## 6. Inspect interface details

Scapy stores interface information in its interface collection.

```python
import scapy.all as scapy

scapy.show_interfaces()

for interface in scapy.conf.ifaces.values():
    print("Name:", interface.name)
    print("Description:", interface.description)
    print("Index:", interface.index)
    print("MAC:", interface.mac)
    print("IPv4:", interface.ip)
    print("-" * 40)
```

The attributes available can vary with Scapy version and operating system. If a particular attribute is unavailable, inspect the interface object:

```python
print(interface)
print(interface.__dict__)
```

You can also inspect Scapy's interface collection directly:

```python
print(scapy.conf.ifaces)
```

## 7. Choose an interface

After viewing the interface list, choose the interface that corresponds to the network you intend to use.

For example:

```python
import scapy.all as scapy

scapy.show_interfaces()

selected_interface = scapy.conf.iface

print("Selected interface:", selected_interface)
```

You can also choose an exact name shown by Scapy:

```python
selected_interface = "Wi-Fi"  # Example only: use your actual Scapy interface name.
```

Do not copy the example name blindly. On your computer it may be called something else.

## 8. Use the interface when sending packets

Scapy's send/receive functions use the parameter **`iface`** to select an interface.

### `sr1()` — send and receive one response

```python
import scapy.all as scapy

target_ip = "192.168.1.1"
selected_interface = scapy.conf.iface

packet = scapy.IP(dst=target_ip) / scapy.ICMP()

response = scapy.sr1(
    packet,
    iface=selected_interface,
    timeout=2,
    verbose=False
)

if response is None:
    print("No response received")
else:
    response.show()
```

Important:

- `sr1()` sends a Layer 3 packet and returns one matching response packet, or `None` if no response is received before timeout.
- `iface=` selects the interface.
- `timeout=2` waits up to two seconds.
- `verbose=False` suppresses Scapy's progress output.

### `sr()` — send and collect answered and unanswered packets

```python
answered, unanswered = scapy.sr(
    packet,
    iface=selected_interface,
    timeout=2,
    verbose=False
)

print("Answered:", answered.summary())
print("Unanswered:", unanswered.summary())
```

The variable names `answered` and `unanswered` are chosen by you. `sr()` returns two collections:

- Answered: sent packets for which Scapy received a matching response.
- Unanswered: sent packets for which no matching response was received.

### `send()` — send at Layer 3 without waiting for a response

```python
scapy.send(
    scapy.IP(dst="192.168.1.1") / scapy.ICMP(),
    iface=selected_interface,
    verbose=False
)
```

`send()` does not return a response packet.

### `sendp()` — send at Layer 2

```python
scapy.sendp(
    scapy.Ether() / scapy.ARP(pdst="192.168.1.1"),
    iface=selected_interface,
    verbose=False
)
```

`sendp()` sends a packet at Layer 2. The packet normally includes an Ethernet layer when sending on Ethernet-style networks.

### `srp1()` — Layer 2 send and receive one response

```python
packet = (
    scapy.Ether()
    / scapy.ARP(pdst="192.168.1.1")
)

response = scapy.srp1(
    packet,
    iface=selected_interface,
    timeout=2,
    verbose=False
)

if response:
    response.show()
else:
    print("No ARP response")
```

## 9. `iface` versus `interface`

For Scapy's common packet send/receive functions, the documented parameter is normally named `iface`, for example:

```python
scapy.sr1(packet, iface=selected_interface)
```

Do not assume this is equivalent:

```python
scapy.sr1(packet, interface=selected_interface)
```

The latter may raise an unexpected-keyword error. Use `iface=` for functions such as `sr1()`, `sr()`, `send()`, `srp()`, and `sendp()`.

## 10. Get interface information with `psutil` (optional)

Scapy is useful for packet work. `psutil` is another library that can list operating-system network interface addresses.

Install it:

```powershell
py -m pip install psutil
```

Then:

```python
import psutil
import socket

interfaces = psutil.net_if_addrs()

for interface_name, addresses in interfaces.items():
    print(f"\nInterface: {interface_name}")

    for address in addresses:
        if address.family == socket.AF_INET:
            print("IPv4:", address.address)
            print("Netmask:", address.netmask)
            print("Broadcast:", address.broadcast)

        elif address.family == socket.AF_INET6:
            print("IPv6:", address.address)

        elif address.family == psutil.AF_LINK:
            print("MAC:", address.address)
```

`psutil.net_if_addrs()` returns a dictionary:

- Keys: interface names.
- Values: lists of address records.

`socket.AF_INET` identifies IPv4 addresses. `socket.AF_INET6` identifies IPv6 addresses. `psutil.AF_LINK` identifies link-layer addresses such as MAC addresses on supported platforms.

To inspect interface status:

```python
for name, stats in psutil.net_if_stats().items():
    print(name)
    print("Is up:", stats.isup)
    print("Speed (Mbps):", stats.speed)
    print("MTU:", stats.mtu)
```

`isup` indicates whether the interface is reported as up. It does not guarantee internet access.

## 11. Windows commands for checking interfaces

You can also use Windows commands outside Python.

### PowerShell

```powershell
Get-NetAdapter
```

Shows network adapters and their status.

```powershell
Get-NetIPConfiguration
```

Shows IP configuration for network interfaces.

```powershell
ipconfig /all
```

Shows detailed IP configuration, including adapter descriptions, physical addresses, IPv4/IPv6 addresses, and gateways.

These commands help you match the Windows adapter to the interface shown by Scapy.

## 12. A reusable function to display Scapy interfaces

```python
import scapy.all as scapy

def show_scapy_interfaces() -> None:
    """Display interfaces available to Scapy."""

    print("Default interface:", scapy.conf.iface)
    print("\nAvailable interfaces:")
    scapy.show_interfaces()


if __name__ == "__main__":
    show_scapy_interfaces()
```

## 13. A reusable function to select an interface

```python
import scapy.all as scapy

def choose_interface(interface_name: str):
    """Return a Scapy interface matching the supplied name."""

    for interface in scapy.conf.ifaces.values():
        if interface.name == interface_name:
            return interface

    raise ValueError(f"Interface not found: {interface_name}")


scapy.show_interfaces()

# Replace this with an exact name shown by Scapy.
selected_interface = choose_interface("Wi-Fi")

print("Selected:", selected_interface)
```

Some Scapy versions or operating systems may identify interfaces differently. If matching by `.name` does not work, inspect the output of `show_interfaces()` and the interface objects, then use the exact identifier Scapy reports.

## 14. Common problems on Windows

### Problem: Scapy cannot send packets

Possible causes:

- The terminal is not running with sufficient privileges.
- Npcap is not installed correctly.
- The Npcap service is not running.
- The selected interface is incorrect.
- A firewall or security product blocks the traffic.
- The packet type or Windows networking mode does not support the requested operation.

What to check:

1. Confirm Npcap is installed from its official source.
2. Check that the Npcap service is running.
3. Try an elevated PowerShell or terminal when required.
4. Run `scapy.show_interfaces()` and select the correct adapter.
5. Test with a simple packet on a network you own or are authorized to test.

### Problem: `OSError` says Windows native Layer 3 raw sockets are only usable as administrator

Windows has restrictions around raw sockets. Scapy on Windows commonly relies on Npcap for packet operations.

Check Npcap installation and permissions. Depending on the Scapy version and operation, you may need to configure or use its Npcap-backed capture/send support. Do not assume that changing the interface name alone will solve a raw-socket problem.

### Problem: No response is received

A timeout does not prove that a host is offline or a port is closed.

Possible reasons:

- The target is offline.
- The target or a firewall drops the packet.
- The wrong interface or route was selected.
- The response is blocked on its way back.
- The timeout is too short.
- The target does not respond to that packet type.

Check routing and test with a permitted target.

## 15. Important networking concepts

### Interface versus IP address

An interface is the adapter/connection point. An IP address is a network-layer address configured on that interface.

One interface can have multiple IP addresses. An IP address may also change over time.

### Interface versus default gateway

The interface is your adapter. The default gateway is the router next hop used for traffic destined outside the local network.

### Interface versus MAC address

The interface is the adapter. The MAC address is a link-layer identifier associated with it. A device can have multiple MAC addresses because it can have multiple interfaces.

### Layer 2 versus Layer 3

- Layer 2: Ethernet frames and MAC addresses. Scapy functions include `sendp()`, `srp()`, and `srp1()`.
- Layer 3: IP packets and IP addresses. Scapy functions include `send()`, `sr()`, and `sr1()`.

The interface still matters at both layers, but the packet structure and send/receive function differ.

## 16. Practical workflow for your Scapy projects

1. Display all Scapy interfaces with `scapy.show_interfaces()`.
2. Identify the adapter that connects to your intended network.
3. Compare it with `Get-NetAdapter` or `ipconfig /all` on Windows.
4. Select the exact interface identifier reported by Scapy.
5. Pass it using `iface=selected_interface`.
6. Start with a simple ICMP or ARP test against a device you own or are authorized to test.
7. Inspect the response and handle the possibility that no response arrives.

## 17. Quick reference

| Goal | Scapy expression |
|---|---|
| Display interfaces | `scapy.show_interfaces()` |
| Get interface names | `scapy.get_if_list()` |
| Display interface collection | `scapy.conf.ifaces.show()` |
| Get default interface | `scapy.conf.iface` |
| Send Layer 3 packet | `scapy.send(packet, iface=...)` |
| Send and receive at Layer 3 | `scapy.sr(packet, iface=...)` |
| Receive one Layer 3 response | `scapy.sr1(packet, iface=...)` |
| Send Layer 2 frame | `scapy.sendp(frame, iface=...)` |
| Send and receive at Layer 2 | `scapy.srp(frame, iface=...)` |
| Receive one Layer 2 response | `scapy.srp1(frame, iface=...)` |

## 18. Official documentation

- Scapy documentation: https://scapy.readthedocs.io/
- Scapy usage guide: https://scapy.readthedocs.io/en/latest/usage.html
- Scapy API reference: https://scapy.readthedocs.io/en/latest/api/
- Npcap: https://npcap.com/
- psutil documentation: https://psutil.readthedocs.io/
