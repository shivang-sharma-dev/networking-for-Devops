# 05 — HTTP & HTTPS

## HTTP — HyperText Transfer Protocol

**HTTP** is a stateless, request-response protocol used to transfer data on the web.

```
Client ──► GET /api/users HTTP/1.1 ──► Server
Client ◄── 200 OK + JSON body       ◄── Server
```

---

## HTTP Methods

| Method | Purpose | Has Body? |
|---|---|---|
| **GET** | Retrieve data | No |
| **POST** | Create / submit data | Yes |
| **PUT** | Replace a resource | Yes |
| **PATCH** | Partially update a resource | Yes |
| **DELETE** | Delete a resource | No |
| **HEAD** | Like GET but no response body | No |
| **OPTIONS** | Check allowed methods (CORS preflight) | No |

---

## HTTP Status Codes

| Range | Category | Examples |
|---|---|---|
| 1xx | Informational | 100 Continue |
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirection | 301 Moved, 302 Found, 304 Not Modified |
| 4xx | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| 5xx | Server Error | 500 Internal Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

---

## HTTP/1.1 vs HTTP/2 vs HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|
| Multiplexing | No (one req/conn) | Yes | Yes |
| Header compression | No | HPACK | QPACK |
| Transport | TCP | TCP | QUIC (UDP) |
| Server push | No | Yes | Yes |
| Binary protocol | No (text) | Yes | Yes |

---

## HTTPS — HTTP + TLS

**HTTPS** adds encryption via **TLS (Transport Layer Security)** on top of HTTP.

### What TLS provides:
- **Confidentiality** — data is encrypted
- **Integrity** — data can't be tampered with
- **Authentication** — you're talking to the right server (via certificates)

---

## TLS Handshake (Simplified)

```
Client                              Server
  │──── ClientHello (TLS ver, ciphers) ──►│
  │◄─── ServerHello + Certificate ─────────│
  │                                        │
  │  [Client verifies cert against CA]      │
  │                                        │
  │──── Pre-master secret (encrypted) ────►│
  │                                        │
  │  [Both sides derive session keys]       │
  │                                        │
  │◄═══════ Encrypted HTTP traffic ════════│
```

### Key Terms
- **Certificate**: Contains the server's public key + identity, signed by a CA
- **CA (Certificate Authority)**: Trusted third party (Let's Encrypt, DigiCert)
- **Cipher suite**: Algorithm combo for key exchange, encryption, hashing
- **mTLS**: Both client AND server authenticate with certificates

---

## Certificates

```bash
# View a website's certificate
openssl s_client -connect example.com:443

# Inspect a certificate file
openssl x509 -in cert.pem -text -noout

# Check cert expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Generate a self-signed cert (for local dev)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

---

## HTTP Headers (Important Ones)

### Request Headers
```
Host: api.example.com
Authorization: Bearer <token>
Content-Type: application/json
Accept: application/json
User-Agent: Mozilla/5.0 ...
```

### Response Headers
```
Content-Type: application/json
Cache-Control: max-age=3600
X-Request-Id: abc123
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: DENY
```

---

## CORS — Cross-Origin Resource Sharing

**CORS** controls which domains can make requests to your API from a browser.

```
Browser: "Can example.com make a request to api.otherdomain.com?"
         Sends: OPTIONS preflight

Server responds with:
  Access-Control-Allow-Origin: https://example.com
  Access-Control-Allow-Methods: GET, POST
```

---

## DevOps Relevance

```bash
# Test HTTP endpoint
curl -v https://api.example.com/health

# Test with custom headers
curl -H "Authorization: Bearer token" https://api.example.com/users

# Follow redirects
curl -L https://example.com

# Time a request
curl -w "%{time_total}s\n" -o /dev/null -s https://example.com

# Check TLS cert chain
curl --cacert ca.pem https://internal-api.company.com
```
