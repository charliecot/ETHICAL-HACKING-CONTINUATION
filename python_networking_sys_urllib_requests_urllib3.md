# Python Networking Reference: `sys`, `urllib`, `requests`, and `urllib3`

**Purpose:** A practical, step-by-step reference for understanding
Python's command-line/system module and its HTTP/networking libraries.

**Scope:** `sys`, the standard-library `urllib` package, third-party
`requests`, third-party `urllib3`, HTTP methods, headers, sessions,
cookies, encoding/decoding, proxies, TLS certificates, timeouts,
responses, and exceptions.

> Use these techniques only with systems you own or are explicitly
> authorized to test. When inspecting traffic through Burp Suite or
> another proxy, use a lab or an approved test environment.

------------------------------------------------------------------------

# 1. The big picture

Python has several ways to communicate over HTTP. They overlap, but they
are not the same library.

  ------------------------------------------------------------------------
  Tool                         Included with Python? Main purpose
  --------------------- ---------------------------- ---------------------
  `sys`                                          Yes Interpreter,
                                                     command-line
                                                     arguments, standard
                                                     input/output, exit
                                                     status, paths

  `urllib`                                       Yes Standard-library URL
                                                     parsing, encoding,
                                                     and HTTP requests

  `requests`                                      No Convenient high-level
                                                     HTTP client

  `urllib3`                                       No Lower-level HTTP
                                                     client with
                                                     connection pooling
                                                     and advanced
                                                     transport controls
  ------------------------------------------------------------------------

Install the third-party packages:

``` bash
python -m pip install requests urllib3
```

Check installed versions:

``` bash
python -m pip show requests urllib3
```

Check the Python version:

``` bash
python --version
```

## 1.1 How they relate

-   `urllib` is part of Python's standard library. It does not need a
    separate installation.
-   `requests` provides a friendly API such as `requests.get()` and
    `requests.post()`.
-   `urllib3` provides lower-level HTTP connection management. Requests
    uses urllib3 internally in its standard implementation, but you
    normally use Requests directly unless you need urllib3's lower-level
    controls.
-   `sys` is not an HTTP library. It helps your script receive
    arguments, print progress, handle program exit, and inspect the
    Python environment.

Do not confuse `urllib` with `urllib3`: their names are similar, but
their APIs are different.

------------------------------------------------------------------------

# 2. The `sys` module

`sys` provides access to variables and functions used by the Python
interpreter.

``` python
import sys

print(sys.version)
print(sys.platform)
```

## 2.1 Important attributes

  -----------------------------------------------------------------------
  Attribute                           Meaning
  ----------------------------------- -----------------------------------
  `sys.argv`                          List of command-line arguments
                                      passed to the script

  `sys.version`                       Python version and build
                                      information

  `sys.version_info`                  Python version as structured values

  `sys.platform`                      Identifier for the operating-system
                                      platform

  `sys.executable`                    Path to the Python interpreter
                                      running the script

  `sys.path`                          Directories Python searches when
                                      importing modules

  `sys.modules`                       Dictionary of modules currently
                                      loaded

  `sys.stdin`                         Standard input stream

  `sys.stdout`                        Standard output stream

  `sys.stderr`                        Standard error stream

  `sys.exit`                          Function that requests program
                                      termination

  `sys.getsizeof()`                   Returns the shallow size of an
                                      object in bytes

  `sys.exc_info()`                    Returns information about the
                                      currently handled exception

  `sys.exception()`                   Returns the currently handled
                                      exception, or `None`

  `sys.maxsize`                       Largest value of a Python
                                      `Py_ssize_t`, often used as an
                                      indication of platform integer size

  `sys.getrecursionlimit()`           Gets the interpreter recursion
                                      limit

  `sys.setrecursionlimit()`           Changes the recursion limit; use
                                      carefully
  -----------------------------------------------------------------------

### `sys.argv`

`sys.argv` is a list. The first item is normally the script filename;
the remaining items are arguments supplied by the user.

Example file: `app.py`

``` python
import sys

print("All arguments:", sys.argv)

if len(sys.argv) != 2:
    print(f"Usage: python {sys.argv[0]} <url>")
    sys.exit(1)

url = sys.argv[1]
print("URL:", url)
```

Run:

``` bash
python app.py https://example.com
```

Possible output:

``` text
All arguments: ['app.py', 'https://example.com']
URL: https://example.com
```

Important points:

-   `sys.argv[0]` is normally the script name/path.
-   `sys.argv[1]` is the first argument supplied by the user.
-   `len(sys.argv)` counts the script name too.
-   Arguments are strings. Convert them when you need integers or other
    types.

``` python
port = int(sys.argv[1])
```

For complex command-line interfaces, prefer the `argparse` module. It
supports named options, help text, validation, and defaults.

### `sys.exit()`

Requests that the program terminate.

``` python
import sys

if not configuration_is_valid:
    sys.exit(1)

print("Configuration is valid.")
```

Common conventions:

-   `sys.exit()` or `sys.exit(0)`: successful termination.
-   `sys.exit(1)`: general failure.
-   Other non-zero codes can represent different errors.

`sys.exit()` raises the `SystemExit` exception. It can be caught,
although most programs should allow it to terminate the process.

### `sys.stdout`, `sys.stderr`, and `sys.stdin`

These are text streams.

``` python
import sys

sys.stdout.write("Normal output\n")
sys.stderr.write("An error message\n")

name = input("Your name: ")
```

-   `stdout`: normal program output.
-   `stderr`: diagnostic/error output.
-   `stdin`: input stream, often connected to the keyboard or another
    process.

`print()` writes to `sys.stdout` by default.

### Updating a progress line with `\r`

``` python
import sys
import time

for number in range(1, 6):
    sys.stdout.write(f"\rProgress: {number}/5")
    sys.stdout.flush()
    time.sleep(0.5)

print()
```

-   `\r` means carriage return: move the cursor to the beginning of the
    current line.
-   `\n` means newline: move to the next line.
-   `flush()` asks Python to send buffered output to the stream
    immediately.

`'/r'` is not a carriage return; it is just the characters `/` and `r`.

### `sys.exc_info()` and `sys.exception()`

Use these while handling an exception.

``` python
import sys

try:
    number = 10 / 0
except ZeroDivisionError:
    exception_type, exception_value, traceback_object = sys.exc_info()

    print("Type:", exception_type)
    print("Value:", exception_value)
    print("Traceback object:", traceback_object)

    print("Current exception:", sys.exception())
```

-   `sys.exc_info()` returns a three-item tuple: exception type,
    exception instance, and traceback.
-   `sys.exception()` returns the active exception instance in the
    current handled-exception context.
-   In modern Python, inside an `except` block, `sys.exception()` is
    often the simpler choice when you only need the exception object.
-   Outside an active exception handler, `sys.exception()` returns
    `None`; `sys.exc_info()` returns `(None, None, None)`.

Usually, prefer binding the exception directly:

``` python
try:
    int("not a number")
except ValueError as error:
    print(error)
```

### `sys.path`

Python searches these locations when importing modules.

``` python
import sys

for path in sys.path:
    print(path)
```

Temporarily add a directory:

``` python
import sys

sys.path.append(r"E:\my_python_modules")
```

Do not use `sys.path` modifications as a substitute for proper package
installation or project structure. Also avoid naming your own files
after standard modules, for example `requests.py`, `sys.py`, or
`urllib.py`, because that can interfere with imports.

------------------------------------------------------------------------

# 3. URLs and the `urllib` package

`urllib` is a standard-library package for working with URLs and
HTTP-related operations.

Its commonly used modules are:

  -----------------------------------------------------------------------
  Module                              Purpose
  ----------------------------------- -----------------------------------
  `urllib.parse`                      Split, construct, quote, unquote,
                                      and parse URLs/query strings

  `urllib.request`                    Open URLs and send HTTP requests

  `urllib.error`                      HTTP and URL-related exceptions

  `urllib.robotparser`                Read and interpret `robots.txt`
                                      rules
  -----------------------------------------------------------------------

Import individual modules:

``` python
import urllib.parse
import urllib.request
import urllib.error
```

## 3.1 Understand the parts of a URL

Example:

``` text
https://example.com:443/products?id=25&sort=price#details
```

  -----------------------------------------------------------------------
  Part                    Value                   Meaning
  ----------------------- ----------------------- -----------------------
  Scheme                  `https`                 Protocol

  Hostname                `example.com`           Server name

  Port                    `443`                   Network port

  Path                    `/products`             Resource path

  Query                   `id=25&sort=price`      Parameters sent in the
                                                  URL

  Fragment                `details`               Client-side fragment;
                                                  normally not sent in
                                                  the HTTP request
  -----------------------------------------------------------------------

## 3.2 `urllib.parse` methods

### `urlsplit(url)`

Splits a URL into structured components.

``` python
from urllib.parse import urlsplit

url = "https://example.com:443/products?id=25#details"
parts = urlsplit(url)

print(parts.scheme)
print(parts.netloc)
print(parts.hostname)
print(parts.port)
print(parts.path)
print(parts.query)
print(parts.fragment)
```

`urlsplit()` returns a `SplitResult` with attributes:

-   `.scheme`
-   `.netloc`
-   `.hostname`
-   `.port`
-   `.path`
-   `.query`
-   `.fragment`

### `urlparse(url)`

Similar to `urlsplit()`, but returns a `ParseResult` that also has a
`.params` component for the final path segment.

``` python
from urllib.parse import urlparse

parts = urlparse("https://example.com/item;mode=full?id=10")

print(parts.scheme)
print(parts.netloc)
print(parts.path)
print(parts.params)
print(parts.query)
```

For most modern URL work, `urlsplit()` is a good default.

### `urlunsplit(parts)` and `urlunparse(parts)`

Rebuild a URL from components.

``` python
from urllib.parse import urlsplit, urlunsplit

parts = urlsplit("https://example.com/products?id=25")
new_url = urlunsplit(parts)

print(new_url)
```

`urlunsplit()` expects five components: scheme, netloc, path, query,
fragment.

`urlunparse()` rebuilds from the six components used by `urlparse()`.

### `urlencode(query)`

Converts a mapping or sequence of key/value pairs into a URL-encoded
query string.

``` python
from urllib.parse import urlencode

payload = {
    "username": "charles",
    "search": "python networking",
    "page": 2
}

encoded = urlencode(payload)
print(encoded)
```

Possible output:

``` text
username=charles&search=python+networking&page=2
```

**Important:** `urlencode()` expects structured key/value data, not one
already-formed string.

Incorrect:

``` python
payload = "username=charles&password=example"
encoded = urlencode(payload)  # Incorrect input type
```

Correct:

``` python
payload = {
    "username": "charles",
    "password": "example"
}

encoded = urlencode(payload)
```

If you already have a query string, do not encode it again without a
specific reason.

### Repeated query parameters: `doseq=True`

``` python
from urllib.parse import urlencode

params = {
    "category": ["books", "computers"],
    "page": 1
}

print(urlencode(params, doseq=True))
```

This produces repeated keys for the list values.

### `quote(string)` and `quote_plus(string)`

These encode text for use in URL components.

``` python
from urllib.parse import quote, quote_plus

value = "python networking & security"

print(quote(value))
print(quote_plus(value))
```

-   `quote()` normally encodes spaces as `%20`.
-   `quote_plus()` encodes spaces as `+`, which is common in form/query
    encoding.
-   Reserved URL characters are encoded according to the function's
    safe-character rules.

Do not apply `quote()` to an entire URL when you intend to preserve its
separators such as `:`, `/`, `?`, and `&`. Encode the individual
component instead.

### `unquote()` and `unquote_plus()`

Decode percent-encoded URL text.

``` python
from urllib.parse import unquote, unquote_plus

print(unquote("python%20networking"))
print(unquote_plus("python+networking"))
```

-   `unquote()` turns percent escapes such as `%20` into their
    characters.
-   `unquote_plus()` also converts `+` into a space, which is useful for
    form-style query strings.

### `parse_qs(query)` and `parse_qsl(query)`

Parse query-string data.

``` python
from urllib.parse import parse_qs, parse_qsl

query = "tag=python&tag=security&page=2"

print(parse_qs(query))
print(parse_qsl(query))
```

-   `parse_qs()` returns a dictionary whose values are lists. Repeated
    keys are preserved.
-   `parse_qsl()` returns a list of `(key, value)` pairs, preserving
    pair order.

### `urljoin(base, url)`

Combines a base URL with a relative URL.

``` python
from urllib.parse import urljoin

base = "https://example.com/articles/"
print(urljoin(base, "python"))
print(urljoin(base, "/login"))
```

A leading slash replaces the path from the site root. Be careful: if the
second argument is an absolute URL, it can replace the base host
entirely. Validate untrusted URLs before fetching them.

### `urldefrag(url)`

Separates the fragment from a URL.

``` python
from urllib.parse import urldefrag

url, fragment = urldefrag("https://example.com/page#section")
print(url)
print(fragment)
```

### `unwrap(url)`

Removes certain wrapper forms around a URL, such as `<URL:...>`.

### `urlencode()` versus `quote()`

  -----------------------------------------------------------------------
  Function                Input                   Typical purpose
  ----------------------- ----------------------- -----------------------
  `urlencode()`           Mapping or key/value    Encode a set of
                          pairs                   query/form fields

  `quote()`               String                  Encode one URL
                                                  component

  `quote_plus()`          String                  Encode one component
                                                  using `+` for spaces

  `unquote()`             String                  Decode percent escapes

  `unquote_plus()`        String                  Decode percent escapes
                                                  and `+` spaces
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 4. Sending HTTP requests with `urllib.request`

`urllib.request` is the standard-library HTTP request interface.

## 4.1 Simple GET request with `urlopen()`

``` python
from urllib.request import urlopen

with urlopen("https://example.com", timeout=10) as response:
    body = response.read()
    print(response.status)
    print(response.headers)
    print(body.decode("utf-8", errors="replace"))
```

Important details:

-   `urlopen()` opens the URL and returns a response-like object.
-   `.read()` reads the response body as bytes.
-   `.status` is the HTTP status code for HTTP responses.
-   `.headers` contains response headers.
-   `.geturl()` returns the final URL after redirects.
-   `.close()` closes the response; `with` handles this automatically.
-   `timeout` limits how long the operation waits for a response at the
    socket level.

### Why `.decode()` is used

`response.read()` returns bytes. Text processing normally needs a
string.

``` python
raw_bytes = response.read()
text_body = raw_bytes.decode("utf-8", errors="replace")
```

The character encoding should ideally match the server's declared
encoding. UTF-8 is common, but not universal.

## 4.2 `Request()` object

Use `Request` when you need to set headers, data, or a method.

``` python
from urllib.request import Request, urlopen

request = Request(
    "https://httpbin.org/anything",
    headers={
        "User-Agent": "Python-learning-client/1.0",
        "Accept": "application/json"
    },
    method="GET"
)

with urlopen(request, timeout=10) as response:
    print(response.status)
    print(response.read().decode("utf-8", errors="replace"))
```

Common `Request` arguments:

  Argument            Meaning
  ------------------- ------------------------------------------------
  `url`               URL to request
  `data`              Request body as bytes, commonly used with POST
  `headers`           Dictionary of request headers
  `origin_req_host`   Origin host associated with the request
  `unverifiable`      Legacy/request-context flag
  `method`            HTTP method such as GET, POST, PUT, DELETE

A `Request` object also has methods/attributes such as:

-   `.get_method()`: returns the method.
-   `.get_full_url()`: returns the full URL.
-   `.get_header(name)`: retrieves a request header.
-   `.add_header(key, val)`: adds a header.
-   `.add_unredirected_header(key, val)`: adds a header not normally
    forwarded during redirects.
-   `.has_header(name)`: checks whether a header exists.
-   `.remove_header(name)`: removes a header.
-   `.data`: request body.
-   `.headers`: request headers.
-   `.full_url`: full URL.

## 4.3 POST form data with `urllib`

``` python
from urllib.parse import urlencode
from urllib.request import Request, urlopen

form_data = {
    "username": "student",
    "topic": "networking"
}

body = urlencode(form_data).encode("utf-8")

request = Request(
    "https://httpbin.org/post",
    data=body,
    headers={
        "Content-Type": "application/x-www-form-urlencoded",
        "User-Agent": "Python-learning-client/1.0"
    },
    method="POST"
)

with urlopen(request, timeout=10) as response:
    print(response.status)
    print(response.read().decode("utf-8", errors="replace"))
```

The sequence is:

1.  Start with a Python dictionary.
2.  `urlencode()` converts the fields into a form-style string.
3.  `.encode("utf-8")` converts the string into bytes.
4.  `Request(data=body)` sends those bytes as the request body.

## 4.4 `urlopen()` and common related tools

-   `urlopen(url_or_request, data=None, timeout=...)`: opens a URL or
    sends a prepared request.
-   `build_opener(...)`: creates an opener with selected handlers.
-   `install_opener(opener)`: installs a global opener for `urlopen()`.
-   `Request(...)`: creates a request object.
-   `HTTPBasicAuthHandler`: handles HTTP Basic authentication.
-   `HTTPPasswordMgrWithDefaultRealm`: stores credentials for
    authentication handlers.
-   `ProxyHandler`: configures a proxy.
-   `HTTPCookieProcessor`: adds cookie handling to an opener.
-   `HTTPSHandler`: handles HTTPS connections.
-   `HTTPRedirectHandler`: handles redirects.
-   `HTTPErrorProcessor`: processes HTTP error responses.

For advanced cookie and proxy workflows, Requests is often simpler.

------------------------------------------------------------------------

# 5. HTTP fundamentals: methods, headers, body, and status codes

HTTP is the protocol used by clients and servers to exchange requests
and responses.

A request commonly contains:

1.  Method: what operation the client wants.
2.  URL: which resource it wants.
3.  Headers: metadata about the request.
4.  Body: optional data sent to the server.

A response commonly contains:

1.  Status code: outcome of the request.
2.  Headers: metadata about the response.
3.  Body: returned content.

## 5.1 Common HTTP methods

  -----------------------------------------------------------------------
  Method                  Meaning                 Typical use
  ----------------------- ----------------------- -----------------------
  `GET`                   Retrieve a resource     Read a page or API
                                                  record

  `POST`                  Submit data or request  Form submission,
                          processing              creating a resource

  `PUT`                   Replace a resource      Replace a complete
                          representation          record

  `PATCH`                 Partially modify a      Update selected fields
                          resource                

  `DELETE`                Delete a resource       Remove a record

  `HEAD`                  Request headers without Check metadata or
                          the normal response     resource availability
                          body                    

  `OPTIONS`               Ask which communication API capability/CORS
                          options are supported   preflight

  `TRACE`                 Diagnostic loop-back    Rarely enabled in
                          method                  production

  `CONNECT`               Establish a tunnel      Commonly used for HTTPS
                          through a proxy         proxy tunnelling
  -----------------------------------------------------------------------

The server decides which methods are supported. A method's intended
semantics do not guarantee that a particular website implements it.

### GET

``` python
import requests

response = requests.get("https://httpbin.org/get", timeout=10)
```

GET data is commonly placed in the URL query string:

``` python
params = {"q": "python", "page": 2}

response = requests.get(
    "https://httpbin.org/get",
    params=params,
    timeout=10
)
```

### POST

``` python
import requests

form = {"username": "student"}

response = requests.post(
    "https://httpbin.org/post",
    data=form,
    timeout=10
)
```

For JSON:

``` python
response = requests.post(
    "https://httpbin.org/post",
    json={"username": "student"},
    timeout=10
)
```

### PUT, PATCH, DELETE, HEAD, OPTIONS

``` python
import requests

base = "https://httpbin.org"

put_response = requests.put(
    f"{base}/put",
    json={"name": "Updated name"},
    timeout=10
)

patch_response = requests.patch(
    f"{base}/patch",
    json={"active": True},
    timeout=10
)

delete_response = requests.delete(
    f"{base}/delete",
    timeout=10
)

head_response = requests.head(
    f"{base}/get",
    timeout=10
)

options_response = requests.options(
    f"{base}/anything",
    timeout=10
)
```

Some servers reject particular methods. A `405 Method Not Allowed`
response means the server does not allow that method for the target
resource.

## 5.2 Common request headers

Headers are key/value metadata. Header names are case-insensitive.

  Header                Meaning
  --------------------- ---------------------------------------------------
  `Host`                Target host (normally managed by the HTTP client)
  `User-Agent`          Identifies the client software
  `Accept`              Content types the client can accept
  `Accept-Language`     Preferred natural languages
  `Accept-Encoding`     Supported response compression encodings
  `Content-Type`        Media type of the request body
  `Content-Length`      Size of the request body in bytes
  `Authorization`       Authentication credentials/token
  `Cookie`              Cookies sent to the server
  `Referer`             Page/address that led to the request
  `Origin`              Origin of a browser-style cross-origin request
  `Cache-Control`       Cache behaviour directives
  `If-None-Match`       Conditional request using an entity tag
  `If-Modified-Since`   Conditional request using a modification date
  `Connection`          Connection-management directive in HTTP/1.1

### `Content-Type` versus `Accept`

These are easy to confuse:

-   `Content-Type`: what format **I am sending** in the request body.
-   `Accept`: what format **I would like to receive** in the response.

Example:

``` python
headers = {
    "Content-Type": "application/json",
    "Accept": "application/json"
}
```

### Common response headers

  Header                          Meaning
  ------------------------------- ----------------------------------------------
  `Content-Type`                  Type of returned content
  `Content-Length`                Response body length, when supplied
  `Set-Cookie`                    Server asks the client to store a cookie
  `Location`                      Redirect target or related resource location
  `Cache-Control`                 Caching rules
  `ETag`                          Identifier for a particular representation
  `Last-Modified`                 Resource modification time
  `Server`                        Server software information, if disclosed
  `Date`                          Response date/time
  `WWW-Authenticate`              Authentication challenge
  `Access-Control-Allow-Origin`   CORS response policy
  `Strict-Transport-Security`     HSTS policy for HTTPS

Do not assume that a header is trustworthy merely because it came from a
server. Treat remote response data as untrusted input.

## 5.3 Common status codes

     Code Meaning
  ------- ----------------------------------------------------
    `200` OK
    `201` Created
    `202` Accepted for processing
    `204` No Content
    `301` Moved Permanently
    `302` Found / temporary redirect
    `304` Not Modified
    `400` Bad Request
    `401` Unauthorized; authentication is required or failed
    `403` Forbidden
    `404` Not Found
    `405` Method Not Allowed
    `408` Request Timeout
    `409` Conflict
    `413` Content Too Large
    `415` Unsupported Media Type
    `418` I'm a teapot (joke/test status)
    `422` Unprocessable Content
    `429` Too Many Requests
    `500` Internal Server Error
    `502` Bad Gateway
    `503` Service Unavailable
    `504` Gateway Timeout

Status-code classes:

-   `1xx`: informational.
-   `2xx`: success.
-   `3xx`: redirection.
-   `4xx`: client/request problem.
-   `5xx`: server or upstream problem.

A successful network connection does not mean the HTTP request
succeeded. Always inspect the status code.

------------------------------------------------------------------------

# 6. The `requests` library

Requests is a third-party HTTP client designed to make common HTTP work
straightforward.

``` python
import requests
```

## 6.1 Main request functions

  --------------------------------------------------------------------------
  Function                               Purpose
  -------------------------------------- -----------------------------------
  `requests.request(method, url, ...)`   General request function

  `requests.get(url, ...)`               GET

  `requests.post(url, ...)`              POST

  `requests.put(url, ...)`               PUT

  `requests.patch(url, ...)`             PATCH

  `requests.delete(url, ...)`            DELETE

  `requests.head(url, ...)`              HEAD

  `requests.options(url, ...)`           OPTIONS

  `requests.Session()`                   Creates a session that can preserve
                                         cookies and configuration
  --------------------------------------------------------------------------

### General `requests.request()`

``` python
import requests

response = requests.request(
    method="GET",
    url="https://httpbin.org/get",
    timeout=10
)

print(response.status_code)
```

The convenience functions call the general request mechanism with a
particular HTTP method.

## 6.2 Important request arguments

These are common arguments accepted by Requests request methods. Not
every argument is useful for every request.

  -----------------------------------------------------------------------
  Argument                            Meaning
  ----------------------------------- -----------------------------------
  `method`                            HTTP method; used by
                                      `requests.request()`

  `url`                               Destination URL

  `params`                            Query-string parameters

  `data`                              Form data or raw request-body data

  `json`                              Python object to serialize as JSON

  `headers`                           Request headers

  `cookies`                           Cookies to send

  `files`                             Files for multipart upload

  `auth`                              Authentication configuration

  `timeout`                           Connection/read timeout

  `allow_redirects`                   Whether to follow redirects

  `proxies`                           Proxy configuration

  `verify`                            TLS certificate verification
                                      setting or CA bundle path

  `cert`                              Client certificate configuration

  `stream`                            Whether to stream response content

  `hooks`                             Event hooks

  `verify=False`                      Disables certificate verification;
                                      unsafe outside controlled testing
  -----------------------------------------------------------------------

### `params` versus `data` versus `json`

``` python
import requests

# Query parameters are placed in the URL.
response = requests.get(
    "https://httpbin.org/get",
    params={"page": 2, "search": "python"},
    timeout=10
)

# Form data is sent in the request body.
response = requests.post(
    "https://httpbin.org/post",
    data={"username": "student"},
    timeout=10
)

# JSON data is serialized and sent in the request body.
response = requests.post(
    "https://httpbin.org/post",
    json={"username": "student"},
    timeout=10
)
```

-   Use `params` for URL query parameters.
-   Use `data` for form-encoded data or a body you have prepared
    yourself.
-   Use `json` for a Python object that should be serialized to JSON.

Avoid passing the same data in both `data` and `json` unless you have a
specific reason.

## 6.3 The Response object

A call such as `requests.get()` returns a `requests.Response` object.

  -----------------------------------------------------------------------
  Attribute/property                  Meaning
  ----------------------------------- -----------------------------------
  `response.status_code`              HTTP status code

  `response.headers`                  Response headers; case-insensitive
                                      lookup

  `response.text`                     Response body decoded into text

  `response.content`                  Raw response body as bytes

  `response.json()`                   Parses a JSON response body into
                                      Python data

  `response.url`                      Final URL after redirects

  `response.request`                  Prepared request that was sent

  `response.history`                  Earlier redirect responses

  `response.cookies`                  Cookies set in the response

  `response.encoding`                 Encoding used by `.text`

  `response.apparent_encoding`        Encoding guessed from the content

  `response.elapsed`                  Time spent waiting for the response

  `response.reason`                   HTTP reason phrase, if provided

  `response.ok`                       True when status is below 400

  `response.is_redirect`              Whether the response is a redirect

  `response.is_permanent_redirect`    Whether it is a permanent redirect
  -----------------------------------------------------------------------

Example:

``` python
import requests

response = requests.get("https://httpbin.org/json", timeout=10)

print("Status:", response.status_code)
print("Headers:", response.headers)
print("URL:", response.url)
print("Text:", response.text)
print("Bytes:", response.content)
print("JSON:", response.json())
```

Important:

-   `.text` is a Python string.
-   `.content` is bytes.
-   `.json()` parses JSON; it does not mean every response is JSON. It
    can raise a JSON decoding error if the body is not valid JSON.
-   `.ok` is not the same as "status code is exactly 200"; for example,
    a 404 response makes it false, while a 201 response makes it true.

### Useful Response methods

  -----------------------------------------------------------------------------
  Method                                    Meaning
  ----------------------------------------- -----------------------------------
  `response.raise_for_status()`             Raises an HTTPError for
                                            unsuccessful HTTP status codes
                                            (4xx/5xx)

  `response.json()`                         Decodes JSON response content

  `response.iter_content(chunk_size=...)`   Iterates over response body chunks

  `response.iter_lines()`                   Iterates over lines in streamed
                                            response content

  `response.close()`                        Closes the response and releases
                                            resources

  `response.__enter__()` /                  Context-manager methods; normally
  `response.__exit__()`                     use `with` instead
  -----------------------------------------------------------------------------

Example with status checking:

``` python
import requests

try:
    response = requests.get(
        "https://httpbin.org/status/404",
        timeout=10
    )

    response.raise_for_status()

except requests.HTTPError as error:
    print("HTTP failure:", error)
```

## 6.4 The `headers` argument

``` python
import requests

headers = {
    "User-Agent": "LearningClient/1.0",
    "Accept": "application/json",
    "X-Lab-Name": "local-practice"
}

response = requests.get(
    "https://httpbin.org/headers",
    headers=headers,
    timeout=10
)

print(response.text)
```

A custom header is simply an additional key/value pair. Use valid header
names and values. Do not manually set transport-managed headers such as
`Host` or `Content-Length` unless you understand the consequences.

------------------------------------------------------------------------

# 7. Sessions: `requests.Session()`

A session is an object that persists useful settings and cookies across
multiple requests.

Without a session:

``` python
import requests

first = requests.get("https://httpbin.org/cookies/set/theme/dark", timeout=10)
second = requests.get("https://httpbin.org/cookies", timeout=10)
```

These are independent top-level calls. They do not automatically share a
persistent cookie jar.

With a session:

``` python
import requests

with requests.Session() as session:
    first = session.get(
        "https://httpbin.org/cookies/set/theme/dark",
        timeout=10
    )

    second = session.get(
        "https://httpbin.org/cookies",
        timeout=10
    )

    print(second.json())
```

The session stores cookies received from the server and can send
matching cookies on later requests.

## 7.1 Why sessions matter

A session can preserve:

-   Cookies between requests.
-   Default headers.
-   Authentication settings.
-   Proxy configuration.
-   TLS verification configuration.
-   Other request settings.
-   Underlying connection pools, allowing connection reuse.

This is useful when an application expects a sequence such as:

1.  Visit a page.
2.  Receive a session cookie.
3.  Submit a form.
4.  Make another request using the same session.

A session does not automatically mean you are logged in. Authentication
depends on the server's rules and the credentials or tokens you supply.

## 7.2 Session attributes

  -----------------------------------------------------------------------
  Attribute                           Meaning
  ----------------------------------- -----------------------------------
  `session.headers`                   Default headers used for requests
                                      from this session

  `session.cookies`                   Persistent cookie jar

  `session.auth`                      Default authentication setting

  `session.proxies`                   Default proxy mapping

  `session.verify`                    Default TLS verification
                                      configuration

  `session.cert`                      Default client certificate

  `session.params`                    Default query parameters

  `session.hooks`                     Event hooks

  `session.trust_env`                 Whether environment settings such
                                      as proxy variables are trusted

  `session.adapters`                  Transport adapters mounted for URL
                                      prefixes
  -----------------------------------------------------------------------

Example:

``` python
import requests

with requests.Session() as session:

    session.headers.update({
        "User-Agent": "LearningClient/1.0",
        "Accept": "application/json"
    })

    session.params.update({
        "source": "python"
    })

    session.verify = True

    response = session.get(
        "https://httpbin.org/get",
        timeout=10
    )

    print(response.status_code)
```

## 7.3 Session methods

  -----------------------------------------------------------------------
  Method                              Meaning
  ----------------------------------- -----------------------------------
  `session.request()`                 Sends a general HTTP request

  `session.get()`                     Sends GET

  `session.post()`                    Sends POST

  `session.put()`                     Sends PUT

  `session.patch()`                   Sends PATCH

  `session.delete()`                  Sends DELETE

  `session.head()`                    Sends HEAD

  `session.options()`                 Sends OPTIONS

  `session.prepare_request()`         Prepares a request using session
                                      settings

  `session.send()`                    Sends a prepared request

  `session.close()`                   Closes session adapters/connections

  `session.mount(prefix, adapter)`    Mounts a transport adapter for a
                                      URL prefix
  -----------------------------------------------------------------------

Use a context manager to close the session automatically:

``` python
with requests.Session() as session:
    response = session.get("https://example.com", timeout=10)
```

## 7.4 Prepared requests

A prepared request lets you inspect a request before sending it.

``` python
import requests

with requests.Session() as session:

    request = requests.Request(
        "GET",
        "https://httpbin.org/get",
        params={"page": 1},
        headers={"X-Lab": "practice"}
    )

    prepared = session.prepare_request(request)

    print("Method:", prepared.method)
    print("URL:", prepared.url)
    print("Headers:", prepared.headers)
    print("Body:", prepared.body)

    response = session.send(prepared, timeout=10)
    print(response.status_code)
```

Important distinction:

-   `requests.Request`: describes a request you want to make.
-   `PreparedRequest`: the prepared form of that request.
-   `Response`: the server's response.

------------------------------------------------------------------------

# 8. Cookies

Cookies are small pieces of data that a server asks a client to store
and send back on later matching requests.

Common uses:

-   Session identification.
-   Login state.
-   User preferences.
-   Shopping-cart state.
-   Security tokens, depending on the application design.

## 8.1 How cookies work

A simplified exchange:

1.  Client requests a page.
2.  Server responds with a `Set-Cookie` header.
3.  Client stores the cookie.
4.  Client sends a matching `Cookie` header on a later request.
5.  Server uses the cookie value according to its application logic.

Example response header:

``` http
Set-Cookie: sessionid=abc123; Path=/; Secure; HttpOnly; SameSite=Lax
```

Example request header:

``` http
Cookie: sessionid=abc123
```

## 8.2 Cookie attributes

  -----------------------------------------------------------------------
  Attribute                           Meaning
  ----------------------------------- -----------------------------------
  Name                                Cookie name

  Value                               Cookie value

  `Domain`                            Hosts to which the cookie applies

  `Path`                              URL path scope

  `Expires`                           Expiry date/time

  `Max-Age`                           Lifetime in seconds

  `Secure`                            Send only over secure HTTPS
                                      connections

  `HttpOnly`                          Prevent browser JavaScript from
                                      reading it

  `SameSite`                          Controls cross-site sending
                                      behaviour (`Strict`, `Lax`, `None`)
  -----------------------------------------------------------------------

`HttpOnly` is enforced by browsers; it does not stop a Python HTTP
client from reading a cookie it receives.

## 8.3 Send a cookie with Requests

``` python
import requests

cookies = {
    "theme": "dark",
    "language": "en"
}

response = requests.get(
    "https://httpbin.org/cookies",
    cookies=cookies,
    timeout=10
)

print(response.text)
```

## 8.4 Read response cookies

``` python
import requests

response = requests.get(
    "https://httpbin.org/cookies/set/theme/dark",
    timeout=10
)

print(response.cookies)
print(response.cookies.get("theme"))
```

Redirect behaviour can affect where a cookie is visible. A session is
often more convenient for multi-request cookie workflows.

## 8.5 Use a session cookie jar

``` python
import requests

with requests.Session() as session:

    session.get(
        "https://httpbin.org/cookies/set/theme/dark",
        timeout=10
    )

    print("Stored cookies:", session.cookies)

    response = session.get(
        "https://httpbin.org/cookies",
        timeout=10
    )

    print(response.json())
```

## 8.6 Cookie jar methods

A Requests session uses a cookie jar based on Python's cookie handling
facilities.

Useful operations include:

``` python
session.cookies.get("sessionid")
session.cookies.set("theme", "dark")
session.cookies.clear()
session.cookies.clear(domain="example.com")
session.cookies.clear(domain="example.com", path="/", name="theme")
```

Cookie jars also support iteration:

``` python
for cookie in session.cookies:
    print(cookie.name, cookie.value, cookie.domain, cookie.path)
```

Do not treat cookies as harmless strings. Session cookies may grant
access to an account. Never publish, log, or share real authentication
cookies.

------------------------------------------------------------------------

# 9. Encoding and decoding

Encoding and decoding are not one single operation. You must know what
kind of data you have.

## 9.1 String versus bytes

A Python string (`str`) represents text.

Bytes (`bytes`) represent raw byte values.

``` python
message = "Hello Python"

raw = message.encode("utf-8")
restored = raw.decode("utf-8")

print(type(message))   # str
print(type(raw))       # bytes
print(restored)        # Hello Python
```

-   `.encode("utf-8")`: string to bytes.
-   `.decode("utf-8")`: bytes to string.

## 9.2 URL encoding

URL encoding represents characters using percent escapes when needed.

``` python
from urllib.parse import quote, unquote, urlencode

value = "python networking"

print(quote(value))
print(unquote("python%20networking"))

params = {"search": value, "page": 2}
print(urlencode(params))
```

Do not confuse URL encoding with encryption. URL encoding is reversible
formatting, not secrecy.

## 9.3 JSON encoding and decoding

``` python
import json

data = {
    "username": "student",
    "active": True
}

json_text = json.dumps(data)  # Python object -> JSON string
python_data = json.loads(json_text)  # JSON string -> Python object
```

In Requests:

``` python
response = requests.get("https://httpbin.org/json", timeout=10)
data = response.json()
```

## 9.4 Base64 encoding and decoding

Base64 represents bytes using printable characters. It is not
encryption.

``` python
import base64

message = "Hello"

encoded = base64.b64encode(message.encode("utf-8"))
decoded = base64.b64decode(encoded).decode("utf-8")

print(encoded)
print(decoded)
```

## 9.5 Common encoding mistakes

  -----------------------------------------------------------------------
  Mistake                 Why it happens          Correct idea
  ----------------------- ----------------------- -----------------------
  Calling `.decode()` on  It is already text      Use it only on bytes
  a `str`                                         

  Calling `.encode()` on  Bytes are already bytes Decode if you need text
  `bytes`                                         

  Passing a string to     It expects key/value    Pass a dict or pairs
  `urlencode()`           data                    

  Encoding the same query Already-escaped `%`     Know whether data is
  twice                   becomes `%25`           already encoded

  Treating Base64 as      Base64 is only a        Use real cryptography
  encryption              representation          for confidentiality

  Assuming every response Servers can use other   Inspect the declared
  is UTF-8                encodings               charset or encoding
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 10. Proxies

A proxy is an intermediary between a client and a destination server.

A normal request:

``` text
Python client  -------------------->  Web server
```

A proxied request:

``` text
Python client  ----->  Proxy  ----->  Web server
```

Common reasons to use a proxy:

-   Inspecting your own test traffic.
-   Debugging HTTP requests.
-   Routing traffic through an approved gateway.
-   Applying network policy.
-   Monitoring traffic in an authorized environment.

## 10.1 Requests proxy configuration

``` python
import requests

proxies = {
    "http": "http://127.0.0.1:8080",
    "https": "http://127.0.0.1:8080"
}

response = requests.get(
    "https://example.com",
    proxies=proxies,
    timeout=10
)
```

The proxy URL scheme describes the connection to the proxy. A local HTTP
proxy commonly uses `http://127.0.0.1:8080` for both HTTP and HTTPS
destinations. HTTPS destinations are typically reached using a CONNECT
tunnel.

## 10.2 Burp Suite and HTTPS

When Burp is configured as an HTTPS interception proxy, it presents
certificates for intercepted sites. Your Python client must trust the
Burp CA certificate if you want TLS verification enabled.

Preferred controlled-lab configuration:

``` python
response = requests.get(
    "https://your-authorized-lab.example",
    proxies=proxies,
    verify=r"C:\path\to\burp-ca-certificate.pem",
    timeout=10
)
```

The path is an example; use the actual certificate file path on your
machine.

Avoid this except for short-lived, isolated troubleshooting:

``` python
verify=False
```

It disables TLS certificate verification and makes the connection
vulnerable to interception. It also causes an `InsecureRequestWarning`.
Suppressing the warning does not make the connection secure.

## 10.3 Proxy authentication

Some proxies require credentials:

``` python
proxies = {
    "http": "http://username:password@proxy.example:8080",
    "https": "http://username:password@proxy.example:8080"
}
```

Do not hard-code real credentials in source code. Use environment
variables or a secure secret store. Be aware that special characters in
credentials may need URL encoding.

## 10.4 Environment proxy variables

Requests may read proxy settings from environment variables, including:

-   `HTTP_PROXY`
-   `HTTPS_PROXY`
-   `ALL_PROXY`
-   `NO_PROXY`

Inspect your environment carefully before assuming a request is
bypassing or using a proxy.

A session can opt out of environment-derived settings:

``` python
session.trust_env = False
```

This also affects other environment-derived behaviour, so use it
intentionally.

## 10.5 `urllib.request.ProxyHandler`

``` python
from urllib.request import ProxyHandler, build_opener

proxy_handler = ProxyHandler({
    "http": "http://127.0.0.1:8080",
    "https": "http://127.0.0.1:8080"
})

opener = build_opener(proxy_handler)

with opener.open("https://example.com", timeout=10) as response:
    print(response.status)
```

------------------------------------------------------------------------

# 11. TLS / HTTPS certificate verification

HTTPS is HTTP carried over TLS. TLS protects confidentiality and
integrity and authenticates the server when certificate verification
succeeds.

Requests verifies server certificates by default.

``` python
requests.get("https://example.com", timeout=10)
```

Use a custom CA bundle when appropriate:

``` python
requests.get(
    "https://example.com",
    verify=r"C:\certificates\trusted-ca.pem",
    timeout=10
)
```

For a client certificate:

``` python
requests.get(
    "https://example.com",
    cert=(
        r"C:\certificates\client-cert.pem",
        r"C:\certificates\client-key.pem"
    ),
    timeout=10
)
```

A certificate file path and a client certificate are not
interchangeable:

-   `verify`: which CA certificates to trust when verifying the server.
-   `cert`: client certificate/key presented to the server when mutual
    TLS is required.

Never solve a certificate problem by permanently disabling verification.

------------------------------------------------------------------------

# 12. Timeouts, redirects, streaming, and downloads

## 12.1 Timeouts

Always consider a timeout for network requests.

``` python
response = requests.get(
    "https://example.com",
    timeout=(5, 20)
)
```

A tuple means:

-   `5`: connection timeout.
-   `20`: read timeout.

A timeout is not necessarily a strict total wall-clock deadline for the
whole download. It primarily controls waiting for connection and
response data.

## 12.2 Redirects

Requests follows redirects by default for most methods.

``` python
response = requests.get(
    "https://example.com",
    allow_redirects=True,
    timeout=10
)

print(response.url)
print(response.history)
```

Disable redirect following when you want to inspect the first response:

``` python
response = requests.get(
    "https://example.com",
    allow_redirects=False,
    timeout=10
)

print(response.status_code)
print(response.headers.get("Location"))
```

## 12.3 Stream a large response

``` python
import requests

with requests.get(
    "https://example.com/large-file",
    stream=True,
    timeout=(5, 30)
) as response:

    response.raise_for_status()

    with open("download.bin", "wb") as file:
        for chunk in response.iter_content(chunk_size=8192):
            if chunk:
                file.write(chunk)
```

-   `stream=True`: do not immediately load the entire response body into
    memory.
-   `iter_content()`: yields chunks of response data.
-   `chunk_size=8192`: asks for chunks around 8 KiB; actual yielded
    sizes can vary.
-   `if chunk`: skips empty keep-alive chunks.

Always validate the source, size, and content of downloaded files before
using them.

------------------------------------------------------------------------

# 13. Exceptions and error handling

Network programs can fail for many reasons:

-   DNS lookup failure.
-   Connection refused.
-   Connection timeout.
-   Read timeout.
-   TLS certificate failure.
-   Proxy failure.
-   Invalid URL.
-   HTTP error status.
-   Invalid JSON.
-   Connection reset or interruption.

## 13.1 Important Requests exceptions

Most Requests exceptions inherit from
`requests.exceptions.RequestException`.

  Exception                Meaning
  ------------------------ -------------------------------------------------
  `RequestException`       Base class for Requests-related exceptions
  `HTTPError`              Raised by `raise_for_status()` for HTTP 4xx/5xx
  `ConnectionError`        A connection-related failure
  `Timeout`                A timeout occurred
  `ConnectTimeout`         Connection attempt timed out
  `ReadTimeout`            Waiting for response data timed out
  `TooManyRedirects`       Redirect limit exceeded
  `URLRequired`            A valid URL was required
  `MissingSchema`          URL has no scheme such as `https://`
  `InvalidSchema`          Unsupported or invalid URL scheme
  `InvalidURL`             Invalid URL
  `InvalidHeader`          Invalid header value/name
  `SSLError`               TLS/SSL-related error
  `ProxyError`             Proxy-related failure
  `ChunkedEncodingError`   Invalid or incomplete chunked response
  `ContentDecodingError`   Failed to decode response content
  `JSONDecodeError`        Response body could not be parsed as JSON

## 13.2 A practical Requests exception pattern

``` python
import requests

try:
    response = requests.get(
        "https://example.com",
        timeout=(5, 15)
    )

    response.raise_for_status()

except requests.exceptions.Timeout:
    print("The request timed out.")

except requests.exceptions.SSLError as error:
    print("TLS certificate or SSL problem:", error)

except requests.exceptions.ProxyError as error:
    print("Proxy problem:", error)

except requests.exceptions.ConnectionError as error:
    print("Could not establish or maintain the connection:", error)

except requests.exceptions.HTTPError as error:
    print("The server returned an unsuccessful HTTP status:", error)

except requests.exceptions.RequestException as error:
    print("Other Requests error:", error)

else:
    print("Request succeeded:", response.status_code)
```

Why order matters:

-   Catch more specific exceptions before broader parent exceptions.
-   `Timeout` and `ConnectionError` are categories within the Requests
    exception hierarchy.
-   `RequestException` is a broad fallback.

## 13.3 JSON parsing exceptions

``` python
import requests

try:
    response = requests.get("https://example.com", timeout=10)
    response.raise_for_status()
    data = response.json()

except requests.exceptions.JSONDecodeError:
    print("The response body was not valid JSON.")

except requests.exceptions.RequestException as error:
    print("HTTP request failed:", error)
```

## 13.4 `urllib.error` exceptions

`urllib.request` commonly raises exceptions from `urllib.error`.

  Exception                Meaning
  ------------------------ -------------------------------------------
  `URLError`               General URL-opening failure
  `HTTPError`              HTTP error response, such as 404 or 500
  `ContentTooShortError`   Downloaded data was shorter than expected

`HTTPError` is also a kind of `URLError`.

Example:

``` python
from urllib.request import urlopen
from urllib.error import HTTPError, URLError

try:
    with urlopen("https://example.com", timeout=10) as response:
        print(response.status)

except HTTPError as error:
    print("HTTP status:", error.code)
    print("Reason:", error.reason)

except URLError as error:
    print("URL/network problem:", error.reason)
```

## 13.5 urllib3 exceptions

Common urllib3 exception classes include:

-   `HTTPError`: base class for many urllib3 HTTP-related errors.
-   `MaxRetryError`: retry attempts were exhausted.
-   `NewConnectionError`: a new connection could not be established.
-   `ConnectTimeoutError`: connection timed out.
-   `ReadTimeoutError`: reading response data timed out.
-   `SSLError`: TLS/SSL failure.
-   `ProxyError`: proxy-related failure.
-   `ProtocolError`: protocol-level or connection interruption problem.
-   `LocationParseError`: invalid URL/location parsing.

Exact exception behaviour depends on the operation and urllib3 version.
Catch the narrowest useful exception and retain the original error
details for diagnosis.

------------------------------------------------------------------------

# 14. `urllib3`

`urllib3` is a lower-level HTTP client. It is useful when you need
direct control over connection pools, retries, TLS configuration, and
request transport.

``` python
import urllib3
```

## 14.1 `PoolManager`

`PoolManager` manages connection pools and reuses connections.

``` python
import urllib3

http = urllib3.PoolManager()

response = http.request(
    "GET",
    "https://httpbin.org/get",
    timeout=urllib3.Timeout(connect=5.0, read=10.0)
)

print(response.status)
print(response.headers)
print(response.data.decode("utf-8", errors="replace"))
```

Important response fields:

  -----------------------------------------------------------------------
  Attribute                           Meaning
  ----------------------------------- -----------------------------------
  `response.status`                   HTTP status code

  `response.headers`                  Response headers

  `response.data`                     Response body as bytes

  `response.reason`                   Reason phrase

  `response.version`                  HTTP protocol version information

  `response.url`                      Response URL

  `response.retries`                  Retry history/metadata, depending
                                      on response configuration
  -----------------------------------------------------------------------

Common response methods/properties include:

-   `.read(amt=...)`: read response data.
-   `.stream(amt=...)`: iterate over response data.
-   `.release_conn()`: return the connection to the pool when
    appropriate.
-   `.drain_conn()`: consume remaining data so the connection can be
    reused.
-   `.get_redirect_location()`: get a redirect location when applicable.

When using streaming or `preload_content=False`, make sure response data
is consumed or the connection is released/closed properly.

## 14.2 `PoolManager.request()`

General form:

``` python
response = http.request(
    method="GET",
    url="https://example.com",
    fields={"page": 1},
    headers={"Accept": "application/json"},
    timeout=10.0
)
```

Common arguments include:

  -----------------------------------------------------------------------
  Argument                            Meaning
  ----------------------------------- -----------------------------------
  `method`                            HTTP method

  `url`                               Destination URL

  `fields`                            Form fields or query fields,
                                      depending on method/encoding
                                      options

  `headers`                           Request headers

  `body`                              Request body

  `encode_multipart`                  Whether to encode fields as
                                      multipart form data

  `multipart_boundary`                Boundary for multipart encoding

  `preload_content`                   Whether to read the response body
                                      before returning

  `decode_content`                    Whether to decode content encodings

  `redirect`                          Whether to follow redirects

  `retries`                           Retry configuration

  `timeout`                           Timeout configuration

  `pool_timeout`                      How long to wait for a connection
                                      from the pool

  `release_conn`                      Whether to release the connection
                                      automatically

  `chunked`                           Whether to send a chunked request
                                      body
  -----------------------------------------------------------------------

The precise accepted arguments can vary by urllib3 version and request
method. Check the installed version's documentation when using advanced
options.

## 14.3 `ProxyManager`

Use `ProxyManager` to send requests through a proxy.

``` python
import urllib3

http = urllib3.ProxyManager(
    "http://127.0.0.1:8080"
)

response = http.request(
    "GET",
    "https://example.com",
    timeout=10.0
)

print(response.status)
```

For HTTPS destinations through an HTTP proxy, urllib3 uses proxy
tunnelling as needed.

## 14.4 `Timeout`

``` python
timeout = urllib3.Timeout(
    connect=5.0,
    read=15.0
)
```

-   `connect`: maximum time for establishing the connection.
-   `read`: maximum wait for response data.
-   A timeout object can be reused as request configuration.

## 14.5 `Retry`

Retries can help with temporary network or server conditions. Do not
blindly retry every request: retrying a non-idempotent operation such as
a payment submission could duplicate an action if the server processed
the first request but the response was lost.

``` python
import urllib3
from urllib3.util import Retry

retry_policy = Retry(
    total=3,
    connect=3,
    read=2,
    status=2,
    backoff_factor=0.3,
    status_forcelist=[429, 500, 502, 503, 504],
    allowed_methods={"GET", "HEAD", "OPTIONS"}
)

http = urllib3.PoolManager(
    retries=retry_policy
)

response = http.request(
    "GET",
    "https://example.com",
    timeout=10.0
)

print(response.status)
```

Important `Retry` options:

  -----------------------------------------------------------------------
  Option                              Meaning
  ----------------------------------- -----------------------------------
  `total`                             Overall retry limit

  `connect`                           Retry limit for connection errors

  `read`                              Retry limit for read errors

  `status`                            Retry limit for selected HTTP
                                      status responses

  `redirect`                          Redirect limit

  `backoff_factor`                    Controls delay growth between
                                      retries

  `status_forcelist`                  Status codes that may trigger a
                                      retry

  `allowed_methods`                   HTTP methods eligible for status
                                      retries

  `raise_on_status`                   Whether to raise after status
                                      retries are exhausted

  `respect_retry_after_header`        Whether to respect a server's
                                      `Retry-After` header
  -----------------------------------------------------------------------

Retry policy behaviour is version-sensitive. Prefer retrying
safe/idempotent methods unless the application has an explicit
idempotency mechanism.

## 14.6 TLS configuration in urllib3

`PoolManager` verifies TLS certificates by default. Configure a trusted
CA bundle when needed:

``` python
import urllib3

http = urllib3.PoolManager(
    cert_reqs="CERT_REQUIRED",
    ca_certs=r"C:\certificates\trusted-ca.pem"
)
```

Do not turn off certificate verification in normal use.

## 14.7 Suppressing warnings

You may see `InsecureRequestWarning` after disabling certificate
verification.

``` python
import urllib3

urllib3.disable_warnings(
    urllib3.exceptions.InsecureRequestWarning
)
```

This only hides the warning. It does not restore certificate
verification or protect the connection. Prefer fixing the CA trust
configuration.

------------------------------------------------------------------------

# 15. Requests versus urllib3 versus urllib

  ----------------------------------------------------------------------------------
  Task              Requests          urllib3           urllib
  ----------------- ----------------- ----------------- ----------------------------
  Simple GET        Very convenient   More explicit     Available

  Simple POST       Very convenient   More low-level    Available

  Session cookies   Built-in          Requires more     Cookie processor/opener
                    `Session`         manual handling   

  Connection pools  Through adapters  Direct control    Different opener/handler
                                                        model

  Query encoding    Can use `params`  Can use fields    `urllib.parse.urlencode()`

  Proxy             `proxies=`        `ProxyManager`    `ProxyHandler`

  TLS CA bundle     `verify=`         CA configuration  SSL context/handlers

  Retry policy      Adapter           Direct `Retry`    Usually custom handling
                    configuration     configuration     

  Included with     No                No                Yes
  Python                                                
  ----------------------------------------------------------------------------------

**Practical recommendation:**

-   Learn `urllib.parse` because URL parsing and encoding are useful
    everywhere.
-   Learn Requests for everyday API clients, web testing in authorized
    labs, and automation.
-   Understand urllib3 when you need lower-level connection, pool, and
    retry behaviour.
-   Learn `sys.argv`, `sys.exit()`, and standard streams for building
    command-line tools.

------------------------------------------------------------------------

# 16. A complete, safe learning example using Requests

This example sends a GET request, uses query parameters, supplies
headers, checks the status, and handles common errors.

``` python
import sys
import requests


def fetch_page(url):

    headers = {
        "User-Agent": "PythonNetworkingStudy/1.0",
        "Accept": "text/html,application/json"
    }

    try:
        response = requests.get(
            url,
            headers=headers,
            params={"source": "learning"},
            timeout=(5, 15)
        )

        response.raise_for_status()

    except requests.exceptions.Timeout:
        print("[-] The request timed out.", file=sys.stderr)
        return None

    except requests.exceptions.RequestException as error:
        print(f"[-] Request failed: {error}", file=sys.stderr)
        return None

    print("[+] Status:", response.status_code)
    print("[+] Final URL:", response.url)
    print("[+] Content type:", response.headers.get("Content-Type"))
    print("[+] Response length:", len(response.content))

    return response


def main():

    if len(sys.argv) != 2:
        print(f"Usage: python {sys.argv[0]} <url>")
        sys.exit(1)

    url = sys.argv[1]

    response = fetch_page(url)

    if response is None:
        sys.exit(1)

    print(response.text[:500])


if __name__ == "__main__":
    main()
```

Run:

``` bash
python app.py https://example.com
```

This example is for learning ordinary HTTP requests. It does not attempt
to bypass authentication or extract private information.

------------------------------------------------------------------------

# 17. Troubleshooting guide

  --------------------------------------------------------------------------
  Symptom                    Possible cause          What to inspect
  -------------------------- ----------------------- -----------------------
  `MissingSchema`            URL has no `http://` or Check the URL
                             `https://`              

  `ConnectionError`          Server unreachable,     Host, port, DNS,
                             refused connection, DNS firewall
                             or network issue        

  `ConnectTimeout`           Connection could not be Network, proxy, timeout
                             established in time     

  `ReadTimeout`              Server did not send     Server response,
                             data quickly enough     timeout

  `ProxyError`               Proxy is unavailable or Proxy host, port,
                             misconfigured           listener

  `SSLError`                 Certificate or TLS      CA trust, hostname,
                             negotiation issue       system time

  `InsecureRequestWarning`   TLS verification        Restore verification
                             disabled                and configure CA

  `404`                      Resource path not found URL path

  `401`                      Authentication missing  Approved
                             or invalid              credentials/token

  `403`                      Server refuses access   Permissions and
                                                     application policy

  `405`                      Method not supported    Allowed methods

  `.json()` fails            Body is not valid JSON  Content-Type and
                                                     response body

  Cookie not retained        Independent calls or    Use one session;
                             cookie scope issue      inspect domain/path

  Request not visible in     Proxy not configured or Proxy mapping,
  Burp                       request bypasses proxy  environment variables,
                                                     session settings

  HTTPS fails through Burp   Client does not trust   Configure CA bundle; do
                             Burp CA                 not permanently disable
                                                     verification
  --------------------------------------------------------------------------

A good debugging sequence:

1.  Print the URL you are requesting.
2.  Print the HTTP method.
3.  Inspect the request headers (do not expose secrets).
4.  Check whether a proxy is configured.
5.  Check the exception type and message.
6.  If a response exists, inspect its status code and content type.
7.  Check whether the server returned a redirect.
8.  Check TLS verification and CA configuration.
9.  Reproduce the issue with a simple request before adding sessions,
    cookies, or custom headers.

------------------------------------------------------------------------

# 18. Security and good practices

1.  **Use HTTPS** for sensitive information.
2.  **Keep TLS verification enabled.** Use a trusted CA bundle for a lab
    proxy.
3.  **Set timeouts.** A network request should not wait forever.
4.  **Do not hard-code secrets.** Use environment variables or a secret
    manager.
5.  **Do not log session cookies, passwords, authorization headers, or
    API tokens.**
6.  **Validate URLs** if users can supply them. This helps prevent
    server-side request forgery (SSRF) in applications that fetch
    user-provided URLs.
7.  **Limit response sizes** when downloading untrusted content.
8.  **Handle redirects carefully.** A redirect can change the
    destination host.
9.  **Use retries carefully.** Avoid retrying non-idempotent operations
    without an idempotency strategy.
10. **Treat response bodies as untrusted input.** Validate before
    parsing, saving, or executing anything.
11. **Use a session for related requests** so cookies and connection
    reuse work as expected.
12. **Use authorized test targets only.** A proxy or HTTP library does
    not grant permission to access a system.

------------------------------------------------------------------------

# 19. Quick reference / cheat sheet

``` python
# Standard library
import sys
import urllib.parse
import urllib.request
import urllib.error

# Third-party libraries
import requests
import urllib3

# sys
sys.argv
sys.exit(1)
sys.stdout.write("message")
sys.stdout.flush()
sys.stderr.write("error")
sys.path
sys.executable
sys.version
sys.platform
sys.exception()
sys.exc_info()

# urllib.parse
urllib.parse.urlsplit(url)
urllib.parse.urlparse(url)
urllib.parse.urlencode({"q": "python"})
urllib.parse.quote("python networking")
urllib.parse.unquote("python%20networking")
urllib.parse.parse_qs("a=1&a=2")
urllib.parse.parse_qsl("a=1&a=2")
urllib.parse.urljoin(base_url, relative_url)
urllib.parse.urldefrag(url)

# urllib.request
urllib.request.Request(url, headers={"Accept": "application/json"})
urllib.request.urlopen(url, timeout=10)

# Requests
requests.get(url, params={"page": 1}, timeout=10)
requests.post(url, data={"name": "student"}, timeout=10)
requests.post(url, json={"name": "student"}, timeout=10)
requests.request("GET", url, timeout=10)

# Response
response.status_code
response.headers
response.text
response.content
response.json()
response.url
response.cookies
response.history
response.elapsed
response.raise_for_status()
response.iter_content(chunk_size=8192)
response.close()

# Session
session = requests.Session()
session.headers.update({"User-Agent": "StudyClient/1.0"})
session.cookies
session.proxies
session.verify
session.get(url, timeout=10)
session.close()

# Proxy
proxies = {
    "http": "http://127.0.0.1:8080",
    "https": "http://127.0.0.1:8080"
}

# urllib3
http = urllib3.PoolManager()
response = http.request("GET", url, timeout=10.0)
response.status
response.headers
response.data

proxy_http = urllib3.ProxyManager("http://127.0.0.1:8080")
```

------------------------------------------------------------------------

# 20. Recommended learning order

Study these topics in this order so that the ideas build on each other:

1.  **HTTP basics:** URL, request, response, methods, headers, status
    codes, body.
2.  **Python strings and bytes:** `str`, `bytes`, `.encode()`,
    `.decode()`.
3.  **`urllib.parse`:** `urlsplit`, `urlencode`, `quote`, `unquote`,
    `parse_qs`.
4.  **`urllib.request`:** `Request`, `urlopen`, headers, POST body,
    exceptions.
5.  **Requests basics:** `get`, `post`, `params`, `data`, `json`,
    response attributes.
6.  **Sessions and cookies:** session persistence, cookie jars,
    authentication concepts.
7.  **Proxies and TLS:** Burp proxy configuration, CA certificates,
    HTTPS verification.
8.  **Exceptions and timeouts:** diagnose failures rather than guessing.
9.  **urllib3:** connection pools, proxy manager, timeout and retry
    policies.
10. **Build a small HTTP client:** command-line arguments, session,
    headers, timeout, error handling, and safe logging.

## Final concept to remember

A typical HTTP workflow looks like this:

``` text
Python program
    |
    |  sys.argv reads the URL supplied by the user
    |
    v
Build request
    |
    |  URL + method + headers + parameters/body + cookies
    |
    v
Optional proxy
    |
    |  Burp Suite or another approved proxy
    |
    v
Web server
    |
    |  HTTP status + response headers + response body
    |
    v
Python response object
    |
    |  inspect status, headers, cookies, text/bytes/JSON
    |
    v
Handle errors and display useful results
```

The most important distinctions:

-   `urllib.parse.urlencode()` converts key/value data into a query/form
    string.
-   `.encode()` converts text (`str`) into bytes (`bytes`).
-   `.decode()` converts bytes into text.
-   `requests.get()` sends a GET request.
-   `requests.Session()` preserves cookies and configuration across
    related requests.
-   `response.headers` contains response metadata; `headers=` supplies
    request metadata.
-   `cookies=` sends cookies; `response.cookies` reads cookies from a
    response.
-   `proxies=` routes a Requests call through a proxy.
-   `verify=` controls TLS certificate verification.
-   `timeout=` limits how long the client waits.
-   `raise_for_status()` turns HTTP error status codes into exceptions.
