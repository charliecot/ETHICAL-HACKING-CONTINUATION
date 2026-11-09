# Python BeautifulSoup (BS4) and Regular Expressions (Regex)
## A Practical Guide: Beginner to Advanced

**Author: Nzegge Charles Ngong**

---

# PART 1: BEAUTIFULSOUP (BS4)

## 1. Introduction to BeautifulSoup

BeautifulSoup is a Python library used to parse HTML and XML documents.

It helps you:
- Extract information from websites.
- Locate HTML elements.
- Extract text from HTML tags.
- Retrieve HTML attributes.
- Navigate through HTML structures.
- Search for specific elements.
- Extract links, images, tables, and other information.

For example, imagine a website returns this HTML:

```html
<html>
    <body>
        <h1>Welcome</h1>

        <p>Learning Python</p>

        <a href="https://example.com">Visit website</a>
    </body>
</html>
```

BeautifulSoup allows you to extract:

```text
Welcome
Learning Python
https://example.com
```

### Installation

```bash
pip install beautifulsoup4
```

Using UV:

```bash
uv add beautifulsoup4
```

Import:

```python
from bs4 import BeautifulSoup
```

---

# 2. Understanding HTML Before Using BS4

BeautifulSoup works with HTML structures, so understand these basic elements.

```html
<html>
    <head>
        <title>My Website</title>
    </head>

    <body>
        <h1 id="heading">Welcome</h1>

        <p class="description">
            Learning Python
        </p>

        <a href="https://example.com">
            Visit website
        </a>

    </body>
</html>
```

| HTML element | Meaning |
|---|---|
| `<html>` | Root HTML element |
| `<head>` | Contains document metadata |
| `<title>` | Page title |
| `<body>` | Main page content |
| `<h1>` | Main heading |
| `<p>` | Paragraph |
| `<a>` | Hyperlink |
| `<div>` | General container |
| `<span>` | Inline container |
| `<table>` | Table |
| `<tr>` | Table row |
| `<th>` | Table heading cell |
| `<td>` | Table data cell |
| `<img>` | Image |
| `<ul>` | Unordered list |
| `<li>` | List item |

### HTML attributes

Attributes provide additional information about elements.

Example:

```html
<p class="description" id="intro">
    Learning Python
</p>
```

Here:

- `class="description"` is a class attribute.
- `id="intro"` is an ID attribute.

These attributes are useful when searching with BeautifulSoup.

---

# 3. Creating a BeautifulSoup Object

BeautifulSoup needs two things:

1. HTML content.
2. A parser.

Example:

```python
from bs4 import BeautifulSoup

html = """
<html>
    <body>
        <h1>Welcome</h1>
        <p>Learning Python</p>
    </body>
</html>
"""

soup = BeautifulSoup(html, "html.parser")
```

Explanation:

```python
BeautifulSoup(html, "html.parser")
```

- `html`: The HTML document.
- `"html.parser"`: Python's built-in HTML parser.
- `soup`: The parsed document object.

Other parsers:

```python
BeautifulSoup(html, "html.parser")
BeautifulSoup(html, "lxml")
BeautifulSoup(html, "html5lib")
```

Install optional parsers:

```bash
pip install lxml html5lib
```

Recommended starting point:

```python
soup = BeautifulSoup(response.text, "html.parser")
```

---

# 4. Finding HTML Elements

This is one of the most important parts of BeautifulSoup.

There are two commonly used methods:

- `find()`
- `find_all()`

## 4.1 find()

Finds the first matching element.

Example:

```python
heading = soup.find("h1")

print(heading)
```

Output:

```html
<h1>Welcome</h1>
```

Extract its text:

```python
print(heading.get_text())
```

Output:

```text
Welcome
```

If no matching element exists:

```python
result = soup.find("h2")

print(result)
```

Output:

```text
None
```

Important: `find()` can return `None`, so check before calling methods on the result.

```python
result = soup.find("h2")

if result:
    print(result.get_text(strip=True))
else:
    print("Element not found")
```

## 4.2 find_all()

Finds all matching elements.

Example:

```html
<body>
    <p>Python</p>
    <p>Django</p>
    <p>Networking</p>
</body>
```

Python:

```python
paragraphs = soup.find_all("p")

for paragraph in paragraphs:
    print(paragraph.get_text(strip=True))
```

Output:

```text
Python
Django
Networking
```

Important difference:

```python
soup.find("p")
```

Returns one element.

```python
soup.find_all("p")
```

Returns a list-like collection of matching elements.

You can also specify multiple tags:

```python
elements = soup.find_all(["h1", "h2", "p", "a"])
```

This finds all matching headings, paragraphs, and links.

---

# 5. Searching with HTML Attributes

You can search using attributes such as `id`, `class`, `href`, and `src`.

## 5.1 Searching by ID

HTML:

```html
<h1 id="main-heading">Welcome</h1>
```

Python:

```python
heading = soup.find(id="main-heading")

print(heading.get_text(strip=True))
```

Output:

```text
Welcome
```

You can also write:

```python
heading = soup.find("h1", id="main-heading")
```

## 5.2 Searching by class

HTML:

```html
<p class="description">Learning Python</p>
```

Python:

```python
paragraph = soup.find("p", class_="description")

print(paragraph.get_text(strip=True))
```

Notice `class_`, not `class`.

This is because `class` is a reserved keyword in Python.

Find all elements with a class:

```python
elements = soup.find_all(class_="description")
```

An element can have multiple classes:

```html
<p class="description important">Learning Python</p>
```

You can search using:

```python
soup.find_all(class_="description")
```

For more flexible class combinations, use CSS selectors.

---

# 6. CSS Selectors with select() and select_one()

BeautifulSoup supports CSS selectors.

These methods are often easier and more readable when dealing with complex HTML structures.

## 6.1 select_one()

Returns the first matching element.

```python
heading = soup.select_one("h1")
```

## 6.2 select()

Returns all matching elements.

```python
paragraphs = soup.select("p")
```

### Common CSS selectors

| Selector | Meaning |
|---|---|
| `p` | All paragraph elements |
| `h1` | All h1 elements |
| `#main` | Element with ID main |
| `.description` | Elements with class description |
| `div p` | Paragraphs inside div elements |
| `div > p` | Direct child paragraphs of div |
| `a[href]` | Links containing href |
| `input[type="text"]` | Text input elements |
| `table tr` | Table rows inside tables |
| `table th` | Table headings inside tables |
| `table td` | Table data cells inside tables |

Example:

```python
links = soup.select("a[href]")

for link in links:
    print(link.get("href"))
```

Find a paragraph inside a particular container:

```python
paragraph = soup.select_one("div.container p")
```

### When should you use CSS selectors?

Use:

```python
soup.select_one()
soup.select()
```

when the HTML structure is complex.

Use:

```python
soup.find()
soup.find_all()
```

when you want straightforward tag or attribute searches.

---

# 7. Extracting Text

There are several ways to extract text.

## 7.1 get_text()

```python
element = soup.find("p")

text = element.get_text()

print(text)
```

## 7.2 get_text(strip=True)

Removes surrounding whitespace.

```python
text = element.get_text(strip=True)
```

## 7.3 get_text(separator=" ")

Useful when an element contains multiple nested elements.

HTML:

```html
<p>
    Learning
    <strong>Python</strong>
    programming
</p>
```

Python:

```python
text = soup.find("p").get_text(separator=" ", strip=True)

print(text)
```

Output:

```text
Learning Python programming
```

## 7.4 strings and stripped_strings

Extract text nodes separately:

```python
element = soup.find("div")

for text in element.stripped_strings:
    print(text)
```

`stripped_strings` removes surrounding whitespace from individual text nodes.

---

# 8. Extracting HTML Attributes

Sometimes you don't need the text inside an element. You need its attributes.

HTML:

```html
<a href="https://example.com" target="_blank">
    Visit website
</a>
```

Python:

```python
link = soup.find("a")

print(link.get("href"))
print(link.get("target"))
```

Output:

```text
https://example.com
_blank
```

You can also access attributes using dictionary-style access:

```python
print(link["href"])
```

Difference:

```python
link.get("href")
```

Returns `None` if the attribute is missing, unless you provide a default.

```python
link.get("href", "")
```

Returns an empty string if missing.

```python
link["href"]
```

Raises `KeyError` if the attribute is missing.

### Extracting links

```python
for link in soup.find_all("a"):
    href = link.get("href")
    text = link.get_text(strip=True)

    print(text, href)
```

### Extracting images

```python
for image in soup.find_all("img"):
    src = image.get("src")
    alt = image.get("alt")

    print(src, alt)
```

---

# 9. Navigating the HTML Tree

BeautifulSoup allows you to move between parent and child elements.

Example:

```html
<div class="container">
    <h1>Products</h1>

    <div class="product">
        <h2>Keyboard</h2>
        <p>Mechanical keyboard</p>
    </div>
</div>
```

## 9.1 parent

Gets the parent element.

```python
heading = soup.find("h2")

print(heading.parent)
```

## 9.2 parents

Iterates through all ancestors.

```python
for parent in heading.parents:
    print(parent.name)
```

## 9.3 children

Gets direct children.

```python
container = soup.find("div", class_="container")

for child in container.children:
    print(child)
```

## 9.4 descendants

Gets all nested elements and text nodes.

```python
for item in container.descendants:
    print(item)
```

## 9.5 find_parent()

Finds the first matching ancestor.

```python
heading = soup.find("h2")

product = heading.find_parent("div", class_="product")
```

## 9.6 find_next_sibling()

Finds the next sibling element matching the search.

```python
heading = soup.find("h2")

next_element = heading.find_next_sibling("p")
```

## 9.7 find_previous_sibling()

Finds a previous sibling.

```python
paragraph = soup.find("p")

previous_element = paragraph.find_previous_sibling("h2")
```

Other useful navigation methods:

```python
element.find_next()
element.find_all_next()

element.find_previous()
element.find_all_previous()

element.find_next_sibling()
element.find_previous_sibling()
```

---

# 10. Working with Tables

Tables are particularly useful when extracting structured information.

HTML:

```html
<table>
    <tr>
        <th>Username</th>
        <th>Role</th>
    </tr>

    <tr>
        <td>admin</td>
        <td>Administrator</td>
    </tr>

    <tr>
        <td>john</td>
        <td>Customer</td>
    </tr>
</table>
```

Extract all rows:

```python
rows = soup.find_all("tr")

for row in rows:
    print(row.get_text(" ", strip=True))
```

Extract all table headings:

```python
headings = soup.find_all("th")

for heading in headings:
    print(heading.get_text(strip=True))
```

Extract all table data cells:

```python
cells = soup.find_all("td")

for cell in cells:
    print(cell.get_text(strip=True))
```

Extract both headings and data cells:

```python
elements = soup.find_all(["th", "td"])

for element in elements:
    print(element.get_text(strip=True))
```

The list:

```python
["th", "td"]
```

means find either of these HTML tags.

### Extracting each row into a list

```python
for row in soup.find_all("tr"):

    values = [
        cell.get_text(strip=True)
        for cell in row.find_all(["th", "td"])
    ]

    print(values)
```

Output:

```text
['Username', 'Role']
['admin', 'Administrator']
['john', 'Customer']
```

This technique is useful when extracting structured table information.

---

# 11. Checking Whether an Element Exists

Never assume an element exists.

Incorrect:

```python
version = soup.find("th").get_text(strip=True)
```

If `find("th")` returns `None`, this causes:

```text
AttributeError: 'NoneType' object has no attribute 'get_text'
```

Correct:

```python
element = soup.find("th")

if element is not None:
    version = element.get_text(strip=True)
    print(version)
else:
    print("Element not found")
```

You can also use:

```python
element = soup.find("th")

if element:
    print(element.get_text(strip=True))
```

The explicit `is not None` check is clearer when you specifically want to check whether an element was found.

---

# 12. Useful BeautifulSoup Properties and Methods

## Properties

| Property | Purpose |
|---|---|
| `.name` | Tag name |
| `.attrs` | All attributes |
| `.parent` | Parent element |
| `.contents` | Direct children as a list |
| `.children` | Iterator over direct children |
| `.descendants` | Iterator over nested descendants |
| `.strings` | Text nodes |
| `.stripped_strings` | Text nodes without surrounding whitespace |
| `.text` | Text contained in the element |
| `.next_sibling` | Next sibling node |
| `.previous_sibling` | Previous sibling node |

## Methods

| Method | Purpose |
|---|---|
| `find()` | Finds first matching element |
| `find_all()` | Finds all matching elements |
| `select_one()` | First CSS selector match |
| `select()` | All CSS selector matches |
| `get()` | Retrieves an attribute |
| `get_text()` | Extracts text |
| `prettify()` | Formats HTML for readability |
| `decode()` | Returns HTML representation |
| `encode()` | Encodes the HTML representation |
| `has_attr()` | Checks whether an attribute exists |
| `get_attribute_list()` | Gets attribute values as a list |
| `find_parent()` | Finds a matching ancestor |
| `find_next()` | Finds a subsequent matching element |
| `find_previous()` | Finds a previous matching element |
| `find_next_sibling()` | Finds a matching next sibling |
| `find_previous_sibling()` | Finds a matching previous sibling |

### Inspecting HTML

When you don't understand the returned structure:

```python
print(soup.prettify())
```

Inspect the tag name:

```python
print(element.name)
```

Inspect its attributes:

```python
print(element.attrs)
```

Inspect its text:

```python
print(element.get_text(strip=True))
```

---

# 13. Combining Requests with BeautifulSoup

BeautifulSoup does not download web pages by itself.

You can use Requests to retrieve a page and BeautifulSoup to parse its HTML.

```python
import requests
from bs4 import BeautifulSoup

url = "https://example.com"

try:
    response = requests.get(url, timeout=(10, 30))

    response.raise_for_status()

except requests.exceptions.RequestException as error:
    print("Request failed:", error)

else:
    soup = BeautifulSoup(response.text, "html.parser")

    title = soup.find("title")

    if title:
        print(title.get_text(strip=True))
```

Understand the responsibilities:

| Library | Responsibility |
|---|---|
| Requests | HTTP communication |
| BeautifulSoup | HTML parsing and extraction |
| Regex | Pattern matching within text |

Important: BeautifulSoup parses the HTML it receives. It does not execute JavaScript. If the page generates its content dynamically, the HTML returned by Requests might not contain that content.

---

# 14. Best Practices for BeautifulSoup

1. Use `find()` when you need one element.
2. Use `find_all()` when you need multiple elements.
3. Use CSS selectors for complex HTML structures.
4. Prefer `get_text(strip=True)` for clean text.
5. Use `.get()` when accessing optional attributes.
6. Check whether an element exists before accessing its methods.
7. Inspect `soup.prettify()` when extraction fails.
8. Avoid assuming the first table row contains the information you need.
9. Keep HTML parsing separate from HTTP requests.
10. Use the actual HTML structure to determine what you extract.

---

# PART 2: REGULAR EXPRESSIONS (REGEX)

# 15. Introduction to Regex

Regular Expressions are patterns used to search, match, validate, and extract text.

Python provides the `re` module.

```python
import re
```

Regex is useful for extracting:

- IP addresses.
- Email addresses.
- URLs.
- Version numbers.
- Usernames.
- Dates.
- Phone numbers.
- File extensions.
- Specific patterns inside HTML text.

Example:

```python
text = "My database version is 8.0.42"

match = re.search(r"\d+", text)

if match:
    print(match.group())
```

Output:

```text
8
```

Why?

Because `\d+` matches one or more digits, and `re.search()` returns the first matching sequence.

---

# 16. Understanding Python Raw Strings

You will frequently see Regex written like this:

```python
r"\d+\.\d+"
```

The `r` means raw string.

Raw strings are useful because Regex uses backslashes extensively.

Compare:

```python
"\d+"
r"\d+"
```

Prefer:

```python
r"\d+"
```

This makes it easier to write and understand Regex patterns without Python interpreting many backslashes as string escapes.

---

# 17. Important Regex Special Characters

| Character | Meaning |
|---|---|
| `.` | Any character except newline by default |
| `^` | Beginning of string or line |
| `$` | End of string or line |
| `*` | Zero or more repetitions |
| `+` | One or more repetitions |
| `?` | Zero or one repetition |
| `{n}` | Exactly n repetitions |
| `{n,m}` | Between n and m repetitions |
| `[]` | Character set |
| `()` | Capturing group |
| `(?:...)` | Non-capturing group |
| `|` | OR |
| `\` | Escape character |
| `\b` | Word boundary |

Examples:

```python
r"\d+"        # One or more digits
r"\d{3}"      # Exactly three digits
r"\d{2,4}"    # Between two and four digits
r"ab*"        # a followed by zero or more b characters
r"ab+"        # a followed by one or more b characters
r"colou?r"    # color or colour
r"cat|dog"    # cat OR dog
r"^Python"    # Starts with Python
r"Python$"    # Ends with Python
```

---

# 18. Character Classes

Character classes define which characters are allowed.

| Pattern | Meaning |
|---|---|
| `\d` | Digit: 0–9 |
| `\D` | Non-digit |
| `\w` | Word character: letters, digits, underscore |
| `\W` | Non-word character |
| `\s` | Whitespace |
| `\S` | Non-whitespace |
| `[abc]` | a, b, or c |
| `[a-z]` | Lowercase letters |
| `[A-Z]` | Uppercase letters |
| `[0-9]` | Digits |
| `[^abc]` | Anything except a, b, or c |

Examples:

```python
r"\d+"          # 12345
r"\w+"          # Python_123
r"[A-Z]+"       # PYTHON
r"[a-z]+"       # python
r"[A-Za-z]+"    # Python
r"[^0-9]+"      # Characters that are not digits
```

---

# 19. Important Regex Functions

Python's `re` module provides several useful functions.

## 19.1 re.search()

Searches anywhere in a string and returns the first match.

```python
text = "Database version: 8.0.42"

match = re.search(r"\d+\.\d+\.\d+", text)

if match:
    print(match.group())
```

Output:

```text
8.0.42
```

## 19.2 re.match()

Checks for a match at the beginning of the string.

```python
text = "8.0.42 MySQL"

match = re.match(r"\d+\.\d+\.\d+", text)

if match:
    print(match.group())
```

Output:

```text
8.0.42
```

If the string begins with another character, it won't match.

## 19.3 re.fullmatch()

Requires the entire string to match.

```python
version = "8.0.42"

match = re.fullmatch(r"\d+\.\d+\.\d+", version)

if match:
    print("Valid version format")
```

## 19.4 re.findall()

Returns all matches as a list.

```python
text = "Versions: 8.0.42, 10.4.2, 9.1.5"

versions = re.findall(r"\d+\.\d+\.\d+", text)

print(versions)
```

Output:

```text
['8.0.42', '10.4.2', '9.1.5']
```

## 19.5 re.finditer()

Returns an iterator containing match objects.

```python
text = "Versions: 8.0.42, 10.4.2"

for match in re.finditer(r"\d+\.\d+\.\d+", text):
    print(match.group())
    print(match.start())
    print(match.end())
```

Useful when you need the positions of matches.

## 19.6 re.sub()

Replaces matching text.

```python
text = "My phone number is 123456789"

result = re.sub(r"\d", "*", text)

print(result)
```

Output:

```text
My phone number is *********
```

## 19.7 re.split()

Splits text using a Regex pattern.

```python
text = "Python,Django;Flask PostgreSQL"

result = re.split(r"[,;\s]+", text)

print(result)
```

Output:

```text
['Python', 'Django', 'Flask', 'PostgreSQL']
```

---

# 20. Understanding Match Objects

Functions like `search()`, `match()`, and `fullmatch()` return a match object when successful.

Example:

```python
text = "Version: 8.0.42"

match = re.search(r"\d+\.\d+\.\d+", text)
```

Useful match object methods:

| Method | Purpose |
|---|---|
| `.group()` | Returns the complete match |
| `.group(0)` | Returns the complete match |
| `.group(1)` | Returns the first capturing group |
| `.groups()` | Returns all captured groups |
| `.start()` | Starting position |
| `.end()` | Ending position |
| `.span()` | Starting and ending positions |

Example:

```python
text = "Version: 8.0.42"

match = re.search(r"(\d+)\.(\d+)\.(\d+)", text)

if match:
    print(match.group(0))
    print(match.group(1))
    print(match.group(2))
    print(match.group(3))
    print(match.groups())
```

Output:

```text
8.0.42
8
0
42
('8', '0', '42')
```

The parentheses create capturing groups.

---

# 21. Regex Flags

Flags modify how a Regex pattern behaves.

| Flag | Meaning |
|---|---|
| `re.IGNORECASE` or `re.I` | Ignore uppercase/lowercase differences |
| `re.MULTILINE` or `re.M` | Make `^` and `$` work per line |
| `re.DOTALL` or `re.S` | Make `.` match newline characters |
| `re.VERBOSE` or `re.X` | Allow readable, commented Regex patterns |
| `re.ASCII` or `re.A` | Restrict certain character classes to ASCII |

Example:

```python
text = "PYTHON is powerful"

match = re.search(r"python", text, re.IGNORECASE)

if match:
    print(match.group())
```

Output:

```text
PYTHON
```

### Using re.VERBOSE

Useful for longer patterns:

```python
pattern = re.compile(
    r"""
    \d+       # One or more digits
    \.        # Literal dot
    \d+       # One or more digits
    \.        # Literal dot
    \d+       # One or more digits
    """,
    re.VERBOSE
)
```

---

# 22. Working with Groups

Groups allow you to extract specific sections of a match.

Example:

```python
text = "Username: admin, Role: administrator"

pattern = r"Username:\s*(\w+),\s*Role:\s*(\w+)"

match = re.search(pattern, text)

if match:
    username = match.group(1)
    role = match.group(2)

    print(username)
    print(role)
```

Output:

```text
admin
administrator
```

### Named groups

Named groups make patterns easier to understand.

```python
pattern = (
    r"Username:\s*(?P<username>\w+), "
    r"Role:\s*(?P<role>\w+)"
)

match = re.search(pattern, text)

if match:
    print(match.group("username"))
    print(match.group("role"))
```

---

# PART 3: COMBINING BEAUTIFULSOUP AND REGEX

# 23. Why Combine BS4 and Regex?

BeautifulSoup understands HTML structure.

Regex understands text patterns.

They solve different problems.

Imagine this HTML:

```html
<table>
    <tr>
        <th>Database version</th>
    </tr>

    <tr>
        <th>8.0.42-0ubuntu0.20.04.1</th>
    </tr>
</table>
```

You want to extract the version.

Step 1: BeautifulSoup finds the HTML element.

Step 2: BeautifulSoup extracts its text.

Step 3: Regex searches for the version format.

## Example 1: Extracting a version

```python
import re
from bs4 import BeautifulSoup

html = """
<table>
    <tr>
        <th>Database version</th>
    </tr>

    <tr>
        <th>8.0.42-0ubuntu0.20.04.1</th>
    </tr>
</table>
"""

soup = BeautifulSoup(html, "html.parser")

pattern = re.compile(
    r"\b\d+(?:\.\d+)+(?:-[\w.]+)?"
)

for element in soup.find_all(["th", "td"]):

    text = element.get_text(strip=True)

    match = pattern.search(text)

    if match:
        print(match.group())
```

Output:

```text
8.0.42-0ubuntu0.20.04.1
```

### Understanding the pattern

```python
r"\b\d+(?:\.\d+)+(?:-[\w.]+)?"
```

| Pattern | Meaning |
|---|---|
| `\b` | Word boundary |
| `\d+` | One or more digits |
| `(?:...)` | Non-capturing group |
| `\.` | Literal dot |
| `+` | One or more repetitions |
| `(?:-[\w.]+)?` | Optional hyphen followed by letters, digits, underscores, or dots |

This pattern matches:

```text
8.0.42
8.0.42-0ubuntu0.20.04.1
10.4.2
```

It is a practical version-shaped pattern, not a universal validator for every software version scheme.

## Example 2: Extracting email addresses

HTML:

```html
<div class="users">
    <p>admin@example.com</p>
    <p>john@example.org</p>
</div>
```

Python:

```python
soup = BeautifulSoup(html, "html.parser")

pattern = re.compile(
    r"[\w.+-]+@[\w.-]+\.[A-Za-z]{2,}"
)

for paragraph in soup.find_all("p"):

    text = paragraph.get_text(strip=True)

    emails = pattern.findall(text)

    for email in emails:
        print(email)
```

## Example 3: Extracting IP addresses

HTML:

```html
<p>Server IP: 192.168.1.100</p>
```

Python:

```python
pattern = re.compile(
    r"\b(?:\d{1,3}\.){3}\d{1,3}\b"
)
```

Important: This pattern finds IPv4-shaped strings but does not validate that every octet is between 0 and 255. Use Python's `ipaddress` module for proper validation.

## Example 4: Extracting usernames and values from table cells

HTML:

```html
<table>
    <tr>
        <th>admin*administrator</th>
    </tr>

    <tr>
        <th>john*customer</th>
    </tr>
</table>
```

Python:

```python
for element in soup.find_all("th"):

    value = element.get_text(strip=True)

    if "*" in value:

        username, separator, role = value.partition("*")

        if separator:
            print("Username:", username)
            print("Role:", role)
```

Notice that we don't need Regex here.

Sometimes a normal string method like `partition()` is simpler and more efficient.

---

# PART 4: BEST PRACTICES AND LEARNING APPROACH

## 24. When Should You Use BS4, Regex, or Both?

| Situation | Recommended tool |
|---|---|
| Find an HTML heading | BeautifulSoup |
| Extract a link | BeautifulSoup |
| Extract an image source | BeautifulSoup |
| Find all table rows | BeautifulSoup |
| Extract an IP address from text | Regex |
| Find a version number in text | Regex |
| Extract usernames from HTML table cells | Both |
| Extract emails from HTML paragraphs | Both |
| Validate an IPv4 address | `ipaddress` |
| Split a string by delimiters | `str.split()` or Regex |

### Important principle

Do not use Regex to parse an entire HTML document when BeautifulSoup can understand its structure.

Instead:

```text
HTML document
     |
     v
BeautifulSoup
     |
     v
Find relevant HTML element
     |
     v
Extract text
     |
     v
Regex or normal Python string methods
     |
     v
Final result
```

## 25. Common Mistakes

### Mistake 1: Calling a method on None

Incorrect:

```python
version = soup.find("th").get_text(strip=True)
```

Correct:

```python
element = soup.find("th")

if element is not None:
    version = element.get_text(strip=True)
```

### Mistake 2: Confusing find() and find_all()

Incorrect:

```python
elements = soup.find_all("th")

print(elements.get_text())
```

`find_all()` returns a collection, not one element.

Correct:

```python
for element in elements:
    print(element.get_text(strip=True))
```

### Mistake 3: Using the wrong Regex escaping

Incorrect:

```python
r"\d+\\.\d+"
```

Correct:

```python
r"\d+\.\d+"
```

### Mistake 4: Forgetting that search() returns a match object

```python
match = re.search(r"\d+", "Version 123")
```

To get the matched text:

```python
print(match.group())
```

### Mistake 5: Assuming every HTML page contains the expected element

Always inspect:

```python
print(soup.prettify())
```

and:

```python
print(response.status_code)
```

---

# PART 5: PRACTICAL EXERCISES

## Exercise 1: BeautifulSoup Fundamentals

Given:

```html
<div class="container">
    <h1>Cybersecurity</h1>

    <p id="description">Learning ethical hacking</p>

    <a href="https://example.com">Visit</a>

    <ul>
        <li>Python</li>
        <li>Networking</li>
        <li>Security</li>
    </ul>
</div>
```

Tasks:

1. Extract the h1 heading.
2. Extract the paragraph.
3. Extract its ID.
4. Extract the link URL.
5. Extract all list items.
6. Extract all text inside the container.
7. Find the container using its class.

## Exercise 2: Table Extraction

Given:

```html
<table>
    <tr>
        <th>Name</th>
        <th>Age</th>
    </tr>

    <tr>
        <td>John</td>
        <td>25</td>
    </tr>

    <tr>
        <td>Mary</td>
        <td>30</td>
    </tr>
</table>
```

Tasks:

1. Extract all rows.
2. Extract all headings.
3. Extract all data cells.
4. Extract the names only.
5. Extract the ages only.
6. Put each row into a Python list.

## Exercise 3: Regex Fundamentals

Write patterns to extract:

1. All numbers from a string.
2. All words from a string.
3. All uppercase letters.
4. All lowercase letters.
5. A version such as `8.0.42`.
6. An IPv4-shaped string.
7. An email address.
8. A date in `YYYY-MM-DD` format.

## Exercise 4: Combine BS4 and Regex

Given:

```html
<table>
    <tr>
        <th>Server: 192.168.1.100</th>
    </tr>

    <tr>
        <th>Version: 8.0.42-0ubuntu0.20.04.1</th>
    </tr>

    <tr>
        <th>Email: admin@example.com</th>
    </tr>
</table>
```

Tasks:

1. Parse the HTML using BeautifulSoup.
2. Extract all `<th>` elements.
3. Extract the text from each element.
4. Use Regex to extract the IP address.
5. Use Regex to extract the version.
6. Use Regex to extract the email address.

---

# FINAL REFERENCE: IMPORTS

```python
import re
import requests

from bs4 import BeautifulSoup
```

## BeautifulSoup: Methods to Practise First

```python
soup.find()
soup.find_all()

soup.select_one()
soup.select()

element.get_text()
element.get()

element.find_parent()
element.find_next_sibling()
element.find_previous_sibling()

soup.prettify()
```

## Regex: Functions to Practise First

```python
re.search()
re.match()
re.fullmatch()

re.findall()
re.finditer()

re.sub()
re.split()

re.compile()
```

## My Recommended Learning Order

1. Understand HTML tags and attributes.
2. Learn `BeautifulSoup()`.
3. Practise `find()` and `find_all()`.
4. Learn `get_text()` and `get()`.
5. Practise CSS selectors.
6. Learn HTML tree navigation.
7. Practise extracting tables and links.
8. Learn Regex character classes.
9. Learn Regex quantifiers and special characters.
10. Practise `search()`, `findall()`, and `sub()`.
11. Learn capturing groups.
12. Combine BeautifulSoup and Regex in small projects.

**Final advice:** Do not rush into combining libraries. Become comfortable extracting clean text with BeautifulSoup first. Then learn Regex independently. Finally, combine them when you need to extract specific patterns from the HTML content.
