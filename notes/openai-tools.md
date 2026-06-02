# OpenAI Function Calling Cheatsheet

## The Flow

1. Send request with `tools` array
2. Model returns either text OR function call (not both)
3. If function call: execute it, send result back, get final response

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
                "location": {
                    "type": "string",
                    "description": "City name"
                }
            },
            "required": ["location"]
        }
    }
]
```

## Streaming + Tools Pattern

```python
def generate():
    function_call = None

    for event in stream:
        # Capture function call
        if event.type == "response.function_call_arguments.done":
            function_call = event

        # Yield text (for non-tool responses)
        if event.type == "response.output_text.delta":
            yield event.delta

    # After stream ends, handle function call
    if function_call:
        args = json.loads(function_call.arguments)
        result = get_weather(args["location"])

        # Second API call with result
        final = client.responses.create(
            model=os.getenv("OPENAI_MODEL"),
            input=[
                {"role": "user", "content": request.prompt},
                {
                    "type": "function_call",
                    "id": function_call.item_id,
                    "name": function_call.name,
                    "arguments": function_call.arguments
                },
                {
                    "type": "function_call_output",
                    "call_id": function_call.item_id,
                    "output": result
                }
            ],
            stream=True
        )

        for event in final:
            if event.type == "response.output_text.delta":
                yield event.delta
```

## Key Attributes on function_call event

- `function_call.name` - function name
- `function_call.arguments` - JSON string of args
- `function_call.item_id` - use as call_id

## Docs

- https://platform.openai.com/docs/guides/function-calling
