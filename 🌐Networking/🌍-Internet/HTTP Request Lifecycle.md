---
tags: ["Networking"]
---


> What happens when you type `https://example.com` and hit Enter.

---
**7 phases** — DNS → TCP → TLS → Request → Server → Response → Render

## Phase 1 — DNS Resolution
| Step | What Happens |
|------|-------------|
| 1 | Browser checks **local cache** (recently visited?) |
| 2 | Asks **OS cache** → then `/etc/hosts` file |
| 3 | Asks **Recursive Resolver** (your ISP or 8.8.8.8) |
| 4 | Resolver asks **Root nameserver** → TLD (`.com`) → **Authoritative DNS** |
| 5 | Returns **IP address** → cached with TTL |

---

## Phase 2 — TCP Handshake (3-Way)
| Step | Client | Server |
|------|--------|--------|
| 1 | → SYN | |
| 2 | | ← SYN-ACK |
| 3 | → ACK | |
> Connection established. Ready to send data.

---

## Phase 3 — TLS Handshake (HTTPS only)
| Step | What Happens |
|------|-------------|
| 1 | Client → **ClientHello** (TLS version, cipher suites) |
| 2 | Server → **Certificate** + **ServerHello** |
| 3 | Client verifies cert with **CA** |
| 4 | Both derive **session keys** → encrypted tunnel ready |

---

## Phase 4 — HTTP Request
```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
Accept-Encoding: gzip
Connection: keep-alive
Cookie: session=abc123
```

| Part | Description |
|------|-------------|
| Method | `GET` `POST` `PUT` `DELETE` `PATCH` |
| Path | Resource being requested |
| Headers | Metadata (auth, content type, cookies) |
| Body | Present in `POST`/`PUT` — contains payload |

---

## Phase 5 — Server Processing
| Step | What Happens |
|------|-------------|
| 1 | Request hits **Load Balancer** → routed to server |
| 2 | **Web server** (Nginx/Apache) receives request |
| 3 | App server runs **business logic** |
| 4 | Queries **DB / cache** (Redis, Postgres) if needed |
| 5 | Builds and returns **HTTP Response** |

---

## Phase 6 — HTTP Response
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Encoding: gzip
Cache-Control: max-age=3600
Set-Cookie: session=abc123

<html>...</html>
```

| Status Range | Meaning |
|-------------|---------|
| `1xx` | Informational |
| `2xx` | Success — `200 OK`, `201 Created` |
| `3xx` | Redirect — `301 Moved`, `304 Not Modified` |
| `4xx` | Client Error — `400`, `401`, `403`, `404` |
| `5xx` | Server Error — `500`, `502`, `503` |

---

## Phase 7 — Browser Rendering
| Step | What Happens |
|------|-------------|
| 1 | Parse **HTML** → build DOM tree |
| 2 | Parse **CSS** → build CSSOM |
| 3 | DOM + CSSOM → **Render tree** |
| 4 | **Layout** → calculate positions |
| 5 | **Paint** → pixels on screen |
| 6 | Fetch sub-resources (JS, CSS, images) → repeat lifecycle |

---

## Connection Reuse
| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Multiplexing | ✗ | ✓ | ✓ |
| Header Compression | ✗ | ✓ (HPACK) | ✓ (QPACK) |
| Transport | TCP | TCP | UDP (QUIC) |
| TLS Required | Optional | Optional | Built-in |

