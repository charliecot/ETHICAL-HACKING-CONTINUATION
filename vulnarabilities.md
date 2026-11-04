# CYBERSECURITY PRACTICAL GUIDE
## Vulnerability Scenarios, Python Testing, Exploitation Concepts and Prevention

**Author:** Nzegge Charles Ngong

**Learning level:** Beginner to Advanced  
**Environment:** Linux, Python, PHP, HTTP, networking and local security laboratories

---

# PART 1 — UNDERSTANDING VULNERABILITIES

## 1. What is a vulnerability?

A vulnerability is a weakness in software, hardware, network infrastructure, configuration, or a security process that can be abused to violate confidentiality, integrity, or availability.

For example:

Imagine a website has a login form.

The developer writes:

```python
username = request.form["username"]
password = request.form["password"]

query = f"""
SELECT * FROM users
WHERE username = '{username}'
AND password = '{password}'
"""
```

The problem is not the login form itself.

The problem is how the developer constructs the SQL query.

The user's input is inserted directly into the SQL instruction. This can allow SQL injection.

**Remember:** A vulnerability is usually caused by a weakness in design, implementation, configuration, or operation.

## 2. The three major security objectives

| Objective | Meaning | Example |
|---|---|---|
| Confidentiality | Prevent unauthorized information disclosure | Protect passwords and private files |
| Integrity | Prevent unauthorized modification | Prevent an attacker changing account balances |
| Availability | Keep systems accessible | Prevent resource exhaustion |

These are called the CIA triad:

- Confidentiality
- Integrity
- Availability

A vulnerability can affect one or more of these objectives.

## 3. The testing environment

Before testing vulnerabilities, create a laboratory.

Suggested structure:

```text
CYBERSECURITY-LAB/
│
├── file-upload/
├── sql-injection/
├── xss/
├── command-injection/
├── path-traversal/
├── authentication/
├── csrf/
├── ssrf/
├── dns-security/
├── deserialization/
└── reports/
```

Create it on Linux:

```bash
mkdir -p CYBERSECURITY-LAB
cd CYBERSECURITY-LAB

mkdir file-upload sql-injection xss
mkdir command-injection path-traversal authentication
mkdir csrf ssrf dns-security deserialization reports
```

Install useful Python libraries:

```bash
python3 -m venv venv
source venv/bin/activate

python3 -m pip install requests flask
```

Useful tools:

| Tool | Purpose |
|---|---|
| Python | Build security testing scripts |
| Requests | Send HTTP requests |
| Flask | Build small test applications |
| PHP | Test PHP web applications |
| curl | Send HTTP requests from Linux |
| Wireshark | Inspect network traffic |
| Scapy | Construct and inspect packets |
| Nmap | Authorized network discovery and service assessment |
| SQLite | Practise SQL injection concepts locally |
| Burp Suite | Inspect and test HTTP requests |
| OWASP ZAP | Web application security testing |

---

# PART 2 — WEB APPLICATION VULNERABILITIES

# 4. SQL INJECTION (SQLi)

## 4.1 What is SQL injection?

SQL injection occurs when an application allows untrusted input to change the meaning of a SQL query.

Imagine a login system.

A user enters:

```text
Username: charles
Password: mypassword
```

The application constructs:

```sql
SELECT * FROM users
WHERE username = 'charles'
AND password = 'mypassword';
```

That query is expected.

But what if the application inserts user input directly into SQL without parameterization?

An attacker may supply SQL syntax instead of ordinary data.

The database may then interpret part of the user's input as an instruction.

## 4.2 Vulnerable Python example

```python
import sqlite3

connection = sqlite3.connect("users.db")
cursor = connection.cursor()

username = input("Username: ")
password = input("Password: ")

query = f"""
SELECT * FROM users
WHERE username = '{username}'
AND password = '{password}'
"""

cursor.execute(query)
```

The dangerous line is:

```python
cursor.execute(query)
```

The SQL instruction was assembled using untrusted input.

## 4.3 Safe local Python testing

Suppose you deliberately create a vulnerable login application in your own lab.

You can use Python Requests to send an unusual input and observe whether the application behaves unexpectedly.

```python
import requests

# Local lab only.
BASE_URL = "http://127.0.0.1:5000/login"

payload = {
    "username": "charles' OR '1'='1' -- ",
    "password": "wrong-password"
}

response = requests.post(
    BASE_URL,
    data=payload,
    timeout=5
)

print("Status:", response.status_code)
print("Response:", response.text[:500])
```

This is a controlled SQL injection test string. It is not a general-purpose database exploitation tool.

Observe whether the application rejects the input normally or behaves as though authentication succeeded.

## 4.4 Prevention: parameterized SQL

Correct Python:

```python
import sqlite3

connection = sqlite3.connect("users.db")
cursor = connection.cursor()

username = input("Username: ")
password = input("Password: ")

query = """
SELECT * FROM users
WHERE username = ?
AND password = ?
"""

cursor.execute(query, (username, password))
```

Notice:

```python
cursor.execute(query, (username, password))
```

The question marks are placeholders.

The database receives the SQL structure and values separately.

The values are treated as data rather than being interpreted as SQL syntax.

For PostgreSQL with psycopg:

```python
cursor.execute(
    """
    SELECT *
    FROM users
    WHERE username = %s
    """,
    (username,)
)
```

The placeholder syntax depends on the database driver.

## 4.5 PHP prevention

Use PDO prepared statements:

```php
<?php

$stmt = $pdo->prepare(
    "SELECT * FROM users WHERE username = :username"
);

$stmt->execute([
    "username" => $username
]);

$user = $stmt->fetch();

?>
```

Never assume that escaping strings manually is a replacement for parameterized SQL.

**Root cause:** Mixing SQL instructions and untrusted input.

**Prevention:** Parameterized queries, least-privilege database accounts, secure error handling, and safe authentication design.

---

# 5. FILE UPLOAD VULNERABILITIES

## 5.1 What is file upload vulnerability?

A file upload vulnerability occurs when an application accepts files without adequately validating or controlling them.

Imagine a website that allows users to upload profile pictures.

The developer expects:

```text
profile.png
```

But the application accepts arbitrary files and stores them in a publicly accessible directory.

This creates several possible problems:

- Uploading executable files.
- Uploading files with misleading extensions.
- Uploading extremely large files.
- Uploading malformed images.
- Overwriting existing files.
- Storing files in dangerous directories.

## 5.2 Vulnerable PHP example

```php
<?php

$destination = "uploads/" . $_FILES["file"]["name"];

move_uploaded_file(
    $_FILES["file"]["tmp_name"],
    $destination
);

echo "Upload successful";

?>
```

What is wrong?

1. It trusts the original filename.
2. It does not validate the file's actual contents.
3. It does not limit file size.
4. It does not restrict allowed file types.
5. It may place uploaded content in an executable web directory.

## 5.3 Python testing example

For a local application with a file-upload endpoint:

```python
import requests

url = "http://127.0.0.1:5000/upload"

files = {
    "file": (
        "test.txt",
        b"This is a harmless laboratory test file.",
        "text/plain"
    )
}

response = requests.post(
    url,
    files=files,
    timeout=5
)

print("Status:", response.status_code)
print(response.text)
```

This tests whether the application accepts a normal file.

You should then test how it handles:

- An unsupported extension.
- A file that exceeds the permitted size.
- An empty file.
- A file with misleading MIME metadata.
- A filename containing unusual characters.

Do these tests only against your lab application.

## 5.4 Prevention

A secure upload system should:

1. Allow only required file formats.
2. Verify actual file content, not just its extension.
3. Generate a random server-side filename.
4. Enforce a maximum file size.
5. Store uploads outside the executable web root where practical.
6. Prevent execution in upload directories.
7. Apply appropriate file permissions.
8. Consider malware scanning.
9. Avoid trusting client-provided MIME types.

### Secure Flask example

```python
from flask import Flask, request, abort
from pathlib import Path
from werkzeug.utils import secure_filename
import uuid

app = Flask(__name__)

UPLOAD_DIR = Path("private_uploads")
UPLOAD_DIR.mkdir(exist_ok=True)

ALLOWED_EXTENSIONS = {"png", "jpg", "jpeg"}

app.config["MAX_CONTENT_LENGTH"] = 2 * 1024 * 1024

@app.route("/upload", methods=["POST"])
def upload():

    uploaded_file = request.files.get("file")

    if uploaded_file is None:
        abort(400, "No file provided")

    original_name = secure_filename(uploaded_file.filename)

    if not original_name:
        abort(400, "Invalid filename")

    extension = original_name.rsplit(".", 1)[-1].lower()

    if extension not in ALLOWED_EXTENSIONS:
        abort(400, "File type not allowed")

    # A production image upload should also decode the image
    # and verify its actual format and content.

    new_name = f"{uuid.uuid4().hex}.{extension}"

    uploaded_file.save(UPLOAD_DIR / new_name)

    return {"message": "Upload accepted"}
```

This is a starting point, not a complete production image-validation system. Extension checks alone cannot establish that a file is a genuine image.

---

# 6. CROSS-SITE SCRIPTING (XSS)

## 6.1 What is XSS?

Cross-Site Scripting occurs when an application places untrusted data into a web page in a way that allows it to be interpreted as executable browser content.

Imagine a comment section:

```text
Username: Charles
Comment: Nice website!
```

The website displays the comment.

But if the application inserts user content into HTML without proper output encoding, a malicious comment could execute JavaScript in another visitor's browser.

Possible consequences include:

- Performing actions as the victim.
- Reading information accessible to page scripts.
- Modifying page content.
- Redirecting visitors.
- Stealing session information when protections are inadequate.

## 6.2 Vulnerable PHP example

```php
<?php

$comment = $_GET["comment"];

echo $comment;

?>
```

The problem is direct output of untrusted content.

## 6.3 Harmless local Python test

Use a harmless HTML marker:

```python
import requests

url = "http://127.0.0.1:5000/comment"

payload = {
    "comment": "<b>LAB-TEST</b>"
}

response = requests.post(
    url,
    data=payload,
    timeout=5
)

print(response.status_code)
print(response.text)
```

If the browser interprets the marker as HTML rather than displaying it as text, investigate the output encoding.

For an authorized lab, a harmless JavaScript proof-of-execution marker can also be used to confirm whether an injection point is executable.

## 6.4 Prevention

PHP:

```php
<?php

$comment = $_GET["comment"] ?? "";

echo htmlspecialchars(
    $comment,
    ENT_QUOTES | ENT_SUBSTITUTE,
    "UTF-8"
);

?>
```

Python Flask with Jinja templates:

```python
from flask import Flask, render_template_string, request

app = Flask(__name__)

@app.route("/")
def home():

    comment = request.args.get("comment", "")

    return render_template_string(
        "<h1>Comment</h1><p>{{ comment }}</p>",
        comment=comment
    )
```

Jinja autoescapes HTML templates in normal Flask configuration.

**Important:** Output encoding depends on context. HTML text, HTML attributes, JavaScript, CSS, and URLs have different encoding requirements. A single escaping function is not appropriate for every context.

---

# 7. COMMAND INJECTION

## 7.1 What is command injection?

Command injection happens when an application builds an operating-system command using untrusted input.

Imagine a network diagnostic website that lets users enter a hostname.

The developer wants to run a ping command.

A dangerous implementation might be:

```python
import os

host = input("Enter hostname: ")

os.system("ping -c 1 " + host)
```

The problem is that the user's input becomes part of a shell command.

The operating system may interpret special shell characters as additional commands.

## 7.2 Safe testing concept

In your local lab, examine how an input containing unexpected shell metacharacters is handled.

Do not run destructive commands to test this vulnerability.

The goal is to determine whether user input is being interpreted as shell syntax instead of being treated as a hostname.

## 7.3 Prevention

Avoid the shell whenever possible.

Correct Python:

```python
import subprocess
import ipaddress

host = input("Enter an IP address: ")

# Validate that the input is an IP address.
ipaddress.ip_address(host)

result = subprocess.run(
    ["ping", "-c", "1", host],
    capture_output=True,
    text=True,
    timeout=5,
    check=False
)

print(result.stdout)
```

Important:

- Pass an argument list.
- Avoid `shell=True` for untrusted input.
- Validate the expected input type.
- Apply timeouts.
- Restrict which operations the application is allowed to perform.

---

# 8. PATH TRAVERSAL

## 8.1 What is path traversal?

Path traversal occurs when an application uses untrusted input to access files without properly restricting the resulting path.

Imagine a download feature:

```text
/download?file=report.pdf
```

The developer expects a file inside:

```text
documents/
```

But if the application trusts the filename and joins paths incorrectly, a user may attempt to access files outside the intended directory.

The vulnerability is sometimes called directory traversal.

## 8.2 Vulnerable Python example

```python
from flask import Flask, request, send_file

app = Flask(__name__)

@app.route("/download")
def download():

    filename = request.args.get("file")

    return send_file("documents/" + filename)
```

The application trusts the supplied path.

## 8.3 Python testing example

```python
import requests

url = "http://127.0.0.1:5000/download"

response = requests.get(
    url,
    params={"file": "report.pdf"},
    timeout=5
)

print(response.status_code)
print(response.headers.get("Content-Type"))
```

Start with legitimate files. Then test whether the server rejects path values that resolve outside the permitted directory.

Do not attempt to retrieve sensitive system files.

## 8.4 Prevention

Use a fixed base directory and verify the resolved path.

```python
from pathlib import Path
from flask import abort, send_file, request

BASE_DIR = Path("documents").resolve()

@app.route("/download")
def download():

    filename = request.args.get("file", "")

    requested_path = (BASE_DIR / filename).resolve()

    if not requested_path.is_relative_to(BASE_DIR):
        abort(403)

    if not requested_path.is_file():
        abort(404)

    return send_file(requested_path)
```

The critical check is:

```python
requested_path.is_relative_to(BASE_DIR)
```

It verifies that the resolved path remains inside the intended directory.

---

# 9. INSECURE DIRECT OBJECT REFERENCES (IDOR) / BROKEN OBJECT AUTHORIZATION

## 9.1 What is IDOR?

IDOR occurs when an application allows a user to access another user's resource by changing an object identifier without checking authorization.

Imagine:

```text
/account/1001
/account/1002
/account/1003
```

A user is authorized to view account `1001`.

But the application only checks whether the user is logged in. It does not check whether the requested account belongs to that user.

Changing the ID may expose another person's data.

## 9.2 Python testing example

In your own test application, create two accounts.

Then request each account using the same authenticated test session:

```python
import requests

session = requests.Session()

# Use only a test account in your local lab.
session.post(
    "http://127.0.0.1:5000/login",
    data={
        "username": "test_user",
        "password": "test_password"
    }
)

for account_id in (1001, 1002):

    response = session.get(
        f"http://127.0.0.1:5000/accounts/{account_id}",
        timeout=5
    )

    print(account_id, response.status_code)
```

If the test user can access another user's account, investigate the authorization logic.

## 9.3 Prevention

Do not rely on object IDs being secret or difficult to guess.

Always check ownership or the user's specific permission.

```python
from flask import abort

@app.route("/accounts/<int:account_id>")
def account_detail(account_id):

    account = get_account(account_id)

    if account is None:
        abort(404)

    if account.owner_id != current_user.id:
        abort(403)

    return account.to_dict()
```

The authorization check must happen on the server for every protected object.

---

# 10. CROSS-SITE REQUEST FORGERY (CSRF)

## 10.1 What is CSRF?

CSRF occurs when an attacker tricks a user's browser into sending an unwanted request to a website where that user is already authenticated.

Imagine:

1. You log in to a banking website.
2. Your browser stores an authentication cookie.
3. You visit an untrusted website.
4. That website attempts to cause your browser to submit a request to the banking website.
5. If the banking application does not properly protect state-changing requests, the unwanted action may succeed.

The browser may automatically attach eligible cookies to the request.

## 10.2 Prevention

Use:

- CSRF tokens.
- SameSite cookie settings.
- Origin and Referer validation as additional protections.
- Appropriate authentication design.
- POST, PUT, PATCH or DELETE for state-changing operations rather than GET.

Important: Changing GET to POST alone does not prevent CSRF.

For Flask, use a maintained CSRF protection extension such as Flask-WTF.

For Django, use its built-in CSRF protection.

---

# 11. SERVER-SIDE REQUEST FORGERY (SSRF)

## 11.1 What is SSRF?

SSRF happens when an application makes network requests based on user-controlled URLs without sufficient restrictions.

Imagine an application has an image-preview feature.

The user supplies a URL:

```text
https://example.com/image.png
```

The server fetches the image and returns a preview.

If the application accepts arbitrary URLs, an attacker may attempt to make the server connect to internal resources that are not publicly accessible.

Potential targets include:

- Internal administration services.
- Private network services.
- Cloud metadata endpoints.
- Internal databases or management APIs.

## 11.2 Safe Python testing

For your local lab, make the application fetch only a harmless local test endpoint that you control.

Observe:

- Whether the server accepts the URL.
- Whether it follows redirects.
- Whether it can access unintended network destinations.
- Whether it returns internal response data.

Do not use a public or third-party service to probe internal networks.

## 11.3 Prevention

A secure URL-fetching application should:

1. Allow only required schemes, normally HTTPS.
2. Use an explicit destination allowlist.
3. Reject loopback, private, link-local and reserved IP addresses when they are not required.
4. Resolve DNS and validate destination addresses.
5. Revalidate every redirect.
6. Prevent DNS rebinding from bypassing validation.
7. Apply connection and response timeouts.
8. Restrict outbound network access at the firewall level.

A simple hostname allowlist is often safer than attempting to block known dangerous addresses.

---

# PART 3 — NETWORK AND DNS VULNERABILITIES

# 12. DNS SPOOFING AND DNS CACHE POISONING

## 12.1 What is DNS spoofing?

DNS translates domain names into IP addresses.

For example:

```text
example.com -> 93.184.216.34
```

DNS spoofing involves supplying false DNS information so that a client receives an incorrect address.

An attacker may attempt to make a victim visit a malicious server while believing they are visiting a legitimate domain.

## 12.2 What is DNS cache poisoning?

DNS cache poisoning is a form of attack in which false DNS information is inserted into a resolver's cache.

If successful, other users of that resolver may receive the incorrect answer until the poisoned entry expires or is removed.

## 12.3 Safe Python DNS testing

First, perform ordinary DNS resolution:

```python
import socket

domain = "example.com"

results = socket.getaddrinfo(
    domain,
    443,
    type=socket.SOCK_STREAM
)

for result in results:
    print(result[4][0])
```

You can also use:

```bash
nslookup example.com
```

Or:

```bash
dig example.com
```

And trace the DNS hierarchy:

```bash
dig +trace example.com
```

For a controlled DNS lab, use a private test domain and a DNS server you administer. Compare the expected answer with the answer received by your client.

## 12.4 Prevention

- Use trusted recursive resolvers.
- Enable DNSSEC validation where supported.
- Keep DNS server software updated.
- Restrict zone transfers.
- Protect DNS administration credentials.
- Monitor unexpected DNS changes.
- Use encrypted DNS where appropriate.

Important distinctions:

- DNSSEC authenticates DNS data; it does not encrypt DNS queries.
- DoH encrypts DNS traffic using HTTPS.
- DoT encrypts DNS traffic using TLS.
- Neither DoH nor DoT by itself proves that the requested domain is trustworthy.

---

# 13. ARP SPOOFING / ARP POISONING

## 13.1 What is ARP spoofing?

ARP is used on IPv4 local networks to associate IP addresses with MAC addresses.

For example:

```text
192.168.1.1 -> AA:BB:CC:DD:EE:FF
```

ARP spoofing involves sending misleading ARP messages so that devices associate an IP address with an incorrect MAC address.

Possible consequences include traffic interception or disruption on a local network.

## 13.2 Safe Python inspection

Use Scapy to inspect your own local ARP table:

```python
import subprocess

result = subprocess.run(
    ["ip", "neigh"],
    capture_output=True,
    text=True,
    check=False
)

print(result.stdout)
```

You can also inspect your local network configuration:

```bash
ip address
ip route
ip neigh
```

## 13.3 Prevention

- Dynamic ARP Inspection on supported switches.
- DHCP snooping.
- Network segmentation.
- Secure switch configuration.
- Static ARP entries in limited environments where appropriate.
- Monitoring unexpected IP-to-MAC changes.

Do not conduct ARP poisoning on a network you do not administer.

---

# 14. MAN-IN-THE-MIDDLE (MITM)

## 14.1 What is MITM?

A man-in-the-middle attack occurs when an attacker positions themselves between two communicating parties and can observe or potentially modify their communication.

For example:

```text
Victim <----> Attacker <----> Server
```

Possible consequences:

- Reading unencrypted traffic.
- Modifying messages.
- Stealing exposed credentials.
- Manipulating responses.

## 14.2 Safe Python inspection

Use Requests to inspect the certificate verification behavior of a test HTTPS service.

```python
import requests

response = requests.get(
    "https://example.com",
    timeout=10
)

print(response.status_code)
print(response.url)
```

Requests verifies certificates by default.

Avoid disabling certificate verification with `verify=False` in real applications.

## 14.3 Prevention

- HTTPS everywhere.
- Correct certificate validation.
- Secure Wi-Fi configurations.
- HSTS for suitable websites.
- Avoid accepting invalid certificates.
- Protect DNS and network infrastructure.
- Use secure authentication protocols.

---

# PART 4 — AUTHENTICATION AND APPLICATION LOGIC

# 15. BROKEN AUTHENTICATION

## 15.1 What is broken authentication?

Authentication establishes who a user is.

Weak authentication mechanisms may allow an attacker to take over an account.

Examples:

- Weak passwords.
- No rate limiting.
- Predictable reset tokens.
- Poor session management.
- Insecure password storage.
- Missing multi-factor authentication.

## 15.2 Python testing example

For your local login application, test how it handles repeated invalid attempts:

```python
import requests

url = "http://127.0.0.1:5000/login"

for attempt in range(5):

    response = requests.post(
        url,
        data={
            "username": "test_user",
            "password": "incorrect-password"
        },
        timeout=5
    )

    print(
        "Attempt:", attempt + 1,
        "Status:", response.status_code
    )
```

This is a small controlled rate-limiting test, not a password-cracking tool.

## 15.3 Prevention

- Use password hashing such as Argon2id or bcrypt.
- Apply rate limiting.
- Implement secure password-reset procedures.
- Use MFA where appropriate.
- Protect sessions.
- Avoid revealing whether a username exists.
- Monitor suspicious login activity.

Never store passwords using plain SHA-256 or reversible encryption as a substitute for password hashing.

---

# 16. SESSION HIJACKING AND SESSION FIXATION

## 16.1 Session hijacking

A session is used to maintain a user's authenticated state.

Session hijacking occurs when an attacker obtains or takes control of a valid session identifier.

## 16.2 Session fixation

Session fixation occurs when an attacker causes a victim to authenticate using a session identifier that the attacker already knows.

## 16.3 Prevention

For PHP:

```php
<?php

session_start();

session_regenerate_id(true);

?>
```

Regenerate the session ID after successful authentication.

Set secure cookie properties:

```php
<?php

session_set_cookie_params([
    "secure" => true,
    "httponly" => true,
    "samesite" => "Lax"
]);

session_start();

?>
```

In production, configure HTTPS correctly before requiring the Secure cookie attribute.

For Python web applications, use secure session-cookie settings and rotate session identifiers or authentication tokens as appropriate.

---

# 17. INSECURE DESERIALIZATION

## 17.1 What is deserialization?

Serialization converts an object into a format that can be stored or transmitted.

Deserialization reconstructs the object.

Examples:

- JSON
- XML
- Pickle
- PHP serialized objects

The problem occurs when an application blindly deserializes untrusted data using a format capable of executing dangerous reconstruction behavior.

## 17.2 Dangerous Python example

```python
import pickle

with open("data.pkl", "rb") as file:
    data = pickle.load(file)
```

Never use `pickle.load()` on untrusted files.

A malicious pickle can execute code during deserialization.

## 17.3 Prevention

Use safer formats such as JSON for untrusted data:

```python
import json

with open("data.json", encoding="utf-8") as file:
    data = json.load(file)
```

Then validate the resulting structure and values.

JSON parsing is not a substitute for input validation, but it avoids pickle's arbitrary object reconstruction behavior.

For PHP, avoid unserializing untrusted input. If object deserialization is unavoidable, tightly restrict allowed classes and use integrity protections.

---

# 18. XML EXTERNAL ENTITY (XXE)

## 18.1 What is XXE?

XXE is a vulnerability involving XML parsers that process external entity declarations when they should not.

A poorly configured XML parser may be tricked into attempting to access external resources.

Potential consequences include:

- Local file disclosure.
- Server-side request forgery.
- Information disclosure.
- Denial of service.

## 18.2 Safe Python testing

For an application that accepts XML, first test normal XML documents and inspect how its parser handles entity declarations.

Do not use sensitive file paths or third-party destinations in your tests.

## 18.3 Prevention

- Disable external entity processing.
- Disable external DTD loading where unnecessary.
- Use securely configured XML parsers.
- Limit XML document size and processing complexity.
- Consider JSON instead of XML where suitable.

Python applications can use `defusedxml` for safer XML parsing:

```bash
python3 -m pip install defusedxml
```

Example:

```python
from defusedxml import ElementTree

xml_data = """
<user>
    <name>Charles</name>
</user>
"""

root = ElementTree.fromstring(xml_data)

print(root.findtext("name"))
```

---

# PART 5 — OTHER IMPORTANT VULNERABILITIES

# 19. DENIAL OF SERVICE (DoS)

DoS attempts to make a system unavailable by exhausting its resources or exploiting a weakness.

Examples:

- Excessive requests.
- Extremely large uploads.
- Expensive database queries.
- Unbounded loops.
- Memory exhaustion.
- Slow request processing.

## Prevention

- Rate limiting.
- Request-size limits.
- Connection limits.
- Timeouts.
- Caching.
- Efficient database queries.
- Reverse proxies and network protections.
- Resource monitoring.

For Python:

```python
app.config["MAX_CONTENT_LENGTH"] = 2 * 1024 * 1024
```

This limits incoming request bodies to 2 MB in Flask.

Test resource limits using small, controlled requests in a local environment. Do not perform flooding attacks.

# 20. SECURITY MISCONFIGURATION

Security misconfiguration occurs when software, servers, databases or network services are configured insecurely.

Examples:

- Debug mode enabled in production.
- Default passwords.
- Exposed administrative panels.
- Unnecessary open ports.
- Directory listing enabled.
- Public database services.
- Overly permissive file permissions.
- Detailed error messages exposed to users.

## Python testing

Inspect listening services:

```bash
ss -tulpen
```

Inspect firewall rules:

```bash
sudo ufw status verbose
```

Check file permissions:

```bash
ls -l
```

## Prevention

- Disable production debug mode.
- Remove default accounts.
- Close unnecessary ports.
- Restrict administrative access.
- Apply least privilege.
- Update software.
- Review server configuration regularly.

# 21. INSECURE CRYPTOGRAPHY

This occurs when an application uses cryptography incorrectly or uses weak algorithms.

Examples:

- Plaintext password storage.
- MD5 for password hashing.
- Hard-coded encryption keys.
- Reusing nonces in inappropriate modes.
- Incorrect AES mode selection.
- Missing authentication on encrypted data.
- Poor random-number generation.

## Prevention

Use appropriate modern cryptographic libraries and designs.

For example, Python's `secrets` module generates security-oriented random values:

```python
import secrets

token = secrets.token_urlsafe(32)

print(token)
```

Use authenticated encryption such as AES-GCM or ChaCha20-Poly1305 for suitable encryption tasks.

Use Argon2id, bcrypt or scrypt for password hashing.

Do not invent your own cryptographic protocol.

---

# PART 6 — HOW TO APPROACH A SECURITY ASSESSMENT

# 22. The vulnerability assessment process

Use this workflow for every authorized laboratory or assessment.

## Step 1: Identify the application

Understand:

- What language is used?
- What framework is used?
- What server is running?
- What database is used?
- What inputs does the application accept?

## Step 2: Identify the attack surface

The attack surface consists of the places through which an application accepts input or exposes functionality.

Examples:

- URL parameters.
- HTTP headers.
- Form fields.
- File uploads.
- API endpoints.
- Cookies.
- WebSockets.
- Network services.

## Step 3: Form a hypothesis

Example:

"The upload endpoint may accept executable files because it checks only the filename extension."

## Step 4: Test safely

Use a harmless test file or request against your lab application.

## Step 5: Record evidence

Record:

- Date and time.
- Endpoint.
- Request method.
- Input supplied.
- Response code.
- Observed behavior.
- Whether the test was successful.

## Step 6: Fix the vulnerability

Implement the appropriate security control.

## Step 7: Retest

Repeat the same test and confirm that the vulnerable behavior no longer occurs.

## Step 8: Document the result

A security report should explain:

- Vulnerability title.
- Description.
- Affected component.
- Root cause.
- Impact.
- Evidence.
- Severity.
- Recommended fix.
- Retest result.

---

# PART 7 — YOUR PRACTICAL LEARNING ROADMAP

Study these in order:

| Stage | Vulnerability | What you learn |
|---|---|---|
| 1 | SQL injection | SQL, databases, parameterization |
| 2 | File upload | HTTP multipart, MIME, file validation |
| 3 | XSS | HTML, JavaScript, browser execution |
| 4 | Command injection | Linux shell and subprocess |
| 5 | Path traversal | Linux filesystem and paths |
| 6 | IDOR | Authentication and authorization |
| 7 | CSRF | Cookies and browser request behavior |
| 8 | Broken authentication | Passwords, tokens and sessions |
| 9 | SSRF | HTTP clients, routing and private networks |
| 10 | DNS attacks | DNS hierarchy, caching and DNSSEC |
| 11 | ARP spoofing | Ethernet, ARP and LAN security |
| 12 | MITM | TLS, certificates and network security |
| 13 | Deserialization | Python objects and data formats |
| 14 | XXE | XML parsing |
| 15 | DoS | Resource management and availability |
| 16 | Cryptographic failures | Hashing, encryption and key management |
| 17 | Security misconfiguration | Linux administration and hardening |

# PART 8 — USEFUL SECURITY REFERENCES

1. OWASP Top 10 — common web application security risks.
2. OWASP Web Security Testing Guide — web security testing methodology.
3. OWASP Cheat Sheet Series — implementation and prevention guidance.
4. PortSwigger Web Security Academy — practical web security labs.
5. TryHackMe — guided cybersecurity learning environments.
6. Hack The Box Academy — structured security training.
7. Python documentation — subprocess, socket, secrets, hashlib and security-related libraries.

---

# FINAL PRINCIPLE

Do not learn cybersecurity as a collection of attack commands.

For each vulnerability, understand:

**Input → Processing → Weakness → Impact → Detection → Prevention → Retesting**

A good security professional should be able to explain not only how a vulnerability can be demonstrated, but also why it exists, what damage it can cause, and how to remove it permanently.
