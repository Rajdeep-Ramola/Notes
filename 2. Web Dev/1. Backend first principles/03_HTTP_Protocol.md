# HTTP Protocol and Backend Fundamentals

# HTTP Protocol Introduction

HTTP is the medium through which browsers communicate with servers.

Browsers use HTTP to:

* Send data to servers
* Receive data from servers

There are many protocols that clients and servers can use to communicate, but HTTP is one of the most widely used protocols, especially in web development.

Two core ideas are at the heart of HTTP:

1. Statelessness
2. Client-server model

---

# Statelessness in HTTP

## What Does Stateless Mean?

Statelessness means:

> HTTP has no memory of past interactions.

Every HTTP request carries all the information necessary for the server to process it.

That includes:

* Headers
* URLs
* Methods
* Authentication information
* Session-related data

After the server responds, it forgets about the request.

If the client sends another request later, the server treats it as:

* A completely new request
* An unrelated interaction

---

## Self-Contained Requests

Since the server does not remember previous requests, every request must contain all required data.

For example:

When accessing a user profile:

* The client must send cookies or tokens
* The server uses those credentials to identify the user
* This must happen on every request

The server does not “remember” who the user is between requests.

---

## Benefits of Statelessness

### 1. Simplicity

Stateless design simplifies server architecture because:

* The server does not need to store session information
* Less memory management is required
* System complexity is reduced

---

### 2. Scalability

Statelessness makes systems easier to scale.

Why?

Because:

* Any request can be handled by any server
* No single server needs to track session state
* Requests can easily be distributed across multiple servers

Even if a server crashes:

* Client state is not lost
* No session recovery is needed

---

## State Management Techniques

Even though HTTP itself is stateless, applications often need continuity.

Examples:

* User login sessions
* Shopping carts
* Persistent authentication

To achieve this, developers implement state management techniques such as:

* Cookies
* Sessions
* Tokens

---

# Client-Server Model

In a typical HTTP flow, there are always two parties:

1. Client
2. Server

---

## Client

The client is usually:

* A web browser
* A mobile application
* Another server
* An API consumer

The client initiates communication.

The client is responsible for sending:

* The resource URL
* Headers
* Methods
* Request body data

---

## Server

The server:

* Hosts resources
* Waits for requests
* Processes incoming requests
* Sends responses back

The response could contain:

* HTML pages
* JSON data
* Text files
* Error messages
* APIs
* Other content

---

## Important Principle

HTTP communication is always initiated by the client.

The client sends a request.

The server sends a response.

---

# HTTP vs HTTPS

For most application-level discussions, HTTP and HTTPS can be treated similarly.

HTTPS is essentially:

> A more secure version of HTTP.

HTTPS adds:

* Encryption
* TLS (Transport Layer Security)
* Security certificates

The underlying communication principles remain the same.

---

# TCP and Network Communication

Before clients and servers can exchange HTTP messages, they need a communication medium.

HTTP uses:

# TCP (Transmission Control Protocol)

HTTP requires the transport layer to be:

* Reliable
* Capable of delivering messages correctly
* Able to report errors

---

## TCP vs UDP

The two major internet transport protocols are:

1. TCP
2. UDP

TCP is considered more reliable because it:

* Establishes connections
* Guarantees ordered delivery
* Handles retransmissions

Because of this, HTTP traditionally relies on TCP.

---

# OSI Model Context

Backend engineers mostly work at:

* Layer 7 (Application Layer)

Lower-level networking concepts include:

* TCP handshakes
* TLS encryption
* Packet transmission

These belong more to networking and systems engineering.

For backend development, the important takeaway is:

> Clients and servers establish network connections and exchange messages.

---

# Evolution of HTTP

HTTP evolved through multiple versions over time.

---

## HTTP 1.0

In HTTP 1.0:

* Every request opened a new connection
* Every response closed that connection

This caused inefficiency because:

* Connections had to be repeatedly established
* Performance became slow

---

## HTTP 1.1

HTTP 1.1 introduced:

### Persistent Connections

Multiple requests and responses could use the same TCP connection.

Benefits:

* Reduced overhead
* Better performance

Additional improvements:

* Chunked transfer encoding
* Better caching mechanisms

---

## HTTP 2.0

HTTP 2 introduced:

### Multiplexing

Multiple requests and responses can travel simultaneously over a single connection.

Other improvements:

* Binary framing instead of text
* Header compression (HPACK)
* Server push

### Server Push

Servers can send resources before the client explicitly requests them.

---

## HTTP 3.0

HTTP 3 is built on:

* QUIC protocol
* UDP instead of TCP

Benefits include:

* Faster connection establishment
* Reduced latency
* Better packet loss handling
* Continued multiplexing support
* Reduced head-of-line blocking

---

# HTTP Messages

HTTP communication happens through:

1. Request messages
2. Response messages

---

## Request Message

A request message is sent by the client.

A typical request contains:

### Request Method

Examples:

* GET
* POST
* PUT
* PATCH
* DELETE

---

### Resource URL

The resource the client wants to access.

---

### HTTP Version

Example:

```http
HTTP/1.1
```

---

### Host

The domain or server being contacted.

---

### Headers

Metadata about the request.

---

### Blank Line

Separates headers from the body.

---

### Request Body

Optional data sent to the server.

Commonly used in:

* POST requests
* PATCH requests
* PUT requests

---

## Response Message

A response message is sent by the server.

A typical response contains:

### HTTP Version

Example:

```http
HTTP/1.1
```

---

### Status Code

Example:

```http
200 OK
```

---

### Response Headers

Metadata about the response.

---

### Blank Line

Separates headers from the body.

---

### Response Body

Contains:

* JSON
* HTML
* Text
* Files
* Error messages

---

# HTTP Headers

Headers are one of the most important parts of HTTP communication.

---

## What Are Headers?

Headers are:

> Key-value pairs sent along with requests and responses.

Example:

```http
Content-Type: application/json
Authorization: Bearer token
```

---

# Why Do We Need Headers?

Why not put all information inside:

* The URL?
* The request body?

To understand this, think of a parcel delivery system.

When sending a package:

* Address
* Recipient details
* Phone number
* PIN code

are written on top of the parcel.

Why?

Because delivery systems need quick access to metadata.

If all information were inside the parcel:

* Workers would repeatedly need to open it
* The process would become inefficient

HTTP headers work similarly.

They expose metadata externally so systems can quickly process requests.

---

# Types of HTTP Headers

Headers can be categorized into several groups.

---

## 1. Request Headers

Sent by the client.

They provide information about:

* The client
* Its preferences
* Its capabilities

### Examples

#### User-Agent

Identifies the client type.

Examples:

* Browser
* Mobile app
* Postman
* Another server

---

#### Authorization

Sends credentials.

Examples:

* Bearer tokens
* JWTs

---

#### Accept

Specifies desired response format.

Examples:

* JSON
* HTML
* Text

---

## 2. General Headers

Used in both requests and responses.

Contain metadata about the message itself.

### Examples

* Date
* Cache-Control
* Connection

---

## 3. Representation Headers

Describe the body being transmitted.

### Examples

#### Content-Type

Specifies media type.

Examples:

* application/json
* text/html

---

#### Content-Length

Size of the resource in bytes.

---

#### Content-Encoding

Compression or encoding information.

Examples:

* gzip
* deflate

---

#### ETag

Unique identifier used for caching.

---

## 4. Security Headers

Used to improve security.

---

### HSTS

Strict-Transport-Security

Forces HTTPS communication.

Prevents protocol downgrade attacks.

---

### Content-Security-Policy (CSP)

Restricts allowed content sources.

Helps prevent:

* Cross-site scripting (XSS)

---

### X-Frame-Options

Prevents embedding inside iframes.

Helps mitigate:

* Clickjacking attacks

---

### X-Content-Type-Options

Prevents MIME type sniffing.

---

### Set-Cookies

Using:

* HttpOnly
* Secure flags

Prevents JavaScript access and enforces HTTPS transmission.

---

# Important Concepts Around Headers

## Extensibility

HTTP is highly extensible.

Headers can be:

* Added easily
* Customized
* Extended without changing the protocol itself

Examples:

* Security enhancements
* Custom application headers
* Content negotiation

### Custom Headers

Developers can define headers such as:

```http
X-Custom-Header
```

for application-specific functionality.

---

## Remote Control Concept

Headers behave like a remote control for server behavior.

Clients can influence how servers respond.

### Examples

#### Content Negotiation

Using:

```http
Accept: application/json
```

or:

```http
Accept: text/html
```

The server responds accordingly.

---

#### Caching Control

Headers like:

* Cache-Control
* Expires

control caching behavior.

---

#### Authentication

Authorization headers influence access control.

---

# HTTP Methods

HTTP methods define:

> The intent of the interaction.

Each method has semantic meaning.

---

## GET

Used to:

* Fetch data

GET requests should:

* Not modify server state

---

## POST

Used to:

* Create new data

POST requests usually include:

* Request bodies

Example:

Creating a user or note.

---

## PATCH

Used to:

* Partially update data

Example:

Updating a user's name.

PATCH only changes selected fields.

---

## PUT

Also used for updates.

Difference from PATCH:

* PUT replaces the entire resource
* PATCH updates selected portions

### Rule of Thumb

Use PATCH unless full replacement is specifically required.

---

## DELETE

Used to:

* Remove resources from the server

---

# Idempotent vs Non-Idempotent Methods

## Idempotent Methods

An idempotent method produces the same result no matter how many times it is executed.

### Examples

#### GET

Fetching data repeatedly should not change anything.

---

#### PUT

Replacing a resource multiple times with the same data gives the same result.

---

#### DELETE

Deleting an already deleted resource produces the same final state.

---

## Non-Idempotent Methods

These can produce different outcomes when repeated.

### POST Example

Suppose a user creates a note.

The first POST request:

* Creates one note

The second POST request:

* Creates another note

Same request → different results.

Therefore POST is non-idempotent.

---

# OPTIONS Method and CORS Workflow

The OPTIONS method is commonly used in:

* CORS preflight requests

Developers usually do not use it directly, but it appears frequently in browser network tabs.

---

# Same-Origin Policy

Browsers enforce:

> Same-origin policy

This restricts web pages from making requests to different domains by default.

---

# What Is CORS?

CORS stands for:

> Cross-Origin Resource Sharing

It is a browser-enforced security mechanism controlling cross-origin communication.

Without CORS:

* Browsers block cross-origin requests

Example:

```text
Frontend: example.com
API: api.example.com
```

These are different origins.

---

# Types of CORS Flows

There are two major CORS flows:

1. Simple request flow
2. Preflight request flow

---

# Simple Request Flow

Example:

```text
Frontend: example.com
Backend: api.example.com
```

The browser sends a GET request.

---

## How It Works

### Step 1 — Browser Adds Origin Header

Example:

```http
Origin: example.com
```

---

### Step 2 — Request Reaches Server

The server checks:

* Whether the origin is allowed

---

### Step 3 — Server Responds with CORS Headers

Example:

```http
Access-Control-Allow-Origin: example.com
```

or:

```http
Access-Control-Allow-Origin: *
```

---

### Step 4 — Browser Validates Response

The browser checks:

* Is the Access-Control-Allow-Origin header present?
* Does it match the requesting origin?

If yes:

* Response is allowed through

If no:

* Browser blocks the response
* A CORS error appears

---

# Preflight Request Flow

Browsers sometimes send an additional request before the actual request.

This is called:

> A preflight request.

---

# When Does a Request Become a Preflight Request?

The request must be:

1. Cross-origin
2. AND satisfy at least one additional condition

---

## Condition 1 — Non-Simple Methods

Methods other than:

* GET
* POST
* HEAD

Examples:

* PUT
* DELETE

---

## Condition 2 — Non-Simple Headers

Examples:

* Authorization
* Custom headers

---

## Condition 3 — Non-Simple Content Types

Simple content types include:

* application/x-www-form-urlencoded
* multipart/form-data
* text/plain

If using:

```http
Content-Type: application/json
```

it becomes a preflight request.

Since most modern applications use JSON:

Most API requests trigger preflight requests.

---

# Structure of a Preflight Request

Preflight requests use:

```http
OPTIONS
```

Example components:

* OPTIONS method
* Resource URL
* Origin header
* Access-Control-Request-Method
* Access-Control-Request-Headers

---

## Purpose of Preflight Requests

The browser asks the server:

* Do you allow this method?
* Do you allow these headers?
* Do you allow this origin?

No actual application data is sent.

---

# Preflight Response

If the server supports CORS, it responds with:

### Status Code

```http
204 No Content
```

---

### Important Headers

#### Access-Control-Allow-Origin

Indicates allowed origins.

---

#### Access-Control-Allow-Methods

Specifies allowed methods.

Example:

```http
PUT, DELETE
```

---

#### Access-Control-Allow-Headers

Specifies allowed custom headers.

Example:

```http
Authorization
```

---

#### Access-Control-Max-Age

Tells the browser:

> How long the preflight response can be cached.

This reduces repeated preflight requests.

---

# Final CORS Flow

The complete sequence:

1. Browser sends preflight OPTIONS request
2. Server responds with allowed capabilities
3. Browser validates response
4. Browser sends original request
5. Server returns actual response

---

# CORS Demo Using Burp Suite

The transcript demonstrates CORS using:

* Burp Suite

Burp Suite is commonly used for:

* HTTP interception
* Traffic visualization
* Security testing

---

# Simple Request Demo Explanation

Frontend:

```text
localhost:5173
```

Backend:

```text
localhost:3001
```

Because ports differ, browsers treat them as:

* Different origins

Thus the request becomes cross-origin.

---

## Successful Simple Request

The server includes:

```http
Access-Control-Allow-Origin: localhost:5173
```

The browser allows the response.

---

## Failed Simple Request

When the header is removed:

```http
Access-Control-Allow-Origin
```

The browser blocks the response and reports a CORS error.

---

# Preflight Demo Explanation

A preflight request occurs because:

* Method is PUT
* Authorization header exists
* Content-Type is application/json

All conditions qualify it as a preflight request.

---

## OPTIONS Request Analysis

The browser sends:

```http
OPTIONS
```

with:

* Access-Control-Request-Method
* Access-Control-Request-Headers

---

## Server Response Analysis

The server responds with:

### Allowed Origins

```http
Access-Control-Allow-Origin
```

---

### Allowed Methods

```http
Access-Control-Allow-Methods
```

Examples:

* GET
* POST
* PUT
* DELETE

---

### Allowed Headers

```http
Access-Control-Allow-Headers
```

Examples:

* Content-Type
* Authorization

---

### Max Age

```http
Access-Control-Max-Age
```

Allows browsers to cache preflight results.

---

# HTTP Response Status Codes

HTTP response codes communicate request outcomes in a standardized way.

They help clients determine:

* Success
* Failure
* Required actions
* Error handling

Without status codes:

Clients would need to inspect response bodies manually.

Status codes provide:

* Consistency
* Standardization
* Predictable behavior

---

# Status Code Categories

HTTP status codes are grouped by leading digit.

| Category | Meaning       |
| -------- | ------------- |
| 1xx      | Informational |
| 2xx      | Success       |
| 3xx      | Redirection   |
| 4xx      | Client errors |
| 5xx      | Server errors |

---

# 1xx Informational Responses

## 100 Continue

Indicates:

* Server received headers
* Client may continue sending body

Used in:

* Large uploads

---

## 101 Switching Protocols

Used when:

* Switching protocols

Example:

* HTTP → WebSocket

---

# 2xx Success Responses

## 200 OK

Most common success code.

Means:

* Request succeeded
* Requested resource returned

Common in:

* Successful GET requests

---

## 201 Created

Means:

* New resource successfully created

Common in:

* POST requests

---

## 204 No Content

Means:

* Request succeeded
* No body content returned

Common in:

* OPTIONS responses
* DELETE responses

---

# 3xx Redirection Responses

## 301 Moved Permanently

Resource permanently moved to another URL.

Example:

```text
/user → /person
```

Useful for:

* Backward compatibility

---

## 302 Temporary Redirect

Resource temporarily redirected.

Clients should continue using original URL later.

Useful for:

* Campaigns
* Temporary UI changes
* Routing experiments

---

## 304 Not Modified

Used for caching.

Means:

* Resource has not changed
* Client should use cached version

Often used with:

* ETags

---

# 4xx Client Errors

These errors occur because of client-side problems.

---

## 400 Bad Request

Occurs when:

* Invalid data is sent
* Malformed requests are made

Examples:

* Sending a string instead of a number
* Invalid form fields

---

## 401 Unauthorized

Occurs when:

* Authentication is missing
* Credentials are invalid
* JWT token expired

Common client behavior:

* Redirect user to login page

---

## 403 Forbidden

Occurs when:

* User is authenticated
* But lacks required permissions

Example:

* User A attempting to delete User B’s resource

---

## 404 Not Found

Occurs when:

* Resource does not exist
* URL is incorrect
* Resource was deleted

One of the most recognized status codes.

---

## 405 Method Not Allowed

Occurs when:

* Wrong HTTP method is used

Example:

* Sending PATCH instead of PUT

---

## 409 Conflict

Occurs when:

* Request conflicts with existing state

Example:

* Creating duplicate folder names

---

## 429 Too Many Requests

Used for:

* Rate limiting

Example:

* More than 60 requests per second

---

# 5xx Server Errors

These indicate problems on the server side.

---

## 500 Internal Server Error

Most famous server error.

Occurs when:

* Unexpected exceptions occur
* Server crashes internally
* Unhandled failures happen

---

## 501 Not Implemented

Occurs when:

* Requested functionality is unsupported
* But may be added later

---

## 502 Bad Gateway

Common in:

* Proxies
* Load balancers
* NGINX setups

Occurs when:

* Upstream server returns invalid response

---

## 503 Service Unavailable

Occurs when:

* Server is overloaded
* Maintenance is ongoing
* Service is temporarily unavailable

---

## 504 Gateway Timeout

Occurs when:

* Upstream server failed to respond in time

Example:

* NGINX waiting for backend response

---

# Advanced HTTP Concepts and Practical Demos

# Response Status Codes Demo

A frontend application and a server are used to simulate different HTTP response codes.

The server responds with different status codes so that the behavior of each response can be understood practically.

Because the requests are cross-origin requests, the browser performs:

* Preflight OPTIONS requests

before each original request.

---

# Examining Common HTTP Response Codes

## 200 OK

A `200 OK` response means:

* The request was successful
* The server processed the request correctly

Example:

```http
200 OK
```

This is the most common success response.

---

## 201 Created

A `201 Created` response means:

* A new resource was successfully created

Typically returned after:

* POST requests
* Form submissions
* Resource creation operations

Example response:

```http
201 Created
```

The server usually includes details of the newly created resource.

---

## 400 Bad Request

A `400 Bad Request` response means:

* The request data is invalid
* Required data is missing
* The request format is incorrect

Example:

```http
400 Bad Request
```

Response message example:

```text
Missing required data
```

---

## 401 Unauthorized

A `401 Unauthorized` response means:

* Authentication is missing
* Authentication token is invalid
* JWT token expired
* Required credentials were not provided

Example:

```http
401 Unauthorized
```

---

## 403 Forbidden

A `403 Forbidden` response means:

* The user is authenticated
* But does not have permission to perform the action

Example:

```http
403 Forbidden
```

Response message example:

```text
You do not have access
```

---

## 404 Not Found

A `404 Not Found` response means:

* The requested resource does not exist
* The URL is invalid
* The resource was deleted

Example:

```http
404 Not Found
```

Response message example:

```text
The requested resource could not be found
```

---

## 409 Conflict

A `409 Conflict` response means:

* The request conflicts with existing data

Example:

* Creating a folder with a name that already exists

Response example:

```http
409 Conflict
```

Message:

```text
Resource already exists
```

---

## 500 Internal Server Error

A `500 Internal Server Error` response means:

* Something unexpected failed on the server
* An unhandled exception occurred

Example:

```http
500 Internal Server Error
```

Servers generally avoid exposing internal details for security reasons.

---

## 503 Service Unavailable

A `503 Service Unavailable` response means:

* The server is temporarily unavailable
* The service may be overloaded or under maintenance

Example:

```http
503 Service Unavailable
```

Response example:

```text
Please try again later
```

---

# HTTP Caching

## What Is HTTP Caching?

HTTP caching is a technique where:

* Copies of server responses are stored
* Clients reuse old responses when data has not changed

This reduces:

* Server load
* Network bandwidth usage
* Response time

The client avoids repeatedly downloading unchanged resources.

---

# Understanding HTTP Caching Through Request Flow

A browser performs a fetch request to an API resource.

The server responds with several important caching headers.

---

# Important Caching Headers

## Cache-Control

Example:

```http
Cache-Control: max-age=10
```

Meaning:

* Cache this resource for 10 seconds

The browser can reuse the cached resource during that period.

---

## ETag

An ETag is:

> A unique identifier or hash representing a response.

Example:

```http
ETag: "3141"
```

The server may generate the ETag by hashing the response data.

ETags help determine whether the resource changed.

---

## Last-Modified

Example:

```http
Last-Modified: Wed, 12 May 2026 10:00:00 GMT
```

This tells the client:

* The last time the resource changed

The browser can compare timestamps to determine freshness.

---

# Initial Fetch Request

The browser performs:

```http
GET /api/resource
```

The server responds with:

* `200 OK`
* Resource body
* Cache-Control
* ETag
* Last-Modified

The browser stores the resource in cache.

---

# Subsequent Fetch Request

When the browser fetches the same resource again, it sends conditional headers.

---

## If-None-Match Header

Example:

```http
If-None-Match: "3141"
```

The browser says:

> If the current ETag differs from this value, send updated data.

---

## If-Modified-Since Header

Example:

```http
If-Modified-Since: Wed, 12 May 2026 10:00:00 GMT
```

The browser says:

> If the resource changed after this time, send updated data.

---

# 304 Not Modified Response

If the resource has not changed:

The server responds with:

```http
304 Not Modified
```

This means:

* The client should continue using the cached version
* No new response body is needed

This saves bandwidth and improves performance.

---

# Updating the Resource

Suppose the resource is updated.

The server:

* Generates a new ETag
* Updates the Last-Modified value

Example:

```http
ETag: "2943"
```

Now when the browser sends the old ETag:

```http
If-None-Match: "3141"
```

The server detects the mismatch and responds with:

```http
200 OK
```

along with:

* Updated resource body
* New ETag
* Updated Last-Modified value

---

# HTTP Caching Challenges

Traditional HTTP caching can become complex because:

* Servers must manually manage ETags
* Incorrect ETag updates may cause stale data
* Cache invalidation becomes difficult

---

# Modern Client-Side Caching

Modern frontend solutions like:

* React Query
* TanStack Query

provide advanced client-side caching.

Advantages include:

* Automatic refetching
* Fine-grained cache control
* Interval-based updates
* Better cache invalidation

However, understanding HTTP-level caching is still important.

---

# HTTP Content Negotiation

Content negotiation is the process where:

> Clients and servers agree on the best format for exchanging data.

The client specifies preferences.

The server attempts to respond accordingly.

---

# Types of Content Negotiation

There are three major types.

---

## 1. Media Type Negotiation

The client specifies desired formats using:

```http
Accept
```

Examples:

```http
Accept: application/json
Accept: application/xml
Accept: text/html
```

---

## 2. Language Negotiation

The client specifies preferred languages using:

```http
Accept-Language
```

Examples:

```http
Accept-Language: en
Accept-Language: es
```

---

## 3. Encoding Negotiation

The client specifies supported compression formats using:

```http
Accept-Encoding
```

Examples:

```http
Accept-Encoding: gzip
Accept-Encoding: deflate
```

---

# Content Negotiation Demo

A frontend client communicates with a server.

The client specifies:

* Language
* Response format
* Compression encoding

through headers.

---

# Example Request Headers

## Accept-Language

Example:

```http
Accept-Language: en
```

The client requests English content.

---

## Accept

Example:

```http
Accept: application/json
```

The client requests JSON.

---

## Accept-Encoding

Example:

```http
Accept-Encoding: gzip, deflate, br, zstd
```

The browser specifies supported compression algorithms.

---

# Language Negotiation Example

If the client changes:

```http
Accept-Language: es
```

The server responds in Spanish instead of English.

This demonstrates how the server adapts content based on client preferences.

---

# Format Negotiation Example

If the client changes:

```http
Accept: application/xml
```

The server responds with XML instead of JSON.

The same resource can therefore be represented in multiple formats.

---

# Benefits of Content Negotiation

Content negotiation allows:

* Flexible response formats
* Better localization
* Efficient client-server communication
* Adaptation to client capabilities and preferences

---

# HTTP Compression

HTTP compression reduces response size before transmission.

Common compression formats include:

* gzip
* deflate
* Brotli (br)
* zstd

---

# Why Compression Is Important

A large JSON response with 11,000 entries is used as an example.

---

## With Compression Enabled

Response size:

```text
3.8 MB
```

The server responds with:

```http
Content-Encoding: gzip
```

This means:

* The response body is compressed using gzip

The browser automatically decompresses it.

---

## With Compression Disabled

Response size increases dramatically:

```text
26 MB
```

This demonstrates why compression matters.

Without compression:

* More bandwidth is consumed
* Downloads become slower
* Server costs increase

---

# Persistent Connections and Keep-Alive

In early HTTP versions such as HTTP 1.0:

* Every request required a new TCP connection
* Connections were closed immediately after responses

This was inefficient because TCP connection setup is expensive.

---

# Persistent Connections in HTTP 1.1

HTTP 1.1 introduced:

> Persistent connections

A single TCP connection can handle:

* Multiple requests
* Multiple responses

Benefits:

* Reduced latency
* Better performance
* Lower overhead

---

# Keep-Alive Mechanism

Persistent connections are managed using:

```http
Connection: keep-alive
```

This tells the server:

> Keep the connection open for future requests.

---

# Important Keep-Alive Concepts

## Persistent by Default

In HTTP 1.1:

* Connections are persistent by default
* Explicit configuration is usually unnecessary

---

## Optional Parameters

Keep-alive can specify:

### Timeout

How long the connection stays open.

---

### Max

How many requests are allowed before closing the connection.

---

## Closing Connections

Example:

```http
Connection: close
```

This forces the connection to close after the response.

This was the default behavior in HTTP 1.0.

---

# Handling Large Requests and Responses

Applications often need to transfer:

* Images
* Videos
* Audio files
* Large documents

These are much larger than typical JSON payloads.

---

# Multipart Form Data

Multipart requests are commonly used for file uploads.

The request body is split into multiple parts.

This allows binary file data to be transmitted efficiently.

---

# Multipart Upload Demo

A file upload request is performed.

---

## Important Headers

### Content-Length

Specifies total request size.

---

### Content-Type

Example:

```http
Content-Type: multipart/form-data
```

---

### Boundary Parameter

Example:

```http
boundary=----WebKitFormBoundaryXYZ
```

The boundary acts as:

> A delimiter separating different parts of the multipart request.

---

# Why Boundaries Are Needed

Binary data is transmitted in chunks.

The server must know:

* Where one part ends
* Where another part begins

Boundaries define those separations.

---

# Structure of Multipart Data

The boundary appears:

* At the beginning of a part
* At the end of a part

This allows the server to reconstruct uploaded files correctly.

---

# Streaming Large Responses

Servers can also stream large responses back to clients.

Instead of sending the entire file at once:

* The server sends chunks continuously

This is more memory efficient.

---

# Text Event Streams

The response uses:

```http
Content-Type: text/event-stream
```

This tells the client:

* Data will arrive incrementally through streamed events

---

# Keep-Alive During Streaming

The response also includes:

```http
Connection: keep-alive
```

This keeps the TCP connection open while chunks continue arriving.

---

# Chunked Transfer Process

The server continuously sends:

* Small chunks of data

The client:

* Receives chunks incrementally
* Appends them together
* Reconstructs the complete file

This approach is useful for:

* Large text files
* Real-time updates
* Streaming systems
* Server-sent events

---

# SSL, TLS, and HTTPS

Even though backend developers may not directly work with these protocols frequently, understanding them is important.

---

# SSL (Secure Sockets Layer)

SSL was the original protocol used to secure communication between:

* Clients
* Servers

Its purpose was to:

* Encrypt sensitive data
* Prevent interception

Examples of protected data:

* Passwords
* Credit card numbers

---

# Why SSL Was Replaced

SSL became outdated because of:

* Security vulnerabilities

It was replaced by:

# TLS (Transport Layer Security)

---

# TLS (Transport Layer Security)

TLS is the modern secure communication protocol.

TLS provides:

* Encryption
* Authentication
* Data integrity

---

# How TLS Works

TLS uses:

* Certificates

Certificates help:

* Authenticate servers
* Establish encrypted connections

TLS prevents:

* Eavesdropping
* Data tampering
* Interception attacks

---

# TLS Versions

TLS evolves continuously.

The currently recommended version is:

```text
TLS 1.3
```

---

# HTTPS

HTTPS is essentially:

> HTTP combined with TLS security.

Initially HTTPS used SSL.

Modern HTTPS uses TLS.

---

# How HTTPS Works

When visiting an HTTPS website:

1. TLS establishes a secure encrypted connection
2. HTTP communication happens through that encrypted channel

This protects:

* Login credentials
* Sensitive user data
* Financial information

from attackers.

---