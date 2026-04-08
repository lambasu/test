# Getting Started

This guide walks you through the four steps you need to go from zero to a working integration: authenticate, make your first request, handle the response, and resolve common issues.

---

## Step 1 – Set Up Auth

All API requests must include a valid bearer token in the `Authorization` header.

**Obtain your API key**

1. Log in to the dashboard.
2. Navigate to **Settings → API Keys**.
3. Click **Create new key**, give it a name, and copy the value. Store it securely — it will not be shown again.

**Token setup — configure the key as an environment variable**

Never hard-code your API key in source files. Instead, set it as an environment variable and read it at runtime.

*Linux / macOS (current shell session)*

```bash
export API_KEY="your_api_key_here"
```

To persist across sessions, add that line to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.) and reload it:

```bash
echo 'export API_KEY="your_api_key_here"' >> ~/.bashrc
source ~/.bashrc
```

*Windows (Command Prompt)*

```cmd
set API_KEY=your_api_key_here
```

*Windows (PowerShell)*

```powershell
$env:API_KEY = "your_api_key_here"
```

*Using a `.env` file (recommended for local development)*

Create a `.env` file in your project root:

```
API_KEY=your_api_key_here
```

Then load it with your framework's dotenv support (e.g. `dotenv` for Node.js, `python-dotenv` for Python). **Add `.env` to your `.gitignore`** so the key is never committed to version control.

**Include the token in every request**

```http
Authorization: Bearer <YOUR_API_KEY>
```

---

## Step 2 – Make the First Request

Use the `GET /v1/status` endpoint to verify your credentials and confirm the API is reachable before writing any business logic.

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
const data = await response.json();
console.log(data);
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
print(response.json())
```

A successful response looks like:

```json
{
  "status": "ok",
  "version": "1.0.0"
}
```

---

## Step 3 – Handle Success and Errors

### Success (2xx)

| Code | Meaning |
|------|---------|
| `200 OK` | Request succeeded; body contains the result. |
| `201 Created` | Resource created; `Location` header points to the new resource. |
| `204 No Content` | Request succeeded; no body returned (e.g. DELETE). |

Always check `Content-Type: application/json` before parsing the body.

### Client errors (4xx)

| Code | Meaning | Action |
|------|---------|--------|
| `400 Bad Request` | Malformed request or missing required field. | Check the `errors` array in the response body for field-level details. |
| `401 Unauthorized` | Missing or invalid token. | Re-check your `Authorization` header and token expiry. |
| `403 Forbidden` | Token valid but lacks permission. | Verify the key's scopes in **Settings → API Keys**. |
| `404 Not Found` | Resource does not exist. | Confirm the ID or path. |
| `429 Too Many Requests` | Rate limit exceeded. | Respect the `Retry-After` header before retrying. |

### Server errors (5xx)

Retry with exponential back-off. If a `500` or `503` persists beyond a few minutes, check the [status page](https://status.example.com).

### Example error body

```json
{
  "error": {
    "code": "validation_failed",
    "message": "The 'email' field is required.",
    "errors": [
      { "field": "email", "message": "This field may not be blank." }
    ]
  }
}
```

### Minimal error-handling snippet

```js
const response = await fetch(url, { headers });

if (!response.ok) {
  const err = await response.json();
  throw new Error(`API error ${response.status}: ${err.error.message}`);
}

const data = await response.json();
```

---

## Step 4 – Debug Common Issues

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `401 Unauthorized` on every request | Token not sent or malformed | Confirm the header is `Authorization: Bearer <token>` with no extra spaces or quotes. |
| `401 Unauthorized` after the token was working | Token expired or revoked | Generate a new key in **Settings → API Keys**. |
| `403 Forbidden` | Insufficient scope | Open the key in the dashboard and add the required scopes. |
| Empty or garbled response body | Wrong `Accept` header | Set `Accept: application/json`. |
| Request times out | Network or DNS issue | Test with `curl -v` to isolate the layer; check firewall / proxy rules. |
| `429 Too Many Requests` | Burst rate limit hit | Add retry logic that reads the `Retry-After` response header. |
| `SSL certificate verify failed` | Outdated CA bundle or self-signed cert in dev | Update your CA bundle; never disable TLS verification in production. |

### Enabling debug logging

Pass `-v` to cURL or set your HTTP client to verbose mode to inspect raw request/response headers:

```bash
curl -v -X GET https://api.example.com/v1/status \
  -H "Authorization: Bearer $API_KEY"
```

Look for `< HTTP/1.1 <STATUS_CODE>` in the output to confirm the server received and processed the request.

---

## Next Steps

- Explore the full [API Reference](./api-reference.md) for available endpoints.
- Review [Authentication](./authentication.md) for advanced topics such as token rotation and OAuth flows.
- See [Error Handling](./error-handling.md) for a complete list of error codes and retry strategies.
