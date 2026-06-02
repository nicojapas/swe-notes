---
title: OpenAI Tools
description: Function calling, tool schemas, responses
---
# OpenAI - Function Calling

## Tool Schema

```python
tools = [
    {
        "type": "function",
        "name": "get_weather",
        "description": "Get current temperature for a location",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "City name"}
            },
            "required": ["location"],
            "additionalProperties": False
        },
        "strict": True
    }
]
```

## Non-Streaming

```python
response = client.responses.create(
    model=os.getenv("OPENAI_MODEL"),
    input=[{"role": "user", "content": "What's the weather in Paris?"}],
    tools=tools
)

tool_call = response.output[0]
if tool_call.type == "function_call":
    result = get_weather(json.loads(tool_call.arguments)["location"])

    final = client.responses.create(
        model=os.getenv("OPENAI_MODEL"),
        input=[
            {"role": "user", "content": "What's the weather in Paris?"},
            {"type": "function_call_output", "call_id": tool_call.call_id, "output": result}
        ]
    )
    print(final.output_text)
```

## Streaming

```python
def generate(prompt):
    stream = client.responses.create(
        model=os.getenv("OPENAI_MODEL"),
        input=[{"role": "user", "content": prompt}],
        tools=tools,
        stream=True
    )

    tool_call = None
    for event in stream:
        if event.type == "response.function_call_arguments.done":
            tool_call = event
        if event.type == "response.output_text.delta":
            yield event.delta

    if tool_call:
        result = get_weather(json.loads(tool_call.arguments)["location"])

        final = client.responses.create(
            model=os.getenv("OPENAI_MODEL"),
            input=[
                {"role": "user", "content": prompt},
                {"type": "function_call_output", "call_id": tool_call.call_id, "output": result}
            ],
            stream=True
        )
        for event in final:
            if event.type == "response.output_text.delta":
                yield event.delta
```

## Key Fields

| Field | Description |
|-------|-------------|
| `tool_call.call_id` | ID to reference in output |
| `tool_call.name` | Function name |
| `tool_call.arguments` | JSON string of args |

## Docs

- https://developers.openai.com/api/docs/guides/function-calling
