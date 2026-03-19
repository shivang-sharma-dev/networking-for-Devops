# 05 — HTTP and HTTPS

> Every web application you build or deploy runs on HTTP/HTTPS.
> Understanding this protocol means you can debug anything — from
> slow APIs to broken authentication to failed deployments.

---

## What is HTTP?

HTTP (HyperText Transfer Protocol) is a request-response protocol.
A client sends a request, a server sends a response. That's it.

```
ANALOGY: Ordering at a restaurant

  You (client)         Waiter (HTTP)       Kitchen (server)
  ────────────         ─────────────       ────────────────
  "I want the          Carries your        Prepares the food
   pasta please"  ───► order to        ───► and sends it back
                       kitchen

  Every HTTP interaction follows this exact pattern:
    Request  → "I want this resource"
    Response → "Here it is" (or "Not found" or "Error")
```

---

## HTTP Request Structure

```
GET /api/users/42 HTTP/1.1          ← Request line: method + path + version
Host: api.example.com               ← Headers start here
Accept: application/json
Authorization: Bearer abc123token
Content-Type: application/json
User-Agent: Mozilla/5.0

{"filter": "active"}                ← Body (optional, for POST/PUT)
```

### HTTP Methods

```
GET     → Retrieve a resource (no body, safe, idempotent)
POST    → Create a resource (has body, not idempotent)
PUT     → Replace a resource entirely (has body, idempotent)
PATCH   → Update part of a resource (has body)
DELETE  → Remove a resource (no body, idempotent)
HEAD    → Same as GET but no body in response (get headers only)
OPTIONS → What methods does this endpoint support?

Idempotent = calling it multiple times = same result as calling once
  GET /users/42      → always returns same user (idempotent)
  DELETE /users/42   → first call deletes, second call = 404 (idempotent result)
  POST /users        → each call creates a NEW user (NOT idempotent)
```

---

## HTTP Response Structure

```
HTTP/1.1 200 OK                     ← Status line: version + code + message
Content-Type: application/json      ← Headers
Content-Length: 82
Cache-Control: max-age=3600
X-Request-ID: abc-123

{"id": 42, "name": "Alice", "email": "alice@example.com"}   ← Body
```

### HTTP Status Codes — Know These Cold

```
1xx — Informational
  100 Continue           Keep sending the request body

2xx — Success
  200 OK                 Request succeeded
  201 Created            Resource was created (POST/PUT)
  204 No Content         Success, no body (DELETE)

3xx — Redirection
  301 Moved Permanently  URL has changed forever (update bookmarks)
  302 Found              Temporary redirect
  304 Not Modified       Cached version is still valid

4xx — Client Errors (YOU did something wrong)
  400 Bad Request        Malformed request syntax
  401 Unauthorized       Not authenticated (no/bad credentials)
  403 Forbidden          Authenticated but not allowed
  404 Not Found          Resource does not exist
  405 Method Not Allowed Wrong HTTP method
  409 Conflict           Resource state conflict
  422 Unprocessable      Validation failed
  429 Too Many Requests  Rate limited

5xx — Server Errors (THE SERVER did something wrong)
  500 Internal Server Error   Unhandled exception
  502 Bad Gateway             Upstream server gave bad response
  503 Service Unavailable     Server overloaded or down
  504 Gateway Timeout         Upstream server timed out

Memory trick:
  4xx = Client's fault  ("You messed up")
  5xx = Server's fault  ("We messed up")
```

---

## HTTP Headers — The Important Ones

```
REQUEST HEADERS
───────────────
Host              Which server to connect to (required in HTTP/1.1)
Authorization     Credentials: "Bearer TOKEN" or "Basic base64"
Content-Type      Format of the request body: "application/json"
Accept            What format you want back: "application/json"
Cookie            Stored cookies sent to server
User-Agent        What client is making the request
Cache-Control     Caching instructions

RESPONSE HEADERS
────────────────
Content-Type      Format of the response body
Content-Length    Size of response body in bytes
Set-Cookie        Tell the client to store a cookie
Cache-Control     How long to cache: "max-age=3600" or "no-cache"
Location          Redirect destination (used with 3xx)
WWW-Authenticate  How to authenticate (used with 401)
X-RateLimit-*     Rate limit info (custom headers)
CORS headers      Cross-origin resource sharing permissions
```

---

## HTTPS — HTTP + TLS

HTTP sends data in plain text. Anyone between you and the server
can read it (man-in-the-middle attack).

```
HTTP (plain text):
  You ────────────────────────────────► Server
        "password=mysecret123"
                         ↑
                  Attacker can read this

HTTPS (encrypted):
  You ────────────────────────────────► Server
        "▓▒░█╔╗▒░▓▒▓░▒█▓▒░"
                         ↑
                  Attacker sees gibberish
```

HTTPS = HTTP running inside a **TLS** (Transport Layer Security) tunnel.

---

## The TLS Handshake — How HTTPS Works

```
CLIENT                                    SERVER
──────                                    ──────

1. ClientHello ──────────────────────────►
   "I support TLS 1.3, here are my
    cipher suites"

2.                         ◄──────────────  ServerHello
                                            "Let's use TLS 1.3 + AES-256"
                                            + sends its CERTIFICATE

3. Verify certificate ──────────────────►
   (Is it signed by a trusted CA?
    Is the domain name correct?
    Is it expired?)

4. Key exchange ─────────────────────────►
   (Diffie-Hellman — both sides derive
    the same secret key without ever
    sending it over the network)

5. Both sides now have the same key.
   All further communication is encrypted.

6. Encrypted HTTP request ───────────────►
   ◄─────────────────────── Encrypted HTTP response
```

---

## TLS Certificates

A certificate proves who you are. It contains:
- Domain name it's valid for
- Public key
- Issuer (Certificate Authority)
- Expiry date
- Digital signature from the CA

```
Certificate chain:
  Your cert (example.com)
       ↑ signed by
  Intermediate CA (Let's Encrypt R3)
       ↑ signed by
  Root CA (ISRG Root X1)
       ↑ trusted by
  Your browser/OS (pre-installed trust store)
```

```bash
# View a site's certificate
openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -text

# Check expiry date
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Check certificate from a file
openssl x509 -in cert.pem -noout -text

# Get Let's Encrypt certificate (free)
sudo certbot --nginx -d example.com
sudo certbot renew --dry-run     # test auto-renewal
```

---

## HTTP Versions

```
HTTP/1.0   One request per connection. Slow.

HTTP/1.1   Persistent connections (reuse TCP connection).
           Pipelining (send multiple requests without waiting).
           Still most widely supported.

HTTP/2     Multiplexing: multiple requests over ONE TCP connection
           in parallel. Header compression. Server push.
           Requires HTTPS.

HTTP/3     Uses QUIC (UDP-based) instead of TCP.
           Faster connection setup. Better on lossy networks.
           Used by major CDNs and Google.

For DevOps:
  Enable HTTP/2 in nginx → multiply throughput for free
  HTTP/3 → consider for CDN/edge, not yet universally supported
```

---

## curl — Your HTTP Swiss Army Knife

```bash
# Basic GET
curl https://api.example.com/users

# With headers
curl -H "Authorization: Bearer TOKEN" https://api.example.com/users
curl -H "Content-Type: application/json" https://api.example.com

# POST with JSON body
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'

# Show response headers
curl -I https://example.com              # headers only
curl -v https://example.com             # verbose: full request + response
curl -D - https://example.com           # dump headers then body

# Follow redirects
curl -L https://short.url/abc

# Save response to file
curl -o output.html https://example.com

# Check HTTP status code only
curl -s -o /dev/null -w "%{http_code}" https://example.com

# Measure response time
curl -w "\nDNS: %{time_namelookup}s\nConnect: %{time_connect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" \
  -s -o /dev/null https://example.com

# Test with different HTTP version
curl --http2 https://example.com
curl --http1.1 https://example.com
```

---

## Common HTTP Issues in DevOps

```
401 Unauthorized
  → Check Authorization header format
  → Is the token expired?
  → Wrong credentials

403 Forbidden
  → User authenticated but lacks permission
  → Check file permissions if serving static files (chmod 644)
  → Check nginx/apache access rules

404 Not Found
  → Wrong URL path
  → Resource doesn't exist
  → Routing misconfiguration in nginx/app

502 Bad Gateway
  → Upstream app is down
  → nginx can't reach your app server
  → Check if the app is running: systemctl status myapp
  → Check the port: ss -tulnp | grep 8080

504 Gateway Timeout
  → Upstream app is too slow to respond
  → Increase proxy_read_timeout in nginx
  → Profile the slow endpoint

CORS error (browser console)
  → Server not sending Access-Control-Allow-Origin header
  → Add CORS headers in nginx or your app
  → "No 'Access-Control-Allow-Origin' header" = server-side fix needed
```
