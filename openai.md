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
    input="Write a short bedtime story about a unicorn."
)

print(response.output_text)

```

## Docs

- https://developers.openai.com/api/docs?lang=python
