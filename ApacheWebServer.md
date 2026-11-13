# Apache HTTP Server vs Nginx — Complete Learning Guide

## 1. Introduction

Apache and Nginx are two popular open-source web servers used to deliver websites and web applications over HTTP and HTTPS.

Both can:

- Receive HTTP/HTTPS requests.
- Serve static files such as HTML, CSS, JavaScript, and images.
- Handle SSL/TLS encryption.
- Act as reverse proxies.
- Route requests to backend applications.
- Produce access and error logs.
- Support virtual hosting.

However, they have different architectures, configuration approaches, and connection-handling mechanisms.

### What is Apache?

Apache HTTP Server is an open-source web server developed by the Apache Software Foundation.

It is known for:

- Flexible configuration.
- Modular architecture.
- `.htaccess` support.
- Process-based and threaded request handling through its Multi-Processing Modules (MPMs).
- Extensive compatibility with web applications.

### What is Nginx?

Nginx (pronounced Engine-X) is an open-source web server originally developed by Igor Sysoev.

It is known for:

- Event-driven architecture.
- Efficient handling of many concurrent connections.
- Reverse proxy functionality.
- Load balancing.
- Efficient static file delivery.
- Low memory overhead in many workloads.

---

# 2. Understanding the Architecture

Architecture describes how a web server internally handles incoming connections and requests.

This is one of the most important differences between Apache and Nginx.

## 2.1 Apache architecture

Apache uses Multi-Processing Modules (MPMs) to determine how it handles incoming connections.

Important MPMs include:

| MPM | Description |
|---|---|
| prefork | Uses multiple processes, generally with one request-handling process per connection |
| worker | Uses multiple processes, each containing multiple threads |
| event | Uses multiple processes and threads, with event-driven handling of connection activity |

### Prefork MPM

Each child process handles one connection at a time.

Conceptually:

```text
             Apache
                |
       -----------------
       |       |       |
    Process Process Process
       |       |       |
    Client  Client  Client
```

Advantages:

- Process isolation.
- Useful for compatibility with certain non-thread-safe modules.

Disadvantages:

- Higher memory consumption when many processes are needed.
- Less efficient for large numbers of idle connections.

### Worker MPM

Worker uses multiple processes, and each process can contain multiple threads.

```text
              Apache
                 |
          ---------------
          |             |
       Process 1     Process 2
          |             |
       Threads        Threads
       /  |  \        /  |  \
      C1  C2  C3     C4  C5  C6
```

Advantages:

- Can handle multiple connections per process.
- Generally uses less memory than prefork for comparable workloads.

### Event MPM

Event MPM builds on the threaded architecture and uses event-driven handling for connection management.

It can handle idle keep-alive connections more efficiently without tying up a worker thread for every idle connection.

**Important:** Apache is not exclusively process-based. Its architecture depends on the selected MPM.

## 2.2 Nginx architecture

Nginx uses an event-driven, asynchronous architecture.

It typically has:

- One master process.
- Multiple worker processes.
- An event-processing mechanism within each worker.

```text
                 Nginx
                   |
             Master Process
                   |
          -------------------
          |        |        |
       Worker   Worker   Worker
       Process  Process  Process
          |        |        |
       Multiple concurrent connections
```

The master process manages worker processes.

Worker processes handle incoming connections and requests using event-driven mechanisms.

Instead of creating a separate thread or process for every connection, workers can manage many connections asynchronously.

Advantages:

- Efficient connection handling.
- Low overhead for many concurrent connections.
- Good performance for static file delivery.
- Well suited to reverse proxying.

---

# 3. Apache vs Nginx: Main Differences

| Feature | Apache | Nginx |
|---|---|---|
| Architecture | MPM-based | Event-driven |
| Connection handling | Processes and/or threads | Asynchronous event handling |
| Static files | Excellent | Excellent, often very efficient |
| Dynamic applications | Modules or application gateways | Usually proxies to application servers |
| `.htaccess` | Supported | Not supported |
| Reverse proxy | Supported | Supported |
| Load balancing | Supported | Supported |
| Virtual hosting | Supported | Supported |
| SSL/TLS | Supported | Supported |
| Configuration | Flexible, directory-level options | Centralized configuration |
| Memory usage | Depends on MPM and workload | Often relatively low |
| Concurrent connections | Supported | Particularly efficient for high concurrency |

Neither server is universally better.

The correct choice depends on workload, application requirements, configuration preferences, and infrastructure.

---

# 4. Understanding Static and Dynamic Content

Before comparing their practical applications, understand these two types of content.

## 4.1 Static content

Static content consists of files that can be served directly without executing application logic.

Examples:

```text
index.html
style.css
app.js
logo.png
document.pdf
```

When a browser requests:

```http
GET /index.html HTTP/1.1
Host: example.com
```

The web server locates the file and returns it.

Both Apache and Nginx can handle this.

## 4.2 Dynamic content

Dynamic content requires application logic to generate a response.

Examples:

- User authentication.
- Database queries.
- Shopping cart operations.
- REST API endpoints.
- User registration.
- Payment processing.

For example, a Django application might receive:

```http
GET /api/products/ HTTP/1.1
Host: example.com
```

Django processes the request, queries the database, and returns JSON.

Apache or Nginx can sit in front of Django and forward requests to it.

---

# 5. Apache and Nginx as Reverse Proxies

A reverse proxy receives requests from clients and forwards them to backend servers.

Consider your Django REST API.

```text
             React Frontend
                   |
                   |
                Browser
                   |
                   | HTTPS
                   |
             Apache / Nginx
                   |
                   | Reverse Proxy
                   |
              Django API
                   |
                   |
              PostgreSQL
```

The web server acts as the public entry point.

Django handles application logic.

PostgreSQL stores application data.

### Why use a reverse proxy?

1. Hide internal application server details.
2. Handle HTTPS termination.
3. Route requests to backend applications.
4. Distribute requests across multiple backend servers.
5. Serve static files.
6. Apply certain request and connection restrictions.

---

# 6. Installing Apache on Ubuntu

Update package information:

```bash
sudo apt update
```

Install Apache:

```bash
sudo apt install apache2
```

Start Apache:

```bash
sudo systemctl start apache2
```

Enable it at startup:

```bash
sudo systemctl enable apache2
```

Check its status:

```bash
sudo systemctl status apache2
```

Test the server:

```text
http://localhost
```

You should see the Apache default welcome page.

## Important Apache commands

```bash
# Start Apache
sudo systemctl start apache2

# Stop Apache
sudo systemctl stop apache2

# Restart Apache
sudo systemctl restart apache2

# Reload configuration without a full restart
sudo systemctl reload apache2

# Check status
sudo systemctl status apache2

# Test configuration syntax
sudo apache2ctl configtest

# Display compiled configuration information
sudo apache2ctl -V

# List loaded modules
sudo apache2ctl -M
```

---

# 7. Installing Nginx on Ubuntu

Update package information:

```bash
sudo apt update
```

Install Nginx:

```bash
sudo apt install nginx
```

Start Nginx:

```bash
sudo systemctl start nginx
```

Enable it at startup:

```bash
sudo systemctl enable nginx
```

Check its status:

```bash
sudo systemctl status nginx
```

Test:

```text
http://localhost
```

You should see the Nginx default welcome page.

## Important Nginx commands

```bash
# Start Nginx
sudo systemctl start nginx

# Stop Nginx
sudo systemctl stop nginx

# Restart Nginx
sudo systemctl restart nginx

# Reload configuration
sudo systemctl reload nginx

# Check status
sudo systemctl status nginx

# Test configuration syntax
sudo nginx -t

# Display compiled configuration information
sudo nginx -V
```

---

# 8. Important Configuration Directories

## 8.1 Apache directories

| Path | Purpose |
|---|---|
| `/etc/apache2/` | Main configuration directory |
| `/etc/apache2/apache2.conf` | Main configuration |
| `/etc/apache2/ports.conf` | Listening port configuration |
| `/etc/apache2/sites-available/` | Available virtual host configurations |
| `/etc/apache2/sites-enabled/` | Enabled virtual host configurations |
| `/etc/apache2/mods-available/` | Available modules |
| `/etc/apache2/mods-enabled/` | Enabled modules |
| `/var/www/html/` | Default document root |
| `/var/log/apache2/access.log` | Access logs |
| `/var/log/apache2/error.log` | Error logs |

## 8.2
