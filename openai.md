---
title: OpenAI
description: Some basic snippets
---
# OpenAI

## Construct a Client instance

```python
from openai import OpenAI
client = OpenAI(

    # Optional, to use different providers, i.e. groq (free) -> OPENAI_BASE_URL=https://api.groq.com/openai/v1
    base_url=os.getenv("OPENAI_BASE_URL")
)

response = client.responses.create(
    # Cheap model in OpenAI -> OPENAI_MODEL=gpt-5.4-nano
    # Free model in groq -> OPENAI_MODEL=llama-3.3-70b-versatile
    model=os.getenv("OPENAI_MODEL"),
    input=[
        # {"role": "system", "content": "You are a helpful assistant."},
        # {"role": "user", "content": "My name is Alice"},
        # {"role": "assistant", "content": "Nice to meet you, Alice!"},
        {"role": "user", "content": "What's my name?"}
    ]
)

print(response.output_text)

```

## Streaming

```python
stream = client.responses.create(
    model=os.getenv("OPENAI_MODEL"),
    input=[{"role": "user", "content": "What's my name?"}],
    stream=True
)

def generate():
    for event in stream:
        if event.type == "response.output_text.delta":
            yield event.delta

return StreamingResponse(generate())
```

## Docs

- https://developers.openai.com/api/docs?lang=python
