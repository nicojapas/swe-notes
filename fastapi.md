# FastAPI Cheatsheet

## Skeleton

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from pydantic import BaseModel

app = FastAPI()

class ChatRequest(BaseModel):
    prompt: str

@app.get("/health")
async def health():
    return {"status": "ok"}

@app.post("/chat")
async def chat(request: ChatRequest):
    return {"response": request.prompt}
```

## Run

```bash
pip install fastapi uvicorn
uvicorn main:app --reload
```

## StreamingResponse

```python
from fastapi.responses import StreamingResponse

def generate():
    for chunk in some_iterator:
        yield chunk

return StreamingResponse(generate(), media_type="text/plain")
```

## Docs

- https://fastapi.tiangolo.com/
- https://fastapi.tiangolo.com/advanced/custom-response/#streamingresponse
