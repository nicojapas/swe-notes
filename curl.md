---
title: curl
description: HTTP requests, headers, auth, JSON, file uploads
---

# curl Cheatsheet

## GET Request

```bash
curl https://api.example.com/users
curl -s https://api.example.com/users  # silent (no progress)
curl -i https://api.example.com/users  # include response headers
curl -v https://api.example.com/users  # verbose (debug)
```

## POST JSON

```bash
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'
```

## POST Form Data

```bash
curl -X POST https://api.example.com/login \
  -d "username=alice&password=secret"
```

## Headers

```bash
curl https://api.example.com/users \
  -H "Authorization: Bearer eyJhbGc..." \
  -H "Accept: application/json"
```

## Query Parameters

```bash
curl "https://api.example.com/users?page=1&limit=10"
curl -G https://api.example.com/users \
  --data-urlencode "query=hello world"
```

## PUT / PATCH / DELETE

```bash
curl -X PUT https://api.example.com/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Bob"}'

curl -X PATCH https://api.example.com/users/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "active"}'

curl -X DELETE https://api.example.com/users/1
```

## Save Response to File

```bash
curl -o response.json https://api.example.com/data
curl -O https://example.com/file.zip  # use remote filename
```

## Upload File

```bash
# multipart form
curl -X POST https://api.example.com/upload \
  -F "file=@/path/to/file.pdf"

# with additional fields
curl -X POST https://api.example.com/upload \
  -F "file=@photo.jpg" \
  -F "description=Profile photo"

# raw file body
curl -X POST https://api.example.com/upload \
  -H "Content-Type: application/octet-stream" \
  --data-binary @file.bin
```

## Authentication

```bash
# Bearer token
curl -H "Authorization: Bearer TOKEN" https://api.example.com/me

# Basic auth
curl -u username:password https://api.example.com/me

# API key in header
curl -H "X-API-Key: your-api-key" https://api.example.com/data
```

## Follow Redirects

```bash
curl -L https://example.com/redirect
```

## Timeout

```bash
curl --connect-timeout 5 --max-time 10 https://api.example.com/slow
```

## Pretty Print JSON

```bash
curl -s https://api.example.com/users | jq .
curl -s https://api.example.com/users | jq '.data[0]'
curl -s https://api.example.com/users | python -m json.tool
```

## Show Only Status Code

```bash
curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health
```

## Show Response Time

```bash
curl -s -o /dev/null -w "Time: %{time_total}s\n" https://api.example.com
```

## Retry on Failure

```bash
curl --retry 3 --retry-delay 2 https://api.example.com/flaky
```

## Ignore SSL Errors (dev only)

```bash
curl -k https://localhost:8443/api
```

## Send JSON from File

```bash
curl -X POST https://api.example.com/data \
  -H "Content-Type: application/json" \
  -d @payload.json
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-s` | Silent mode (no progress) |
| `-S` | Show errors (use with -s) |
| `-i` | Include response headers |
| `-I` | Headers only (HEAD request) |
| `-v` | Verbose output |
| `-L` | Follow redirects |
| `-o` | Output to file |
| `-O` | Save with remote filename |
| `-X` | HTTP method |
| `-H` | Add header |
| `-d` | POST data |
| `-F` | Form data (multipart) |
| `-u` | Basic auth credentials |
| `-k` | Ignore SSL errors |

## Notes

- Use `-s` for scripts to suppress progress bar
- Always quote URLs with query params
- Use `jq` for JSON parsing in scripts
- `--data-urlencode` handles special characters

## Docs

- https://curl.se/docs/manpage.html
