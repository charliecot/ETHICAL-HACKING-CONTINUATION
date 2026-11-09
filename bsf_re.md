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
head
