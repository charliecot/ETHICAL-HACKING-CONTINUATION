# DNS RECORD TYPES — COMPLETE BEGINNER-TO-ADVANCED GUIDE

**Author:** Nzegge Charles Ngong

# 1. Introduction to DNS Records

DNS records are entries stored in DNS zones that provide information about domain names and network services.

For example:

```text
example.com.    3600    IN    A    192.0.2.10
```

This is a DNS record.

Let's understand its structure:

| Part | Meaning |
|---|---|
| `example.com.` | Domain name (owner name) |
| `3600` | TTL in seconds |
| `IN` | Internet DNS class |
| `A` | DNS record type |
| `192.0.2.10` | Record data (IPv4 address) |

The general structure is:

```text
NAME    TTL    CLASS    TYPE    VALUE
```

Not every DNS tool displays all these fields in exactly this format.

---

# 2. A Record — Address Record

The A record maps a domain name or hostname to an IPv4 address.

IPv4 addresses use 32 bits.

Example:

```text
example.com.    IN    A    192.0.2.10
```

This means:

```text
example.com → 192.0.2.10
```

When a computer requests the IPv4 address of `example.com`, DNS can return this A record.

## Example with multiple A records

```text
example.com.    IN    A    192.0.2.10
example.com.    IN    A    192.0.2.20
example.com.    IN    A    192.0.2.30
```

One hostname can have multiple A records.

This may be used for:

- Load distribution.
- Multiple server addresses.
- Redundancy.
- Different service infrastructure.

The DNS resolver may return multiple addresses. The client or other infrastructure determines which address to use.

## Query an A record

Windows:

```powershell
nslookup -type=A example.com
```

PowerShell:

```powershell
Resolve-DnsName example.com -Type A
```

Linux:

```bash
dig example.com A
```

---

# 3. AAAA Record — IPv6 Address Record

The AAAA record maps a domain name to an IPv6 address.

IPv6 addresses use 128 bits.

Example:

```text
example.com.    IN    AAAA    2001:db8::10
```

This means:

```text
example.com → 2001:db8::10
```

Notice the difference:

| Record | Address version | Address size |
|---|---|---:|
| A | IPv4 | 32 bits |
| AAAA | IPv6 | 128 bits |

Why is it called AAAA?

It is pronounced "quad A". It represents an IPv6 address record.

## Example

A domain can have both A and AAAA records:

```text
example.com.    IN    A       192.0.2.10

example.com.    IN    AAAA    2001:db8::10
```

This allows clients using IPv4 or IPv6 to discover an appropriate address.

## Query an AAAA record

Windows:

```powershell
nslookup -type=AAAA example.com
```

PowerShell:

```powershell
Resolve-DnsName example.com -Type AAAA
```

Linux:

```bash
dig example.com AAAA
```

---

# 4. CNAME Record — Canonical Name Record

CNAME means **Canonical Name**.

It makes one DNS name an alias of another DNS name.

Example:

```text
www.example.com.    IN    CNAME    example.com.
```

This means:

```text
www.example.com → example.com
```

The CNAME does not directly store an IPv4 or IPv6 address.

Instead, it tells the resolver that the queried name is an alias of another name.

The resolver then looks up the target name to obtain the required address.

## Example

Imagine these records:

```text
example.com.       IN    A        192.0.2.10

www.example.com.   IN    CNAME    example.com.
```

When someone requests:

```text
www.example.com
```

DNS follows the alias to:

```text
example.com
```

It can then obtain:

```text
192.0.2.10
```

## Important CNAME rules

1. A CNAME owner name generally cannot have other ordinary DNS data records alongside it.
2. You should not normally create an A record and a CNAME record for the same owner name.
3. A CNAME target must be a domain name, not an IP address.
4. A CNAME can point to another CNAME, although chains should generally be avoided because they add lookup steps.
5. A zone apex (for example, `example.com`) normally cannot be a CNAME because it must contain required zone records such as SOA and NS.

Some DNS providers offer ALIAS or ANAME features to work around the zone-apex limitation. These are provider-specific features, not standard DNS record types.

## Query a CNAME record

```powershell
nslookup -type=CNAME www.example.com
```

```bash
dig www.example.com CNAME
```

---

# 5. MX Record — Mail Exchange Record

MX means **Mail Exchange**.

It identifies the mail servers responsible for receiving email for a domain.

Suppose you send an email to:

```text
student@example.com
```

The sending mail server needs to discover where email for `example.com` should be delivered.

It queries the MX records.

Example:

```text
example.com.    IN    MX    10    mail.example.com.
```

Meaning:

```text
Domain: example.com
Priority: 10
Mail server: mail.example.com
```

## Understanding MX priority

MX records use preference values.

A lower number means a higher preference.

Example:

```text
example.com.    IN    MX    10    mail1.example.com.
example.com.    IN    MX    20    mail2.example.com.
example.com.    IN    MX    30    mail3.example.com.
```

The sending mail server normally tries:

1. `mail1.example.com`
2. `mail2.example.com`
3. `mail3.example.com`

If the preferred server is unavailable, it can try a less-preferred server.

## Important

An MX record identifies a mail server hostname, not the mail server's IP address directly.

The mail server hostname must itself be resolved using appropriate address records.

## Query MX records

```powershell
nslookup -type=MX example.com
```

```powershell
Resolve-DnsName example.com -Type MX
```

```bash
dig example.com MX
```

---

# 6. NS Record — Name Server Record

NS means **Name Server**.

It identifies the authoritative name servers for a DNS zone.

Example:

```text
example.com.    IN    NS    ns1.example-dns.com.

example.com.    IN    NS    ns2.example-dns.com.
```

This tells DNS which name servers are authoritative for the zone.

## Why are NS records important?

Remember the DNS hierarchy:

```text
ROOT
  |
  v
.COM TLD
  |
  v
EXAMPLE.COM AUTHORITATIVE SERVERS
```

The `.com` registry maintains delegation information identifying the authoritative name servers for `example.com`.

The authoritative zone also contains its own NS records.

This is how DNS resolvers discover where to ask for authoritative information.

## Query NS records

```powershell
nslookup -type=NS example.com
```

```powershell
Resolve-DnsName example.com -Type NS
```

```bash
dig example.com NS
```

---

# 7. TXT Record — Text Record

TXT records store text associated with a domain.

They are widely used for domain verification and email security policies.

Example:

```text
example.com.    IN    TXT    "some verification value"
```

The content is a text string. The meaning depends on the application using it.

## Common uses of TXT records

### A. SPF

SPF means Sender Policy Framework.

It specifies which mail servers are authorized to send email for a domain.

Example:

```text
example.com. IN TXT "v=spf1 ip4:192.0.2.10 -all"
```

Understanding the example:

- `v=spf1`: SPF version.
- `ip4:192.0.2.10`: Authorizes the specified IPv4 address.
- `-all`: Indicates that other sources should fail the SPF policy.

This is an illustrative policy, not a recommended configuration for an actual domain without understanding its mail infrastructure.

### B. DKIM

DKIM means DomainKeys Identified Mail.

It uses cryptographic signatures to help receiving mail servers verify that an email was authorized by the sending domain and that signed message content has not been altered.

A public DKIM key is published in DNS, commonly using a TXT record.

Example owner name:

```text
selector1._domainkey.example.com
```

### C. Domain verification

Services may ask you to create a TXT record to prove you control a domain.

Example:

```text
example.com. IN TXT "service-verification=abc123"
```

The service checks whether the expected value exists.

## Query TXT records

```powershell
nslookup -type=TXT example.com
```

```bash
dig example.com TXT
```

---

# 8. PTR Record — Pointer Record

PTR records are used for reverse DNS lookups.

Ordinary DNS resolution:

```text
Domain name → IP address
```

Reverse DNS:

```text
IP address → Domain name
```

Example:

```text
8.8.8.8 → dns.google
```

A PTR record stores the reverse mapping.

## How does reverse DNS work?

For IPv4, DNS uses a special domain:

```text
in-addr.arpa
```

The IPv4 address is reversed.

For example:

```text
8.8.8.8
```

Becomes:

```text
8.8.8.8.in-addr.arpa
```

For a real PTR query, the address is represented by the reversed octets:

```text
8.8.8.8
```

Example DNS owner name:

```text
8.8.8.8.in-addr.arpa.
```

A PTR record might look like:

```text
8.8.8.8.in-addr.arpa. IN PTR dns.google.
```

## IPv6 reverse DNS

IPv6 reverse DNS uses:

```text
ip6.arpa
```

It reverses individual hexadecimal nibbles of the IPv6 address.

## Why is PTR useful?

- Mail-server reputation and reverse DNS checks.
- Network troubleshooting.
- Identifying hostnames associated with IP addresses.
- Network administration.

Important: A PTR record does not prove ownership or guarantee that the hostname is the original website domain.

## Query PTR records

```powershell
nslookup 8.8.8.8
```

```bash
dig -x 8.8.8.8
```

---

# 9. SOA Record — Start of Authority

SOA means **Start of Authority**.

Every DNS zone has an SOA record.

It provides important administrative information about the zone.

Example:

```text
example.com. IN SOA ns1.example.com. admin.example.com. (
    2026110401
    3600
    1800
    1209600
    300
)
```

Let's understand the fields.

| Field | Example | Meaning |
|---|---:|---|
| MNAME | ns1.example.com. | Primary source name server |
| RNAME | admin.example.com. | Responsible party's email represented in DNS format |
| SERIAL | 2026110401 | Zone version number |
| REFRESH | 3600 | How often a secondary checks for updates |
| RETRY | 1800 | How long to wait before retrying a failed check |
| EXPIRE | 1209600 | How long secondary data may remain valid without successful refresh |
| MINIMUM | 300 | Negative caching TTL in modern SOA usage |

The RNAME field represents an email address.

For example:

```text
admin.example.com.
```

represents:

```text
admin@example.com
```

The first dot represents the `@` separator.

## Understanding the SERIAL

The serial number helps secondary DNS servers identify whether the zone has changed.

Example:

```text
2026110401
```

An administrator might use a date-based numbering convention.

The serial must be updated appropriately when zone data changes.

## Query SOA

```powershell
nslookup -type=SOA example.com
```

```bash
dig example.com SOA
```

---

# 10. SRV Record — Service Record

SRV records identify the hostname and port where a particular service can be found.

Example:

```text
_service._tcp.example.com. IN SRV 10 5 443 server.example.com.
```

The fields are:

```text
PRIORITY WEIGHT PORT TARGET
```

Understanding the example:

| Field | Value | Meaning |
|---|---:|---|
| Priority | 10 | Lower value is preferred |
| Weight | 5 | Helps distribute connections among equal-priority targets |
| Port | 443 | Service port |
| Target | server.example.com | Hostname providing the service |

The owner name includes:

```text
_service._tcp
```

This identifies the service and transport protocol.

For example:

```text
_sip._tcp.example.com
```

could identify a SIP service using TCP.

Common SRV record use cases include:

- VoIP services.
- Microsoft Active Directory.
- Service discovery.
- Messaging systems.

## Query SRV records

```powershell
nslookup -type=SRV _sip._tcp.example.com
```

```bash
dig _sip._tcp.example.com SRV
```

---

# 11. CAA Record — Certification Authority Authorization

CAA records specify which certificate authorities are permitted to issue TLS certificates for a domain.

Example:

```text
example.com. IN CAA 0 issue "letsencrypt.org"
```

This indicates that Let's Encrypt is authorized to issue certificates for the domain under the applicable CAA policy.

Common CAA tags:

| Tag | Purpose |
|---|---|
| issue | Authorizes a CA to issue normal certificates |
| issuewild | Authorizes a CA to issue wildcard certificates |
| iodef | Specifies a reporting destination for policy violations |

Example:

```text
example.com. IN CAA 0 issue "letsencrypt.org"
example.com. IN CAA 0 issuewild ";"
```

The second record indicates that wildcard certificate issuance is not authorized through that policy.

CAA helps reduce the risk of unauthorized certificate issuance.

## Query CAA records

```powershell
nslookup -type=CAA example.com
```

```bash
dig example.com CAA
```

---

# 12. NAPTR Record — Naming Authority Pointer

NAPTR records are used to rewrite or transform domain names into information about services.

They are particularly useful in service discovery and communication systems.

They can be used with:

- SIP.
- ENUM.
- VoIP systems.
- Service discovery.

Example:

```text
example.com. IN NAPTR 100 10 "U" "E2U+sip" "!^.*$!sip:user@example.com!" .
```

NAPTR fields include:

```text
ORDER PREFERENCE FLAGS SERVICE REGEXP REPLACEMENT
```

The regular expression can describe how to transform a name into another identifier.

NAPTR is more specialized than A, AAAA, MX, or CNAME.

---

# 13. DNSSEC-Related Records

DNSSEC adds cryptographic validation to DNS data.

Several DNS record types support DNSSEC.

## A. DNSKEY

Contains public keys used for DNSSEC validation.

Example:

```text
example.com. IN DNSKEY ...
```

## B. RRSIG

Contains a digital signature associated with a DNS record set.

It helps a resolver verify that the data is authentic and has not been modified.

## C. DS

Delegation Signer.

It stores a digest of a child zone's DNSKEY information in the parent zone.

This helps establish the chain of trust between DNS zones.

## D. NSEC

Provides authenticated information about the next existing name in a DNS zone and helps prove that a queried name or record does not exist.

## E. NSEC3

Provides a hashed form of authenticated denial-of-existence information.

It reduces some of the information exposure associated with ordinary NSEC, although it does not make zone enumeration impossible in all cases.

## F. CDS and CDNSKEY

These records support automated DNSSEC delegation maintenance.

---

# 14. Other DNS Record Types Worth Knowing

| Record | Purpose |
|---|---|
| TLSA | Associates TLS certificate information with a service using DANE |
| SSHFP | Publishes SSH public-key fingerprints |
| LOC | Stores geographic location information |
| HINFO | Stores host information, such as hardware and operating system descriptions |
| RP | Identifies the responsible person and mailbox for a domain or host |
| CERT | Stores certificate-related information |
| URI | Associates a domain name with a URI and priority/weight |
| SVCB | Provides service binding information |
| HTTPS | Provides HTTPS service binding information |

SVCB and HTTPS records are useful in modern service discovery and web connection optimization.

The HTTPS record type is not the same thing as the HTTPS protocol itself.

---

# 15. Complete DNS Record Comparison

| Record | Main purpose | Example data |
|---|---|---|
| A | IPv4 address | `192.0.2.10` |
| AAAA | IPv6 address | `2001:db8::10` |
| CNAME | Alias | `example.com.` |
| MX | Mail server | `10 mail.example.com.` |
| NS | Name server | `ns1.example.com.` |
| TXT | Text data | `"verification=abc123"` |
| PTR | Reverse DNS | `dns.google.` |
| SOA | Zone authority | Serial, timers and administrator |
| SRV | Service location | Priority, weight, port and target |
| CAA | Certificate issuance policy | `issue "letsencrypt.org"` |
| NAPTR | Name/service rewriting | Order, flags, service and rules |
| DNSKEY | DNSSEC public key | Public key data |
| RRSIG | DNSSEC signature | Digital signature |
| DS | DNSSEC delegation | Key digest |
| NSEC | Authenticated denial | Next existing name |
| NSEC3 | Hashed denial | Hashed name information |
| TLSA | DANE TLS association | Certificate association data |
| SSHFP | SSH fingerprint | Algorithm and fingerprint |
| SVCB | Service binding | Service parameters |
| HTTPS | HTTPS service binding | HTTPS parameters |

---

# 16. Practical DNS Record Exercises

Use these commands on Windows.

### A record

```powershell
nslookup -type=A google.com
```

### AAAA record

```powershell
nslookup -type=AAAA google.com
```

### MX record

```powershell
nslookup -type=MX gmail.com
```

### NS record

```powershell
nslookup -type=NS google.com
```

### TXT record

```powershell
nslookup -type=TXT google.com
```

### CNAME record

```powershell
nslookup -type=CNAME www.example.com
```

The domain may not have a CNAME at that exact hostname. A failed lookup does not mean DNS is broken.

### SOA record

```powershell
nslookup -type=SOA google.com
```

### SRV record

```powershell
nslookup -type=SRV _sip._tcp.example.com
```

### CAA record

```powershell
nslookup -type=CAA google.com
```

For more complete record-type support, use `dig`:

```bash
dig google.com A
dig google.com AAAA
dig google.com MX
dig google.com NS
dig google.com TXT
dig google.com SOA
dig google.com CAA
```

---

# 17. Important Differences to Remember

## A vs AAAA

- A returns an IPv4 address.
- AAAA returns an IPv6 address.

## CNAME vs A

- A stores an IPv4 address.
- CNAME points to another DNS name.

## MX vs NS

- MX identifies mail servers.
- NS identifies authoritative name servers.

## TXT vs SOA

- TXT stores text data for different applications.
- SOA stores important zone administration information.

## PTR vs A

- A maps a name to an IPv4 address.
- PTR maps an IP address to a name.

## NS vs SOA

- NS identifies authoritative name servers.
- SOA identifies important information about the DNS zone and its administration.

---

# 18. Final Learning Advice

Learn the following records first because you will encounter them frequently:

1. A
2. AAAA
3. CNAME
4. MX
5. NS
6. TXT
7. PTR
8. SOA
9. SRV
10. CAA

After understanding these, move to DNSSEC-related records such as DNSKEY, RRSIG, DS, NSEC, and NSEC3.

Remember:

**DNS records are not all designed to translate domain names into IP addresses. They provide many different kinds of information that make domain names, email, service discovery, and DNS security work.**
