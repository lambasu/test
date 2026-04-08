# Token Setup

This guide explains how to create, configure, and manage API tokens so you can authenticate against the API.

---

## Overview

Every API request must include a bearer token in the `Authorization` header. Tokens are created in the dashboard and are scoped to the permissions your integration requires.

---

## Step 1 – Create a Token

1. Log in to the dashboard.
2. Navigate to **Settings → API Keys**.
3. Click **Create new key**.
4. Enter a descriptive name (e.g. `production-backend` or `ci-pipeline`).
5. Select the **scopes** your integration needs (see [Token Scopes](#token-scopes) below).
6. Click **Save**.
7. **Copy the token value immediately** — it is shown only once. If you lose it, you must revoke and regenerate the key.

---

## Step 2 – Store the Token Securely

Never hard-code a token in source code or commit it to version control.

| Environment | Recommended storage |
|-------------|---------------------|
| Local development | `.env` file (add `.env` to `.gitignore`) |
| CI / CD | Encrypted secret variables in your CI provider |
| Production server | Environment variable injected at runtime, or a secrets manager (e.g. AWS Secrets Manager, HashiCorp Vault) |

**Example `.env` file**

```
API_KEY=your_token_value_here
```

Load it in your application before making requests:

```bash
# bash / zsh
export API_KEY=$(grep API_KEY .env | cut -d '=' -f2)
```

```js
// Node.js (with dotenv)
require('dotenv').config();
const token = process.env.API_KEY;
```

```python
# Python (with python-dotenv)
from dotenv import load_dotenv
import os

load_dotenv()
token = os.environ["API_KEY"]
```

---

## Step 3 – Use the Token in Requests

Include the token as a bearer token in the `Authorization` header of every request:

```http
Authorization: Bearer <YOUR_API_KEY>
```

**cURL**

```bash
curl -X GET https://api.example.com/v1/status \
  -H "Authorization: Bearer $API_KEY" \
  -H "Accept: application/json"
```

**Node.js**

```js
const response = await fetch('https://api.example.com/v1/status', {
  headers: {
    Authorization: `Bearer ${process.env.API_KEY}`,
    Accept: 'application/json',
  },
});
```

**Python**

```python
import os, requests

response = requests.get(
    'https://api.example.com/v1/status',
    headers={
        'Authorization': f'Bearer {os.environ["API_KEY"]}',
        'Accept': 'application/json',
    },
)
```

---

## Token Scopes

When creating a token, select only the scopes your integration requires (principle of least privilege).

| Scope | Access granted |
|-------|----------------|
| `read:data` | Read resources (GET requests) |
| `write:data` | Create and update resources (POST, PUT, PATCH) |
| `delete:data` | Delete resources (DELETE requests) |
| `admin` | Full access including settings and key management |

You can view and update a token's scopes at any time from **Settings → API Keys** without regenerating the token.

---

## Rotating a Token

Rotate tokens periodically or immediately after a suspected exposure:

1. Go to **Settings → API Keys**.
2. Click the token you want to rotate.
3. Click **Regenerate** and copy the new value.
4. Update the token in every environment that uses it.
5. Verify that your integration still works, then click **Revoke old key** to invalidate the previous value.

---

## Revoking a Token

To permanently invalidate a token:

1. Go to **Settings → API Keys**.
2. Click the token you want to revoke.
3. Click **Revoke** and confirm.

Any requests using the revoked token will immediately start receiving `401 Unauthorized` responses.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `401 Unauthorized` on every request | Token not sent or header is malformed | Confirm the header is exactly `Authorization: Bearer <token>` with no extra quotes or spaces. |
| `401 Unauthorized` after the token was working | Token expired or revoked | Generate a new key in **Settings → API Keys**. |
| `403 Forbidden` | Token lacks the required scope | Open the key in the dashboard and add the missing scope. |
| Token value not visible | You navigated away before copying it | Revoke the existing key and create a new one. |

---

## Next Steps

- Follow the [Getting Started guide](./getting-started.md) to make your first API request.
- Read the full [API Reference](./api-reference.md) for all available endpoints.
