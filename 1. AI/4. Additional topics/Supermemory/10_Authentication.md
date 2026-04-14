# 🔐 Authentication

> All API requests require authentication using a Bearer token.

---

## 📑 Table of Contents

1. [API Keys](#1-api-keys)
2. [Connector Branding](#2-connector-branding)
3. [Scoped API Keys](#3-scoped-api-keys)

---

## 1. API Keys

All API requests require authentication using a Bearer token. Our key is included in all requests.

---

## 2. Connector Branding

When users want to connect to external services (Google Drive, OneDrive) and want to see their own app name during authorization instead of "Log in to Supermemory", we provide our credentials via the settings endpoint (part of the Supermemory SDK) to connect our apps to Supermemory.

```javascript
await client.settings.update({
  googleDriveCustomKeyEnabled: true,
  googleDriveClientId: "your-client-id.apps.googleusercontent.com",
  googleDriveClientSecret: "your-client-secret"
});
```

---

## 3. Scoped API Keys

If we want to limit access to Supermemory, we use a scoped API key which restricts Supermemory to a single `containerTag`.

**Allowed endpoints:** `/v3/documents`, `/v3/memories`, `/v4/memories`, `/v3/search`, `/v4/search`, `/v4/profile`

**Creating a scoped key:**

```bash
curl https://api.supermemory.ai/v3/auth/scoped-key \
  --request POST \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  -d '{
    "containerTag": "my-project",
    "name": "my-key-name",
    "expiresInDays": 30
  }'
```

**To disable a scoped key:**

Send a `DELETE` request with the `id` returned at creation time.

```bash
curl https://api.supermemory.ai/v3/auth/scoped-key/KEY_ID \
  --request DELETE \
  --header 'Authorization: Bearer YOUR_API_KEY'
```
