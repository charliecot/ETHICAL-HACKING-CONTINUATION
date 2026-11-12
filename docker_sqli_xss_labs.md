# Docker Vulnerable Web Applications: SQL Injection (SQLi) and XSS Labs

**Purpose:** A reference list of intentionally vulnerable web applications for practising SQL injection and Cross-Site Scripting (XSS) with Docker and Burp Suite.

> **Safety:** Run these labs only on your own computer or an isolated training network. The commands below bind published ports to `127.0.0.1` so the applications are not exposed to other machines on your network. Never deploy these vulnerable applications to a public server.

## 1. Quick comparison

| # | Application | Docker image | SQLi | XSS | Suggested host port |
|---:|---|---|:---:|:---:|---:|
| 1 | DVWA | `vulnerables/web-dvwa` | Yes | Yes | 8082 |
| 2 | OWASP Juice Shop | `bkimminich/juice-shop` | Yes | Yes | 3000 |
| 3 | bWAPP | `raesene/bwapp` | Yes | Yes | 8084 |
| 4 | OWASP Mutillidae II | `szsecurity/mutillidae` | Yes | Yes | 8083 |
| 5 | Damn Vulnerable NodeJS Application (DVNA) | `appsecco/dvna:sqlite` | Yes | Yes | 9090* |
| 6 | OWASP WebGoat | `webgoat/webgoat` | Yes (lessons) | Yes (lessons) | 8081 |
| 7 | SQLi-Labs | See project instructions | Yes | Not its focus | Check project setup |

\*DVNA's exact port mapping depends on the selected image version and its startup configuration. Follow the repository README before publishing a port.

The vulnerability coverage can vary by version and challenge/lesson. The table is a learning guide, not a guarantee that every category appears in every release.

---

## 2. DVWA — Damn Vulnerable Web Application

**Good for:** Learning SQL injection and XSS fundamentals in a deliberately vulnerable PHP/MySQL application.

### Topics

- SQL injection
- Blind SQL injection
- Reflected XSS
- Stored XSS
- Command injection
- File upload vulnerabilities
- CSRF

### Docker image

```text
vulnerables/web-dvwa
```

### Pull and run

```bash
docker pull vulnerables/web-dvwa

docker run -d \
  --name dvwa \
  -p 127.0.0.1:8082:80 \
  vulnerables/web-dvwa
```

### Open in browser

```text
http://127.0.0.1:8082
```

Complete the application's database setup page if prompted. The Docker Hub page documents the default login for this image; change credentials if you customize the lab.

**Project links:**
- Docker Hub: https://hub.docker.com/r/vulnerables/web-dvwa
- Source code: https://github.com/digininja/DVWA

---

## 3. OWASP Juice Shop

**Good for:** Testing a modern JavaScript e-commerce application and learning how to inspect browser/API traffic.

### Topics

- SQL injection challenges
- Cross-Site Scripting (XSS) challenges
- Authentication and authorization weaknesses
- Broken access control
- Insecure APIs
- Business logic flaws

### Docker image

```text
bkimminich/juice-shop
```

### Pull and run

```bash
docker pull bkimminich/juice-shop

docker run -d \
  --name juice-shop \
  -p 127.0.0.1:3000:3000 \
  bkimminich/juice-shop
```

### Open in browser

```text
http://127.0.0.1:3000
```

**Project links:**
- Docker Hub: https://hub.docker.com/r/bkimminich/juice-shop
- Source code: https://github.com/juice-shop/juice-shop
- OWASP project guide: https://owasp.org/www-project-juice-shop/

---

## 4. bWAPP — Buggy Web Application

**Good for:** Exploring a broad collection of common web application vulnerabilities.

### Topics

- SQL injection
- Blind SQL injection
- Reflected XSS
- Stored XSS
- DOM-based XSS
- Command injection
- File inclusion and other web vulnerabilities

### Docker image

```text
raesene/bwapp
```

### Pull and run

```bash
docker pull raesene/bwapp

docker run -d \
  --name bwapp \
  -p 127.0.0.1:8084:80 \
  raesene/bwapp
```

### Open in browser

```text
http://127.0.0.1:8084
```

Some bWAPP images require initial application/database setup. Check the image documentation if the application does not open immediately.

**Project links:**
- Docker Hub: https://hub.docker.com/r/raesene/bwapp
- Project information: http://www.itsecgames.com/
- Docker project: https://github.com/raesene/bwapp-docker

---

## 5. OWASP Mutillidae II

**Good for:** Practising common OWASP web vulnerabilities in a PHP application.

### Topics

- SQL injection
- Blind SQL injection
- Reflected XSS
- Stored XSS
- DOM-based XSS
- Authentication and session weaknesses
- Security misconfiguration

### Docker image

```text
szsecurity/mutillidae
```

### Pull and run

```bash
docker pull szsecurity/mutillidae

docker run -d \
  --name mutillidae \
  -p 127.0.0.1:8083:80 \
  szsecurity/mutillidae
```

### Open in browser

```text
http://127.0.0.1:8083
```

**Project links:**
- Docker Hub: https://hub.docker.com/r/szsecurity/mutillidae
- Source code: https://github.com/webpwnized/mutillidae

---

## 6. DVNA — Damn Vulnerable NodeJS Application

**Good for:** Studying vulnerabilities in a Node.js web application.

### Topics

- SQL injection
- Cross-Site Scripting (XSS)
- Authentication weaknesses
- Access control issues
- Other Node.js application security mistakes

### Docker image

```text
appsecco/dvna:sqlite
```

### Pull image

```bash
docker pull appsecco/dvna:sqlite
```

Read the project's current README for the correct run command, required environment variables, and container port. Do not assume a host port without checking the image configuration.

**Project links:**
- Docker Hub: https://hub.docker.com/r/appsecco/dvna
- Source code and setup: https://github.com/appsecco/dvna

---

## 7. OWASP WebGoat

**Good for:** Guided, lesson-based web security learning. It includes lessons covering injection and cross-site scripting.

### Topics

- SQL injection lessons
- Cross-Site Scripting lessons
- Authentication and session management
- Access control
- Other common web application security topics

### Docker image

```text
webgoat/webgoat
```

### Pull and run

```bash
docker pull webgoat/webgoat

docker run -d \
  --name webgoat \
  -p 127.0.0.1:8081:8080 \
  -p 127.0.0.1:9091:9090 \
  webgoat/webgoat
```

### Open in browser

```text
http://127.0.0.1:8081/WebGoat
```

**Project links:**
- Docker Hub: https://hub.docker.com/r/webgoat/webgoat
- Source code: https://github.com/WebGoat/WebGoat

---

## 8. SQLi-Labs

**Good for:** Concentrated SQL injection practice. This project focuses on SQL injection rather than being a general XSS lab.

### Topics

- Error-based SQL injection
- UNION-based SQL injection
- Boolean-based blind SQL injection
- Time-based blind SQL injection
- Different SQL injection scenarios

### Project

```text
https://github.com/Audi-1/sqli-labs
```

This project is commonly run with a web server and database. Docker images and third-party wrappers may differ, so use the repository's current instructions or inspect the image publisher's documentation before running an image. Avoid assuming an unofficial image is maintained or trustworthy.

---

## 9. Docker management commands

```bash
# Show running containers
docker ps

# Show all containers, including stopped ones
docker ps -a

# Show downloaded images
docker images

# View logs
docker logs dvwa

# Follow logs continuously
docker logs -f dvwa

# Stop a container
docker stop dvwa

# Start an existing stopped container
docker start dvwa

# Remove a stopped container
docker rm dvwa

# Remove an image (only after no container depends on it)
docker rmi vulnerables/web-dvwa
```

Replace `dvwa` with the container name you want to manage.

## 10. Using these labs with Burp Suite

For each local lab:

1. Start the Docker container.
2. Start Burp Suite.
3. Confirm the Burp proxy listener is running on `127.0.0.1:8080`.
4. Enable your FoxyProxy profile pointing to `127.0.0.1:8080`.
5. Keep **Proxy → Intercept** OFF while browsing normally.
6. Open the lab URL in Chrome.
7. Inspect requests under **Proxy → HTTP history**.
8. Send an interesting request to **Repeater** to study and modify it.
9. Keep testing limited to your local lab.

### Suggested learning order

1. **DVWA:** Begin with simple SQL injection and reflected/stored XSS lessons.
2. **Juice Shop:** Move to a modern e-commerce application and API traffic.
3. **bWAPP:** Explore more variations of SQLi and XSS.
4. **Mutillidae II:** Practise additional OWASP-style scenarios.
5. **WebGoat:** Use its guided lessons to strengthen your understanding.
6. **SQLi-Labs:** Focus deeply on SQL injection techniques.
7. **DVNA:** Compare how vulnerabilities appear in a Node.js application.

### Important note about ports

The host-side ports in this document are examples chosen to avoid conflicts with Burp Suite's usual port `8080`. The number before the colon is the port on your computer; the number after the colon is the port inside the container.

For example:

```bash
-p 127.0.0.1:8082:80
```

means:

- `127.0.0.1`: only your own computer can access the published port.
- `8082`: port on your computer.
- `80`: port inside the container.

If a port is already in use, choose another unused host port.

---

## References

- OWASP Vulnerable Web Applications Directory: https://owasp.org/www-project-vulnerable-web-applications-directory/
- OWASP Juice Shop: https://owasp.org/www-project-juice-shop/
- DVWA: https://github.com/digininja/DVWA
- Mutillidae II: https://github.com/webpwnized/mutillidae
- WebGoat: https://github.com/WebGoat/WebGoat
- DVNA: https://github.com/appsecco/dvna
- SQLi-Labs: https://github.com/Audi-1/sqli-labs
