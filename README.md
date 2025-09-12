# NGINX Configuration

## **1. Worker Processes**
### **What Are Worker Processes?**
- NGINX uses a **master process** to control **worker processes**.
- **Worker processes** handle incoming client requests (HTTP, TCP/UDP) asynchronously using an event-driven model.
- **Key Features**:
  - Single-threaded but non-blocking (handles thousands of connections per worker).
  - Stateless by design (no shared memory between workers).

### **Key Directives**
| Directive               | Description                                                                 | Common Values              |
|-------------------------|-----------------------------------------------------------------------------|----------------------------|
| `worker_processes`      | Number of worker processes.                                                 | `auto` (default), `2`, `4` |
| `worker_connections`    | Max simultaneous connections **per worker**.                                | `1024`, `2048`             |
| `worker_rlimit_nofile`  | Sets the max number of open files (sockets) per worker (overrides OS limits). | `65535`                    |

### **Formula for Max Concurrent Connections**
```
Max connections = worker_processes × worker_connections
```
- Example: `4 workers × 1024 connections = 4096 concurrent connections`.

---

## **2. Configuration Blocks**
NGINX configurations are hierarchical, organized into **blocks** (contexts). Each block inherits settings from its parent.

### **A. Main Context (Global Settings)**
- **Scope**: Top-level configuration affecting the entire NGINX instance.
- **Directives**:
  ```nginx
  user nginx;                     # Run workers as the `nginx` user
  worker_processes auto;          # Auto-detect CPU cores
  error_log /var/log/nginx/error.log warn; # Log errors with severity ≥ "warn"
  pid /run/nginx.pid;             # Process ID file location
  include /path/to/modules.conf;  # Load dynamic modules (e.g., HTTP/2, GeoIP)
  ```

### **B. Events Block (Connection Handling)**
- **Scope**: Controls how workers handle connections.
- **Directives**:
  ```nginx
  events {
    worker_connections 1024;    # Connections per worker
    use epoll;                   # Event model (epoll for Linux)
    multi_accept on;             # Accept all pending connections at once
    accept_mutex off;            # Disable mutex for high traffic
  }
  ```

### **C. HTTP Block (Web Traffic)**
- **Scope**: Settings for HTTP/S traffic (e.g., compression, logging, MIME types).
- **Directives**:
  ```nginx
  http {
    include /etc/nginx/mime.types;     # File extension → MIME type mapping
    default_type application/octet-stream; # Default MIME type
    sendfile on;                # Efficient file transfer
    tcp_nopush on;              # Optimize TCP packets
    keepalive_timeout 65;       # Keep-alive connection timeout
    client_max_body_size 100M;  # Max upload size (e.g., for file uploads)
    gzip on;                    # Enable compression
    access_log /var/log/nginx/access.log; # Request logging
  }
  ```

### **D. Server Block (Virtual Host)**
- **Scope**: Defines a virtual server (similar to Apache's `<VirtualHost>`).
- **Directives**:
  ```nginx
  server {
    listen 80;                  # Port to listen on
    server_name example.com;    # Domain name
    root /var/www/html;         # Website root directory
    index index.html;           # Default index file

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
  }
  ```

### **E. Location Block (URI Routing)**
- **Scope**: Routes requests based on URI (e.g., `/api`, `/static`).
- **Directives**:
  ```nginx
  location / {
    try_files $uri $uri/ /index.html; # Serve static files or fallback
  }

  location /api {
    proxy_pass http://backend;  # Reverse proxy to backend servers
    proxy_set_header Host $host;
  }

  location ~* \.(jpg|png|css)$ {
    expires 7d;                 # Browser caching for static assets
  }
  ```

### **F. Stream Block (TCP/UDP Traffic)**
- **Scope**: For non-HTTP traffic (e.g., database load balancing).
- **Directives**:
  ```nginx
  stream {
    upstream db_servers {
      server 10.0.0.1:3306;    # MySQL server 1
      server 10.0.0.2:3306;    # MySQL server 2
    }

    server {
      listen 3306;             # MySQL port
      proxy_pass db_servers;
    }
  }
  ```

---

## **3. Key Variables**
NGINX provides **built-in variables** for dynamic configurations and logging:

### **Common HTTP Variables**
| Variable                  | Example Value               | Use Case                          |
|---------------------------|-----------------------------|-----------------------------------|
| `$host`                   | `example.com`               | Requested hostname.               |
| `$uri`                    | `/api/users`                | Current request URI.              |
| `$args`                   | `name=john&age=30`          | Query string parameters.          |
| `$request_method`         | `GET`, `POST`               | HTTP method.                      |
| `$remote_addr`            | `192.168.1.100`             | Client IP address.                |
| `$status`                 | `200`, `404`                | HTTP response status code.        |
| `$request_time`           | `0.005` (seconds)           | Time taken to process the request.|
| `$upstream_addr`          | `10.0.0.1:8080`             | Backend server address.           |
| `$http_[header]`          | `$http_user_agent`          | Access request headers.           |
| `$sent_http_[header]`     | `$sent_http_content_type`   | Access response headers.          |

### **Example: Advanced Logging**
```nginx
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" "$http_user_agent" '
                'Upstream: $upstream_addr Response time: $request_time';
```

---

## **4. Advanced Parameters & Tuning**
### **Performance Optimization**
| Directive                | Purpose                                                                 | Example Value      |
|--------------------------|-------------------------------------------------------------------------|--------------------|
| `keepalive_requests`     | Max requests per keep-alive connection.                                 | `1000`             |
| `open_file_cache`        | Cache metadata for frequently accessed files (e.g., static assets).     | `max=1000 inactive=20s` |
| `gzip_min_length`        | Skip compressing files smaller than this size.                          | `256` (bytes)      |
| `client_body_buffer_size`| Buffer size for client request bodies.                                  | `16k`              |

### **Security Hardening**
| Directive                | Purpose                                                                 | Example Value      |
|--------------------------|-------------------------------------------------------------------------|--------------------|
| `add_header`             | Add security headers (e.g., CSP, HSTS).                                 | `add_header X-Content-Type-Options "nosniff";` |
| `ssl_protocols`          | Restrict TLS versions.                                                  | `TLSv1.2 TLSv1.3` |
| `limit_req_zone`         | Rate limiting to prevent DDoS.                                          | `zone=mylimit:10m rate=10r/s;` |

### **Reverse Proxy**
| Directive                | Purpose                                                                 | Example Value      |
|--------------------------|-------------------------------------------------------------------------|--------------------|
| `proxy_buffer_size`      | Buffer size for proxied responses.                                      | `4k`               |
| `proxy_connect_timeout`  | Timeout for connecting to upstream.                                     | `60s`              |
| `proxy_cache_path`       | Define a cache zone for proxied content.                                | `path=/var/cache/nginx keys_zone=mycache:10m` |

---

## **5. Example: Full Configuration Skeleton**
```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /run/nginx.pid;

events {
  worker_connections 2048;
  multi_accept on;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" "$http_user_agent"';

  access_log /var/log/nginx/access.log main;

  sendfile on;
  tcp_nopush on;
  keepalive_timeout 65;
  client_max_body_size 100M;

  gzip on;
  gzip_types text/plain text/css application/json;

  server {
    listen 80;
    server_name example.com;

    location / {
      root /var/www/html;
      index index.html;
    }

    location /api {
      proxy_pass http://backend;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  }
}
```

---

## **6. Troubleshooting Tips**
1. **Check Error Logs**: `tail -f /var/log/nginx/error.log`.
2. **Validate Config**: `nginx -t`.
3. **Monitor Connections**: `ss -tan | grep :80 | wc -l`.
4. **Tune Kernel Parameters**: Adjust `net.core.somaxconn` and `ulimit -n`.

---

### **Global Context (Main Block)**
```nginx
user nginx;
```
- **What**: Runs NGINX worker processes under the `nginx` user account.
- **Why**: Security best practice (avoid running as root).
- **Parameters**: 
  - `user www-data;` (common on Debian systems)
  - `user nobody;` (minimal privileges)

```nginx
worker_processes auto;
```
- **What**: Sets the number of worker processes.
- **Why**: `auto` sets it to match CPU cores for optimal performance.
- **Parameters**: 
  - Number: `2`, `4`, `8` (manual override)
  - `auto` (default recommended)

```nginx
error_log /var/log/nginx/error.log;
```
- **What**: Defines error log location and severity level.
- **Why**: Critical for debugging issues.
- **Parameters**: 
  - `error_log /path/to/file.log warn;` (log warnings and above)
  - Levels: `debug`, `info`, `notice`, `warn`, `error`, `crit`

```nginx
pid /run/nginx.pid;
```
- **What**: Stores the master process ID.
- **Why**: Required for process management (e.g., `systemctl reload nginx`).

```nginx
include /usr/share/nginx/modules/*.conf;
```
- **What**: Loads dynamic modules (e.g., HTTP/2, GeoIP).
- **Why**: Modular architecture for extensibility.

---

### **Events Block**
```nginx
events {
    worker_connections 1024;
}
```
- **What**: Sets max simultaneous connections per worker.
- **Why**: Key for handling high traffic.
- **Parameters**:
  - `worker_connections 2048;` (increase if `ulimit -n` allows)
  - Total connections = `worker_processes * worker_connections`

---

### **HTTP Block**
#### **Logging**
```nginx
log_format main '...custom format...';
```
- **What**: Custom access log format with extended fields.
- **Why**: Enhanced monitoring (response times, rate limits, user tracking).
- **Variables**:
  - `$request_time`: Total request processing time.
  - `$upstream_http_*`: Headers from backend servers.

```nginx
access_log /var/log/nginx/access.log main;
```
- **What**: Enables access logging with the `main` format.
- **Why**: Traffic analysis and auditing.

#### **Compression**
```nginx
gzip on;
gzip_proxied any;
gzip_comp_level 6;
gzip_types text/plain ...;
```
- **What**: Enables GZIP compression for specific MIME types.
- **Why**: Red bandwidth usage and improves performance.
- **Parameters**:
  - `gzip_comp_level 1-9` (1=fastest, 9=best compression)
  - `gzip_min_length 256;` (skip small files)
  - Add MIME types like `font/woff2`

#### **Performance Tuning**
```nginx
sendfile on;
tcp_nopush on;
tcp_nodelay on;
```
- **What**: Optimizes data transfer over TCP.
- **Why**: Reduces latency and packet count.
- **Parameters**:
  - `sendfile_max_chunk 128k;` (prevents worker blocking)

```nginx
keepalive_timeout 65;
```
- **What**: Keep-alive connection timeout.
- **Why**: Reduces TCP handshake overhead.
- **Parameters**:
  - `keepalive_timeout 30s;` (adjust based on traffic)

```nginx
types_hash_max_size 2048;
```
- **What**: Sets hash table size for MIME types.
- **Why**: Prevents hash collisions with many types.

```nginx
client_max_body_size 30M;
```
- **What**: Max upload size for client requests.
- **Why**: Avoids `413 Payload Too Large` errors.
- **Parameters**:
  - `100M` (for file upload services)

#### **MIME Types**
```nginx
include /etc/nginx/mime.types;
default_type application/octet-stream;
```
- **What**: Maps file extensions to MIME types.
- **Why**: Ensures browsers handle files correctly.

---

### **Modular Configuration**
```nginx
include /etc/nginx/conf.d/*.conf;
```
- **What**: Loads server blocks or additional configs.
- **Why**: Organize configurations (e.g., `example.com.conf`).

---

### **Key Parameters & Tuning Considerations**
| Directive                | Common Parameters                     | Use Case Example                   |
|--------------------------|---------------------------------------|------------------------------------|
| `worker_processes`       | `auto`, `2`, `4`                      | Match CPU cores for parallelism    |
| `worker_connections`     | `1024`, `2048`                        | High-traffic servers               |
| `keepalive_timeout`      | `30`, `65` (seconds)                  | API servers with frequent requests |
| `gzip_comp_level`        | `1` (fast) to `9` (best compression)  | Static content vs. CPU tradeoff    |
| `client_max_body_size`   | `10M`, `100M`                         | File upload services               |
| `access_log`/`error_log` | `off`, custom paths                   | Disable logging for performance    |

### **Security & Optimization Tips**
1. **User Context**: Always run workers as non-root.
2. **Rate Limiting**: Add `limit_req_zone` in `http` block.
3. **SSL**: Enable TLS 1.3, HSTS, and modern ciphers.
4. **Caching**: Configure `proxy_cache` for static assets.
5. **Headers**: Add security headers (CSP, X-Content-Type).

---
