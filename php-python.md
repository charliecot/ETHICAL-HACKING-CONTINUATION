# PHP AND PYTHON ON LINUX — COMMAND-LINE COMPARISON GUIDE

**Author:** Nzegge Charles Ngong  
**Purpose:** Learn PHP command-line commands and understand their closest Python equivalents.

---

# 1. First, understand the command structure

A command generally has this structure:

```bash
program [options] [arguments]
```

For example:

```bash
php -S localhost:8000
```

Breakdown:

- `php`: the program being executed.
- `-S`: an option telling PHP to start its built-in web server.
- `localhost:8000`: the address and port on which the server listens.

Python equivalent:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Breakdown:

- `python3`: executes Python.
- `-m`: runs a Python module as a script.
- `http.server`: the module providing the HTTP server.
- `8000`: the port.
- `--bind 127.0.0.1`: the local address to listen on.

**Important:** These commands do not provide identical functionality. PHP's built-in server executes PHP scripts, whereas Python's `http.server` serves static files. It does not execute Python files or PHP files as web applications.

# 2. Check whether PHP and Python are installed

| Task | PHP | Python |
|---|---|---|
| Display version | `php -v` | `python3 --version` |
| Display help | `php --help` | `python3 --help` |
| Find executable | `which php` | `which python3` |
| Find executable path | `command -v php` | `command -v python3` |

Examples:

```bash
php -v
python3 --version

which php
which python3
```

To display more detailed PHP configuration:

```bash
php -i
```

Python alternatives:

```bash
python3 -c "import sys; print(sys.version)"
python3 -m sysconfig
```

The first Python command displays the interpreter version. The second displays Python installation and build configuration information.

# 3. Run a PHP file and a Python file

Suppose you have these files:

```text
my-project/
├── index.php
└── index.py
```

### PHP

```bash
php index.php
```

You can also explicitly specify the file option:

```bash
php -f index.php
```

`-f` tells PHP which file to execute.

### Python

```bash
python3 index.py
```

Python also has a module-based way to execute files:

```bash
python3 -m mymodule
```

**Difference:**

- PHP `-f` executes a PHP file.
- Python `-m` executes an importable module.
- Python normally executes a script directly using its filename.

# 4. Execute code directly from the terminal

You do not always need to create a file before testing code.

### PHP: `-r`

```bash
php -r 'echo "Hello from PHP\n";'
```

`-r` executes PHP code directly from the command line.

Do not include `<?php` and `?>` tags when using `-r`.

### Python: `-c`

```bash
python3 -c 'print("Hello from Python")'
```

`-c` executes Python code supplied as a command-line string.

More examples:

```bash
php -r 'echo 10 + 20;'

python3 -c 'print(10 + 20)'
```

```bash
php -r '$name = "Charles"; echo $name;'

python3 -c 'name = "Charles"; print(name)'
```

**Important shell lesson:** PHP variables use `$`. In Bash, double quotes allow shell variable expansion. Prefer single quotes around PHP code containing `$variables`.

# 5. Interactive mode

Interactive mode allows you to type instructions and receive results immediately.

### PHP

```bash
php -a
```

### Python

```bash
python3
```

Python also has:

```bash
python3 -i script.py
```

`-i` runs the script and then leaves you in the interactive interpreter.

Example:

```bash
python3 -i
```

Now type:

```python
>>> name = "Charles"
>>> print(name)
Charles
>>> 10 + 20
30
```

To leave the interactive interpreter:

```python
exit()
```

Or press `Ctrl + D` in Linux.

# 6. Start a local development web server

This is one of the most important comparisons.

## PHP built-in server

```bash
php -S localhost:8000
```

Starts a PHP development server on port 8000.

Open:

```text
http://localhost:8000
```

If your project is:

```text
my-project/
├── index.php
└── about.php
```

First enter the project directory:

```bash
cd my-project
```

Then:

```bash
php -S localhost:8000
```

PHP can execute `index.php` when you visit the site.

## Python equivalent: static HTTP server

```bash
python3 -m http.server 8000
```

Open:

```text
http://localhost:8000
```

Python serves files from your current directory.

### Select a different port

PHP:

```bash
php -S localhost:8080
```

Python:

```bash
python3 -m http.server 8080
```

### Select a specific document root

PHP:

```bash
php -S localhost:8000 -t public/
```

Python:

```bash
python3 -m http.server 8000 --directory public/
```

The `-t` option in PHP selects the document root.

The `--directory` option in Python selects the directory from which files are served.

### Select a specific network interface

PHP:

```bash
php -S 127.0.0.1:8000
```

Python:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

To listen on all IPv4 interfaces:

PHP:

```bash
php -S 0.0.0.0:8000
```

Python:

```bash
python3 -m http.server 8000 --bind 0.0.0.0
```

**Security warning:** Binding to `0.0.0.0` can make the server reachable from other devices, depending on your network and firewall. Only do this in a controlled environment. Neither PHP's built-in server nor Python's basic HTTP server is intended as a production web server.

### Stop either server

Press:

```text
Ctrl + C
```

# 7. PHP and Python syntax checking

You should check your code before running it.

### PHP

```bash
php -l index.php
```

`-l` means syntax check or lint.

It checks whether the PHP file contains syntax errors without executing the program.

### Python

```bash
python3 -m py_compile index.py
```

This compiles the Python file to bytecode and reports syntax errors.

You can also compile all Python files in a directory:

```bash
python3 -m compileall .
```

Useful distinction:

- `php -l`: checks PHP syntax.
- `python3 -m py_compile`: compiles and checks a Python source file.
- Neither guarantees that the program will run correctly at runtime.

# 8. PHP modules and Python modules

### PHP

```bash
php -m
```

Displays loaded PHP modules and extensions.

To display detailed information:

```bash
php --ri json
```

This displays configuration information for the PHP JSON extension.

### Python

List installed third-party packages:

```bash
python3 -m pip list
```

Display information about a package:

```bash
python3 -m pip show requests
```

List packages:

```bash
python3 -m pip freeze
```

Important distinction:

PHP extensions and Python packages/modules are related concepts, but they are not exactly the same.

Python's standard library also contains modules that do not need to be installed separately.

For example:

```bash
python3 -c 'import os, sys, json; print("Modules available")'
```

# 9. Display PHP configuration and Python configuration

### PHP

```bash
php --ini
```

Displays configuration file locations.

```bash
php -i
```

Displays extensive PHP configuration information.

```bash
php -n -i
```

Displays PHP information without loading the normal `php.ini` configuration.

### Python

```bash
python3 -c 'import sys; print(sys.path)'
```

Displays Python's module search paths.

```bash
python3 -c 'import sys; print(sys.executable)'
```

Displays the interpreter executable path.

```bash
python3 -c 'import sysconfig; print(sysconfig.get_paths())'
```

Displays Python installation paths.

# 10. Useful Python command-line options

These options are especially useful when working with Python on Linux.

| Option | Meaning | Example |
|---|---|---|
| `-c` | Execute code directly | `python3 -c 'print(10)'` |
| `-m` | Run a module | `python3 -m http.server 8000` |
| `-i` | Enter interactive mode after execution | `python3 -i app.py` |
| `-h` | Display help | `python3 -h` |
| `-V` | Display Python version | `python3 -V` |
| `-B` | Do not write `.pyc` files | `python3 -B app.py` |
| `-O` | Enable basic optimizations | `python3 -O app.py` |
| `-OO` | Enable more optimizations, including removal of docstrings | `python3 -OO app.py` |
| `-u` | Unbuffer standard output and error | `python3 -u app.py` |
| `-v` | Verbose import messages | `python3 -v app.py` |
| `-q` | Suppress introductory interactive message | `python3 -q` |
| `-E` | Ignore Python environment variables | `python3 -E app.py` |
| `-I` | Run in isolated mode | `python3 -I app.py` |
| `-S` | Do not automatically import `site` | `python3 -S app.py` |
| `-W` | Configure warning behavior | `python3 -W error app.py` |
| `-X` | Set implementation-specific options | `python3 -X dev app.py` |

Some options deserve extra attention.

### `-B`: Do not create bytecode files

```bash
python3 -B app.py
```

Normally Python may create `__pycache__` directories containing `.pyc` files.

`-B` prevents Python from writing these bytecode files.

### `-u`: Unbuffered output

```bash
python3 -u app.py
```

Useful when you want output to appear immediately, particularly when monitoring scripts or running processes through containers and logging systems.

### `-I`: Isolated mode

```bash
python3 -I app.py
```

Runs Python in isolated mode, ignoring Python environment variables and user site-packages, among other isolation measures.

Useful when you want a more controlled execution environment.

### `-W`: Warnings

```bash
python3 -W error app.py
```

Treats warnings as errors.

This can help you discover problems during development.

### `-X`: Advanced interpreter options

```bash
python3 -X dev app.py
```

Enables Python development mode, which adds runtime checks and warnings useful during development.

# 11. More useful Python modules that have command-line interfaces

Python's `-m` option is very powerful. It allows you to run many modules directly.

| Command | Purpose |
|---|---|
| `python3 -m http.server 8000` | Start a static HTTP server |
| `python3 -m pip install requests` | Install a package |
| `python3 -m pip list` | List installed packages |
| `python3 -m pip show requests` | Display package information |
| `python3 -m venv venv` | Create a virtual environment |
| `python3 -m unittest` | Run unit tests |
| `python3 -m pytest` | Run pytest if installed |
| `python3 -m json.tool data.json` | Format JSON |
| `python3 -m py_compile app.py` | Compile a Python file |
| `python3 -m compileall .` | Compile Python files recursively |
| `python3 -m pdb app.py` | Start the Python debugger |
| `python3 -m cProfile app.py` | Profile program execution |
| `python3 -m timeit 'sum(range(100))'` | Benchmark a small expression |
| `python3 -m zipfile -c archive.zip file.txt` | Create a ZIP archive |

Note: `pytest` is a third-party package. Install it before using `python3 -m pytest`.

# 12. Build your own comparison project

Create this directory:

```bash
mkdir php-python-lab
cd php-python-lab

mkdir php-project
mkdir python-project
```

Create a PHP file:

```bash
nano php-project/index.php
```

Put this inside:

```php
<?php

$name = "Charles";

echo "Hello from PHP, $name\n";

?>
```

Run:

```bash
php php-project/index.php
```

Create a Python file:

```bash
nano python-project/index.py
```

Put this inside:

```python
name = "Charles"

print(f"Hello from Python, {name}")
```

Run:

```bash
python3 python-project/index.py
```

Now compare the two programs.

They perform the same task, but their syntax and execution differ.

# 13. A practical server comparison exercise

Create these directories:

```bash
mkdir -p php-site
mkdir -p python-site
```

Create a PHP page:

```bash
nano php-site/index.php
```

```php
<?php

echo "<h1>Hello from PHP</h1>";

?>
```

Create a Python static page:

```bash
nano python-site/index.html
```

```html
<h1>Hello from a Python HTTP server</h1>
```

Start the PHP server:

```bash
php -S localhost:8000 -t php-site
```

In another terminal, start Python:

```bash
python3 -m http.server 8001 --bind 127.0.0.1 --directory python-site
```

Visit:

```text
http://localhost:8000
http://localhost:8001
```

Observe:

- PHP executes the PHP code and returns HTML.
- Python's basic server reads and returns the HTML file.
- Python's basic server does not execute PHP or Python application code.

If you want Python to behave like a dynamic web application server, study Django, Flask or FastAPI.

---

# 14. Quick command reference

```bash
# VERSION
php -v
python3 --version

# HELP
php --help
python3 --help

# RUN A FILE
php index.php
python3 index.py

# RUN INLINE CODE
php -r 'echo "Hello";'
python3 -c 'print("Hello")'

# INTERACTIVE MODE
php -a
python3

# START A DEVELOPMENT SERVER
php -S localhost:8000
python3 -m http.server 8000

# CHOOSE DOCUMENT ROOT
php -S localhost:8000 -t public/
python3 -m http.server 8000 --directory public/

# CHOOSE PORT
php -S localhost:8080
python3 -m http.server 8080

# CHOOSE BIND ADDRESS
php -S 127.0.0.1:8000
python3 -m http.server 8000 --bind 127.0.0.1

# CHECK SYNTAX
php -l index.php
python3 -m py_compile index.py

# MODULES AND PACKAGES
php -m
python3 -m pip list

# CONFIGURATION
php --ini
python3 -m sysconfig

# STOP SERVER
Ctrl + C
```

# 15. What you should remember

1. `php -S` starts PHP's built-in development server.
2. `python3 -m http.server` starts Python's basic static file server.
3. `php -r` executes inline PHP code.
4. `python3 -c` executes inline Python code.
5. `php -l` checks PHP syntax.
6. `python3 -m py_compile` checks and compiles Python source.
7. `php -m` displays loaded PHP extensions.
8. `python3 -m pip list` displays installed Python packages.
9. PHP's `-t` selects its document root.
10. Python's `--directory` selects its serving directory.

**Best learning approach:** Do not only memorize commands. Run the PHP command, run its Python counterpart, observe the output, and understand what each option changes.
