# Nginx Reverse Proxy Lab

## 1. Project Overview

This project demonstrates the implementation of an Nginx reverse proxy in front of two simple Python HTTP backend applications.

The laboratory was built using VMware Workstation, CentOS Stream 9 and Kali Linux.

**Main objectives:**
- Understand how a reverse proxy works.
- Configure Nginx to forward HTTP requests.
- Deploy two simple Python HTTP servers.
- Restrict direct network access to backend applications.
- Verify the architecture using network and HTTP tests.

## 2. Lab Architecture

| Component | IP / Port | Role |
|---|---|---|
| Kali Linux | 192.168.56.20 | Client |
| CentOS Stream 9 | 192.168.56.10 | Server |
| Nginx | Port 80 | Reverse Proxy |
| App1 | 127.0.0.1:8081 | Python Backend |
| App2 | 127.0.0.1:8082 | Python Backend |

### Request Flow

Kali Linux → Nginx → App1 or App2

The client accesses the applications through Nginx. The backend applications listen only on the local loopback interface.

## 3. Technologies Used

- VMware Workstation
- CentOS Stream 9
- Kali Linux
- Nginx
- Python 3
- HTTP
- Linux Firewall (firewalld)

## 4. Backend Applications

Two directories were created:

```bash
mkdir -p /home/khalil/lab/app1
mkdir -p /home/khalil/lab/app2
```

Simple HTML pages were created for testing:

```html
<h1>App1 - interne</h1>
```

```html
<h1>App2 - admin</h1>
```

### Starting the Python HTTP Servers

App1:

```bash
cd /home/khalil/lab/app1
python3 -m http.server 8081 --bind 127.0.0.1 &
```

App2:

```bash
cd /home/khalil/lab/app2
python3 -m http.server 8082 --bind 127.0.0.1 &
```

Python's built-in `http.server` module provides a simple HTTP server for testing and learning purposes.

## 5. Nginx Installation

```bash
sudo dnf install nginx -y
```

Enable and start Nginx:

```bash
sudo systemctl enable --now nginx
```

Check the service:

```bash
sudo systemctl status nginx
```

## 6. Nginx Reverse Proxy Configuration

Configuration file:

`/etc/nginx/conf.d/reverse-proxy.conf`

```nginx
server {
    listen 80;
    server_name _;

    location /app1/ {
        proxy_pass http://127.0.0.1:8081/;
    }

    location /app2/ {
        proxy_pass http://127.0.0.1:8082/;
    }
}
```

### Configuration Explanation

- `listen 80`: Nginx listens on HTTP port 80.
- `server_name _`: Generic server name for the lab.
- `location /app1/`: Handles requests for App1.
- `location /app2/`: Handles requests for App2.
- `proxy_pass`: Forwards requests to the corresponding backend.

Validate the configuration:

```bash
sudo nginx -t
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

## 7. Backend Network Isolation

Initially, the Python servers were listening on all IPv4 interfaces using:

```bash
--bind 0.0.0.0
```

This was changed to:

```bash
--bind 127.0.0.1
```

The loopback address ensures that the backend services accept connections only from the local machine.

Nginx can still access both applications because it runs on the same CentOS server.

This reduces the network exposure of the backend services.

## 8. Testing and Verification

### Check Listening Ports

```bash
ss -lntp | grep -E ':80|:8081|:8082'
```

Expected listening addresses:

```text
0.0.0.0:80
127.0.0.1:8081
127.0.0.1:8082
```

### Test Backend Applications Locally

```bash
curl http://127.0.0.1:8081/
curl http://127.0.0.1:8082/
```

### Test Through Nginx

```bash
curl http://127.0.0.1/app1/
curl http://127.0.0.1/app2/
```

### Test from Kali Linux

Access through Nginx:

```bash
curl http://192.168.56.10/app1/
curl http://192.168.56.10/app2/
```

Direct backend access attempts:

```bash
curl --connect-timeout 3 http://192.168.56.10:8081/
curl --connect-timeout 3 http://192.168.56.10:8082/
```

Expected results:

- Access through Nginx: successful.
- Direct access to backend ports: unsuccessful.

## 9. Security Improvements

The laboratory demonstrates two important security principles:

1. Reverse proxy: Nginx acts as the controlled entry point for client requests.
2. Backend isolation: Python applications listen only on the loopback interface.

This reduces the exposed network attack surface.

## 10. Future Improvements

Planned improvements:

- HTTPS and TLS certificates.
- Stronger firewall configuration.
- Web Application Firewall (ModSecurity + OWASP CRS).
- Secure remote access using WireGuard.
- Deployment of more realistic backend applications.

## Conclusion

This laboratory demonstrates how Nginx can act as a reverse proxy between clients and backend applications.

The backend services are isolated from direct network access, while clients can still access them through Nginx.

**Environment:** VMware Workstation — CentOS Stream 9 — Kali Linux.
