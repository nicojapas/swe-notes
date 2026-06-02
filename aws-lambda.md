# AWS Lambda Cheatsheet

## Handler Pattern

```python
import json

def handler(event, context):
    # context.function_name, context.aws_request_id, context.get_remaining_time_in_millis()

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"message": "ok"})
    }
```

## API Gateway Event

```python
def handler(event, context):
    # Path params: /users/{id}
    user_id = event["pathParameters"]["id"]

    # Query string: ?page=1
    page = event.get("queryStringParameters", {}).get("page", "1")

    # Body (JSON)
    body = json.loads(event.get("body") or "{}")

    # Headers
    auth = event["headers"].get("Authorization", "")
```

## EventBridge Event

```python
def handler(event, context):
    detail_type = event["detail-type"]  # e.g., "order.created"
    source = event["source"]            # e.g., "myapp.orders"
    detail = event["detail"]            # your payload
```

## Environment Variables

```python
import os

TABLE_NAME = os.environ["TABLE_NAME"]
STAGE = os.environ.get("STAGE", "dev")
```

## Async Handler (for I/O heavy)

```python
import asyncio

def handler(event, context):
    return asyncio.get_event_loop().run_until_complete(async_handler(event))

async def async_handler(event):
    # async code here
    return {"statusCode": 200, "body": "ok"}
```

## Logging

```python
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def handler(event, context):
    logger.info(f"Request ID: {context.aws_request_id}")
    logger.info(json.dumps(event))  # structured logging
```

## Notes

- Cold start: first invocation slower (~100-500ms for Python)
- Max execution: 15 minutes
- Max memory: 10GB (CPU scales with memory)
- `/tmp` has 512MB-10GB ephemeral storage
- Return `None` for async invocations (EventBridge, SNS, S3)

## Docs

- https://docs.aws.amazon.com/lambda/latest/dg/python-handler.html
