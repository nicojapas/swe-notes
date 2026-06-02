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

for item in response.output:
    if item.type == "function_call":
        result = get_weather(json.loads(item.arguments)["location"])
        print("r", result)
        final = client.responses.create(
            model=os.getenv("OPENAI_MODEL"),
            input=[
                {"role": "user", "content": "What's the weather in Paris?"},
                item,
                {"type": "function_call_output", "call_id": item.call_id, "output": result}
            ]
        )
        
        return final.output_text

return response.output_text    

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
                {"type": "function_call", "call_id": tool_call.item_id, "name": tool_call.name, "arguments": tool_call.arguments},
                {"type": "function_call_output", "call_id": tool_call.item_id, "output": result}
            ],
            stream=True
        )
        for event in final:
            if event.type == "response.output_text.delta":
                yield event.delta

return StreamingResponse(generate("What's the weather in Paris?"))
```

## Docs

- [Function calling guide](https://developers.openai.com/api/docs/guides/function-calling)
- [Response object schema](https://developers.openai.com/api/reference/resources/responses/methods/create)
- [Streaming events schema](https://developers.openai.com/api/reference/resources/responses/streaming-events)
