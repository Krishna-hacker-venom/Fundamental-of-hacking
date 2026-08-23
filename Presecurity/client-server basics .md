# Client-Server Basics: Comprehensive Guide to Web Architecture

## Table of Contents
1. [Introduction](#introduction)
2. [Client-Server Architecture](#client-server-architecture)
3. [Request-Response Model](#request-response-model)
4. [Networking Fundamentals](#networking-fundamentals)
5. [HTTP Methods Deep Dive](#http-methods-deep-dive)
6. [URL Anatomy](#url-anatomy)
7. [HTTP Status Codes](#http-status-codes)
8. [Identifying Requests in DevTools](#identifying-requests-in-devtools)
9. [Security Implications](#security-implications)

---

## Introduction

The client-server model forms the foundation of modern web applications and cybersecurity. Understanding how data flows between clients and servers is critical for identifying vulnerabilities, performing security testing, and implementing secure communication protocols.

### Key Definitions

**Client**: An application or device that initiates a request for resources or services. Examples include web browsers, mobile applications, API clients.

**Server**: A computer system that provides resources, services, or data in response to client requests. Runs continuously to handle incoming requests.

**Protocol**: A set of rules governing communication between devices. Defines message format, transmission methods, error handling, and data synchronization.

**Port**: A logical endpoint on a server identified by a 16-bit number (0-65535) that allows multiple services to run on a single IP address.

---

## Client-Server Architecture

### Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                      INTERNET                            │
└──────────────────────┬──────────────────────────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
   ┌────▼────┐                   ┌────▼────┐
   │  CLIENT  │                   │  SERVER │
   │ (Browser)│◄────────────────►│ (Hosts  │
   │          │   Request        │  Data)  │
   └──────────┘   Response        └─────────┘
```

### How It Works

**1. Client Initiates**: User performs an action (clicks link, submits form) → Browser creates request

**2. Transmission**: Request travels through the internet to the server's IP address and specific port

**3. Server Processing**: Server receives request → Processes it → Prepares response

**4. Response Delivery**: Server sends response back to client with status code and requested resources

**5. Client Rendering**: Browser receives response → Parses HTML/CSS/JavaScript → Renders web page

---

## Request-Response Model

### Request Structure

A typical HTTP request contains:

| Component | Description | Example |
|-----------|-------------|---------|
| **Method** | Action to perform on resource | GET, POST, PUT, DELETE |
| **URL** | Target resource location | `https://example.com/api/users` |
| **Headers** | Metadata about request | Content-Type, Authorization |
| **Body** | Data sent with request (optional) | JSON, form data |
| **Cookies** | Client-side state data | Session tokens, preferences |

### Response Structure

A typical HTTP response contains:

| Component | Description | Example |
|-----------|-------------|---------|
| **Status Code** | Success/failure indicator | 200, 404, 500 |
| **Headers** | Metadata about response | Content-Type, Set-Cookie |
| **Body** | Response data | HTML, JSON, file content |
| **Cookies** | Data for client storage | Session ID, tracking ID |

---

## Networking Fundamentals

### Protocol Layers

**Application Layer Protocols** (Cybersecurity Focus)

| Protocol | Purpose | Port | Security |
|----------|---------|------|----------|
| **HTTP** | Unencrypted web communication | 80 | No encryption - vulnerable to interception |
| **HTTPS** | Encrypted web communication | 443 | TLS/SSL encryption - secure |
| **FTP** | File transfer (unencrypted) | 21 | No encryption - avoid for sensitive data |
| **SFTP** | Secure file transfer | 22 | SSH encryption - secure |
| **DNS** | Domain name resolution | 53 | No encryption by default - can be spoofed |
| **SMTP** | Email sending | 25, 587 | Optional TLS encryption |
| **IMAP/POP3** | Email retrieval | 143, 110 | Optional encryption |
| **SSH** | Secure shell access | 22 | Public-key cryptography |

### Ports Explained

**Definition**: A port is a logical communication endpoint, identified by a number (0-65535), allowing a single IP address to support multiple services simultaneously.

**Port Categories**:

**Well-Known Ports (0-1023)**
- Require administrative privileges to use
- Reserved for standard services
- Examples: HTTP (80), HTTPS (443), SSH (22), FTP (21)

**Registered Ports (1024-49151)**
- Assigned by IANA for specific services
- Can be used by any application with appropriate permissions
- Examples: MySQL (3306), PostgreSQL (5432)

**Dynamic/Private Ports (49152-65535)**
- Not assigned to any service
- Ephemeral ports for temporary connections
- Used for client-side connections

### DNS (Domain Name System)

**Definition**: A hierarchical naming system that translates human-readable domain names into IP addresses.

**DNS Resolution Process**:

1. **User Input**: User enters URL in browser (example.com)
2. **Recursive Query**: Browser queries local DNS resolver
3. **Root Nameserver**: Resolver queries root server for TLD location
4. **TLD Nameserver**: Root directs to TLD nameserver (.com, .org, etc.)
5. **Authoritative Nameserver**: TLD directs to domain's authoritative nameserver
6. **IP Response**: Authoritative nameserver returns IP address
7. **Caching**: Resolver caches result and returns to browser
8. **Connection**: Browser connects to server using returned IP address

**DNS Records**:

| Record Type | Purpose | Example |
|------------|---------|---------|
| **A** | Maps domain to IPv4 address | 192.168.1.1 |
| **AAAA** | Maps domain to IPv6 address | 2001:0db8::1 |
| **CNAME** | Alias for another domain | www → example.com |
| **MX** | Mail exchange server location | priority 10 mail.example.com |
| **TXT** | Arbitrary text (SPF, DKIM) | v=spf1 include:_spf.example.com |
| **NS** | Nameserver for domain | ns1.example.com |

**Security Implications**: DNS lacks encryption by default. Attackers can perform DNS spoofing, poisoning, or hijacking attacks.

### IP Addresses

**Definition**: Internet Protocol address, a unique numerical identifier for devices on a network.

**IPv4**: 32-bit address (example: 192.168.1.1)
- Format: Four octets (0-255) separated by dots
- Total addresses: ~4.3 billion

**IPv6**: 128-bit address (example: 2001:0db8::1)
- Format: Eight 16-bit hexadecimal groups
- Total addresses: ~340 undecillion
- Deployment: Increasing due to IPv4 exhaustion

**Special IP Ranges**:

| Range | Purpose |
|-------|---------|
| **127.0.0.1** | Localhost (same device) |
| **192.168.x.x** | Private network (RFC 1918) |
| **10.0.0.0 - 10.255.255.255** | Private network (RFC 1918) |
| **172.16.0.0 - 172.31.255.255** | Private network (RFC 1918) |
| **0.0.0.0** | Current network/default route |

---

## HTTP Methods Deep Dive

### Overview of HTTP Methods

HTTP defines 9 core methods (RFC 7231) for client-server communication. Each method has specific use cases and security implications.

| Method | Purpose | Safe | Idempotent | Body |
|--------|---------|------|-----------|------|
| **GET** | Retrieve resource | Yes | Yes | No |
| **POST** | Create new resource | No | No | Yes |
| **PUT** | Replace resource entirely | No | Yes | Yes |
| **DELETE** | Remove resource | No | Yes | No |
| **PATCH** | Partial resource update | No | No | Yes |
| **HEAD** | Like GET, but no response body | Yes | Yes | No |
| **OPTIONS** | Describe communication options | Yes | Yes | No |
| **CONNECT** | Establish tunnel (proxies) | No | No | No |
| **TRACE** | Echo request (diagnostic) | Yes | Yes | No |

**Safe Methods**: Do not modify server state (GET, HEAD, OPTIONS, TRACE)

**Idempotent Methods**: Multiple identical requests produce same result (GET, HEAD, PUT, DELETE, OPTIONS, TRACE)

### GET Method - Detailed Analysis

**Definition**: Retrieves data from the server without modifying resources. The request data is encoded in the URL query string.

**Characteristics**:
- **Safe**: Does not modify server state
- **Idempotent**: Multiple identical requests return same result
- **Cacheable**: Response can be cached by browser and intermediaries
- **Body**: Never contains a request body
- **URL Length Limit**: Typically 2000-8000 characters depending on browser/server

**GET Request Example**:
```
GET /api/users?id=123&role=admin HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0
Accept: application/json
```

**URL Query String Components**:
```
https://api.example.com/api/users?id=123&role=admin

protocol: https
host: api.example.com
path: /api/users
query string: ?id=123&role=admin
  - parameter 1: id=123
  - parameter 2: role=admin
```

**Security Concerns**:
- Data visible in URL history and browser logs
- Parameters logged by server (access logs)
- No encryption of data (use HTTPS to mitigate)
- Vulnerable to browser cache poisoning
- Subject to CSRF attacks

**Use Cases**:
- Fetching web pages
- Retrieving data listings
- Filtering data
- Pagination
- Search operations

### POST Method - Detailed Analysis

**Definition**: Submits data to server for processing, typically creating new resources. Data is sent in the request body, not the URL.

**Characteristics**:
- **Not Safe**: Modifies server state (creates resources)
- **Not Idempotent**: Multiple identical requests may create duplicate resources
- **Cacheable**: Only with explicit cache headers (rarely cached)
- **Body**: Contains form data, JSON, XML, or other data
- **No URL Length Limit**: Body size limited by server configuration

**POST Request Example**:
```
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 42
Authorization: Bearer token123

{"name":"John Doe","email":"john@example.com"}
```

**Security Advantages Over GET**:
- Data not exposed in URL
- Data not stored in browser history
- Data not logged in server access logs by default
- Allows larger data transmission
- Can include authentication tokens

**Security Concerns**:
- Request body visible in DevTools/proxy tools
- Use HTTPS to encrypt data in transit
- Vulnerable to CSRF attacks (requires token validation)
- No guarantee data is encrypted at rest on server

**Use Cases**:
- User registration/login
- Form submissions
- Creating resources
- File uploads
- Webhook callbacks

### Comparison: GET vs POST

**URL Visibility**:
```
GET Method:
URL: https://api.example.com/login?username=admin&password=secret123
Risk: Password visible in URL, browser history, logs

POST Method:
URL: https://api.example.com/login
Body: {"username":"admin","password":"secret123"}
Risk: Data hidden from URL but visible in request body (use HTTPS)
```

**Identifying Methods in URLs**

**Visual Indicators in Browser**:

| Location | What to Look For |
|----------|------------------|
| **URL Bar** | GET requests show all data in URL. POST requests show only path |
| **DevTools Network Tab** | Method column explicitly shows GET, POST, PUT, DELETE, etc. |
| **Querystring** | Presence of `?` and parameters = GET method |
| **Page Load** | Initial navigation = typically GET. Form submission = POST |

**DevTools Analysis**:

1. **Open DevTools**: F12 or Right-click → Inspect
2. **Navigate to Network Tab**: Shows all requests/responses
3. **Look for Method Column**: First column displays HTTP method
4. **Filter by Type**: Click "XHR" to show API calls only
5. **Inspect Request**: Click request → Headers tab shows full details

**Network Tab Columns**:

| Column | Information |
|--------|-------------|
| **Name** | Resource name/endpoint |
| **Method** | HTTP method (GET, POST, etc.) |
| **Status** | HTTP status code |
| **Type** | Document, XHR, Fetch, Stylesheet, Script, Image |
| **Size** | Response body size |
| **Time** | Request duration |
| **Initiator** | What triggered the request |

---

## URL Anatomy

### Complete URL Structure

```
https://subdomain.example.com:8080/path/to/resource?param=value&foo=bar#section

│      │  │                │    │  │                 │                 │
│      │  │                │    │  │                 │                 └─ Fragment (anchor)
│      │  │                │    │  │                 └─ Query String
│      │  │                │    │  └─ Path
│      │  │                │    └─ Port
│      │  │                └─ Host
│      │  └─ Domain Extension (TLD)
│      └─ Subdomain
└─ Scheme (Protocol)
```

### URL Component Details

**1. Scheme**

**Definition**: Specifies the protocol used for communication.

| Scheme | Protocol | Security | Port |
|--------|----------|----------|------|
| **http** | HyperText Transfer Protocol | Unencrypted | 80 |
| **https** | HTTP Secure (TLS/SSL) | Encrypted | 443 |
| **ftp** | File Transfer Protocol | Unencrypted | 21 |
| **ftps** | FTP Secure | Encrypted | 990 |

**Security Note**: Always use HTTPS for any transmission of sensitive data (passwords, tokens, personal information).

**2. Host (Domain)**

**Definition**: The server's domain name or IP address to which the request is sent.

**Components**:
- **Subdomain**: Optional prefix (www, api, mail, cdn)
- **Domain Name**: Main domain (example)
- **Top-Level Domain (TLD)**: Extension (.com, .org, .gov, .io, .uk)

**Examples**:
- www.example.com (subdomain: www, domain: example, TLD: com)
- api.github.com (subdomain: api, domain: github, TLD: com)
- localhost (local development)
- 192.168.1.1 (IP address instead of hostname)

**3. Port**

**Definition**: The network endpoint on the server.

**Rules**:
- If port is omitted, default port for scheme is used
- Default HTTP port: 80
- Default HTTPS port: 443
- Explicit port format: `:` followed by port number

**Examples**:
- https://example.com (default port 443)
- http://example.com:8080 (custom port 8080)
- localhost:3000 (local development server)

**4. Path**

**Definition**: Specifies the location of the resource on the server.

**Format**:
- Starts with forward slash `/`
- Case-sensitive on Unix-based servers, case-insensitive on Windows
- Separated by slashes representing directory hierarchy

**Examples**:
- `/` = root directory (index.html)
- `/api/users` = path to users endpoint
- `/static/css/style.css` = path to stylesheet

**5. Query String**

**Definition**: Optional parameters passed to the server, used to filter, sort, or customize the response.

**Format**:
- Starts with `?` after the path
- Multiple parameters separated by `&`
- Each parameter: `key=value`
- Special characters URL-encoded

**Examples**:
```
/search?q=cybersecurity&sort=date&limit=10

Components:
- q=cybersecurity (search query)
- sort=date (sort by date)
- limit=10 (return 10 results)
```

**URL Encoding** (Special Characters):

| Character | Encoded |
|-----------|---------|
| Space | %20 or + |
| & | %26 |
| = | %3D |
| # | %23 |
| / | %2F |
| ? | %3F |

**6. Fragment (Anchor)**

**Definition**: Client-side reference to a section within the document. Not sent to the server.

**Format**:
- Starts with `#` after query string
- Used for navigation within page

**Example**:
```
https://example.com/docs#section-authentication

Fragment: section-authentication
→ Browser navigates to heading with id="section-authentication"
→ NOT sent in HTTP request to server
```

---

## HTTP Status Codes

### Status Code Categories

HTTP responses include a three-digit status code indicating the outcome of the request.

### 1xx Informational

Temporary responses indicating the request is being processed.

| Code | Meaning | Use Case |
|------|---------|----------|
| **100** | Continue | Client waiting for permission to send body |
| **101** | Switching Protocols | Upgrading from HTTP to WebSocket |

### 2xx Success

Request succeeded and server returned expected response.

| Code | Meaning | Significance |
|------|---------|--------------|
| **200** | OK | Request successful, response contains data |
| **201** | Created | Resource successfully created (POST) |
| **202** | Accepted | Request accepted but processing continues async |
| **204** | No Content | Request successful but no response body |
| **206** | Partial Content | Partial resource returned (ranges, resumable downloads) |

### 3xx Redirection

Client must perform additional action to complete request.

| Code | Meaning | Security Note |
|------|---------|--------------|
| **300** | Multiple Choices | Multiple resources match request |
| **301** | Moved Permanently | Resource permanently moved (update bookmarks) |
| **302** | Found | Temporary redirect (don't update bookmarks) |
| **304** | Not Modified | Cached version still valid (no data transfer) |
| **307** | Temporary Redirect | Temporary redirect (preserve method) |

**Security Concern**: Redirects can be exploited in phishing attacks. Always verify redirect destination.

### 4xx Client Error

Client made an invalid request.

| Code | Meaning | Common Causes |
|------|---------|--------------|
| **400** | Bad Request | Malformed request syntax |
| **401** | Unauthorized | Authentication required but not provided |
| **403** | Forbidden | Authenticated but lacks permission |
| **404** | Not Found | Resource does not exist |
| **405** | Method Not Allowed | HTTP method not supported for this resource |
| **408** | Request Timeout | Request took too long to complete |
| **429** | Too Many Requests | Rate limit exceeded |

**Cybersecurity Note**: 401 vs 403
- **401**: "Who are you?" (missing authentication)
- **403**: "I know you, but you can't access this" (insufficient privileges)

### 5xx Server Error

Server failed to fulfill valid request.

| Code | Meaning | Severity |
|------|---------|----------|
| **500** | Internal Server Error | Unexpected server error |
| **501** | Not Implemented | Feature not implemented |
| **502** | Bad Gateway | Invalid response from upstream server |
| **503** | Service Unavailable | Server temporarily unavailable (maintenance, overload) |
| **504** | Gateway Timeout | Upstream server not responding |

---

## Identifying Requests in DevTools

### Step-by-Step DevTools Analysis

**Opening DevTools**:
- **Chrome/Edge**: F12 or Ctrl+Shift+I (Windows) / Cmd+Shift+I (Mac)
- **Firefox**: F12 or Ctrl+Shift+I (Windows) / Cmd+Shift+I (Mac)
- **Safari**: Cmd+Option+U (Mac), then Develop → Show Web Inspector
- **Alternative**: Right-click on page → Inspect or Inspect Element

### Network Tab Overview

**Initial Page Load Requests** (HTTPS Example):

```
Name             Method  Status  Type        Size     Time
─────────────────────────────────────────────────────────────
example.com      GET     200     document    45.2 KB  234 ms
/styles.css      GET     200     stylesheet  12.5 KB  156 ms
/script.js       GET     200     script      89.3 KB  445 ms
/api/data        GET     200     xhr         2.1 KB   678 ms
/image.png       GET     200     image       234 KB   123 ms
```

### Inspecting Individual Requests

**1. Headers Tab**

Shows request and response metadata:

**Request Section**:
```
Request URL: https://api.example.com/users/123
Request Method: GET
Status Code: 200 OK
Remote Address: 192.168.1.100:443
Referrer Policy: strict-origin-when-cross-origin

General Headers:
Host: api.example.com
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
```

**Response Headers Section**:
```
Content-Type: application/json; charset=utf-8
Content-Length: 1024
Cache-Control: private, max-age=3600
Set-Cookie: sessionid=abc123; Path=/; HttpOnly
Date: Mon, 23 Aug 2024 10:30:45 GMT
Server: nginx/1.19.0
```

**2. Preview Tab**

Shows formatted response data:

```json
{
  "id": 123,
  "username": "john_doe",
  "email": "john@example.com",
  "role": "user",
  "created_at": "2024-01-15T10:30:00Z"
}
```

**3. Response Tab**

Shows raw response content (unformatted):

```
{"id":123,"username":"john_doe","email":"john@example.com","role":"user","created_at":"2024-01-15T10:30:00Z"}
```

**4. Cookies Tab**

Displays cookies sent and received:

| Name | Value | Domain | Path | Expires | HttpOnly | Secure | SameSite |
|------|-------|--------|------|---------|----------|--------|----------|
| sessionid | abc123def456 | example.com | / | Session | Yes | Yes | Strict |
| preferences | theme=dark | example.com | / | 2025-08-23 | No | Yes | Lax |

### Filtering and Searching

**Filter Types in Network Tab**:

| Filter | Shows |
|--------|-------|
| **XHR/Fetch** | API calls only (XMLHttpRequest) |
| **JS** | JavaScript files |
| **CSS** | Stylesheet files |
| **Img** | Image files |
| **Media** | Audio/video files |
| **Font** | Font files |
| **Doc** | Documents |
| **WS** | WebSocket connections |
| **Manifest** | Web manifest files |

**Search Box**: Filter requests by name, URL, or domain

**Example**:
- Type "api" to show only API calls
- Type ".json" to show JSON requests

### Analyzing POST Requests

**Submitted Form Data Example**:

```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 42

username=admin&password=secretpassword123
```

**DevTools View** (Form Data Tab):

```
Form Data:
username: admin
password: secretpassword123
```

**JSON Payload Example**:

```
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "securepass123"
}
```

### JavaScript Resource Loading

When JavaScript files are loaded via GET method:

**DevTools Network Tab shows**:
```
Name              Method  Status  Type      Size
─────────────────────────────────────────────────
index.html        GET     200     document  15.3 KB
styles.css        GET     200     stylesheet 8.2 KB
script.js         GET     200     script    45.1 KB
jquery.min.js     GET     200     script    28.5 KB
react.js          GET     200     script    120.3 KB
bundle.js         GET     200     script    256.7 KB
```

**Characteristics**:
- All use GET method
- Type column shows "document", "stylesheet", or "script"
- Status 200 indicates successful loading
- Files downloaded sequentially or in parallel
- Browser caches these files for future visits

### Common DevTools Observations

**Resource Loading Analysis**:

| Observation | Meaning | Implication |
|-------------|---------|------------|
| High "Time" value | Request took long to complete | Network latency or server processing delay |
| "304 Not Modified" | Cached version used | Reduces bandwidth |
| "404 Not Found" | Resource doesn't exist | Broken link or 404 error |
| "429 Too Many Requests" | Rate limited | Exceeded server request limits |
| "403 Forbidden" | Access denied | Insufficient permissions |
| Large file size | Resource is large | May impact performance |

---

## Security Implications

### Key Security Concepts Related to Client-Server Communication

**1. HTTPS vs HTTP**

**HTTP (Unencrypted)**:
- Data transmitted in plaintext
- Vulnerable to man-in-the-middle (MITM) attacks
- Attacker can intercept credentials, tokens, personal data
- No server authenticity verification

**HTTPS (Encrypted)**:
- Uses TLS/SSL encryption
- Data encrypted during transit
- Server authenticity verified via certificates
- Protects against MITM attacks
- Standard for all modern web applications

**2. Authentication vs Authorization**

**Authentication (401)**:
- Verifies user identity
- "Who are you?"
- Methods: Username/password, OAuth, API keys, JWT tokens

**Authorization (403)**:
- Verifies user permissions
- "What can you access?"
- Role-based access control (RBAC)
- Attribute-based access control (ABAC)

**3. Cross-Site Request Forgery (CSRF)**

**Definition**: Attacker tricks authenticated user into performing unintended actions.

**Example**:
```
User logged into bank.com
Attacker tricks user into visiting attacker.com
attacker.com contains: <img src="bank.com/transfer?amount=1000&to=attacker">
Browser automatically includes bank.com cookies
Unintended transfer occurs
```

**Prevention**:
- CSRF tokens in forms
- SameSite cookie attribute
- Origin/Referer header validation

**4. SQL Injection (via GET/POST)**

**Vulnerable GET Request**:
```
GET /search?query=' OR '1'='1
```

**Server-side Query**:
```sql
SELECT * FROM products WHERE name = '' OR '1'='1'
```

**Result**: Returns all products instead of search results

**Prevention**:
- Use parameterized queries
- Input validation
- Escape special characters

**5. Cookie Security**

**Secure Cookie Flags**:

| Flag | Purpose |
|------|---------|
| **HttpOnly** | Prevents JavaScript access (prevents XSS cookie theft) |
| **Secure** | Only sent over HTTPS |
| **SameSite** | Controls cross-site transmission (prevents CSRF) |
| **Domain** | Restricts to specific domain |
| **Path** | Restricts to specific path |

**Example Secure Cookie**:
```
Set-Cookie: sessionid=abc123; HttpOnly; Secure; SameSite=Strict; Path=/; Domain=example.com
```

**6. Sensitive Data in URLs**

**Never Put in Query String**:
- Passwords
- API keys
- Authentication tokens
- Personally identifiable information (PII)
- Credit card numbers

**Why**:
- Visible in browser history
- Logged in server access logs
- Exposed in browser cache
- Visible in referrer headers

**Better Approach**: Use POST with HTTPS and send in request body

**7. Port Scanning for Reconnaissance**

**Attacker Perspective**:
```
Open port 22 (SSH) → Likely Linux server
Open port 3389 (RDP) → Likely Windows server
Open port 80, 443 → Web server running
Open port 3306 (MySQL) → Database exposed
Open port 27017 (MongoDB) → NoSQL database exposed
```

**Security Practice**: Close unnecessary ports, run services on non-standard ports

**8. DNS Security Threats**

**DNS Spoofing**:
- Attacker returns fake IP address
- User redirected to attacker's server
- Credentials/data stolen

**DNS Poisoning**:
- Attacker corrupts DNS cache
- Multiple users affected

**Prevention**:
- Use DNSSEC (DNS Security Extensions)
- Use trusted DNS resolvers (1.1.1.1, 8.8.8.8)
- Monitor DNS queries

---

## Practical Cybersecurity Testing

### Testing Checklist

**1. Verify HTTPS Usage**
- [ ] Check scheme is HTTPS, not HTTP
- [ ] Verify SSL certificate validity
- [ ] Check certificate expiration date

**2. Inspect Request Methods**
- [ ] Sensitive operations use POST, not GET
- [ ] Credentials not in query strings
- [ ] Proper method used for each operation (GET for retrieval, POST for modification)

**3. Analyze Response Codes**
- [ ] Success responses are 2xx
- [ ] Proper error codes returned (401 for auth, 403 for permission)
- [ ] No information disclosure in error messages

**4. Cookie Security**
- [ ] HttpOnly flag set
- [ ] Secure flag set for HTTPS
- [ ] SameSite attribute configured
- [ ] Session cookies don't persist

**5. Input Validation**
- [ ] Test with special characters
- [ ] Test with SQL injection payloads
- [ ] Test with cross-site scripting (XSS) payloads
- [ ] Verify server-side validation occurs

**6. Rate Limiting**
- [ ] Rapid requests return 429 status
- [ ] Brute force attacks are mitigated
- [ ] DoS attacks are limited

---

## References and Further Reading

- RFC 7231: Hypertext Transfer Protocol (HTTP/1.1) Semantics and Content
- RFC 7230: Hypertext Transfer Protocol (HTTP/1.1) Message Syntax and Routing
- OWASP Top 10: Web Application Security Risks
- PortSwigger Web Security Academy: HTTP Basics
- MDN Web Docs: HTTP Protocol Documentation
