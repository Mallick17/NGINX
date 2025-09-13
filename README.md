# NGINX
nginx ("engine x") is an HTTP web server, reverse proxy, content cache, load balancer, TCP/UDP proxy server, and mail proxy server.

<details>
    <summary>Click to view the guide to nginx</summary>

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
### Module `ngx_http_core_module`
### Module `ngx_http_api_module`

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
The ngx_http_addition_module module is a filter that adds text before and after a response. This module is not built by default, it should be enabled with the --with-http_addition_module configuration parameter.

* “Before sending the normal response body, insert the response from `/before_action`.”
* “After sending the normal response body, append the response from `/after_action`.”

So NGINX will make **subrequests** to those URIs (or files) and include their responses inline.

<details>
    <summary>Click to view the Example and How to Use</summary>

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

<details>
    <summary>Real Life Scenario</summary>

### Scenario

You want to show a **“Maintenance Notice”** on top of every web page **without touching your Laravel/Swoole app**.

---

### NGINX Config Example

```nginx
server {
    listen 80;
    server_name myapp.local;

    root /var/www/html/public;

    # Main application
    location / {
        add_before_body /maintenance_notice;
        add_after_body  /footer_notice;
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Maintenance banner (prepended to all responses)
    location /maintenance_notice {
        default_type text/html;
        return 200 "<div style='background:red;color:white;padding:10px;text-align:center;'>
                      🚧 Maintenance ongoing: some features may be unavailable 🚧
                    </div>";
    }

    # Footer banner (appended to all responses)
    location /footer_notice {
        default_type text/html;
        return 200 "<div style='background:#333;color:#ccc;padding:10px;text-align:center;'>
                      © 2025 My Company – All Rights Reserved
                    </div>";
    }
}
```

---

### What Happens

#### Original App Response (`/`)

```html
<html>
<body>
<h1>Welcome to My App</h1>
<p>Main content goes here.</p>
</body>
</html>
```

#### Modified Response with `ngx_http_addition_module`

```html
<div style='background:red;color:white;padding:10px;text-align:center;'>
  🚧 Maintenance ongoing: some features may be unavailable 🚧
</div>

<html>
<body>
<h1>Welcome to My App</h1>
<p>Main content goes here.</p>
</body>
</html>

<div style='background:#333;color:#ccc;padding:10px;text-align:center;'>
  © 2025 My Company – All Rights Reserved
</div>
```

---

### Why It’s Useful

* **Zero code changes** in your app.
* Can be **enabled/disabled quickly** at the NGINX layer.
* Works for **all requests** (HTML responses).
* Great for banners, warnings, or compliance notices.

---

### Limitations

* Don’t use for **JSON APIs** → extra HTML will break clients.
* Subrequests (`/maintenance_notice`, `/footer_notice`) are full NGINX requests → avoid making them heavy.
* Best suited for **HTML websites**, not REST/GraphQL APIs.
* This gives you a fast, reversible way to **inject messages** into every page served by NGINX.
   
</details>

</details>

---

## Module `ngx_http_auth_basic_module`
It lets you **protect a page or a whole site with a username and password**.

* The browser will pop up a login box 🪪
* The user must enter the correct username + password before they can see the page.
* This is called **HTTP Basic Authentication**.

<details>
    <summary>Click to view Configuration </summary>

## Example Configuration

```nginx
location / {
    auth_basic "closed site";
    auth_basic_user_file conf/htpasswd;
}
```

* `auth_basic "closed site";` → Enables the login prompt.

  * `"closed site"` is the *realm* → it’s just the message that shows in the login popup.
* `auth_basic_user_file conf/htpasswd;` → The file where valid usernames and passwords are stored.

---

## Example of the password file (`conf/htpasswd`)

```
# comment line
alice:$apr1$ajf92sl.$X9ZcL8zj1fP0J8hiUZkS/
bob:$apr1$98fh29s.$kZpTjUq2IefB3R5hNQkty0
```

* `alice` and `bob` are usernames.
* The long string is the encrypted password.
* This file can be created with tools like:

  * `htpasswd` (from Apache utils)
  * `openssl passwd -apr1`

---

## How it looks to the user

1. User opens `http://example.com/`.
2. Browser shows a popup:

   ```
   Authentication Required
   Username: [   ]
   Password: [   ]
   ```
3. If user enters `alice` + correct password → access granted.
4. If wrong → NGINX returns `401 Unauthorized`.

---

## When it’s useful

* Protecting **admin panels** or **test environments**.
* Quickly securing a site without adding login logic in the app.
* Restricting **internal dashboards** (like `/nginx_status`).
* Adding an extra layer of protection before exposing something sensitive.

---

## Security Notes

* Passwords are **hashed** in the `htpasswd` file (not plain text).
* Avoid using SHA-1 (`SHA`, `SSHA`) because it’s weak. Use **MD5 (`apr1`)** or **bcrypt (via Apache htpasswd with `-B`)**.
* Use HTTPS when enabling basic auth, otherwise credentials are sent as plain text over the network.

> _In short:_
> `ngx_http_auth_basic_module` = **simple password gate** you can put in front of any location in NGINX.

<details>
    <summary>Click to view the configuration</summary>

## Configuration
### Step 1. Create a password file

On your server, run:

```bash
sudo sh -c "echo -n 'admin:' >> /etc/nginx/.htpasswd"
sudo sh -c "openssl passwd -apr1 'StrongPassword123' >> /etc/nginx/.htpasswd"
```

This creates a user `admin` with password `StrongPassword123`.
File: `/etc/nginx/.htpasswd`

---

### Step 2. Update NGINX config

In your `server {}` block:

```nginx
location /nginx_status {
    stub_status;

    # Step 1: Restrict by IP
    allow 127.0.0.1;       # localhost
    allow 192.168.1.100;   # your office IP (example)
    deny all;              # everyone else denied

    # Step 2: Require Basic Auth
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;

    # Optional: log hits
    access_log /var/log/nginx/access.log main;
}
```

---

### How it works

1. If someone from an unauthorized IP tries → they get **403 Forbidden**.
2. If someone from an allowed IP tries:

   * They see a **username/password popup**.
   * Must enter `admin / StrongPassword123`.
   * If correct → NGINX shows the stub\_status page.
3. Access attempts are logged in `/var/log/nginx/access.log`.

---

### Example Flow

#### Correct client (127.0.0.1)

```bash
curl -u admin:StrongPassword123 http://127.0.0.1/nginx_status
```

Output:

```
Active connections: 3
server accepts handled requests
 1042 1042 1094
Reading: 0 Writing: 1 Waiting: 2
```

### Wrong password

```bash
curl -u admin:wrong http://127.0.0.1/nginx_status
```

Output:

```
401 Authorization Required
```

### Unauthorized IP

```bash
curl http://203.0.113.50/nginx_status
```

Output:

```
403 Forbidden
```

---

> This way `/nginx_status` is protected by **two layers**:

* **IP filtering** (only trusted networks allowed)
* **Basic auth** (username/password gate)
    
</details>

<details>
    <summary>Supported Password Types</summary>

## Supported password types
### Password file basics

* The file (`htpasswd`) holds usernames and passwords.
* Each line looks like:

  ```
  username:encrypted_password[:comment]
  ```
* NGINX checks the password the user enters against the encrypted value in the file.

---

### Supported password types

1. **crypt() function (traditional UNIX crypt)**

   * Oldest method.
   * Example entry:

     ```
     alice:SaQYwJz6hG0wQ
     ```
   * Generated using:

     ```bash
     openssl passwd mypassword
     ```
   * Not recommended today (weak).

---

2. **Apache MD5 (`apr1`)**

   * Safer than plain `crypt`.
   * Example entry:

     ```
     bob:$apr1$1d9a7dC2$7DXMcKbnY4Qo1r4CuYxvL.
     ```
   * Generated using:

     ```bash
     htpasswd -m /etc/nginx/.htpasswd bob
     # or
     openssl passwd -apr1 "mypassword"
     ```
   *  Commonly used and supported.

3. **RFC 2307 style → `{scheme}data`**

   * Format: `{SCHEME}encoded_password`
   * Schemes supported in NGINX:

     * **PLAIN** → `{PLAIN}mypassword` (⚠️ never use, password is cleartext).
     * **SHA** → `{SHA}base64_of_sha1(password)` (⚠️ unsafe, unsalted SHA-1).
     * **SSHA** → `{SSHA}base64_of_sha1(password+salt)` (slightly better, used in LDAP/Dovecot).

   Example (SHA-1 hashed):

   ```
   carol:{SHA}W6ph5Mm5Pz8GgiULbPgzG37mj9g=
   ```

---

### Security Notes

* **Best today:**

  * Use **MD5-apr1 (`-m`)** or **bcrypt (`-B`)** if your `htpasswd` tool supports it.
  * Example bcrypt entry:

    ```
    david:$2y$05$zA5O4/Ph3EZ/CMECnM8Xe.3tC8zJ5o4kOt/yHcSvTj5R3AN/uzqZW
    ```
* **Avoid:**

  * PLAIN (obvious reasons)
  * SHA (unsalted SHA-1, easily cracked with rainbow tables)

---

> So in simple terms:

* You can use `htpasswd` or `openssl passwd` to create these entries.
* NGINX supports **crypt, apr1-MD5, SHA/SSHA, and RFC 2307-style formats**.
* For modern security, stick with **apr1-MD5** or **bcrypt**.
    
</details>

</details>

---

## Module `ngx_http_auth_jwt_module`
* This module lets NGINX check incoming requests for a **JWT token**.
* JWTs are often used in modern APIs, OAuth2, and OpenID Connect (OIDC).
* The token is usually sent in the `Authorization: Bearer <token>` header, a cookie, or query parameter.
* NGINX validates the JWT:

  * Verifies the **signature** with a public/private key.
  * Optionally decrypts it (if JWE).
  * Checks expiry (`exp`) and “not before” (`nbf`) times.
  * You can enforce claims (like `iss`, `aud`, `role`).

> If the token is valid → request continues.
> If invalid → NGINX rejects with `401 Unauthorized` (or `403 Forbidden` if configured).

<details>
    <summary>Click to view the Configuration</summary>

### Example configuration

```nginx
server {
    listen 80;

    location /api/ {
        auth_jwt "Secure API";                     # Realm name
        auth_jwt_key_file /etc/nginx/jwt-keys.json; # Public keys in JWKS format

        # Example: enforce a claim
        auth_jwt_claim_set $email email;
        auth_jwt_require $email;   # only allow if JWT has an "email" claim
    }
}
```

* Now if a request comes without a **valid JWT**, NGINX returns `401 Unauthorized`.
* If JWT is valid, request passes through to your backend.

---

### Supported Algorithms

This module is pretty complete (but only with **NGINX Plus**, not OSS):

* **JWS (signed tokens):**

  * HMAC (HS256, HS384, HS512)
  * RSA (RS256, RS384, RS512)
  * ECDSA (ES256, ES384, ES512)
  * EdDSA (Ed25519, Ed448)
  * Probabilistic RSA (PS256, PS384, PS512)
* **JWE (encrypted tokens):**

  * AES-CBC + HMAC, AES-GCM, AES Key Wrap, RSA-OAEP, direct mode.
* **Nested JWTs**: signed first, then encrypted.

---

### Real-world use cases

* **API Gateway / Reverse Proxy**: NGINX checks the token *before* it hits your app.
* **OIDC / OAuth2 integration**: Validate tokens issued by Keycloak, Okta, Auth0, etc.
* **Microservices**: Each request carries a JWT → NGINX enforces validity at the edge.
* **Zero Trust setups**: Combine with IP restriction (`ngx_http_access_module`) or Basic Auth.

---

### How it differs from `auth_basic`

* `auth_basic` → static username/password from a file (`htpasswd`).
* `auth_jwt` → dynamic tokens signed by an Identity Provider (IdP).

  * More secure for APIs.
  * Supports role-based access (`claims`).
  * Works with modern OAuth2/OpenID systems.

---

### Why it’s commercial (NGINX Plus only)

* JWT validation requires parsing, crypto verification, possibly decryption.
* That’s more advanced than OSS NGINX modules.
* That’s why **`ngx_http_auth_jwt_module` is only in NGINX Plus**, not free OSS NGINX.

---

> In simple words:
* **`auth_basic`** = “password gate” (static).
* **`auth_jwt`** = “token gate” (dynamic, for modern APIs).

</details>

<details>
    <summary>Click to view the Setup</summary>

### Goal

We want to protect `/api/*` routes behind **JWT authentication** using OSS NGINX.
Since OSS NGINX doesn’t support JWT directly, we’ll use:

* **NGINX `auth_request`** → asks a helper service if a token is valid.
* **Validator service** → checks the JWT (signature, expiry, claims).
* **Backend app** → only gets the request if token is valid.

---

### Step 1. Install tools

* NGINX OSS installed (e.g. `apt install nginx` or `yum install nginx`).
* Python (for our simple JWT validator demo).

  ```bash
  pip install flask pyjwt
  ```

---

### Step 2. Create JWT Validator Service

Make a small Python script `validator.py`:

```python
from flask import Flask, request, jsonify
import jwt, datetime

app = Flask(__name__)
SECRET = "mysecret"  # same secret used when creating JWTs

@app.route("/validate", methods=["GET"])
def validate():
    auth_header = request.headers.get("Authorization")
    if not auth_header or not auth_header.startswith("Bearer "):
        return jsonify({"error": "Missing token"}), 401

    token = auth_header.split(" ")[1]

    try:
        payload = jwt.decode(token, SECRET, algorithms=["HS256"])
        return jsonify({"ok": True, "user": payload.get("sub")}), 200
    except jwt.ExpiredSignatureError:
        return jsonify({"error": "Token expired"}), 401
    except jwt.InvalidTokenError:
        return jsonify({"error": "Invalid token"}), 401

if __name__ == "__main__":
    app.run(port=9000)
```

Run it:

```bash
python validator.py
```

> Now you have a service at `http://127.0.0.1:9000/validate` that says if a JWT is good or bad.

---

#### Step 3. Configure NGINX

Edit `/etc/nginx/conf.d/jwt.conf`:

```nginx
server {
    listen 80;

    # Protect all /api routes
    location /api/ {
        auth_request /jwt-auth;          # check token first
        proxy_pass http://127.0.0.1:8080; # your backend app
    }

    # Internal location to validate token
    location = /jwt-auth {
        internal;
        proxy_pass http://127.0.0.1:9000/validate;  # validator service
        proxy_set_header Authorization $http_authorization;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
    }

    # Handle failed auth (401/403)
    error_page 401 = @error401;
    location @error401 {
        return 401 '{"error":"Invalid or expired token"}';
        add_header Content-Type application/json;
    }
}
```

Reload NGINX:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

### Step 4. Generate a JWT for testing

Make a Python script `make_token.py`:

```python
import jwt, datetime

SECRET = "mysecret"
payload = {
    "sub": "user123",
    "role": "admin",
    "exp": datetime.datetime.utcnow() + datetime.timedelta(minutes=5) # expires in 5 minutes
}

token = jwt.encode(payload, SECRET, algorithm="HS256")
print(token)
```

Run it:

```bash
python make_token.py
```

Copy the printed token.

---

### Step 5. Test the Setup

1. **Valid Token**

```bash
curl -H "Authorization: Bearer <PASTE_TOKEN>" http://localhost/api/test
```

* NGINX calls validator → validator says token is OK → NGINX forwards to your backend app (port 8080).

2. **No Token**

```bash
curl http://localhost/api/test
```

Response:

```json
{"error":"Invalid or expired token"}
```

3. **Expired/Invalid Token**

```bash
curl -H "Authorization: Bearer BADTOKEN" http://localhost/api/test
```

Response:

```json
{"error":"Invalid or expired token"}
```

---

### Step 6. Where new tokens come from

> NGINX **does not** create tokens.

* Your **app (Laravel, Flask, etc.)** must provide a `/auth/login` endpoint.
* That endpoint issues a JWT after username/password login.
* Clients store the token and send it in every request.
* If token expires, client must call `/auth/refresh` or `/auth/login` again.

---

### Summary

1. **NGINX** → gatekeeper. Uses `auth_request` to check JWTs.
2. **Validator service** → checks if JWT is valid.
3. **Backend app** → only gets traffic if JWT is valid.
4. **NGINX never creates tokens** → your app or IdP must issue them.

</details>

---

## Module `ngx_http_auth_request_module`
* It lets NGINX **ask another service** (via a subrequest) whether a client request should be allowed.
* Think of it like:

  > "Hey validator, is this request allowed?"

  * If the validator says **200 OK** → NGINX allows the request.
  * If the validator says **401 or 403** → NGINX blocks the request with that code.
  * Any other response code → NGINX treats it as an error.

  > This is how we plug JWT validation into OSS NGINX.

<details>
    <summary>Click to view Scenario, Examples and Uses</summary>

### Example from docs

```nginx
location /private/ {
    auth_request /auth;
    proxy_pass http://backend_app;
}

location = /auth {
    proxy_pass http://127.0.0.1:9000/validate;
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
    proxy_set_header X-Original-URI $request_uri;
}
```

* `/private/` → protected resource.
* `/auth` → subrequest target, forwards to a validator service.
* If validator says 200 → `/private/` request continues.
* If validator says 401/403 → access denied.

---

### Scenario

You have a **Laravel app** running on `http://127.0.0.1:8000/api/hello`.
You only want users with a **valid JWT** to access this endpoint.

So:

* **Client** sends request with `Authorization: Bearer <token>`.
* **NGINX** asks a **validator service**: *“Is this token valid?”*
* **Validator** checks the token:

  * If valid → returns `200 OK`.
  * If invalid/expired → returns `401 Unauthorized`.
* **NGINX**:

  * If validator said 200 → forwards request to Laravel.
  * If validator said 401 → blocks the request.

---

### Useful Directives

#### `auth_request`

* Enables the subrequest check.
* Example:

  ```nginx
  auth_request /auth;
  ```

  → every request to this location will be checked against `/auth`.

#### `auth_request_set`

* Lets you capture values from the validator response headers and use them as NGINX variables.
* Example:

  ```nginx
  auth_request_set $user_id $upstream_http_x_user_id;
  proxy_set_header X-User-ID $user_id;
  ```

  → if the validator includes `X-User-ID: 123` in its response headers, NGINX can forward that to the backend app.

This is powerful: you can pass **user\_id, roles, claims** from the JWT validator into your backend.

---

### Why It’s Useful

* Works with **JWT, OAuth2, API keys, custom logic** → anything your validator service can check.
* Decouples NGINX from application logic.
* You can reuse the validator for multiple apps.
* Can combine with:

  * **`ngx_http_access_module`** (IP restriction)
  * **`ngx_http_auth_basic_module`** (username/password)
  * **satisfy directive** (e.g., allow if *either* IP or JWT is valid).

> So in simple terms:
* `auth_request` = **NGINX asks another service if request is allowed**.
* `auth_request_set` = **grab details from that answer and pass to backend**.

</details>

---

<details>
    <summary>Click to view Setup</summary>

### Step 1. JWT Validator Service

Create a small Python file `validator.py`:

```python
from flask import Flask, request, jsonify
import jwt, datetime

app = Flask(__name__)
SECRET = "mysecret"

@app.route("/validate", methods=["GET"])
def validate():
    auth_header = request.headers.get("Authorization", "")
    if not auth_header.startswith("Bearer "):
        return jsonify({"error": "Missing token"}), 401

    token = auth_header.split(" ")[1]
    try:
        payload = jwt.decode(token, SECRET, algorithms=["HS256"])
        return jsonify({"ok": True}), 200
    except jwt.ExpiredSignatureError:
        return jsonify({"error": "Expired"}), 401
    except jwt.InvalidTokenError:
        return jsonify({"error": "Invalid"}), 401

if __name__ == "__main__":
    app.run(port=9000)
```

Run it:

```bash
pip install flask pyjwt
python validator.py
```

---

### Step 2. NGINX Config

Edit `/etc/nginx/conf.d/jwt.conf`:

```nginx
server {
    listen 80;

    # Protected API
    location /api/ {
        auth_request /jwt-auth;          # ask validator first
        proxy_pass http://127.0.0.1:8000; # your Laravel app
    }

    # Internal location for auth check
    location = /jwt-auth {
        internal;
        proxy_pass http://127.0.0.1:9000/validate;
        proxy_set_header Authorization $http_authorization;

        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
    }

    # Custom error for unauthorized
    error_page 401 = @error401;
    location @error401 {
        return 401 '{"error":"Invalid or expired token"}';
        add_header Content-Type application/json;
    }
}
```

Reload:

```bash
nginx -t && systemctl reload nginx
```

---

### Step 3. Generate a Test JWT

Make `make_token.py`:

```python
import jwt, datetime
SECRET = "mysecret"

payload = {
    "sub": "user123",
    "exp": datetime.datetime.utcnow() + datetime.timedelta(minutes=5)
}
token = jwt.encode(payload, SECRET, algorithm="HS256")
print(token)
```

Run:

```bash
python make_token.py
```

---

### Step 4. Test

1. **Valid token**:

```bash
TOKEN=$(python make_token.py)
curl -H "Authorization: Bearer $TOKEN" http://localhost/api/hello
```

> Goes through to Laravel.

2. **No token**:

```bash
curl http://localhost/api/hello
```

❌ Response:

```json
{"error":"Invalid or expired token"}
```

3. **Expired/invalid token**:

```bash
curl -H "Authorization: Bearer badtoken" http://localhost/api/hello
```

❌ Response:

```json
{"error":"Invalid or expired token"}
```

---

### In Simple Words

* Client knocks on `/api/hello`.
* NGINX whispers to validator: *“Hey, is this token good?”*
* Validator replies:

  * *Yes (200)* → Laravel gets the request.
  * *No (401)* → NGINX stops it.
    
</details>

---

## Module `ngx_http_auth_require_module` (NGINX Plus)
* Lets you **enforce access rules based on variables**.
* Works well with variables from:

  * `auth_jwt` (JWT claims),
  * `auth_oidc` (OIDC claims),
  * `auth_request` (custom validator headers).
* Access is allowed only if the variable is **non-empty** and not equal to `"0"`.
* Otherwise → request is denied with `403` (or another error you specify).

> Think of it like:
> “Even if the token is valid, don’t let the request pass unless the user’s claim/role matches my rule.”

<details>
    <summary>Click to view Examples</summary>

### Example Scenario 1 — Admin-only Dashboard

Your app has `/admin/*` routes. You only want users with `"role": "admin"` in their JWT to access it.

### Config

```nginx
http {
    # Extract role claim from JWT (requires ngx_http_auth_jwt_module)
    auth_jwt          "restricted";
    auth_jwt_key_file conf/keys.json;

    # Map JWT role → allowed flag
    map $jwt_claim_role $is_admin {
        "admin" 1;
    }

    server {
        listen 80;

        location /admin/ {
            # Require admin role
            auth_require $is_admin;
        }

        location / {
            return 200 "Hello, public!\n";
        }
    }
}
```

#### Flow

* JWT decoded → `role=admin` → `$is_admin=1` → request allowed.
* JWT decoded → `role=user` → `$is_admin=0` → NGINX returns `403 Forbidden`.

---

### Example Scenario 2 — Multiple Conditions

Require both:

* role must be `"manager"`,
* department must be `"finance"`.

```nginx
map $jwt_claim_role $is_manager {
    "manager" 1;
}

map $jwt_claim_department $is_finance {
    "finance" 1;
}

server {
    listen 80;

    location /finance-reports/ {
        auth_require $is_manager;
        auth_require $is_finance error=401; # custom error
    }
}
```

* If both match → allow.
* If role fails → `403`.
* If department fails → `401`.

---

### Example Scenario 3 — Combine with `auth_request`

If you don’t use NGINX Plus JWT/OIDC modules, you can still use `auth_request` + `auth_require`.

1. Validator service returns headers:

   ```
   X-Role: admin
   X-Department: finance
   ```

2. NGINX captures them:

   ```nginx
   auth_request_set $user_role $upstream_http_x_role;
   auth_request_set $user_dept $upstream_http_x_department;

   map $user_role $is_admin {
       "admin" 1;
   }

   map $user_dept $is_finance {
       "finance" 1;
   }

   location /admin {
       auth_request /jwt-auth;
       auth_require $is_admin;
   }
   ```

---

### Real-life Use Cases

* **Admin dashboards** → allow only if claim `role=admin`.
* **Premium content** → allow only if claim `plan=premium`.
* **Geo restrictions** → allow only if claim `region=eu`.
* **Multi-factor** → combine rules, e.g., `auth_require $is_verified; auth_require $is_admin;`.

---

> In simple words:
* `auth_jwt`/`auth_oidc` → validates JWT/OIDC and extracts claims.
* `auth_request` → lets an external service decide validity.
* `auth_require` → enforces *fine-grained rules* (like role, plan, department).

</details>

---

## Module `ngx_http_autoindex_module`

The `ngx_http_autoindex_module` is what lets NGINX **show a directory listing** (like Apache does) when there’s **no index file** (`index.html`, `index.php`, etc.) in a folder.

> Example:
> If you go to `http://example.com/files/` and there’s no `index.html` in `/files/`, NGINX can automatically generate a list of files in that directory.

<details>
    <summary>Click to view Examples, Use Cases</summary>

### Example Configuration

```nginx
server {
    listen 80;
    server_name example.com;

    location /files/ {
        root /var/www/html;
        autoindex on;                 # enable directory listing
        autoindex_exact_size off;     # show human-readable sizes
        autoindex_format html;        # directory listing in HTML
        autoindex_localtime on;       # show file times in local timezone
    }
}
```

If `/var/www/html/files/` has files:

```
report.pdf
photo.jpg
backup.zip
```

Visiting `http://example.com/files/` will show a **file browser page** generated by NGINX.

---

### Directives (easy explanation)

#### 1. `autoindex on | off`

* Turns **directory listing** on or off.
* Default: `off`.
* Example:

  ```nginx
  location /downloads/ {
      autoindex on;
  }
  ```

---

#### 2. `autoindex_exact_size on | off`

* **on** → shows exact file size in bytes.
* **off** → shows human-readable (KB, MB, GB).
* Example:

  ```nginx
  autoindex_exact_size off;  # “2.3 MB” instead of “2381123”
  ```

---

#### 3. `autoindex_format html | xml | json | jsonp`

* Choose how the directory listing looks:

  * **html** (default): like a simple webpage with links.
  * **xml**: good for scripts/parsers.
  * **json**: structured file listing for APIs.
  * **jsonp**: like JSON, but wrapped in a callback function (for browsers that don’t support CORS).
* Example:

  ```nginx
  autoindex_format json;
  ```

Result for `/files/` might look like:

```json
[
  {"name":"report.pdf","size":234123,"mtime":"2025-09-13T10:22:00Z"},
  {"name":"photo.jpg","size":129034,"mtime":"2025-09-12T15:12:00Z"}
]
```

---

#### 4. `autoindex_localtime on | off`

* **on** → show file modification times in **server’s local timezone**.
* **off** (default) → show in **UTC**.
* Example:

  ```nginx
  autoindex_localtime on;
  ```

---

### Real-Life Use Cases

1. **File Server**

   * Share documents, images, or videos quickly.
   * Example: `http://example.com/files/` → autoindex shows all files.

2. **Static Backups**

   * Expose a folder of backup `.zip` or `.tar.gz` files for download.

3. **Developer Builds**

   * CI/CD systems dump build artifacts into `/builds/`.
   * Autoindex shows latest build files without writing custom HTML.

4. **API-style file listing**

   * Use `autoindex_format json` to provide a JSON file list for frontend apps.

> In simple words:
* `autoindex` = makes NGINX behave like a **file browser** for directories.
* You can control **size format, time format, and output format** (HTML, JSON, XML).
* Very handy for **file servers, downloads, backups, or APIs**.

</details>

---

## Module `ngx_http_status_module`

* This module gives you **detailed status info about NGINX**: connections, requests, upstreams, caches, SSL stats, etc.
* Think of it as an **advanced version of `stub_status`**.
* Unlike `stub_status` (which only shows basic counters), this one outputs **JSON (or JSONP)** with rich data.

<details>
    <summary>Click to view Example, Config, and Output</summary>

### Example Scenario

Imagine you’re running an API behind NGINX with **2 upstream servers**.
You want to:

* See how many requests each server handled.
* See active connections, response codes, errors.
* Monitor cache usage (if using proxy\_cache).

---

### Example Configuration

```nginx
http {
    ##
    ## 1. Define upstream with "zone" for per-peer stats
    ##
    upstream backend {
        zone http_backend 64k;                 # enables upstream metrics (→ "upstreams" in JSON)
        server backend1.example.com weight=5;
        server backend2.example.com;
    }

    ##
    ## 2. Define cache path with "keys_zone" for cache stats
    ##
    proxy_cache_path /data/nginx/cache_backend keys_zone=cache_backend:10m; 
    # enables cache metrics (→ "caches" in JSON)

    ##
    ## 3. Main server that uses upstream and cache
    ##
    server {
        listen 80;
        server_name api.example.com;

        location / {
            proxy_pass http://backend;
            proxy_cache cache_backend;
            health_check;                      # enables health check info in upstream stats
        }

        status_zone server_backend;            # enables per-server stats (→ "server_zones" in JSON)
    }

    ##
    ## 4. Monitoring endpoint
    ##
    server {
        listen 127.0.0.1;

        location /status {
            status;                            # enable status
            status_format json;                # JSON format (machine-readable)
        }

        location = /status.html { }            # optional HTML page for simple view
    }
}
```

---

### Example Outputs

### If you curl `/status`

```bash
curl http://127.0.0.1/status
```

You get JSON like:

```json
{
  "version": 8,                          ##--> Format version of the status JSON (used by monitoring tools to parse correctly), From status module itself
  "nginx_version": "1.25.3",             ##--> Version of NGINX running
  "connections": {                     ## From NGINX core
    "accepted": 5000,                    ##--> Total number of TCP connections accepted by NGINX since start
    "dropped": 10,                       ##--> Connections dropped (e.g., resource limits, overload)
    "active": 120,                       ##--> Currently open/active client connections
    "idle": 20                           ##--> Connections that are open but not currently sending requests
  },
  "requests": {                        ## From NGINX core
    "total": 20000,                      ##--> Total number of HTTP requests handled since start
    "current": 15                         ##--> Number of requests currently being processed at this moment
  },
  "server_zones": {                    ## Because of "status_zone server_backend;"
    "server_backend": {
      "processing": 5,                   ##--> Requests currently being handled by this server zone
      "requests": 15000,                 ##--> Total requests received for this server block
      "responses": {
        "total": 15000,                  ##--> Total number of responses sent to clients
        "2xx": 14000,                    ##--> Successful responses (status codes 200–299)
        "4xx": 500,                      ##--> Client error responses (e.g., 404, 403, 400)
        "5xx": 500                       ##--> Server error responses (e.g., 500, 502, 504)
      },
      "received": 12345678,              ##--> Total bytes received from clients (uploads, request bodies)
      "sent": 98765432                   ##--> Total bytes sent to clients (responses, files, etc.)
    }
  },
  "upstreams": {                       ## Because of "zone http_backend 64k;" in upstream
    "backend": {
      "peers": [
        {
          "id": 1,                       ##--> ID of the upstream peer (unique identifier)
          "server": "backend1.example.com", ##--> Upstream server address
          "state": "up",                 ##--> Current health status (up, down, unhealthy, etc.)
          "active": 10,                  ##--> Active (ongoing) requests currently being handled by this peer
          "requests": 8000,              ##--> Total requests forwarded to this peer since NGINX started
          "responses": { 
            "2xx": 7800,                 ##--> Number of successful responses returned by this peer
            "5xx": 200                   ##--> Number of server error responses returned by this peer
          },
          "fails": 5                     ##--> Failed attempts to connect or communicate with this peer
        },
        {
          "id": 2,
          "server": "backend2.example.com",
          "state": "up",
          "active": 5,
          "requests": 7000,              ##--> Total requests sent to this peer (counts every request routed here)
          "responses": { 
            "2xx": 6900,                 ##--> Successful responses returned by this peer
            "5xx": 100                   ##--> Server-side errors returned by this peer
          },
          "fails": 3                     ##--> Number of failed connection attempts or retries for this peer
        }
      ]
    }
  },

"caches": {                          ## Because of "proxy_cache_path ... keys_zone=cache_backend"
    "cache_backend": {
      "size": 1234567,                  ## Current size of cache
      "max_size": 10485760,             ## Max allowed size (10m)
      "cold": false,                    ## If cache loader still initializing
      "hit": { "responses": 5000, "bytes": 10000000 },
      "miss": { "responses": 2000, "bytes": 4000000 },
      "expired": { "responses": 100, "bytes": 20000 }
    }
  }
}

```

---

### Why It’s Useful

* **Better than `stub_status`** → you see request counts, per-upstream stats, response codes, cache hits/misses.
* **Monitoring tools** (like Grafana, Prometheus, Datadog) can easily parse this JSON.
* Lets you set up **dashboards**:

  * Active connections.
  * Request rate.
  * Per-upstream health & failures.
  * Cache usage.

> Inshort
* Use `status` (NGINX Plus) for full **JSON API-style monitoring** with rich metrics.

</details>

---


