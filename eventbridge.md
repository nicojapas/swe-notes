---
title: EventBridge
description: Event structure, rules, patterns, boto3
---
# EventBridge Cheatsheet

## Event Structure

```json
{
  "version": "0",
  "id": "uuid",
  "detail-type": "order.created",
  "source": "myapp.orders",
  "account": "123456789",
  "time": "2024-01-01T00:00:00Z",
  "region": "us-east-1",
  "detail": {
    "order_id": "123",
    "user_id": "456",
    "total": 9999
  }
}
```

## Put Events (boto3)

```python
import boto3
import json

client = boto3.client("events")

client.put_events(
    Entries=[
        {
            "Source": "myapp.orders",
            "DetailType": "order.created",
            "Detail": json.dumps({
                "order_id": "123",
                "user_id": "456",
                "total": 9999
            }),
            "EventBusName": "default"  # or custom bus name
        }
    ]
)
```

## Lambda Handler for EventBridge

```python
def handler(event, context):
    detail_type = event["detail-type"]
    source = event["source"]
    detail = event["detail"]

    if detail_type == "order.created":
        order_id = detail["order_id"]
        process_order(order_id)

    return {"status": "ok"}
```

## Terraform Rule

```hcl
resource "aws_cloudwatch_event_rule" "order_created" {
  name        = "order-created-rule"
  description = "Trigger on order.created events"

  event_pattern = jsonencode({
    source      = ["myapp.orders"]
    detail-type = ["order.created"]
  })
}

resource "aws_cloudwatch_event_target" "lambda" {
  rule      = aws_cloudwatch_event_rule.order_created.name
  target_id = "process-order"
  arn       = aws_lambda_function.processor.arn
}

resource "aws_lambda_permission" "eventbridge" {
  statement_id  = "AllowEventBridge"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.processor.function_name
  principal     = "events.amazonaws.com"
  source_arn    = aws_cloudwatch_event_rule.order_created.arn
}
```

## Event Patterns

```json
// Match exact source and detail-type
{
  "source": ["myapp.orders"],
  "detail-type": ["order.created"]
}

// Match multiple detail-types
{
  "source": ["myapp.orders"],
  "detail-type": ["order.created", "order.updated"]
}

// Match field in detail
{
  "source": ["myapp.orders"],
  "detail": {
    "status": ["pending", "processing"]
  }
}

// Prefix matching
{
  "source": [{"prefix": "myapp."}]
}

// Numeric matching
{
  "detail": {
    "total": [{"numeric": [">", 100]}]
  }
}

// Exists check
{
  "detail": {
    "discount_code": [{"exists": true}]
  }
}
```

## Scheduled Events (Cron)

```hcl
resource "aws_cloudwatch_event_rule" "daily" {
  name                = "daily-cleanup"
  schedule_expression = "cron(0 2 * * ? *)"  # 2 AM UTC daily
}

# rate expressions
# rate(1 minute)
# rate(5 minutes)
# rate(1 hour)
# rate(1 day)
```

## Input Transformation

```hcl
resource "aws_cloudwatch_event_target" "lambda" {
  rule      = aws_cloudwatch_event_rule.order_created.name
  target_id = "process-order"
  arn       = aws_lambda_function.processor.arn

  input_transformer {
    input_paths = {
      order_id = "$.detail.order_id"
      user_id  = "$.detail.user_id"
    }
    input_template = <<EOF
{
  "orderId": "<order_id>",
  "userId": "<user_id>",
  "source": "eventbridge"
}
EOF
  }
}
```

## Dead Letter Queue

```hcl
resource "aws_cloudwatch_event_target" "lambda" {
  rule      = aws_cloudwatch_event_rule.order_created.name
  target_id = "process-order"
  arn       = aws_lambda_function.processor.arn

  dead_letter_config {
    arn = aws_sqs_queue.dlq.arn
  }

  retry_policy {
    maximum_retry_attempts       = 3
    maximum_event_age_in_seconds = 3600
  }
}
```

## Custom Event Bus

```hcl
resource "aws_cloudwatch_event_bus" "orders" {
  name = "orders-bus"
}

resource "aws_cloudwatch_event_rule" "order_rule" {
  name           = "order-rule"
  event_bus_name = aws_cloudwatch_event_bus.orders.name
  event_pattern  = jsonencode({...})
}
```

## Notes

- `Detail` must be a JSON string (use `json.dumps()`)
- Default bus is free; custom buses have costs
- Max event size: 256 KB
- Events are at-least-once delivery
- Use `detail-type` for event names (e.g., `order.created`)
- Use `source` for service/app identifier (e.g., `myapp.orders`)

## Docs

- https://docs.aws.amazon.com/eventbridge/latest/userguide/
