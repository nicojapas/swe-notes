# LangChain Cheatsheet

## Basic Chat

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")
response = llm.invoke("What is 2+2?")
print(response.content)
```

## With Messages

```python
from langchain_core.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="What is the capital of France?"),
]
response = llm.invoke(messages)
```

## Prompt Templates

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {role}."),
    ("human", "{question}"),
])

chain = prompt | llm
response = chain.invoke({"role": "pirate", "question": "What's the weather?"})
```

## Streaming

```python
for chunk in llm.stream("Tell me a story"):
    print(chunk.content, end="", flush=True)
```

## Structured Output

```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int

structured_llm = llm.with_structured_output(Person)
person = structured_llm.invoke("John is 25 years old")
print(person.name, person.age)
```

## Tool Calling

```python
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """Get weather for a city."""
    return f"Weather in {city}: 72°F"

llm_with_tools = llm.bind_tools([get_weather])
response = llm_with_tools.invoke("What's the weather in Paris?")

# Check for tool calls
if response.tool_calls:
    for call in response.tool_calls:
        print(call["name"], call["args"])
```

## RAG Pattern

```python
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import FAISS
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

# Create vector store
embeddings = OpenAIEmbeddings()
texts = ["Paris is the capital of France.", "Berlin is the capital of Germany."]
vectorstore = FAISS.from_texts(texts, embeddings)
retriever = vectorstore.as_retriever()

# RAG chain
template = """Answer based on context:
{context}

Question: {question}"""

prompt = ChatPromptTemplate.from_template(template)

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
)

response = chain.invoke("What is the capital of France?")
```

## Document Loaders

```python
from langchain_community.document_loaders import TextLoader, PyPDFLoader

# Text file
loader = TextLoader("file.txt")
docs = loader.load()

# PDF
loader = PyPDFLoader("file.pdf")
docs = loader.load()
```

## Text Splitting

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(docs)
```

## Memory (Conversation History)

```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

store = {}

def get_session_history(session_id: str):
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

chain_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="question",
    history_messages_key="history",
)

response = chain_with_history.invoke(
    {"question": "Hi, I'm Alice"},
    config={"configurable": {"session_id": "abc123"}},
)
```

## Output Parsers

```python
from langchain_core.output_parsers import StrOutputParser, JsonOutputParser

# String output
chain = prompt | llm | StrOutputParser()
text = chain.invoke({"question": "Hello"})  # returns str, not AIMessage

# JSON output
chain = prompt | llm | JsonOutputParser()
data = chain.invoke({"question": "Return JSON"})  # returns dict
```

## LCEL Basics

```python
# Pipe operator chains components
chain = prompt | llm | parser

# RunnablePassthrough passes input through
chain = {"question": RunnablePassthrough()} | prompt | llm

# RunnableParallel runs multiple chains
from langchain_core.runnables import RunnableParallel

chain = RunnableParallel(
    answer=prompt | llm,
    source=retriever,
)
```

## Notes

- Use `langchain_openai` not `langchain.chat_models`
- `invoke()` for single calls, `stream()` for streaming, `batch()` for multiple
- LCEL chains are lazy - nothing runs until `invoke()`
- `|` operator builds the chain, `invoke()` runs it

## Docs

- https://python.langchain.com/docs/
