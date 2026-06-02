# OpenAI Streaming Cheatsheet

## Basic Pattern

```python
from openai import OpenAI

client = OpenAI(base_url=os.getenv("OPENAI_BASE_URL"))

stream = client.responses.create(
    model=os.getenv("OPENAI_MODEL"),
    input=[{"role": "user", "content": prompt}],
    stream=True
)

def generate():
    for event in stream:
        if event.type == "response.output_text.delta":
            yield event.delta

return StreamingResponse(generate())
```

## Key Event Types

| Event Type | When |
|------------|------|
| `response.created` | Stream started |
| `response.output_text.delta` | Text chunk arrived |
| `response.function_call_arguments.done` | Tool call complete |
| `response.completed` | Stream finished |

## Environment Variables

```
OPENAI_BASE_URL=https://api.openai.com/v1
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o-mini
```

Load with:
```python
from dotenv import load_dotenv
load_dotenv()
```

## Docs

- https://platform.openai.com/docs/api-reference/responses
