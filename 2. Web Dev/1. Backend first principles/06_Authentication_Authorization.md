# Authentication and Authorization

# Introduction to Authentication and Authorization

Authentication and authorization are concepts we encounter every day.

Whenever you:

- Log into a platform
- Sign up for an application
- Access a protected system

you are interacting with authentication and authorization systems.

These are fundamental concepts in backend engineering, security, distributed systems, APIs, cloud platforms, and modern web applications.

---

# Authentication vs Authorization

Although these terms are often used together, they solve two completely different problems.

---

## Authentication

Authentication answers the question:

> Who are you?

Authentication is:

> A mechanism used to assign an identity to a subject.

The subject can be:

- A user
- A browser
- A mobile application
- A server
- A device
- A service

The “context” can be:

- A website
- An operating system
- A mobile phone
- A cloud platform
- An API

Authentication is the process of verifying identity.

---

## Authorization

Authorization answers the question:

> What can you do?

Authorization determines:

- Permissions
- Access levels
- Capabilities
- Roles
- Allowed actions

Examples:

- Can the user delete data?
- Can the user edit content?
- Can the user access admin routes?
- Can the user call a protected API?

Authorization happens **after authentication**.

The system first identifies the user, then determines what the user is allowed to do.

---

# High-Level Goal of This Topic

In this discussion, we explore:

- Historical evolution of authentication
- Stateful vs stateless authentication
- Sessions
- Cookies
- JWTs
- API keys
- OAuth 2.0
- Distributed authentication systems
- Authentication security tradeoffs
- Modern authentication architecture

---

# Historical Evolution of Authentication

To understand modern authentication systems, it is useful to understand how authentication evolved historically.

---

# Early Human Trust Systems

In pre-industrial societies, authentication was mostly:

> Implicit authentication based on trust.

Identity verification depended on:

- Human relationships
- Social trust
- Reputation
- Familiarity

---

## Example: Village Elder Trust Model

Suppose someone in a village wanted to verify another person’s identity.

A respected individual — such as a village elder — could:

- Vouch for the person
- Validate agreements
- Confirm identity

Agreements were often sealed using:

- Handshakes
- Mutual recognition
- Community trust

This was an early form of authentication based entirely on:

> Human contextual trust.

---

# Why This Model Failed to Scale

As societies expanded:

- Populations grew
- Trade increased
- Travel expanded
- Interactions extended beyond local communities

The trust model broke down.

Why?

Because a trusted individual in one village:

- Was not trusted everywhere
- Had no authority across distant regions
- Could not validate identities globally

This limitation introduced the need for:

> Explicit authentication mechanisms.

---

# Medieval Authentication and Wax Seals

During medieval times, societies introduced:

> Physical authentication tokens.

One of the earliest forms was:

# Wax Seals

Wax seals were attached to:

- Letters
- Agreements
- Trade documents
- Official communication

These seals acted as:

- Identity verification mechanisms
- Signatures
- Early authentication tokens

---

## Authentication Principle Used Here

The authentication principle became:

> Something you have.

If a person possessed the seal, they could authenticate themselves.

This is conceptually similar to:

- Security cards
- Smart cards
- Hardware tokens
- Physical authentication devices

---

# Early Authentication Attacks

Wax seals introduced a major problem:

# Forgery

Attackers could:

- Copy seals
- Forge seals
- Bypass verification systems

This became one of the earliest examples of:

> Authentication bypass attacks.

---

# Birth of Cryptographic Thinking

Because seals could be forged, societies began exploring:

- Watermarks
- Secret patterns
- Encrypted markings
- Trade verification mechanisms

This laid the foundation for:

> Cryptographic security systems.

---

# Industrial Revolution and Shared Secrets

During the Industrial Revolution:

- Machines evolved rapidly
- Communication systems advanced
- Telegraph systems emerged

Telegraphs became critical communication infrastructure.

This created the need for:

> Secure message validation.

---

# Telegraph Operators and Passphrases

Telegraph operators began using:

- Pre-agreed passphrases
- Shared secret strings
- Static authentication phrases

This was an early version of:

# Password-based authentication

---

## Authentication Principle Shift

Authentication evolved from:

> Something you have

to:

> Something you know

This became the foundation of modern:

- Passwords
- PIN systems
- Shared secret authentication

---

# Mainframes and Digital Authentication

In the mid-20th century, computing systems evolved into:

# Mainframe systems

Authentication entered its:

> First digital phase.

---

# MIT CTSS Password System (1961)

Researchers at MIT’s Project MAC worked on:

# Compatible Time-Sharing System (CTSS)

They introduced passwords for:

> Multi-user computing systems.

The goal was to allow multiple users to use the same computer without exposing each other’s data.

---

# Major Security Vulnerability

Passwords were stored in:

> Plain text.

This became a serious security issue when:

- Someone printed the password file directly

This incident became historically important because it introduced awareness around:

# Secure password storage.

---

# Birth of Password Hashing

This incident motivated the development of:

- Password hashing
- Cryptographic password storage
- Irreversible transformations

---

# What Is Hashing?

Hashing is a cryptographic process where:

- Input text is transformed
- Into a fixed-length output
- Using a deterministic algorithm

Important properties:

- Same input → same hash
- Output length remains fixed
- Hashes are computationally difficult to reverse

Example idea:

```text
password123 → 5f4dcc3b5aa765d61d8327deb882cf99
```

---

# CIA Triad and Security Principles

Authentication systems became aligned with the core principles of information security:

- Confidentiality
- Integrity
- Availability

These are commonly known as the:

# CIA Triad

---

# Rise of Modern Cryptography

In the 1970s, cryptographic research accelerated.

Researchers like:

- Whitfield Diffie
- Martin Hellman

introduced revolutionary concepts.

---

# Diffie-Hellman Key Exchange

Diffie-Hellman introduced:

> Asymmetric cryptography.

This allowed two parties to:

- Establish a shared secret
- Over an untrusted network

without previously sharing credentials.

---

# Asymmetric Cryptography

Asymmetric cryptography uses:

- Public keys
- Private keys

This became the foundation of:

- Modern authentication protocols
- TLS
- HTTPS
- PKI systems
- Digital signatures

---

# Kerberos and Ticket-Based Authentication

Protocols like:

# Kerberos

introduced:

> Ticket-based authentication.

Kerberos used:

- Trusted third parties
- Issued authentication tickets
- User-service identity validation

This became a precursor to modern:

- Token-based authentication
- JWT systems
- OAuth systems

---

# Internet Era and MFA

As the internet expanded during the 1990s:

- Username/password systems became insufficient
- Brute force attacks increased
- Dictionary attacks emerged

This led to:

# Multi-Factor Authentication (MFA)

---

# MFA Authentication Factors

MFA combines multiple authentication principles.

---

## 1. Something You Know

Examples:

- Passwords
- PINs
- Passphrases

---

## 2. Something You Have

Examples:

- Smart cards
- OTP generators
- Security tokens
- Hardware devices

---

## 3. Something You Are

Examples:

- Fingerprints
- Retina scans
- Face recognition
- Voice recognition

---

# Biometric Authentication

Biometric systems introduced:

- Pattern recognition algorithms
- Statistical identification models
- Physical identity verification

However, biometric systems also introduced challenges:

- False positives
- False negatives
- Template theft
- Biometric data security concerns

---

# Modern Authentication Era

The 21st century introduced:

- Cloud computing
- Distributed systems
- Mobile devices
- APIs
- Microservices

Traditional authentication systems were no longer sufficient.

This led to modern authentication technologies such as:

- OAuth 2.0
- OpenID Connect
- JWTs
- Zero Trust Architecture
- Passwordless authentication
- WebAuthn

---

# Future of Authentication

Several modern technologies may shape the future of authentication.

---

# Decentralized Identity

Technologies like blockchain are being explored for:

- Distributed identity systems
- Self-sovereign identity
- Trustless verification

These systems are still evolving.

---

# Behavioral Biometrics

Behavioral authentication analyzes:

- Typing patterns
- Mouse movement
- User behavior
- Interaction habits

to identify users continuously.

---

# Post-Quantum Cryptography

Quantum computers may eventually break current cryptographic algorithms such as:

- RSA
- Traditional public-key systems

This led to research into:

# Post-Quantum Cryptography

These are cryptographic systems designed to remain secure against quantum computing attacks.

---

# Important Authentication Components

Before discussing authentication architectures, three critical components must be understood:

1. Sessions
2. JWTs
3. Cookies

These concepts appear repeatedly in authentication systems.

---

# Sessions

When HTTP was designed, it was:

# Stateless

This means:

> Every request is treated independently.

HTTP servers do not remember previous interactions automatically.

---

# Why Statelessness Became a Problem

Early websites mostly contained:

- Static pages
- Read-only content
- Minimal interaction

This worked fine.

But modern applications needed:

- Login persistence
- Shopping carts
- User continuity
- Stateful interactions

Examples:

- E-commerce carts
- User dashboards
- Logged-in experiences

---

# Birth of Sessions

Sessions introduced:

> Temporary server-side memory.

A session allows the server to remember users across requests.

---

# Typical Session Workflow

---

## Step 1 — User Logs In

The client sends:

- Username
- Password

to the server.

---

## Step 2 — Session Creation

The server:

- Generates a session ID
- Stores user-related data

Examples:

- User ID
- Roles
- Cart items
- Authentication state

This data is stored in:

- Databases
- Redis
- Memory stores

---

## Step 3 — Session ID Sent to Client

The server sends the session ID back using:

# Cookies

The browser stores this session ID.

---

## Step 4 — Subsequent Requests

Every future request automatically includes:

- The session cookie

The server:

- Extracts the session ID
- Looks up user data
- Identifies the user

This creates:

> Stateful web interactions.

---

# Session Expiration

Sessions usually expire after a certain duration.

Example:

```text
15 minutes
```

After expiration:

- A new session must be created
- Users may need to log in again

---

# Session Storage Evolution

---

## File-Based Sessions

Initially, sessions were stored in:

- Server files

Problems:

- Poor scalability
- Difficult management

---

## Database-Based Sessions

Later, sessions moved to:

- Databases

Advantages:

- Persistence
- Better reliability
- Centralized storage

---

## Distributed Session Stores

Modern systems use:

- Redis
- Memcached

These are:

> In-memory distributed stores.

Advantages:

- Faster lookups
- Scalability
- Distributed access

---

# JWTs (JSON Web Tokens)

JWTs became extremely popular in modern distributed systems.

They were introduced to solve scaling issues created by sessions.

---

# Problems with Stateful Sessions

As applications became globally distributed:

- Session replication became expensive
- Synchronization introduced latency
- Distributed systems became complex

Developers wanted:

> Stateless authentication systems.

---

# What Is a JWT?

A JWT is:

> A self-contained authentication token.

It stores:

- User information
- Claims
- Cryptographic signatures

inside the token itself.

---

# Structure of a JWT

A JWT has three parts:

```text
header.payload.signature
```

---

# 1. Header

The header stores metadata such as:

- Signing algorithm
- Token type

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

# 2. Payload

The payload stores user claims.

Examples:

- User ID
- Roles
- Token issue time

Common fields:

---

## sub

Subject identifier (usually user ID).

---

## iat

Issued-at timestamp.

---

## Additional Custom Fields

Examples:

- name
- role
- permissions

---

# 3. Signature

The signature ensures:

- Integrity
- Authenticity
- Tamper detection

The server signs the JWT using:

- Secret keys
- Cryptographic algorithms

If the token is modified:

- Signature verification fails

---

# Advantages of JWTs

---

## 1. Statelessness

Servers do not need to store session data.

---

## 2. Scalability

JWTs work extremely well in:

- Microservices
- Distributed systems
- Multi-region deployments

Any server can verify the token independently.

---

## 3. Portability

JWTs are lightweight and portable.

They can be:

- Sent in headers
- Stored in cookies
- Passed between systems

---

# Problems with JWTs

Despite their advantages, JWTs introduce important challenges.

---

# Problem 1 — Token Theft

If an attacker steals a JWT:

- They can impersonate the user
- Until the token expires

---

# Problem 2 — Revocation Difficulty

JWTs are stateless.

This means:

- Servers cannot easily revoke tokens
- Tokens remain valid until expiration

---

# Hybrid JWT Approach

Modern systems often use:

> Hybrid authentication systems.

Workflow:

1. JWTs provide stateless scalability
2. A server-side blacklist stores revoked tokens

This introduces partial statefulness again.

---

# Important Tradeoff

If token validation still requires database lookups:

- Why not use sessions entirely?

This is one of the major architectural debates in authentication engineering.

---

# Recommendation for Production Systems

For medium-to-large systems:

> Use a professional authentication provider.

Examples:

- Auth0
- Clerk
- Firebase Auth
- Cognito

Authentication is extremely security-sensitive.

Building authentication incorrectly can introduce severe vulnerabilities.

---

# Cookies

Cookies are another foundational concept in web authentication.

---

# What Is a Cookie?

A cookie is:

> A mechanism for storing data in the browser.

Servers can store information directly inside the client browser.

---

# Important Security Principle

A cookie created by one server:

- Cannot be accessed by another server

This isolation improves security.

---

# Cookie Workflow

---

## Step 1 — Authentication

The user logs in.

---

## Step 2 — Server Sets Cookie

The server sends:

```http
Set-Cookie
```

containing:

- Session ID
- JWT
- Authentication token

---

## Step 3 — Browser Stores Cookie

The browser automatically stores the cookie.

---

## Step 4 — Automatic Request Attachment

All future requests automatically include the cookie.

This allows:

- User identification
- Session continuity
- Automatic authentication

---

# HttpOnly Cookies

A common security practice is using:

# HttpOnly cookies

Advantages:

- JavaScript cannot access them
- Helps reduce XSS token theft

---

# Types of Authentication

Modern backend systems commonly use:

1. Stateful authentication
2. Stateless authentication
3. API key authentication
4. OAuth-based authentication

---

# Stateful Authentication

Stateful authentication uses:

- Sessions
- Persistent session storage

---

# Stateful Authentication Workflow

1. User logs in
2. Server creates session
3. Session stored in Redis/database
4. Session ID stored in cookie
5. Future requests include cookie
6. Server validates session

---

# Advantages of Stateful Authentication

- Centralized control
- Easy revocation
- Real-time session management
- Strong security model

---

# Disadvantages

- Scaling complexity
- Session synchronization
- Distributed infrastructure overhead

---

# Stateless Authentication

Stateless authentication uses:

- JWTs
- Self-contained tokens

---

# Stateless Workflow

1. User logs in
2. Server generates JWT
3. Client stores token
4. Client sends token in Authorization header
5. Server verifies token signature

---

# Advantages of Stateless Authentication

- High scalability
- No session store dependency
- Excellent for distributed systems

---

# Disadvantages

- Difficult revocation
- Token theft risks
- Reduced centralized control

---

# Hybrid Authentication Strategy

A practical production strategy is:

- Stateful auth for web browsers
- Stateless auth for mobile apps and APIs

This balances:

- Security
- Scalability
- Developer convenience

---

# API Key Authentication

API keys solve a completely different problem.

They are primarily used for:

> Machine-to-machine communication.

---

# Why API Keys Exist

Suppose a company provides:

- A web application
- Backend capabilities
- APIs

Some developers may want:

- Programmatic access
- Server-to-server integration
- Automated workflows

without using the web UI.

API keys solve this problem.

---

# API Key Workflow

---

## Step 1 — Generate API Key

A user generates an API key through a dashboard.

Example:

```text
sk_live_xxxxxxxxx
```

---

## Step 2 — Attach Key to Requests

The client sends the key:

- In headers
- In authorization fields
- Through server-defined mechanisms

---

## Step 3 — Server Validates Key

The server identifies:

- User identity
- Permissions
- Usage limits
- Expiration rules

---

# Real-World Example: OpenAI API

ChatGPT UI is one interface.

But OpenAI also exposes:

- APIs
- Programmatic access

Developers can:

- Build custom applications
- Integrate AI into their systems
- Use machine-to-machine workflows

using API keys.

---

# Human-to-Machine vs Machine-to-Machine Communication

---

## Human-to-Machine

Examples:

- Browser login
- Mobile app authentication
- UI interactions

Requires:

- Forms
- Password entry
- User interaction

---

## Machine-to-Machine

Examples:

- Server integrations
- API consumption
- Automation workflows

Requires:

- Programmatic authentication
- No human interaction

API keys are ideal here.

---

# OAuth 2.0 and OpenID Connect

OAuth 2.0 solves another major authentication problem.

Traditional authentication systems become difficult when:

- Multiple services interact
- Third-party access is needed
- Federated identity systems are required

OAuth enables:

> Delegated authorization.

Examples:

- “Login with Google”
- “Login with GitHub”
- Third-party app access

OAuth and OpenID Connect are foundational in:

- Modern cloud systems
- SaaS platforms
- Enterprise authentication
- Federated identity systems

---

# The Problem with Traditional Authentication

Before OAuth existed, authentication workflows were much simpler — but also much more problematic.

The workflow looked like this:

1. Create an account with a username and password
2. Authenticate using those credentials
3. Receive a token or session
4. Continue interacting with the platform

This worked initially.

But as users started using more websites and more internet platforms, a serious problem emerged:

> Credential overload.

Users now had:

- Hundreds of accounts
- Hundreds of passwords
- Credentials spread across multiple platforms

---

# Problems with Traditional Password-Based Authentication

---

## 1. Password Reuse

During the early internet era, users frequently reused passwords.

Common examples:

```text
12345
password
qwerty
admin
```

This created severe security risks.

If one platform experienced a data breach:

- Attackers gained access to passwords
- Users reused those passwords elsewhere
- Multiple accounts became compromised

One breach could compromise:

- Email accounts
- Banking systems
- Social media accounts
- Cloud services

---

## 2. Credential Fatigue

Managing large numbers of accounts became overwhelming.

Users had to remember:

- Different usernames
- Different passwords
- Different security questions

This introduced:

- Poor security practices
- Weak passwords
- Password reuse
- Unsafe storage methods

---

# The Delegation Problem

As the internet evolved, platforms increasingly needed:

> Access to resources owned by other platforms.

This introduced a major architectural challenge known as:

# Delegation

---

# What Is Delegation?

Delegation means:

> Allowing one platform to access specific resources from another platform on behalf of a user.

---

# Examples of Delegation

---

## Example 1 — Travel Apps Accessing Gmail

A travel booking app may want to:

- Scan flight tickets
- Read booking confirmations
- Detect travel schedules

To do this, it may need access to:

- Your Gmail inbox

---

## Example 2 — Social Media Contact Importing

A social media platform may want to:

- Import contacts
- Find friends
- Sync address books

using:

- Google Contacts
- Other social platforms

---

# Initial Solution: Sharing Passwords

Initially, users solved this problem in the worst possible way:

> They shared passwords.

A platform would literally ask for:

- Your Gmail password
- Your Facebook password
- Your account credentials

This was disastrous from a security perspective.

---

# Why Password Sharing Was Dangerous

Sharing passwords meant:

> Granting complete account access.

If someone had your Google password, they could:

- Read emails
- Delete emails
- Access photos
- View calendar data
- Access maps history
- Read contacts
- Modify account settings

There was:

- No permission isolation
- No limited access
- No granular control

---

# Another Major Problem: Revocation

Suppose you shared your password with a platform.

Now you want to revoke access.

What can you do?

Usually:

> Change your password everywhere.

This was extremely inconvenient.

Especially because users reused passwords across platforms.

---

# Birth of OAuth (2007)

To solve the delegation problem, engineers from companies like:

- Google
- Twitter

began designing a standardized solution.

This became:

# OAuth

OAuth stands for:

> Open Authorization

OAuth was revolutionary because it solved:

> Delegated access without password sharing.

---

# Core Idea Behind OAuth

Instead of sharing passwords:

> Platforms share limited-access tokens.

---

# Why Tokens Were Better

Passwords provide:

- Full account access

Tokens provide:

- Limited permissions
- Scoped access
- Controlled capabilities

---

# Example of Scoped Access

Suppose you share:

- Your Google password

The platform gets complete access.

But suppose you share:

- A token that only allows reading contacts

Now the platform can:

- Read contacts

but cannot:

- Delete contacts
- Modify contacts
- Access emails
- Access photos

This is called:

# Permission Scoping

---

# OAuth 1.0 Components

OAuth introduced several important architectural components.

---

## 1. Resource Owner

The user who owns the data.

Example:

- You

---

## 2. Client

The application requesting access.

Example:

- Facebook
- Travel app
- Note-taking app

---

## 3. Resource Server

The platform containing the protected data.

Example:

- Google servers
- Gmail servers
- Google Contacts

---

## 4. Authorization Server

The server responsible for:

- Authenticating users
- Issuing tokens
- Managing permissions

---

# OAuth 1.0 Workflow

---

## Step 1 — Redirect to Authorization Server

The client redirects the user to the authorization server.

Example:

- Facebook redirects you to Google login.

---

## Step 2 — User Grants Permission

The user:

- Logs in
- Grants permissions
- Approves requested access

Example permissions:

- Read contacts
- Read email
- Access calendar

---

## Step 3 — Authorization Server Issues Token

After successful authorization:

- The authorization server sends a token to the client.

Example:

- Google sends a token to Facebook.

---

## Step 4 — Client Accesses Resource Server

The client uses the token to access resources.

Example:

- Facebook reads Google Contacts.

All this happens:

- Programmatically
- Without password sharing

---

# Why OAuth 1.0 Was Revolutionary

OAuth eliminated:

- Password sharing
- Unlimited third-party access
- Credential exposure

It introduced:

- Permission-based delegation
- Controlled access
- Secure resource sharing

---

# Problems with OAuth 1.0

Despite being revolutionary, OAuth 1.0 had limitations.

---

## 1. Complex Implementation

The protocol was difficult for developers to implement correctly.

---

## 2. Cryptographic Signature Complexity

OAuth 1.0 heavily relied on:

- Cryptographic request signatures

This introduced:

- Developer confusion
- Implementation errors
- Increased complexity

---

# Birth of OAuth 2.0

Around 2010, OAuth evolved into:

# OAuth 2.0

OAuth 2.0 simplified many parts of the original protocol.

---

# Major Improvements in OAuth 2.0

---

## 1. Bearer Tokens

OAuth 2.0 introduced:

# Bearer Tokens

Bearer tokens are simpler than cryptographic request signing.

The basic idea:

> Whoever possesses the token can use it.

This made implementation significantly easier.

---

## 2. Multiple Authentication Flows

OAuth 2.0 introduced different flows for different application types.

---

# OAuth 2.0 Flows

---

## Authorization Code Flow

Used for:

- Server-side applications

Most common modern flow.

---

## Implicit Flow

Originally designed for:

- Browser-based applications

Now largely discouraged because of security risks.

---

## Client Credentials Flow

Used for:

> Machine-to-machine communication.

Examples:

- Server-to-server APIs
- Backend integrations
- Automation systems

---

## Device Code Flow

Used for devices with:

- Limited input capability

Examples:

- Smart TVs
- Console authentication
- IoT devices

---

# Important Limitation of OAuth 2.0

OAuth solved:

> Authorization

But it did NOT solve:

> Authentication

This distinction is extremely important.

---

# OAuth vs Authentication

OAuth answers:

> What can the application access?

It does NOT answer:

> Who is the user?

This gap led to another technology.

---

# OpenID Connect (OIDC)

Around 2014, developers introduced:

# OpenID Connect (OIDC)

OIDC was built:

> On top of OAuth 2.0

Its goal was to add:

# Authentication

to the OAuth ecosystem.

---

# Core Idea of OpenID Connect

OIDC introduced:

# ID Tokens

These tokens are usually:

- JWTs

---

# What Does an ID Token Contain?

An ID token typically contains:

- User ID
- User email
- User name
- Authentication timestamp
- Issuing authority

These are stored inside:

- JWT claims

---

# Real-World Example: Sign In with Google

When you click:

```text
Sign in with Google
```

OIDC is working behind the scenes.

The platform receives:

- Your identity
- Your email
- Your profile picture
- Your user information

without implementing its own authentication system.

---

# Typical OpenID Connect Workflow

---

## Step 1 — Redirect User to Authorization Server

Suppose you visit:

- A note-taking application

You click:

```text
Sign in with Google
```

The client redirects you to:

- Google Authorization Server

---

## Step 2 — User Logs In

You authenticate using:

- Your Google account

not the client application's credentials.

---

## Step 3 — User Grants Permissions

You approve requested permissions.

Examples:

- Access email
- Access profile picture
- Access basic identity

---

## Step 4 — Authorization Server Sends Authorization Code

The authorization server sends:

- Authorization code
- Sometimes an ID token

back to the client.

---

## Step 5 — Client Exchanges Authorization Code

The client exchanges the authorization code for:

- Access token
- ID token

---

# Access Token vs ID Token

---

## Access Token

Used to:

- Access protected resources
- Perform actions on behalf of the user

---

## ID Token

Used to:

- Identify the user
- Authenticate the user

Contains:

- User identity information

---

# Example: Accessing Google Keep Notes

Suppose the note-taking application requested access to:

- Google Keep notes

After successful authorization:

- The application receives an access token

Now it can:

- Read your Google Keep notes
- Access only approved resources
- Operate within granted permissions

---

# Why OAuth + OIDC Changed the Internet

Together, OAuth 2.0 and OpenID Connect transformed the internet from:

> Password-sharing chaos

into:

> Permission-based interconnected systems.

These technologies enabled:

- Login with Google
- Login with Facebook
- Login with GitHub
- Third-party integrations
- Secure delegated access

---

# Choosing the Right Authentication Method

Now that we understand multiple authentication systems, the next question becomes:

> Which one should you use?

---

# 1. Stateful Authentication

Use when:

- Building traditional web apps
- Maintaining server-side sessions
- Managing user session state

Best for:

- SaaS applications
- Session-heavy platforms
- Browser-based applications

---

# 2. Stateless Authentication

Use when:

- Building scalable APIs
- Designing distributed systems
- Using microservices

Best for:

- Global infrastructure
- Mobile applications
- Token-based systems

---

# 3. OAuth Authentication

Use when:

- Integrating third-party services
- Supporting social login
- Delegating access

Examples:

- Sign in with Google
- Sign in with Discord
- Sign in with GitHub

---

# 4. API Key Authentication

Use when:

- Enabling machine-to-machine communication
- Providing API access
- Supporting backend integrations

Best for:

- Developer platforms
- API ecosystems
- Automation systems

---

# Introduction to Authorization

After authentication comes:

# Authorization

Authentication determines:

> Who the user is.

Authorization determines:

> What the user can do.

---

# Why Authorization Exists

Suppose you build:

- A note-taking platform

Regular users can:

- Create notes
- Update notes
- Delete notes

But you, as the platform creator, may need:

- Administrative permissions
- Access to deleted notes
- Special maintenance capabilities

Not every user should have those capabilities.

This introduces the need for:

> Permission systems.

---

# Incorrect Approach: Secret Admin Strings

One naive solution is:

- Sending special secret strings with API requests

Example:

```text
admin-secret-xyz
```

Problems:

- Easy to leak
- Difficult to manage
- Extremely insecure
- Impossible to scale cleanly

---

# Need for Structured Authorization

Modern systems require:

- Different users
- Different permissions
- Different access levels

Examples:

- Admins
- Moderators
- Viewers
- Editors
- Members

This led to modern authorization systems.

---

# RBAC (Role-Based Access Control)

The most common authorization model is:

# RBAC

Role-Based Access Control.

---

# How RBAC Works

Users are assigned:

- Roles

Roles are assigned:

- Permissions

---

# Example Roles

---

## User Role

Permissions:

- Read notes

---

## Editor Role

Permissions:

- Read
- Write

---

## Admin Role

Permissions:

- Read
- Write
- Delete
- Access administrative resources

---

# Granular Permissions

RBAC can become extremely detailed.

Example:

| Resource | User | Moderator | Admin |
|---|---|---|---|
| Notes | Read | Read/Write | Full Access |
| Deleted Notes | No Access | No Access | Full Access |

---

# Typical RBAC Workflow

---

## Step 1 — User Authenticates

The user logs in.

---

## Step 2 — Server Determines Role

The server identifies:

- User role
- Assigned permissions

This information may come from:

- JWT claims
- Database lookup
- Session data

---

## Step 3 — Middleware Evaluates Access

Authorization middleware checks:

- Does the user have permission?

---

## Step 4 — Access Granted or Denied

If allowed:

- Request proceeds

If denied:

- Server responds with:

```http
403 Forbidden
```

Meaning:

> The user is authenticated but lacks permission.

---

# Security Best Practices in Authentication

Two extremely important topics:

1. Authentication error messages
2. Timing attacks

---

# Authentication Error Messages

Suppose a login attempt fails.

A naive implementation may send:

```text
User not found
```

or:

```text
Incorrect password
```

These messages are dangerous.

---

# Why Specific Error Messages Are Dangerous

They leak information to attackers.

---

## Example 1 — User Enumeration

If the server says:

```text
User not found
```

Attackers learn:

- The account does not exist

They move to another username.

---

## Example 2 — Password Confirmation

If the server says:

```text
Incorrect password
```

Attackers learn:

- Username is correct

Now they only need to brute-force the password.

---

# Correct Security Practice

Always use:

> Generic authentication errors.

Example:

```text
Authentication failed
```

regardless of the actual reason.

---

# Timing Attacks

Timing attacks are more advanced but important to understand.

---

# How Timing Attacks Work

Suppose authentication has these steps:

1. Find user
2. Check account lock status
3. Verify password hash

If the username is invalid:

- The server fails immediately

If the password is invalid:

- Password hashing still occurs
- Processing takes longer

Attackers can measure:

> Response timing differences.

This helps them determine:

- Whether usernames are valid
- Which step failed

---

# Defending Against Timing Attacks

---

## 1. Constant-Time Operations

Use:

- Constant-time comparison functions

These prevent timing differences during hash comparison.

---

## 2. Artificial Delays

Even if authentication fails early:

- Introduce fixed delays

Example:

```javascript
setTimeout(...)
```

or:

```go
time.Sleep(...)
```

This prevents attackers from measuring timing differences.

---

# Final Thoughts

Authentication and authorization are foundational pillars of backend engineering.

Modern systems combine concepts from:

- Cryptography
- Networking
- Distributed systems
- Security engineering
- Browser architecture

As a backend engineer, you should deeply understand:

- Sessions
- Cookies
- JWTs
- OAuth 2.0
- OpenID Connect
- RBAC
- Stateful vs Stateless systems
- Authentication security practices

The goal is not simply to memorize technologies.

The goal is to understand:

- Tradeoffs
- Security implications
- Architectural decisions
- Scalability concerns

behind every authentication and authorization system.