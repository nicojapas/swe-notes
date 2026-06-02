---
title: Docker
description: Dockerfile, build, run, volumes, env vars
---
## Dockerfile Template

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Build & Run

```bash
# Build
docker build -t my-service .

# Run
docker run -p 8000:8000 --env-file .env my-service

# Run with volume (for persistent data)
docker run -p 8000:8000 --env-file .env -v $(pwd)/data:/app/data my-service
```

## Port Mapping

```
-p HOST_PORT:CONTAINER_PORT
-p 3000:8000  # access at localhost:3000, container listens on 8000
```

## Environment Variables

```bash
# From file (no quotes in .env values!)
--env-file .env

# Individual
-e MY_API_KEY=sk-...
```

## Volume Mounting

```bash
-v /path/on/host:/path/in/container
-v $(pwd)/data:/app/data  # current dir's data folder
```

## Notes

- Rebuild after code changes: `docker build -t name .`
- `.env` values: NO quotes when using `--env-file`
- Container filesystem is isolated and ephemeral
- Use volumes for persistent data
