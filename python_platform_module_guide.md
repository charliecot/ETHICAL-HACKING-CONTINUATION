# Python `platform` Module — Complete Guide

## 1. What is the `platform` module?

Python's built-in `platform` module provides information about the computer and operating system on which Python is running.

You can use it to discover:

- Operating system name (Windows, Linux, macOS).
- OS release and version.
- Computer or machine name.
- CPU / machine architecture (such as `AMD64` or `x86_64`).
- Processor information when available.
- Python implementation and version.
- Platform-specific details.

It is part of Python's standard library, so **you do not need to install it with pip**.

```python
import platform
```

## 2. Quick machine-information program

```python
import platform

print("Machine name:", platform.node())
print("Operating system:", platform.system())
print("OS release:", platform.release())
print("OS version:", platform.version())
print("Machine architecture:", platform.machine())
print("Processor:", platform.processor())
print("Python version:", platform.python_version())
print("Python implementation:", platform.python_implementation())
```

Example output (your values will differ):

```text
Machine name: DESKTOP-ABC123
Operating system: Windows
OS release: 11
OS version: 10.0.26200
Machine architecture: AMD64
Processor: ...
Python version: 3.x.x
Python implementation: CPython
```

Some values depend on the operating system and Python build. For example, `platform.processor()` can return an empty string on some systems.

## 3. Important functions

### `platform.system()`

Returns the operating-system name.

```python
print(platform.system())
```

Typical values:

- `Windows`
- `Linux`
- `Darwin` (macOS)

Use it when your program needs to make an OS-specific decision.

```python
if platform.system() == "Windows":
    print("Running on Windows")
elif platform.system() == "Linux":
    print("Running on Linux")
elif platform.system() == "Darwin":
    print("Running on macOS")
else:
    print("Unknown operating system")
```

### `platform.release()`

Returns the operating-system release.

```python
print(platform.release())
```

Examples may include `11`, `10`, or a Linux kernel release such as `6.8.0`.

Do not assume that the release string alone tells you every detail about the OS.

### `platform.version()`

Returns a system-specific version string.

```python
print(platform.version())
```

On Windows, this can contain a longer Windows build/version string. On other systems, its format differs.

### `platform.node()`

Returns the network name of the machine (usually its hostname).

```python
computer_name = platform.node()
print(computer_name)
```

This is useful for logging which computer ran a script. It is not an IP address.

### `platform.machine()`

Returns the machine hardware architecture identifier.

```python
print(platform.machine())
```

Common examples:

- `AMD64`: commonly 64-bit x86 architecture on Windows.
- `x86_64`: 64-bit x86 architecture on Linux/macOS.
- `i386` / `i686`: 32-bit x86 identifiers.
- `aarch64`: 64-bit ARM architecture.

This describes the machine architecture as reported by the platform, not necessarily every detail of the CPU.

### `platform.processor()`

Returns a processor identifier where the OS can provide one.

```python
processor = platform.processor()
print(processor or "Processor information not available")
```

It may be blank or generic. If you need detailed CPU information, use an operating-system-specific command or a third-party library such as `psutil`.

### `platform.architecture()`

Returns information about the bit architecture of the executable (usually the Python interpreter).

```python
bits, linkage = platform.architecture()

print("Python executable architecture:", bits)
print("Linkage:", linkage)
```

Example:

```text
Python executable architecture: 64bit
Linkage: WindowsPE
```

Important: this describes the Python executable, not a complete hardware inventory.

### `platform.python_version()`

Returns the Python version as a string.

```python
print(platform.python_version())
```

Example: `3.15.1`.

### `platform.python_version_tuple()`

Returns the Python version as a tuple of strings.

```python
major, minor, micro = platform.python_version_tuple()

print("Major:", major)
print("Minor:", minor)
print("Micro:", micro)
```

For version `3.15.1`, the values are `"3"`, `"15"`, and `"1"`.

### `platform.python_implementation()`

Returns the Python implementation.

```python
print(platform.python_implementation())
```

Common result: `CPython`.

Other implementations include PyPy. This is different from the Python version number.

### `platform.uname()`

Returns a named tuple containing several system details.

```python
details = platform.uname()

print("System:", details.system)
print("Node:", details.node)
print("Release:", details.release)
print("Version:", details.version)
print("Machine:", details.machine)
print("Processor:", details.processor)
```

You can also print the entire result:

```python
print(platform.uname())
```

This is a convenient way to collect several fields in one call.

### `platform.platform()`

Returns a combined, human-readable platform description.

```python
print(platform.platform())
```

It combines several platform details. Its exact format varies by OS and Python version, so do not parse it as if it had a fixed format.

### `platform.machine()` versus `platform.architecture()`

These functions answer related but different questions:

- `machine()` reports the machine architecture identifier.
- `architecture()` reports the bit architecture and linkage of the Python executable.

For example, a 64-bit Windows computer may report `AMD64` from `machine()` and `64bit` from `architecture()`.

## 4. Understanding the difference between machine name, IP address, and interface

These are not the same thing:

| Term | Meaning | Python tool |
|---|---|---|
| Hostname / machine name | Name assigned to the computer | `platform.node()` or `socket.gethostname()` |
| IP address | Network address assigned to an interface | `socket`, `psutil`, or OS commands |
| Network interface | Network adapter such as Wi-Fi or Ethernet | `psutil` or Scapy |
| MAC address | Link-layer address associated with an adapter | `psutil` or Scapy |
| Architecture | CPU/platform architecture identifier | `platform.machine()` |

The `platform` module does **not** provide a complete list of network interfaces, IP addresses, or MAC addresses.

## 5. Collect system information into a dictionary

A dictionary is useful when you want to store, log, or serialize the information.

```python
import platform

machine_info = {
    "hostname": platform.node(),
    "system": platform.system(),
    "release": platform.release(),
    "version": platform.version(),
    "machine": platform.machine(),
    "processor": platform.processor(),
    "python_version": platform.python_version(),
    "python_implementation": platform.python_implementation(),
}

for key, value in machine_info.items():
    print(f"{key}: {value}")
```

You can export the dictionary as JSON:

```python
import json
import platform

machine_info = {
    "hostname": platform.node(),
    "system": platform.system(),
    "release": platform.release(),
    "machine": platform.machine(),
    "python_version": platform.python_version(),
}

with open("machine_info.json", "w", encoding="utf-8") as file:
    json.dump(machine_info, file, indent=4)
```

## 6. Use platform information to choose OS-specific behavior

For example, a script may need to use a different command on Windows and Linux.

```python
import platform
import subprocess

system = platform.system()

if system == "Windows":
    command = ["ipconfig"]
elif system == "Linux":
    command = ["ip", "address"]
elif system == "Darwin":
    command = ["ifconfig"]
else:
    raise RuntimeError(f"Unsupported operating system: {system}")

subprocess.run(command, check=False)
```

Security and reliability notes:

- Prefer a list of command arguments instead of building a shell command string.
- Avoid `shell=True` unless you have a specific, safe reason.
- Do not assume a command exists on every system.
- Handle `FileNotFoundError` if the command may be unavailable.

## 7. Common mistakes

### Mistake 1: Thinking `platform.node()` returns an IP address

```python
print(platform.node())
```

This returns the hostname, not an IP address.

### Mistake 2: Thinking `platform.processor()` always returns a detailed CPU model

It may return an empty or generic value. Do not make your program depend on a detailed CPU name being present.

### Mistake 3: Confusing Python architecture with hardware architecture

`platform.architecture()` describes the Python executable's architecture. `platform.machine()` reports the machine architecture identifier.

### Mistake 4: Trying to install the module

Do not run:

```powershell
py -m pip install platform
```

`platform` is already included in Python's standard library.

## 8. A reusable function

```python
import platform

def get_machine_info() -> dict[str, str]:
    """Return basic information about the current machine."""

    return {
        "hostname": platform.node(),
        "operating_system": platform.system(),
        "os_release": platform.release(),
        "os_version": platform.version(),
        "architecture": platform.machine(),
        "processor": platform.processor(),
        "python_version": platform.python_version(),
        "python_implementation": platform.python_implementation(),
    }


if __name__ == "__main__":
    for name, value in get_machine_info().items():
        print(f"{name}: {value or 'Not available'}")
```

The return annotation `dict[str, str]` says that the function returns a dictionary whose keys and values are strings. The `or 'Not available'` expression handles empty strings.

## 9. When is this module useful?

- Recording which machine ran a program.
- Writing cross-platform Python scripts.
- Collecting basic diagnostic information.
- Logging OS and Python versions when troubleshooting.
- Choosing platform-specific behavior.
- Checking whether a program is running on Windows, Linux, or macOS.

Do not use it as a replacement for a full hardware inventory, network-interface discovery, or security auditing tool.

## 10. Official documentation

- Python documentation: https://docs.python.org/3/library/platform.html
