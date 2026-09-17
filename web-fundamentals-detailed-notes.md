# Web Fundamentals — Detailed Notes (Expanded Edition)

---

## 1. How the Internet and Web Work

### Internet vs Web
- **Internet** = a global network of networks — millions of computers connected via cables, satellites, and routers, all agreeing to use shared protocols (mainly TCP/IP) to move data around.
- **Web (World Wide Web)** = one specific *application* built on top of the Internet — a system of linked "documents" (webpages) accessed using HTTP through a browser.
- Analogy: the Internet is the road system (highways, streets, traffic rules). The Web is just one type of vehicle that uses those roads. Email (SMTP), file transfer (FTP), online gaming, video calls — all also use the Internet, but none of them are "the Web."

### Why this distinction matters
People say "I'm on the internet" when they mean "I'm browsing the web" — but your phone doing a software update, or a smart fridge phoning home, is using the Internet with zero involvement from the Web at all.

### How data actually moves (a light refresher, since you already studied this deeply)
- Data isn't sent as one giant blob — it's broken into small **packets**, each with source/destination info, sent independently, and reassembled at the destination.
- This packet-based design is *why* the Internet is resilient — if one path is broken, packets can be rerouted.

### Client-Server Architecture
- **Client** = the program that *requests* something (your browser, a mobile app, even a script).
- **Server** = a computer that *stores* data/webpages and *responds* when asked.

```
Client (browser) ---- request ----> Server
Client (browser) <--- response ---- Server
```

- A single server machine can serve millions of clients — that's why one Google server cluster can handle the entire world's search traffic.
- Some systems are **peer-to-peer (P2P)** instead — no fixed client/server roles, everyone can be both (e.g., BitTorrent) — but the Web is client-server, not P2P.

### Browser's Job vs Server's Job

**Browser's job:**
1. Take the address you typed
2. Resolve it and send a request to the right server
3. Receive the response (HTML/CSS/JS/data)
4. **Render** it — convert raw code into the visual page, and execute any JavaScript

**Server's job:**
1. Listen for incoming requests
2. Process what's being asked (maybe query a database, run some logic)
3. Send back an appropriate response

> Important nuance: the server never "renders" anything visually — it just sends text/data. All visual interpretation (fonts, colors, layout, running JS) happens exclusively in the browser. This is why the same HTML can look slightly different across browsers — each browser's rendering engine interprets it independently.

---

## 2. URLs and DNS Basics

### Why DNS exists
Computers communicate using numeric **IP addresses** (e.g., `142.250.183.14` for IPv4, or longer for IPv6). Humans can't realistically memorize hundreds of these, so DNS translates human-friendly names into machine-friendly numbers.

```
You type: google.com → DNS lookup happens → Returns: 142.250.183.14 → Browser connects to that IP
```

### A bit more depth on DNS structure (builds on what you already studied)
- Domain names are hierarchical, read right to left: `www.example.com` → `.com` (Top-Level Domain) → `example` (the actual domain) → `www` (a subdomain).
- DNS results are **cached** — by your browser, OS, and ISP — so repeated lookups for the same domain are fast and don't hit the root servers every time.
- Different DNS "record types" store different info (not just IP address): e.g., an `MX` record points to mail servers, a `CNAME` record aliases one domain to another. (You don't need to memorize all types — just know IP resolution is only one of DNS's jobs.)

### Anatomy of a URL

```
https://www.example.com:443/products/shoes?color=red&size=9#reviews
```

| Part | Value | Meaning |
|---|---|---|
| Scheme/Protocol | `https://` | Which protocol to use to talk to the server |
| Host/Domain | `www.example.com` | Which server (resolved via DNS to an IP) |
| Port | `:443` | Which "door" on the server to knock on |
| Path | `/products/shoes` | Which specific resource on that server |
| Query string | `?color=red&size=9` | Extra parameters as key=value pairs |
| Fragment | `#reviews` | A section within the page |

- **Port** is often invisible because browsers assume the default for the scheme (80 for HTTP, 443 for HTTPS) — you only see it written out when a non-default port is used (e.g., a local dev server at `:3000`).
- **Query string** parameters are separated by `&`. These *are* sent to the server (unlike the fragment).
- **Fragment** (`#reviews`) is handled entirely client-side by the browser — it's used to jump to a section of an already-loaded page, and it is **never transmitted to the server** in the request.

### URL Encoding (a commonly missed detail)
URLs can't contain spaces or certain special characters directly — they get **percent-encoded**. A space becomes `%20`, `&` inside a value becomes `%26`, and so on. This is why URLs sometimes look messy with lots of `%XX` sequences — that's just encoding, not corruption.

### Absolute vs Relative URLs
- **Absolute URL**: the full address, e.g. `https://example.com/images/logo.png`
- **Relative URL**: just the path, assumed to be relative to the current page, e.g. `/images/logo.png` or `images/logo.png` — the browser fills in the scheme/host automatically based on the current page's location.

---

## 3. HTTP vs HTTPS

### HTTP (HyperText Transfer Protocol)
The shared rulebook the client and server agree to follow for every exchange — defining exactly how requests and responses must be structured (method, path, headers, body, status code).

### The core weakness of plain HTTP
HTTP transmits everything as **plain, readable text**. If your data crosses a public WiFi network, or passes through your ISP, anyone positioned to intercept that traffic (a "man-in-the-middle") can read it — including passwords, card numbers, or private messages, in full.

### HTTPS = HTTP + TLS
HTTPS is not a separate protocol from scratch — it's ordinary HTTP, wrapped inside a **TLS (Transport Layer Security)** encryption layer.

```
HTTP:  Browser <----plain text----> Server   (readable if intercepted)
HTTPS: Browser <--TLS-encrypted---> Server   (unreadable if intercepted)
```

Key facts:
- HTTPS defaults to port **443**; HTTP defaults to port **80**.
- The server needs a valid **SSL/TLS certificate**, issued by a trusted **Certificate Authority (CA)**, proving it really owns that domain.
- Browsers show a padlock for HTTPS and actively flag plain HTTP sites as "Not Secure" — especially on pages with password/payment fields.
- **"Mixed content" warning**: if an HTTPS page tries to load a resource (like an image or script) over plain HTTP, browsers often block or warn about it, since that one insecure resource could be tampered with even on an otherwise secure page.

### What HTTPS does *not* guarantee
- It does not verify the site's content is trustworthy, honest, or malware-free — only that the *connection* itself can't be read or tampered with in transit.
- It does not hide *that* a connection is happening or which IP/domain you're connecting to — only the actual content of the exchange is hidden.

### A note on versions (context, not required memorization)
HTTP has evolved — HTTP/1.1 (older, one request at a time per connection typically), HTTP/2 (multiplexes multiple requests over one connection, faster), HTTP/3 (built on a different transport, QUIC, for even better performance). The core request/response concepts you're learning apply across all versions — only the underlying transport efficiency differs.

---

## 4. HTTP Methods

Every HTTP request specifies a **method** — a verb describing the intended action on a resource.

| Method | Purpose | Example |
|---|---|---|
| **GET** | Retrieve data, no changes | Fetch a webpage, get order details |
| **POST** | Create new data | Submit a form, place an order |
| **PUT** | Replace/update a resource fully | Overwrite an entire profile |
| **PATCH** | Update part of a resource | Change just the email field |
| **DELETE** | Remove data | Delete an order |
| **HEAD** | Like GET, but only returns headers, no body | Check if a resource exists/its size, without downloading it |
| **OPTIONS** | Ask what methods/permissions are allowed | Used automatically by browsers for CORS preflight checks |

```
GET /orders/45        → "give me order 45"
DELETE /orders/45      → "delete order 45"
POST /orders           → "create a new order"
```

Same path, different method = a completely different action — this is a core REST idea you'll revisit in Topic 7.

### Idempotency
A method is **idempotent** if calling it once or many times leaves the server in the *same resulting state*.

| Method | Idempotent? | Why |
|---|---|---|
| GET | ✅ Yes | Reading doesn't change anything |
| PUT | ✅ Yes | Replacing with the same data repeatedly = same end state |
| DELETE | ✅ Yes | Deleting an already-deleted thing changes nothing further |
| HEAD | ✅ Yes | Just reads metadata |
| POST | ❌ No | Typically creates a *new* thing each call |
| PATCH | ⚠️ Depends | Not guaranteed — depends on what the partial update does |

### Safe vs Idempotent (a subtle distinction worth knowing)
- **Safe** methods don't change server state at all (GET, HEAD).
- **Idempotent** methods may change state, but repeating them doesn't change it *further* (PUT, DELETE).
- All safe methods are idempotent, but not all idempotent methods are safe (PUT changes state on the first call, but repeating it doesn't make it "more changed").

> Practical impact: browsers warn "resubmit this form?" when refreshing a page loaded via POST, since repeating it isn't safe — but never warn about refreshing a GET.

---

## 5. Request/Response Structure + Headers

### Anatomy of an HTTP Request

```
1. Request Line   → GET /orders/45 HTTP/1.1
2. Headers        → Key: Value metadata
3. Body           → Actual data (optional — mainly used with POST/PUT/PATCH)
```

Example:
```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer abc123

{"username": "raj", "password": "1234"}
```

- GET requests almost never have a body — you're asking for something, not sending data.
- The **request line** contains the method, the path (not the full URL — the host is separately sent via the `Host` header), and the HTTP version.

### Anatomy of an HTTP Response

```
1. Status Line   → HTTP/1.1 200 OK
2. Headers       → Key: Value metadata
3. Body          → The actual content returned
```

Example:
```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 58

{"id": 45, "item": "Shoes", "status": "shipped"}
```

### Common Headers

| Header | Used in | Meaning |
|---|---|---|
| `Content-Type` | Request & Response | Format of the body (`application/json`, `text/html`, `multipart/form-data`) |
| `Content-Length` | Response | Size of the body in bytes |
| `Authorization` | Request | Credentials/token proving identity |
| `Host` | Request | Which domain is being targeted (needed since one server IP can host many domains — "virtual hosting") |
| `User-Agent` | Request | Info about the browser/device making the request |
| `Set-Cookie` | Response | Tells the browser to store a cookie |
| `Cache-Control` | Response | Rules for caching |
| `Accept` | Request | What response formats the client can handle (e.g., `Accept: application/json`) |
| `Referer` | Request | The page the request originated from (misspelled historically in the spec, but that's the real header name) |

> Headers describe the data — they aren't the data itself. If a `Content-Type` header claims JSON but the actual body is HTML text, the receiver will try to parse it as JSON, fail, and throw an error, rather than correctly rendering the HTML.

### A quick note on request bodies beyond JSON
Not every body is JSON. Forms submitted the traditional HTML way often use `Content-Type: application/x-www-form-urlencoded` (key=value pairs like a query string) or `multipart/form-data` (used specifically when uploading files, since binary file data can't be represented as plain key=value text).

---

## 6. HTTP Status Codes

The first digit tells you the category:

| Range | Category | Meaning |
|---|---|---|
| **1xx** | Informational | Request received, still processing |
| **2xx** | Success | Everything worked |
| **3xx** | Redirection | Resource moved, go elsewhere |
| **4xx** | Client Error | The client made a mistake |
| **5xx** | Server Error | The server messed up |

### Common codes, expanded

**2xx**
- `200 OK` — standard success
- `201 Created` — new resource successfully created (typically returns the created resource in the body)
- `204 No Content` — success, but nothing to send back (common after DELETE)

**3xx**
- `301 Moved Permanently` — resource has a new permanent URL; clients should update bookmarks/links
- `302 Found` — temporary redirect, current URL still valid long-term

**4xx**
- `400 Bad Request` — malformed/invalid request
- `401 Unauthorized` — not authenticated (no valid credentials at all)
- `403 Forbidden` — authenticated, but not permitted
- `404 Not Found` — resource doesn't exist
- `405 Method Not Allowed` — e.g., trying `DELETE` on an endpoint that only supports `GET`
- `429 Too Many Requests` — client is being rate-limited (sent too many requests too fast)

**5xx**
- `500 Internal Server Error` — generic server crash/bug
- `502 Bad Gateway` — a server acting as a proxy/gateway got an invalid response from an upstream server
- `503 Service Unavailable` — server overloaded or down for maintenance
- `504 Gateway Timeout` — an upstream server took too long to respond

### 401 vs 403 (the classic confusion)
- `401` = "I don't know who you are" — like showing up to a locked building with no ID badge at all.
- `403` = "I know who you are, but you're not allowed" — like showing a valid badge, confirmed genuine, but it doesn't have clearance for this specific room.

---

## 7. REST APIs

### The problem REST solves
Without shared conventions, every API designs URLs/methods differently — one uses `/getUser?id=5`, another `/fetchUserData/5`, another `/user_info/5`. Clients have to learn a new inconsistent style for every API.

### REST (Representational State Transfer)
A convention: **URLs represent resources (nouns)**, and **HTTP methods represent actions (verbs)** on them.

```
Resource: "users"

GET    /users        → get all users
GET    /users/5       → get user 5
POST   /users         → create a new user
PUT    /users/5       → replace user 5 entirely
PATCH  /users/5       → update part of user 5
DELETE /users/5       → delete user 5
```

One URL (`/users/5`), five different actions, decided purely by the method. Good REST style avoids putting verbs in the URL (`/getUser`, `/deleteUser` is bad style).

### Nesting resources for relationships

```
GET /users/5/orders        → all orders belonging to user 5
GET /users/5/orders/45     → a specific order belonging to user 5
```

The path structure itself tells a hierarchy story. Resource names are conventionally **plural** (`/users`, `/products`, `/posts`), since the path represents a collection you're indexing into.

### Breaking down the name "REST"
- **Resource** = a "thing" the API manages (a user, an order, a product)
- **Representation** = the format returned — usually JSON, sometimes XML
- **State Transfer** = client and server exchange the current state of a resource through requests/responses; the server has no memory of past interactions

### Statelessness (a core REST principle)
Every request must carry all information the server needs to understand and process it (e.g., an `Authorization` header on *every* request) — the server does not remember previous requests from that client. This is directly why cookies/tokens (Topics 10, 13) exist: something has to carry identity forward since the protocol itself won't.

### A couple of additional real-world conventions (context)
- **API versioning**: many REST APIs include a version in the URL, e.g. `/v1/users`, so breaking changes don't affect existing clients.
- Query parameters are often used for filtering/sorting/pagination on collection endpoints, e.g. `GET /users?sort=name&page=2`, without needing new "actions" — the resource is still `users`, just filtered.

---

## 8. JSON (JavaScript Object Notation)

A lightweight, text-based format for representing structured data — the most common format exchanged by REST APIs today.

```json
{
  "id": 45,
  "name": "Raj",
  "isActive": true,
  "score": 87.5,
  "tags": ["admin", "verified"],
  "address": {
    "city": "Delhi",
    "pincode": 110001
  },
  "manager": null
}
```

| Type | Example |
|---|---|
| String | `"Raj"` (always double quotes — single quotes are invalid JSON) |
| Number | `45`, `87.5` |
| Boolean | `true` / `false` |
| Array | `["admin", "verified"]` |
| Object | `{ "city": "Delhi" }` — key-value pairs |
| Null | `null` |

Objects and arrays can nest freely, letting you represent arbitrarily complex data.

### Parsing and Stringifying
- **Parse**: JSON text → programming language object
- **Stringify**: programming language object → JSON text

```javascript
let jsonText = '{"id": 45, "name": "Raj"}';
let obj = JSON.parse(jsonText);
console.log(obj.name); // "Raj"

let data = { id: 45, name: "Raj" };
let text = JSON.stringify(data);
console.log(text); // '{"id":45,"name":"Raj"}'
```

This mirrors what happens across an HTTP exchange: client stringifies → sends as request body → server parses → processes → server stringifies its own response → client parses the response.

### Common JSON pitfalls (worth knowing)
- **No trailing commas** allowed — `{"a": 1,}` is invalid JSON, even though it's fine in JavaScript object literals.
- **No comments** allowed in JSON — unlike most programming languages.
- **Keys must be double-quoted strings** — `{name: "Raj"}` (no quotes on the key) is invalid JSON, even though it's valid in plain JS.
- JSON itself is language-agnostic — despite the name (JavaScript Object Notation), it's used by virtually every language (Python, Java, Go, etc.), not just JavaScript.

### JSON vs XML (context — why JSON "won")
Before JSON became dominant, many APIs used XML, which is more verbose (`<name>Raj</name>` vs `"name": "Raj"`). JSON is lighter, easier to read, and maps naturally onto objects/arrays already used in most programming languages — which is largely why it became the default for web APIs.

> `Content-Type: application/json` tells the receiver to treat the body as JSON — forgetting to set it can cause the server to misinterpret the body, often resulting in a `400 Bad Request`.

---

## 9. Caching Basics

**Caching** = storing a copy of a response so future requests can reuse it instead of going back to the server.

```
1st visit:  Browser ---request---> Server ---response (logo.png)---> Browser stores a copy
2nd visit:  Browser checks cache -> found -> uses cached copy, no request sent at all
```

This reduces bandwidth use, speeds up load times, and reduces server load.

### `Cache-Control` directives

| Directive | Meaning |
|---|---|
| `max-age=3600` | Cache valid for 3600 seconds |
| `no-cache` | Must revalidate with server before reuse (still stores it, but always checks freshness first) |
| `no-store` | Don't cache at all — for sensitive data like banking info |
| `public` | Cacheable by the browser *and* shared caches (like CDNs) |
| `private` | Cacheable only by that individual user's browser, not shared/public caches |

### Revalidation
Rather than blindly re-downloading everything after expiry, the browser can ask "has this actually changed?" using:
- **`ETag`** — a unique fingerprint/hash of the content
- **`Last-Modified`** — a timestamp of when it last changed

```
Browser: "GET logo.png, I have ETag 'abc123' — changed?"
Server: unchanged → 304 Not Modified (no body sent, reuse cached copy)
Server: changed   → 200 OK + new content
```

### Where caching can happen (a layer you might not have considered)
- **Browser cache** — stored on your own device.
- **Proxy/shared cache** — e.g., your company or ISP's network cache, shared across many users.
- **CDN (Content Delivery Network)** — servers geographically distributed, caching content closer to users worldwide, so a user in India doesn't have to fetch a static file from a server in the US every time.

### Cache-busting (a practical technique)
If a file's *content* changes but its *URL* stays the same (e.g., `style.css`), caches might keep serving the old version. A common trick: include a hash or version number in the filename (`style.abc123.css`) — when the content changes, the filename changes too, forcing a fresh fetch, while unchanged files keep being served from cache.

---

## 10. Cookies

A cookie is a small key-value piece of data that:
1. The server sends via the `Set-Cookie` header
2. The browser stores it
3. The browser **automatically attaches it to every future request** to that same domain

```
Login:  Browser ---POST /login---> Server
        Browser <---Set-Cookie: sessionId=xyz789--- Server

Later:  Browser ---GET /dashboard, Cookie: sessionId=xyz789---> Server
        Server: "sessionId xyz789 belongs to Raj" -> knows who's asking
```

This solves HTTP's statelessness problem — the cookie carries identity across otherwise-independent requests.

### Cookie Attributes

```
Set-Cookie: sessionId=xyz789; Max-Age=3600; HttpOnly; Secure; SameSite=Strict; Domain=example.com; Path=/
```

| Attribute | Meaning |
|---|---|
| `Max-Age` / `Expires` | How long the cookie lives before auto-deleting |
| `HttpOnly` | JavaScript **cannot** access this cookie — protects against theft via malicious injected scripts |
| `Secure` | Only sent over HTTPS, never plain HTTP |
| `SameSite` | Controls whether the cookie is sent on cross-site requests |
| `Domain` | Which domain(s) the cookie applies to |
| `Path` | Restricts the cookie to a specific path on the site |
| (no expiry) | "Session cookie" — deleted when the browser closes |

### `SameSite` values in more detail
- `Strict` — cookie is never sent on cross-site requests at all (most restrictive).
- `Lax` — cookie is sent on some cross-site cases, like clicking a link to navigate to the site, but not on background cross-site requests (a common default balance).
- `None` — cookie is sent on all cross-site requests (must be paired with `Secure`).

### First-party vs Third-party cookies
- **First-party cookie** — set by the site you're actually visiting.
- **Third-party cookie** — set by a *different* domain embedded in the page you're visiting (e.g., an ad network's tracking script). These are widely used for cross-site tracking, and many browsers now restrict or block them by default for privacy reasons.

### Practical limits
Cookies are small — typically limited to around 4KB, and browsers cap the number of cookies per domain — one reason large amounts of client-side data use localStorage instead (next topic).

---

## 11. localStorage & sessionStorage

Browser storage mechanisms, accessible only via JavaScript, that **never get sent to the server automatically** — unlike cookies.

```javascript
// Saving data
localStorage.setItem("theme", "dark");
sessionStorage.setItem("draftText", "Hello world");

// Reading data
localStorage.getItem("theme");       // "dark"
sessionStorage.getItem("draftText"); // "Hello world"

// Removing data
localStorage.removeItem("theme");
```

Both store simple key-value **string** pairs (if you store an object, you must `JSON.stringify()` it first, and `JSON.parse()` it back when reading).

### Difference: lifetime and scope

| | localStorage | sessionStorage |
|---|---|---|
| Lifetime | Persists **forever**, until explicitly cleared | Cleared when the **tab/window closes** |
| Scope | Shared across all tabs of the same site | Only available in that specific tab |
| Typical use | Theme preference, cart items across visits | Multi-step form data within a single session/tab |

### Same-origin restriction applies here too
Just like cookies and CORS, both localStorage and sessionStorage are scoped to the **origin** — one website's JS cannot read another website's localStorage data, even if both are open in the same browser.

### When to use cookies vs Web Storage
- **Cookies** — when the *server* needs to see the data on every request (e.g., a session ID for authentication).
- **localStorage/sessionStorage** — when only the *browser/JS* needs the data, and the server doesn't care (e.g., UI preferences, draft form text).
- Avoid storing sensitive tokens in Web Storage where possible — there's no `HttpOnly`-equivalent protection; any JavaScript running on the page (including a malicious injected script, in an attack called XSS) can read it freely.

### A storage option you may encounter later (just context)
For larger or more structured client-side data (not just simple key-value strings), browsers also offer **IndexedDB** — a more powerful, database-like storage system. Not something to master right now, just good to know it exists for bigger use cases.

---

## 12. CORS (Cross-Origin Resource Sharing)

### The problem it solves
If you're logged into `bank.com` (holding a valid session cookie) and you visit `evil.com` in another tab, and `evil.com`'s JavaScript could freely call `bank.com/transfer-money`, your browser would auto-attach your `bank.com` cookie to that request — letting the malicious site act as you, without consent.

### Same-Origin Policy
By default, browsers block JavaScript on one **origin** from reading responses from a different origin.

**Origin** = scheme + host + port:
```
https://example.com:443   → one origin
http://example.com:443    → different (scheme differs)
https://api.example.com   → different (host differs)
https://example.com:8080  → different (port differs)
```

### CORS — the server explicitly opts in
The server can tell the browser which origins are allowed to read its responses:
```
Access-Control-Allow-Origin: https://myapp.com
```

```
1. JS on myapp.com sends a request to api.backend.com
2. api.backend.com responds with data + Access-Control-Allow-Origin: https://myapp.com
3. Browser checks: is myapp.com allowed? Yes -> lets JS read the response
```

If that header is missing or mismatched, the browser **blocks JS from reading the response**, even if the server actually processed the request successfully. CORS is enforced by the **browser**, not the server — the server merely declares permissions.

`Access-Control-Allow-Origin: *` means any origin is allowed — common for public, non-sensitive APIs.

### Simple requests vs Preflighted requests
Not every cross-origin request triggers a preflight. Browsers treat some requests as "simple" (allowed without a preflight check) if they meet specific conditions — generally: using GET, POST, or HEAD, with only a small set of "safe" headers, and simple content types like form data. Anything outside that — DELETE/PUT, custom headers like `Authorization`, `Content-Type: application/json` in some cases — triggers a **preflight**.

### Preflight requests (`OPTIONS`)
```
Browser: OPTIONS /orders/45   (asking: "would you allow a DELETE from myapp.com?")
Server:  responds with Access-Control-Allow-Origin, Access-Control-Allow-Methods, etc.
Browser: if allowed -> sends the actual DELETE request
Browser: if not allowed -> blocks it, real request never sent
```

### Credentials with CORS (an extra detail)
By default, cross-origin requests don't send cookies. If a request needs to include cookies/credentials cross-origin, the client must explicitly opt in, and the server must respond with `Access-Control-Allow-Credentials: true` — and in that case, `Access-Control-Allow-Origin` cannot be `*`, it must name the exact origin.

---

## 13. Authentication Basics

**Authentication** = "Who are you?" (proving identity)
**Authorization** = "What are you allowed to do?" (401 vs 403 from Topic 6 maps exactly onto this distinction)

### General flow
```
1. User submits username + password (POST /login)
2. Server checks credentials against its database
3. If valid, server creates proof of identity and sends it back
4. Client includes that proof on every future request
5. Server checks the proof each time, without needing the password resent
```

### Passwords are never stored in plain text (an important related fact)
A well-built server never stores your actual password — it stores a **hashed** version (a one-way scrambled form). When you log in, the server hashes what you typed and compares hashes, rather than comparing raw passwords. This way, even if the database is ever leaked, actual passwords aren't directly exposed.

### Session-based Auth (cookie-based)
```
Login -> Server creates a session, stores it (sessionId -> user) in its own memory/DB
Server sends: Set-Cookie: sessionId=xyz789
Every request -> cookie auto-sent -> server looks up sessionId in its storage -> finds user
```
- Server must **store** session data — this is "stateful" on the server side.
- Relies on cookies (which auto-attach, as covered in Topic 10).

### Token-based Auth (commonly JWT — JSON Web Token)
```
Login -> Server creates a signed token: {"userId": 5, "role": "admin"}
Server sends the token in the response body
Client stores it and sends manually: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Server verifies the token's signature -> trusts the info inside, no DB lookup needed
```
- Server stores nothing about active sessions — this is **stateless authentication**.

### Signing — why tokens can't be forged
A JWT = `header.payload.signature`. The payload is just base64-encoded (readable by anyone, **not encrypted**), but the **signature**, created using a secret key only the server knows, prevents tampering. If someone edits the payload (e.g., changes `userId: 5` to `userId: 1`), the signature no longer matches, and the server rejects the token.

> Because the payload is readable, JWTs should never contain sensitive data like plaintext passwords.

### Token theft and revocation risk
If an attacker steals a valid JWT, they can simply replay it — no need to forge anything — and the server will authenticate them as the victim, since a bearer token essentially means "whoever holds this token is treated as the account owner." If there's no revocation mechanism, the stolen token remains usable until it naturally expires, even if the real user changes their password. This is why many real systems use **short-lived access tokens + longer-lived refresh tokens** — it shrinks the window an attacker can exploit a stolen token.

### A note on OAuth and multi-factor auth (context, not required depth yet)
- **OAuth** is a broader standard for delegated authorization (e.g., "Log in with Google") — it builds on top of concepts like tokens but adds a flow for one service to grant limited access on a user's behalf to another service.
- **Multi-factor authentication (MFA)** adds a second proof of identity beyond just a password (like an OTP or authenticator app), reducing risk if a password alone is compromised.

---

## 14. Browser Rendering Basics

### The Rendering Pipeline

```
1. Parse HTML  -> builds the DOM (Document Object Model) — tree of elements
2. Parse CSS   -> builds the CSSOM (CSS Object Model) — tree of style rules
3. Combine     -> DOM + CSSOM = Render Tree (only visible elements, with computed styles)
4. Layout      -> calculate exact position/size of every element ("reflow")
5. Paint       -> draw pixels — colors, text, borders, images — onto the screen
```

- **DOM** = the structural tree of your HTML (`<body>` contains `<div>` contains `<p>`, etc.) — and it's also what JavaScript manipulates when it changes the page.
- **CSSOM** = the tree of style rules (e.g., "this div is red, this text is 16px").
- **Render Tree** = the merge of both — only elements that will actually be visually displayed; elements with `display: none` are excluded entirely (whereas `visibility: hidden` elements are still in the render tree, just invisible — a subtle but useful distinction).

### Script Loading Behavior
By default, when the HTML parser hits a `<script>` tag, it **stops parsing HTML entirely**, downloads and fully executes the script, then resumes parsing. This is why placing scripts carelessly (especially in `<head>`) can visibly delay when the page appears.

```html
<script src="app.js" defer></script>
```
- `defer` — downloads in the background, but runs only **after** HTML parsing is fully done. Scripts with `defer` run in the order they appear.
- `async` — downloads in the background and runs **as soon as it's ready**, potentially interrupting parsing whenever that happens; execution order relative to other scripts isn't guaranteed.
- No attribute — blocks parsing immediately when hit — the traditional, slowest default.

### Reflow and Repaint
- **Repaint** — a purely visual change (color, background) with no effect on layout/position — the browser just redraws the affected pixels. Cheaper.
- **Reflow** — a layout-affecting change (width, height, adding/removing elements, font size changes) — the browser must recalculate the position/size of potentially many elements, then repaint. More expensive.

> Changing `background-color` via JS → repaint only. Changing `width` via JS → triggers a reflow, since neighboring elements may need to shift as a result.

### A couple of relevant browser events (useful practical context)
- **`DOMContentLoaded`** — fires once the HTML has been fully parsed and the DOM is built, *without* waiting for images/stylesheets to finish loading.
- **`load`** — fires only once absolutely everything (including images, stylesheets, subframes) has finished loading.

### Why this matters practically
Understanding this pipeline explains real, common performance advice you'll hear repeatedly as a developer: "put CSS in the `<head>`, put non-critical scripts at the bottom (or use `defer`)," "avoid excessive DOM changes in a loop," "minimize layout-triggering property changes in animations." All of that advice traces directly back to this rendering pipeline.

---

## Key Cross-Topic Connections
- URL's port field (Topic 2) → directly maps to HTTP vs HTTPS default ports 80/443 (Topic 3).
- `Content-Type` header (Topic 5) → determines how JSON is interpreted (Topic 8) and interacts with CORS "simple request" rules (Topic 12).
- Idempotency (Topic 4) → explains why GET doesn't need a CORS preflight but DELETE does (Topic 12).
- `Authorization` header (Topic 5) → carries the JWT in token-based auth (Topic 13).
- Statelessness in REST (Topic 7) → is exactly the problem cookies (Topic 10) and tokens (Topic 13) exist to solve.
- `304 Not Modified` (Topic 9) → a concrete real-world status code example, building on Topic 6.
- `HttpOnly` cookies (Topic 10) vs localStorage's total JS-readability (Topic 11) → the core security trade-off between the two storage mechanisms.
- Same-Origin Policy (Topic 12) → also applies to localStorage/sessionStorage access (Topic 11), not just to reading HTTP responses.
