# DynamoDB Cheatsheet

## boto3 Setup

```python
import boto3
from boto3.dynamodb.conditions import Key, Attr

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("my-table")
```

## Put Item

```python
table.put_item(Item={
    "PK": "USER#123",
    "SK": "PROFILE",
    "name": "Alice",
    "email": "alice@example.com"
})
```

## Get Item

```python
response = table.get_item(Key={"PK": "USER#123", "SK": "PROFILE"})
item = response.get("Item")  # None if not found
```

## Query (PK + SK condition)

```python
# All items for a user
response = table.query(
    KeyConditionExpression=Key("PK").eq("USER#123")
)
items = response["Items"]

# With SK prefix
response = table.query(
    KeyConditionExpression=Key("PK").eq("USER#123") & Key("SK").begins_with("ORDER#")
)
```

## Query GSI

```python
response = table.query(
    IndexName="GSI1",
    KeyConditionExpression=Key("GSI1PK").eq("STATUS#pending")
)
```

## Update Item

```python
table.update_item(
    Key={"PK": "USER#123", "SK": "PROFILE"},
    UpdateExpression="SET #n = :name, updated_at = :ts",
    ExpressionAttributeNames={"#n": "name"},  # 'name' is reserved
    ExpressionAttributeValues={":name": "Bob", ":ts": "2024-01-01T00:00:00Z"}
)
```

## Delete Item

```python
table.delete_item(Key={"PK": "USER#123", "SK": "PROFILE"})
```

## Batch Write

```python
with table.batch_writer() as batch:
    for item in items:
        batch.put_item(Item=item)
```

## Scan (avoid in production)

```python
response = table.scan(
    FilterExpression=Attr("status").eq("active")
)
```

## Single-Table Pattern Keys

```
PK              SK                  Data
USER#123        PROFILE             {name, email}
USER#123        ORDER#456           {total, status}
ORDER#456       ORDER#456           {user_id, items}  (for GSI access)
```

## Notes

- Always query by PK (scan = full table read = expensive)
- Use `begins_with()` for hierarchical SK
- Reserved words need `ExpressionAttributeNames`
- `get_item` returns `None` if not found, doesn't throw
- Batch write handles retries automatically

## Docs

- https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html
