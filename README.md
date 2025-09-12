# NGINX

## Parameters in NGINX
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

---

Short answer: Use the NGINX stub_status module to see live connection counts and processing states in real time (Active, Reading, Writing, Waiting), and use the cumulative requests counter to infer how many have completed by taking deltas over time; NGINX Plus adds a live API with per‑zone request/response counts for precise “returned” totals by status code class.

## What to use
- Open source: stub_status exposes basic live metrics: Active connections, accepts, handled, requests, Reading, Writing, Waiting, plus embedded variables for these values.
- NGINX Plus: Live Activity Monitoring (dashboard and REST API) exposes detailed request and response counts, including responses by status class and per upstream/server zone, ideal for “how many returned” in real time.

## Key metrics explained
- Active connections: current live client connections, including idle keep‑alive (Waiting).
- Reading: connections where NGINX is reading the request header; Writing: connections where NGINX is sending the response; Waiting: idle keep‑alive connections awaiting a new request.
- requests (counter): total client requests since start; “returned” can be approximated as the increase in this counter over an interval minus any currently in‑flight requests, while NGINX Plus exposes explicit response counts by status class.

## Quick formulas
- In‑process now ≈ Reading + Writing, since these represent requests currently being received or responded to.
- Live connections now = Active connections, noting this includes Waiting (idle keep‑alive) as well as Reading and Writing.
- Completed/returned over interval $$ \Delta t $$ ≈ $$ \Delta\text{requests} $$, i.e., requests(t2) − requests(t1), with NGINX Plus offering direct response counters by class for precision.

## Stub_status parameters and variables
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

## How to enable live view (open source)
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

## How to get exact “returned” counts
- With stub_status, track “requests” as a counter and compute deltas per collection interval to approximate “requests returned,” subtracting in‑flight if needed using Reading + Writing for momentary in‑process counts.
- With NGINX Plus, use the REST API (for example, /api/<version>/http/server_zones and /api/<version>/connections) to obtain precise live request and response counters, including responses by status class, suitable for dashboards and SLOs.
