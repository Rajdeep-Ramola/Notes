# 🔌 Connectors

> Connectors help us connect external platforms to automatically sync documents into Supermemory.

---

## 📑 Table of Contents

1. [Steps](#1-steps)
   - [1.1 Create Connection](#11-create-connection)
   - [1.2 Handle OAuth Callback](#12-handle-oauth-callback)
   - [1.3 Monitor Sync Status](#13-monitor-sync-status)
2. [Connection Management](#2-connection-management)
   - [2.1 List All Connections](#21-list-all-connections)
   - [2.2 Delete Connections](#22-delete-connections)
3. [Different Connectors](#3-different-connectors)

---

## 1. Steps

### 1.1 Create Connection

```python
from supermemory import Supermemory
import os

client = Supermemory(api_key=os.environ.get("SUPERMEMORY_API_KEY"))

connection = client.connections.create(
    'notion',                              # connection type (specifies you want to connect to notion)
    redirect_url='https://yourapp.com/callback',  # where notion will redirect the user
    container_tags=['user-123', 'workspace-alpha'],  # organize and scope documents
    document_limit=5000,                   # max number of docs supermemory will ingest
    metadata={'department': 'sales'}       # attaches custom metadata to entire connection
)

# Redirect user to complete OAuth
print(f'Auth URL: {connection.auth_link}')
print(f'Expires in: {connection.expires_in}')

# Output: Auth URL: https://api.notion.com/v1/oauth/authorize?...
# Output: Expires in: 1 hour
```

### 1.2 Handle OAuth Callback

After user completes OAuth, the connection is automatically established and sync begins.

### 1.3 Monitor Sync Status

```python
from supermemory import Supermemory
import os

client = Supermemory(api_key=os.environ.get("SUPERMEMORY_API_KEY"))

# List all connections using SDK
connections = client.connections.list(
    container_tags=['user-123', 'workspace-alpha']
)

for conn in connections:
    print(f'Connection: {conn.id}')
    print(f'Provider: {conn.provider}')
    print(f'Email: {conn.email}')
    print(f'Created: {conn.created_at}')

# List synced documents (memories) using SDK
memories = client.documents.list(container_tags=['user-123', 'workspace-alpha'])
print(f'Synced {len(memories.memories)} documents')

# Output: Synced 45 documents
```

---

## 2. Connection Management

### 2.1 List All Connections

```python
from supermemory import Supermemory
import os

client = Supermemory(api_key=os.environ.get("SUPERMEMORY_API_KEY"))

connections = client.connections.list(container_tags=['org-123'])

for conn in connections:
    print(f"{conn.provider}: {conn.email} ({conn.id})")
    print(f"Documents: {conn.document_limit or 'unlimited'}")
    print(f"Expires: {conn.expires_at or 'never'}")

# Output: notion: user@company.com (conn_abc123)
# Output: Documents: 5000
# Output: Expires: never
```

### 2.2 Delete Connections

| Parameter        | Type    | Default | Description                                                                                                             |
|------------------|---------|---------|-------------------------------------------------------------------------------------------------------------------------|
| deleteDocuments  | boolean | true    | When `true`, all documents imported by the connection are permanently deleted. When `false`, the connection is removed but documents are kept. |

---

## 3. Different Connectors

- Notion Connector
- Google Drive Connector
- Gmail Connector
- OneDrive Connector
- S3 Connector
- Github Connector
- Web Crawler Connector
