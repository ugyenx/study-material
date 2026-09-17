# Web Fundamentals — Complete Notes

---

## 1. How the Internet and Web Work

### Internet vs Web
- **Internet** = a global network of networks — millions of computers connected via cables/routers using shared protocols (TCP/IP). It's the physical/logical infrastructure.
- **Web (World Wide Web)** = one specific *application* that runs on top of the Internet — the system of linked documents accessed via HTTP and a browser.
- Analogy: Internet = the road system. Web = one type of vehicle driving on it. Email (SMTP) is another example that uses the Internet but is *not* part of the Web.

### Client-Server Architecture
- **Client** = requests something (usually the browser).
- **Server** = stores data and responds when asked.

```
Client (browser) ---- request ----> Server
Client (browser) <--- response ---- Server
```

### Browser's Job vs Server's Job
**Browser:**
1. Takes the address you typed
2. Sends a request to the right server
3. Receives the response (HTML/CSS/JS/data)
4. **Renders** it — turns raw code into the visible page, runs JS

**Server:**
1. Listens for requests
2. Figures out what's being asked
3. Sends back the right response

> The server never renders anything — it just sends raw data. The browser renders.

---

## 2. URLs and DNS Basics

### DNS (Domain Name System)
- Computers only understand IP addresses (e.g., `142.250.183.14`), not names like `google.com`.
- DNS is like a phonebook: name in → IP address out.

```
You type: google.com → DNS lookup → Returns: 142.250.183.14 → Browser connects
```

### Anatomy of a URL

```
https://www.example.com:443/products/shoes?color=red&size=9#reviews
```

| Part | Value | Meaning |
|---|---|---|
| Scheme/Protocol | `https://` | Which protocol to use |
| Host/Domain | `www.example.com` | Which server (resolved via DNS) |
| Port | `:443` | Which "door" on the server |
| Path | `/products/shoes` | Specific resource |
| Query string | `?color=red&size=9` | Extra key=value parameters |
| Fragment | `#reviews` | Section within the page — **never sent to the server**, browser-only |

> If DNS stopped working, you could still reach a site by typing its raw IP directly — the network layer only cares about IPs, DNS is just a convenience layer.

---

## 3. HTTP vs HTTPS

### HTTP (HyperText Transfer Protocol)
- The shared rulebook for client-server communication: defines request format (method, path, headers, body) and response format (status code, headers, body).

### The Problem with Plain HTTP
- Data travels as **plain text** — anyone intercepting it (ISP, shared WiFi) can read it, including passwords.

### HTTPS = HTTP + TLS Encryption
- Same HTTP protocol, wrapped in **TLS** encryption.
- HTTPS uses port **443** by default; HTTP uses port **80**.
- Requires an SSL/TLS certificate on the server.
- TLS encrypts the *payload/content* — it does **not** hide that a connection is happening (IP/domain may still be visible).
- HTTPS does **not** guarantee the site's content is safe/legitimate — only that the connection can't be eavesdropped on.

---

## 4. HTTP Methods

Every request has a **method** (verb) describing the intended action on a resource.

| Method | Purpose |
|---|---|
| **GET** | Retrieve data, no changes |
| **POST** | Create new data |
| **PUT** | Replace/update a resource fully |
| **PATCH** | Update part of a resource |
| **DELETE** | Remove data |

```
GET /orders/45     → "give me order 45"
DELETE /orders/45  → "delete order 45"
POST /orders       → "create a new order"
```

### Idempotency
A method is **idempotent** if calling it once or many times produces the same end result.

| Method | Idempotent? | Why |
|---|---|---|
| GET | ✅ Yes | Reading doesn't change state |
| PUT | ✅ Yes | Replacing with same data = same end state |
| DELETE | ✅ Yes | Already-deleted stays deleted |
| POST | ❌ No | Each call typically creates something new |
| PATCH | ⚠️ Usually not guaranteed | Depends on the update logic |

> This is why browsers warn "resubmit form?" on refresh after a POST, but not after a GET.

---

## 5. Request/Response Structure + Headers

### HTTP Request — 3 parts
```
1. Request Line → GET /orders/45 HTTP/1.1
2. Headers      → Key: Value metadata
3. Body         → Actual data (optional, mainly POST/PUT/PATCH)
```

Example:
```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer abc123

{"username": "raj", "password": "1234"}
```

### HTTP Response — 3 parts
```
1. Status Line → HTTP/1.1 200 OK
2. Headers     → Key: Value metadata
3. Body        → Actual content returned
```

### Common Headers

| Header | Used in | Meaning |
|---|---|---|
| `Content-Type` | Request & Response | Format of the body (`application/json`, `text/html`) |
| `Content-Length` | Response | Size of body in bytes |
| `Authorization` | Request | Credentials/token |
| `Host` | Request | Which domain is being targeted |
| `User-Agent` | Request | Info about the browser/device |
| `Set-Cookie` | Response | Tells browser to store a cookie |
| `Cache-Control` | Response | Caching rules |

> Headers are metadata describing the data — not the data itself. If `Content-Type` says JSON but the body is actually HTML, the browser will try to parse it as JSON and fail/error, rather than rendering it.

---

## 6. HTTP Status Codes

First digit = category:

| Range | Category | Meaning |
|---|---|---|
| 1xx | Informational | Still processing |
| 2xx | Success | Worked |
| 3xx | Redirection | Resource moved |
| 4xx | Client Error | Client's mistake |
| 5xx | Server Error | Server's fault |

### Common codes

**2xx:** `200 OK`, `201 Created` (after POST, usually returns a body), `204 No Content` (success, no body — common after DELETE)
**3xx:** `301 Moved Permanently`, `302 Found` (temporary redirect)
**4xx:** `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`
**5xx:** `500 Internal Server Error`, `503 Service Unavailable`

### 401 vs 403 (key distinction)
- **401** = "I don't know who you are" (not authenticated) — like showing up with no ID badge at all.
- **403** = "I know who you are, but you're not allowed" (authenticated, not authorized) — like showing a valid badge that just lacks clearance for this room.

---

## 7. REST APIs

**REST (Representational State Transfer)** = a set of conventions for API design: **URLs = resources (nouns)**, **HTTP methods = actions (verbs)**.

```
GET    /users        → get all users
GET    /users/5       → get user 5
POST   /users         → create a user
PUT    /users/5       → replace user 5
PATCH  /users/5       → partially update user 5
DELETE /users/5       → delete user 5
```

- Bad REST style: verbs baked into the URL, e.g. `/getUser?id=5`, `/deleteProduct?id=12`.
- Good REST style: `GET /users/5`, `DELETE /products/12` — plural resource names are conventional.

### Nesting resources
```
GET /users/5/orders        → orders belonging to user 5
GET /users/5/orders/45     → a specific order
GET /posts/8/comments      → comments on post 8
```

### Breaking down the name
- **Resource** = a "thing" the API manages
- **Representation** = the format returned (usually JSON)
- **State Transfer** = client/server exchange current state per request

### Statelessness
REST APIs are meant to be **stateless** — every request must carry all info needed (e.g., `Authorization` header on every request), since the server doesn't remember past requests.

---

## 8. JSON (JavaScript Object Notation)

A lightweight, text-based data format for structured data.

```json
{
  "id": 45,
  "name": "Raj",
  "isActive": true,
  "score": 87.5,
  "tags": ["admin", "verified"],
  "address": { "city": "Delhi", "pincode": 110001 },
  "manager": null
}
```

| Type | Example |
|---|---|
| String | `"Raj"` (double quotes only — single quotes are invalid) |
| Number | `45`, `87.5` |
| Boolean | `true` / `false` |
| Array | `["admin", "verified"]` |
| Object | `{ "city": "Delhi" }` |
| Null | `null` |

### Parsing and Stringifying
- **Parse**: JSON text → JS object — `JSON.parse(jsonText)`
- **Stringify**: JS object → JSON text — `JSON.stringify(data)`

```javascript
let obj = JSON.parse('{"id": 45, "name": "Raj"}');
console.log(obj.name); // "Raj"

let text = JSON.stringify({ id: 45, name: "Raj" });
console.log(text); // '{"id":45,"name":"Raj"}'
```

> `Content-Type: application/json` tells the receiver to parse the body as JSON. Forgetting to set it can cause the server to misinterpret the body (often leading to a 400 error).

---

## 9. Caching Basics

**Caching** = storing a copy of a response so future requests reuse it instead of re-fetching.

```
1st visit: Browser -> Server -> response stored
2nd visit: Browser checks cache -> found -> uses cached copy, no request sent
```

### `Cache-Control` directives

| Directive | Meaning |
|---|---|
| `max-age=3600` | Valid for 3600 seconds |
| `no-cache` | Must revalidate with server before reuse |
| `no-store` | Don't cache at all (e.g., banking data) |
| `public` | Cacheable by browser and shared caches (CDNs) |
| `private` | Cacheable only by the individual user's browser |

### Revalidation
Instead of blind re-downloads, browser can ask "has this changed?" using `ETag`/`Last-Modified`.

```
Browser: "GET logo.png, ETag 'abc123' — changed?"
Server: unchanged → 304 Not Modified (no body sent)
Server: changed   → 200 OK + new content
```

> `304 Not Modified` never includes the full body — that's the point, it tells the browser "use what you already have."

---

## 10. Cookies

Small key-value data that:
1. Server sends via `Set-Cookie` header
2. Browser stores it
3. Browser **auto-attaches it to every future request** to that domain

```
Login:  Browser --POST /login--> Server
        Browser <--Set-Cookie: sessionId=xyz789-- Server

Later:  Browser --GET /dashboard, Cookie: sessionId=xyz789--> Server
        Server: "xyz789 belongs to Raj" -> authenticated
```

This solves the "HTTP is stateless" problem — the cookie carries identity across independent requests.

### Cookie Attributes

```
Set-Cookie: sessionId=xyz789; Max-Age=3600; HttpOnly; Secure; SameSite=Strict
```

| Attribute | Meaning |
|---|---|
| `Max-Age`/`Expires` | Lifetime before auto-deletion |
| `HttpOnly` | JavaScript **cannot** access this cookie — protects against theft via malicious scripts |
| `Secure` | Only sent over HTTPS |
| `SameSite` | Controls cross-site sending |
| (no expiry) | "Session cookie" — deleted when browser closes |

> Without `Secure`, a cookie could be sent over plain HTTP and intercepted, even on a site that also supports HTTPS.

---

## 11. localStorage & sessionStorage

Browser storage accessible only via JavaScript — **never automatically sent to the server** (unlike cookies). Larger limit than cookies (~5-10MB vs ~4KB).

```javascript
localStorage.setItem("theme", "dark");
sessionStorage.setItem("draftText", "Hello world");

localStorage.getItem("theme");
localStorage.removeItem("theme");
```

| | localStorage | sessionStorage |
|---|---|---|
| Lifetime | Persists forever until cleared | Cleared when tab/window closes |
| Scope | Shared across all tabs of the same site | Only that specific tab |
| Use case | Theme preference, cart across visits | Multi-step form data within one tab |

### When to use which
- **Cookies** → when the **server** needs to see the data on every request (e.g., session ID).
- **localStorage/sessionStorage** → when only the **browser/JS** needs it (e.g., UI preferences).
- Avoid storing sensitive auth tokens in Web Storage — no `HttpOnly`-equivalent protection exists; any JS on the page can read it.
- "Remember dark mode forever" → use **localStorage** (persists across browser restarts); sessionStorage would lose it when the tab closes.

---

## 12. CORS (Cross-Origin Resource Sharing)

### The problem it solves
If `evil.com`'s JS could freely call `bank.com/transfer-money`, the browser would auto-attach your `bank.com` cookie, letting the malicious site act as you.

### Same-Origin Policy
By default, browsers block JS on one **origin** from reading responses from a different origin.

**Origin** = scheme + host + port:
```
https://example.com:443   → one origin
http://example.com:443    → different (scheme differs)
https://api.example.com   → different (host differs)
https://example.com:8080  → different (port differs)
```

### CORS — the server opts in
Server sends a header explicitly allowing certain origins:
```
Access-Control-Allow-Origin: https://myapp.com
```
- If missing/mismatched, the **browser blocks JS from reading the response** — even though the server may have already processed the request. CORS is enforced by the **browser**, not the server.
- `Access-Control-Allow-Origin: *` = any origin allowed.

### Preflight requests (`OPTIONS`)
For "risky" requests (DELETE, PUT, custom headers like `Authorization`), the browser sends an automatic `OPTIONS` request first, asking permission, before sending the real request.

```
Browser: OPTIONS /orders/45 (asking permission)
Server: responds with allowed origins/methods
Browser: if allowed -> sends real DELETE; if not -> blocks it
```

> Simple GETs are considered "safe"/read-only by long-standing convention and don't need a preflight; state-changing methods (DELETE, PUT) do, since the browser wants permission before a potentially destructive action.

---

## 13. Authentication Basics

**Authentication** = "Who are you?" (proving identity)
**Authorization** = "What are you allowed to do?" (ties back to 401 vs 403)

### General flow
```
1. User submits username + password (POST /login)
2. Server checks credentials
3. If valid, server creates proof of identity
4. Client includes that proof on every future request
5. Server checks the proof each time, without needing the password again
```

### Session-based Auth
```
Login -> Server creates a session, stores it (sessionId -> user)
Server sends: Set-Cookie: sessionId=xyz789
Every request -> cookie auto-sent -> server looks up sessionId -> finds user
```
- Server must **store** session data (stateful).

### Token-based Auth (JWT)
```
Login -> Server creates a signed token: {"userId": 5, "role": "admin"}
Client stores token, sends manually: Authorization: Bearer eyJhbGci...
Server verifies signature -> trusts info inside, no DB lookup needed
```
- Server stores nothing — **stateless** authentication.

### Signing (why tokens can't be forged)
A JWT = `header.payload.signature`. The payload is readable (base64, not encrypted) but the **signature** (made with a server-only secret key) prevents tampering — changing the payload invalidates the signature.

> JWTs should never contain sensitive plaintext data (like passwords) since the payload is readable by anyone.

### Token theft / revocation risk
If an attacker steals a valid JWT, they can replay it as-is (no need to forge it) and impersonate the user until the token expires — the server has no built-in way to revoke it early. This is why systems commonly use **short-lived access tokens + refresh tokens** instead of one long-lived token — it limits the window of exposure if a token is stolen, even if the account password is changed afterward.

---

## 14. Browser Rendering Basics

### The Rendering Pipeline

```
1. Parse HTML  -> builds the DOM (Document Object Model) — tree of elements
2. Parse CSS   -> builds the CSSOM (CSS Object Model) — tree of style rules
3. Combine     -> DOM + CSSOM = Render Tree (only visible elements, computed styles)
4. Layout      -> calculate position/size of every element ("reflow")
5. Paint       -> draw pixels — colors, text, borders, images
```

- **DOM** = structural tree of HTML elements.
- **CSSOM** = tree of style rules.
- **Render Tree** = merge of both, excluding elements like `display: none`.

### Script Loading Behavior
By default, when the HTML parser hits a `<script>` tag, it **stops parsing HTML**, downloads and runs the script, then continues.

```html
<script src="app.js" defer></script>
```
- `defer`: downloads in background, runs **after** HTML parsing finishes.
- `async`: downloads in background, runs **as soon as ready** (order not guaranteed).
- No attribute: blocks parsing immediately at that point (slowest default).

### Reflow vs Repaint
- **Repaint**: visual-only change (e.g., color) — layout unaffected — browser just redraws pixels. Cheaper.
- **Reflow**: layout/position/size changed (e.g., width, adding an element) — browser recalculates layout for potentially many elements, then repaints. More expensive.

> Changing `background-color` → repaint only. Changing `width` → triggers reflow, since it affects the layout/positioning of surrounding elements.

---

## Key Cross-Topic Connections
- URL's port field (Topic 2) → directly maps to HTTP vs HTTPS default ports 80/443 (Topic 3).
- `Content-Type` header (Topic 5) → determines how JSON is interpreted (Topic 8) and how CORS/caching headers behave.
- Idempotency (Topic 4) → explains why GET doesn't need CORS preflight but DELETE does (Topic 12).
- `Authorization` header (Topic 5) → carries the JWT in token-based auth (Topic 13).
- Statelessness in REST (Topic 7) → is exactly the problem cookies (Topic 10) and tokens (Topic 13) solve.
- `304 Not Modified` (Topic 9) → a real status code example building on Topic 6.
