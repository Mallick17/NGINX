# NGINX
nginx ("engine x") is an HTTP web server, reverse proxy, content cache, load balancer, TCP/UDP proxy server, and mail proxy server.

<details>
    <summary>Click to view the guide to nginx</summary>

Here’s your content rewritten into **documentation format**, aligned with the style, headings, and keywords used in the official **nginx beginner’s guide**:

---

# Beginner’s Guide to nginx

This guide provides a basic introduction to **nginx** and describes simple tasks that can be performed with it. It is assumed that nginx is already installed on the system. If it is not, see the [Installing nginx](https://nginx.org/en/docs/install.html) page.

This guide covers:

* Starting, stopping, and reloading nginx
* Understanding the configuration file’s structure
* Serving static content
* Configuring nginx as a proxy server
* Connecting nginx with a FastCGI application

---

## Master and Worker Processes

nginx has **one master process** and several **worker processes**.

* **Master process**:

  * Reads and evaluates configuration
  * Maintains worker processes

* **Worker processes**:

  * Handle actual request processing
  * Use an **event-based model** with OS-dependent mechanisms for efficiency

The number of worker processes is defined in the configuration file using the [`worker_processes`](https://nginx.org/en/docs/ngx_core_module.html#worker_processes) directive. This can either be a fixed number or automatically adjusted to the number of available CPU cores.

---

## Configuration File

The way nginx and its modules behave is determined by the **configuration file**.

* Default name: `nginx.conf`
* Common locations:

  * `/usr/local/nginx/conf`
  * `/etc/nginx`
  * `/usr/local/etc/nginx`

---

## Starting, Stopping, and Reloading Configuration

To start nginx, run the executable:

```bash
nginx
```

Once started, it can be controlled using the **-s** parameter:

```bash
nginx -s signal
```

### Supported signals:

* **stop** — fast shutdown
* **quit** — graceful shutdown
* **reload** — reload configuration
* **reopen** — reopen log files

For example, to gracefully stop nginx:

```bash
nginx -s quit
```

To reload configuration after changes:

```bash
nginx -s reload
```

### Sending signals using `kill`

You can also control nginx processes with standard Unix tools:

```bash
kill -s QUIT <master_process_id>
```

The **PID** of the master process is stored in:

* `/usr/local/nginx/logs/nginx.pid`
* `/var/run/nginx.pid`

To list running nginx processes:

```bash
ps -ax | grep nginx
```

For more, see [Controlling nginx](https://nginx.org/en/docs/control.html).

---

## Configuration File’s Structure

nginx consists of **modules** controlled by **directives**.

* **Simple directives**:

  * Name + parameters, end with `;`
* **Block directives**:

  * Same as simple directives, but with `{ ... }` containing additional instructions

Some block directives can contain other directives — these are called **contexts**. Examples:

* `events`
* `http`
* `server`
* `location`

Directives outside of any context belong to the **main context**.

* `events` and `http` → in main context
* `server` → in `http` context
* `location` → in `server` context

Anything after `#` is a **comment**.

---

## Serving Static Content

Serving static files (HTML, images, etc.) is a core function of nginx.

Example:

* `/data/www` → HTML files
* `/data/images` → image files

Create directories:

```bash
mkdir -p /data/www /data/images
echo "Hello from nginx" > /data/www/index.html
```

### Server configuration:

```nginx
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
    }
}
```

* Requests starting with `/images/` → `/data/images`
* Other requests → `/data/www`

Example:

* `http://localhost/images/example.png` → `/data/images/example.png`
* `http://localhost/page.html` → `/data/www/page.html`

Apply changes:

```bash
nginx -s reload
```

Logs are in:

* `/usr/local/nginx/logs/access.log`
* `/usr/local/nginx/logs/error.log`
* or `/var/log/nginx`

---

## Setting Up a Simple Proxy Server

nginx can act as a **proxy server** — forwarding requests to other servers.

### Step 1: Define proxied server

```nginx
server {
    listen 8080;
    root /data/up1;

    location / {
    }
}
```

* Listens on port **8080**
* Serves files from `/data/up1`

### Step 2: Configure proxy server

```nginx
server {
    location / {
        proxy_pass http://localhost:8080;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

* Requests for `.gif`, `.jpg`, `.png` → served locally
* All other requests → forwarded to proxied server on `localhost:8080`

---

## Setting Up FastCGI Proxying

nginx can also pass requests to **FastCGI servers** (e.g., PHP).

### Example configuration:

```nginx
server {
    location / {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param QUERY_STRING    $query_string;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

* Requests → FastCGI server on **localhost:9000**
* `SCRIPT_FILENAME` → determines script file
* `QUERY_STRING` → passes request parameters
* Static images (`gif/jpg/png`) → served locally

---
    
</details>

NGINX includes a wide range of modules that extend its core functionality. Here is an overview of some key NGINX modules organized by category, including official built-in and dynamic modules:

<details>
    <summary>Click to view all the Modules Supported by nginx</summary>

### Core NGINX HTTP Modules (examples)

| Module Name                   | Description                                      |
|------------------------------|------------------------------------------------|
| ngx_http_core_module         | Core HTTP module, essential for basic processing|
| ngx_http_log_module          | Handles logging of HTTP requests and responses  |
| ngx_http_stub_status_module  | Provides basic live status info for NGINX       |
| ngx_http_access_module       | Controls client access rules                      |
| ngx_http_auth_basic_module   | HTTP Basic Authentication support                |
| ngx_http_limit_conn_module   | Limits number of simultaneous connections        |
| ngx_http_limit_req_module    | Request rate limiting                             |
| ngx_http_rewrite_module      | Supports URL rewriting and redirects             |
| ngx_http_ssl_module          | SSL/TLS support for HTTP                          |
| ngx_http_gzip_module         | Gzip compression                                  |
| ngx_http_proxy_module        | Proxying HTTP requests to backend servers        |
| ngx_http_fastcgi_module      | FastCGI support for PHP and other apps           |
| ngx_http_geoip_module        | GeoIP-based IP location functions                 |
| ngx_http_headers_module      | Manipulates HTTP headers                          |

***

### Mail modules (SMTP, IMAP, POP3)

| Module Name                 | Description                                      |
|----------------------------|------------------------------------------------|
| ngx_mail_core_module       | Core mail streaming module                       |
| ngx_mail_ssl_module        | SSL/TLS support for mail protocols               |
| ngx_mail_auth_http_module  | HTTP-based mail authentication                    |
| ngx_mail_proxy_module      | Mail proxy functionality                          |

***

### Stream (TCP/UDP) Modules

| Module Name            | Description                                        |
|-----------------------|--------------------------------------------------|
| ngx_stream_core_module | Core stream module for TCP/UDP proxying           |
| ngx_stream_ssl_module  | SSL/TLS support in stream module                   |
| ngx_stream_limit_conn_module | Limits connections for stream                      |

***

### Popular Dynamic/Third-party Modules (examples)

| Module Name              | Description                                            |
|--------------------------|--------------------------------------------------------|
| ngx_http_lua_module      | Lua scripting inside NGINX                              |
| ngx_http_perl_module     | Perl scripting                                          |
| ngx_http_js_module       | JavaScript scripting (njs)                             |
| ngx_http_image_filter_module | Image processing                                        |
| ngx_http_geoip2_module   | IP Geolocation support using GeoIP2                    |
| ngx_http_brotli_filter_module | Brotli compression                                     |
| ngx_http_waf_module      | Web Application Firewall (security)                    |
| ngx_http_auth_jwt_module | JWT authentication                                     |
| ngx_http_vts_module      | Virtual host traffic status                             |
| ngx_http_redis_module    | Redis integration                                      |
| ngx_http_pagespeed_module| Google PageSpeed optimizations                          |

***

</details>

### How to list enabled modules on your system

Run this command in the terminal to see all modules NGINX was built with:

```bash
nginx -V 2>&1 | grep _module
```

This will output static and dynamic modules compiled or available.

***

### Official NGINX Documentation source of modules

The official modules list is available in NGINX documentation here:

- https://nginx.org/en/docs/

- https://nginx.org/en/docs/http/ngx_http_core_module.html (example of a core module docs)

***

## Parameters in NGINX
1. NGINX Request & Access Log Parameters
2. NGINX Backend (Upstream) Log Parameters
3. Custom Application-Specific Parameters

<details>
    <summary>Click to view the details Parameters in NGINX</summary>

### 1. **NGINX Request & Access Log Parameters**

These are **client-side and request-related values** directly handled by NGINX.
> What the client does + how NGINX sees it.

| Parameter                 | Explanation                                                            | Example Value                  |
| ------------------------- | ---------------------------------------------------------------------- | ------------------------------ |
| `$remote_addr`            | IP address of the client making the request                            | `192.168.1.1`                  |
| `$remote_user`            | Authenticated username if provided, else `-`                           | `john_doe` or `-`              |
| `$time_local`             | Local server time when request was received                            | `[12/Sep/2025:12:00:01 +0530]` |
| `$request`                | Full HTTP request line (method, URI, protocol)                         | `"GET /index.html HTTP/1.1"`   |
| `$request_method`         | HTTP method used                                                       | `GET`, `POST`                  |
| `$request_uri`            | Requested URI including query string                                   | `/api/v1/users?id=42`          |
| `$status`                 | HTTP response status code sent back to client                          | `200`, `404`                   |
| `$request_length`         | Size in bytes of the HTTP request from client                          | `564`                          |
| `$body_bytes_sent`        | Bytes sent in the response **body only** (not headers)                 | `1024`                         |
| `$bytes_sent`             | Total bytes sent (headers + body)                                      | `1120`                         |
| `$http_referer`           | Referrer URL showing where request came from                           | `"http://example.com/start"`   |
| `$http_user_agent`        | User agent string (browser, bot, CLI tool, etc.)                       | `"Mozilla/5.0..."`             |
| `$http_x_forwarded_for`   | Original client IP if behind a proxy/load balancer                     | `203.0.113.42`                 |
| `$server_name`            | Hostname of the NGINX server processing the request                    | `example.com`                  |
| `$scheme`                 | Protocol scheme used                                                   | `http` or `https`              |
| `$ssl_protocol`           | SSL/TLS version if HTTPS                                               | `TLSv1.3`                      |
| `$ssl_cipher`             | SSL cipher suite used for encryption                                   | `ECDHE-RSA-AES128-GCM-SHA256`  |
| `$request_time`           | Total time (seconds) NGINX took to process the request                 | `0.123`                        |
| `$request_id`             | Unique request identifier (helps with tracing requests across systems) | `abc123xyz`                    |
| `$sent_http_x_request_id` | Request ID passed back to client in `X-Request-Id` header              | `d4998d2e-...`                 |

---

## 2. **NGINX Backend (Upstream) Log Parameters**

> These capture how NGINX interacts with **backend servers** (API, app server, etc.).
> What happens when NGINX calls your backend

| Parameter                 | Explanation                                                         | Example Value         |
| ------------------------- | ------------------------------------------------------------------- | --------------------- |
| `$upstream_addr`          | Address of backend server handling request                          | `192.168.100.10:8000` |
| `$upstream_status`        | HTTP status returned by backend                                     | `200`, `502`          |
| `$upstream_response_time` | Time backend took to respond (seconds)                              | `0.053`               |
| `$upstream_connect_time`  | Time taken to open TCP connection to backend                        | `0.002`               |
| `$upstream_header_time`   | Time until first byte of headers received from backend              | `0.045`               |
| `$proxy_host`             | Hostname of the backend/proxy server                                | `api-backend1.local`  |
| `$proxy_port`             | Port used to connect to backend                                     | `8080`                |
| `$upstream_cache_status`  | Cache result when using NGINX cache (`HIT`, `MISS`, `BYPASS`, etc.) | `MISS`                |

---

### 3. **Custom Application-Specific Parameters**

These come from **headers set by your backend application** to measure internal timings (e.g., DB, analysis).
> What your backend reports back for deeper app insights.

| Parameter                      | Explanation                                             | Example Value |
| ------------------------------ | ------------------------------------------------------- | ------------- |
| `$upstream_http_db_read_time`  | Time backend spent **reading from DB**                  | `0.012`       |
| `$upstream_http_db_write_time` | Time backend spent **writing to DB**                    | `0.008`       |
| `$upstream_http_analysis_time` | Time backend spent on **business logic/analysis**       | `0.015`       |
| `$upstream_http_other_time`    | Time spent on **other backend tasks** not covered above | `0.010`       |

</details>

---

## Module `ngx_http_stub_status_module`

Use the NGINX stub_status module to see live connection counts and processing states in real time (Active, Reading, Writing, Waiting), and use the cumulative requests counter to infer how many have completed by taking deltas over time; NGINX Plus adds a live API with per‑zone request/response counts for precise “returned” totals by status code class.

<details>
    <summary>Click to view in detail, Explaination, Examples and Execution</summary>

### What to use
- Open source: stub_status exposes basic live metrics: Active connections, accepts, handled, requests, Reading, Writing, Waiting, plus embedded variables for these values.
- NGINX Plus: Live Activity Monitoring (dashboard and REST API) exposes detailed request and response counts, including responses by status class and per upstream/server zone, ideal for “how many returned” in real time.

### Key metrics explained
- Active connections: current live client connections, including idle keep‑alive (Waiting).
- Reading: connections where NGINX is reading the request header; Writing: connections where NGINX is sending the response; Waiting: idle keep‑alive connections awaiting a new request.
- requests (counter): total client requests since start; “returned” can be approximated as the increase in this counter over an interval minus any currently in‑flight requests, while NGINX Plus exposes explicit response counts by status class.

### Quick formulas
- In‑process now ≈ Reading + Writing, since these represent requests currently being received or responded to.
- Live connections now = Active connections, noting this includes Waiting (idle keep‑alive) as well as Reading and Writing.
- Completed/returned over interval $$ \Delta t $$ ≈ $$ \Delta\text{requests} $$, i.e., requests(t2) − requests(t1), with NGINX Plus offering direct response counters by class for precision.

### Stub_status parameters and variables
| Parameter/Variable | Explanation | Example |
|---|---|---|
| Active connections | Current number of live client connections, including Waiting (idle keep‑alive)  | 291  |
| accepts | Total number of accepted client connections since start (cumulative)  | 16630948  |
| handled | Total number of handled connections; typically equals accepts unless limits were hit  | 16630948  |
| requests | Total client requests processed since start (cumulative)  | 31070465  |
| Reading | Connections where NGINX is reading the request header (in‑flight)  | 6  |
| Writing | Connections where NGINX is writing the response (in‑flight)  | 179  |
| Waiting | Idle keep‑alive connections waiting for a request  | 106  |
| $connections_active | Embedded variable equal to “Active connections”  | 291  |
| $connections_reading | Embedded variable equal to “Reading”  | 6  |
| $connections_writing | Embedded variable equal to “Writing”  | 179  |
| $connections_waiting | Embedded variable equal to “Waiting”  | 106  |

### How to enable live view (open source)
- Add a protected location and enable stub_status, then reload NGINX, which exposes the counters above as a simple text page.
- A minimal example uses “location = /basic_status { stub_status; }”, producing output like “Active connections: 291 … Reading: 6 Writing: 179 Waiting: 106” for quick at‑a‑glance monitoring.

```nginx
location = /basic_status {
    stub_status;
    allow 127.0.0.1;  # restrict as needed
    deny all;
}
```

This produces the canonical stub_status output with Active/accepts/handled/requests/Reading/Writing/Waiting fields for live inspection and scraping by monitors that understand the format.

### How to get exact “returned” counts
- With stub_status, track “requests” as a counter and compute deltas per collection interval to approximate “requests returned,” subtracting in‑flight if needed using Reading + Writing for momentary in‑process counts.
- With NGINX Plus, use the REST API (for example, /api/<version>/http/server_zones and /api/<version>/connections) to obtain precise live request and response counters, including responses by status class, suitable for dashboards and SLOs.

### Live examples of using the **NGINX stub_status module** including configuration snippets and the output log format it produces.

### Example 1: Minimal stub_status configuration

```nginx
server {
    listen 80;
    
    location = /nginx_status {
        stub_status;        # Enables stub_status module
        allow 127.0.0.1;    # Allow localhost access only for security
        deny all;           # Deny all other IPs
    }
}
```

***

### Accessing the stub_status page

Run this command on the server or from allowed hosts:

```bash
curl http://127.0.0.1/nginx_status
```

***

### Example stub_status output

```
Active connections: 291 
server accepts handled requests
16630948 16630948 31070465
Reading: 6 Writing: 179 Waiting: 106
```

***

### Explanation of output fields:

| Field              | Description                                                                                   |
|--------------------|-----------------------------------------------------------------------------------------------|
| Active connections  | Current total active client connections including those waiting for requests (keep-alive)     |
| accepts            | Total number of accepted client TCP connections since server start                            |
| handled            | Total number of successfully handled connections (usually equals accepts)                    |
| requests           | Total number of HTTP requests processed since server start                                  |
| Reading            | Number of connections actively reading client request headers                               |
| Writing            | Number of connections actively writing responses to clients                                |
| Waiting            | Number of idle keep-alive connections waiting for new requests                             |

***

### Example 2: Adding stub_status to existing site

```nginx
location = /status {
    stub_status on;
    access_log off;             # Disable logging for status requests
    allow 192.168.1.0/24;       # Allow trusted subnet
    deny all;                   # Deny others
}
```

Curling this endpoint would show the same above status metrics.

***

### Embedded Variables (can be used in log_format)

- `$connections_active` — same as Active connections
- `$connections_reading` — same as Reading value
- `$connections_writing` — same as Writing value
- `$connections_waiting` — same as Waiting value

These variables enable inclusion of live connection stats in logs or custom dashboards.

To embed the stub_status **embedded variables** like `$connections_active`, `$connections_reading`, `$connections_writing`, and `$connections_waiting` into an NGINX configuration (e.g., for logs or monitoring), here are clear examples:

***

### Example 1: Embedding in custom access log format

```nginx
http {
    log_format connection_status '$remote_addr - $remote_user [$time_local] '
                                 '"$request" $status $body_bytes_sent '
                                 'active=$connections_active reading=$connections_reading '
                                 'writing=$connections_writing waiting=$connections_waiting '
                                 'request_time=$request_time';

    access_log /var/log/nginx/access.log connection_status;

    server {
        listen 80;
        ...
    }
}
```

This creates log entries that include live connection counts showing how many connections are active, reading request headers, writing response bodies, and waiting idle.

***

### Example log entry output with embedded connection variables

```
192.168.1.100 - - [12/Sep/2025:13:10:00 +0530] "GET /index.html HTTP/1.1" 200 1024 active=267 reading=5 writing=180 waiting=82 request_time=0.123
```

***

### Example 2: Using these variables in a status endpoint custom log

```nginx
server {
    listen 80;

    location = /status_log {
        stub_status on;
        access_log /var/log/nginx/status.log connection_status;
        allow 127.0.0.1;
        deny all;
    }
}
```

- Variables `$connections_active`, `$connections_reading`, `$connections_writing`, `$connections_waiting` can be used like any other NGINX variables in `log_format`.
- Define a `log_format` using these variables alongside other request info.
- Reference that format in `access_log` directive inside `http` or `server` context.
- The values reflect real-time states of connections when the log entry is made.
- This setup is useful for integrating live connection info directly into access or custom logs for monitoring and troubleshooting.

#### Summary
The `ngx_http_stub_status_module` is a simple yet powerful tool for live monitoring of the NGINX server status. It exposes:

- Total active connections
- Total accepted, handled, and processed requests
- Breakdown of connections currently reading requests, writing responses, and waiting idly
- This status info helps operators understand current load, diagnose issues, and tune performance.

</details>

---

## Module `ngx_http_addition_module`

### What it does

The `ngx_http_addition_module` lets you tell NGINX:

* “Before sending the normal response body, insert the response from `/before_action`.”
* “After sending the normal response body, append the response from `/after_action`.”

So NGINX will make **subrequests** to those URIs (or files) and include their responses inline.

---

### Example Configuration

```nginx
location / {
    add_before_body /before_action;
    add_after_body  /after_action;
    root /usr/share/nginx/html;
}

location /before_action {
    return 200 ">>> This is added before main response\n";
}

location /after_action {
    return 200 "\n>>> This is added after main response";
}
```

---

### Example Request/Response

#### Request

```bash
curl http://localhost/index.html
```

#### Normal `index.html` (if served directly)

```
<html>
<body>
Main page content here
</body>
</html>
```

#### With `ngx_http_addition_module` enabled

```
>>> This is added before main response

<html>
<body>
Main page content here
</body>
</html>

>>> This is added after main response
```

---

### How it can be useful

* **Injecting banners, notices, or disclaimers** before/after responses without editing the backend app.

  * Example: “System under maintenance” warning at the top of every page.
* **Debugging** → you can add debug text before or after body responses.
* **Wrapping third-party content** → if you proxy to another backend but want to prepend/append content.
* **Adding footers/headers in HTML APIs** (though for JSON APIs it’s generally not useful, because it breaks strict JSON).

---

### Limitations

* It only works for responses with a body (`text/html`, etc.), not for `HEAD` requests or responses with `Content-Length: 0`.
* It does **not** parse or understand HTML/JSON — it just blindly appends text.
* If you use it with APIs (JSON/XML), it will usually break clients unless they’re designed to handle extra text.
* **Best suited for HTML responses** where you want to inject banners, notices, or wrappers at the NGINX level.

---

