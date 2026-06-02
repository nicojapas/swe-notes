# Asyncio Cheatsheet

## Basic Pattern

```python
import asyncio

async def fetch_data():
    await asyncio.sleep(1)  # simulates I/O
    return "data"

async def main():
    result = await fetch_data()
    print(result)

asyncio.run(main())
```

## Run Multiple Tasks Concurrently

```python
async def main():
    # gather - wait for all, return results in order
    results = await asyncio.gather(
        fetch_data("url1"),
        fetch_data("url2"),
        fetch_data("url3"),
    )
    # results = ["data1", "data2", "data3"]
```

## Handle Exceptions in gather

```python
results = await asyncio.gather(
    task1(),
    task2(),
    return_exceptions=True  # don't fail on first exception
)

for result in results:
    if isinstance(result, Exception):
        print(f"Task failed: {result}")
```

## TaskGroup (Python 3.11+)

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch("url1"))
        task2 = tg.create_task(fetch("url2"))

    # both tasks done here
    print(task1.result(), task2.result())
```

## Timeout

```python
async def main():
    try:
        result = await asyncio.wait_for(slow_operation(), timeout=5.0)
    except asyncio.TimeoutError:
        print("Operation timed out")
```

## Create Task (Fire and Forget)

```python
async def main():
    # Start task without waiting
    task = asyncio.create_task(background_job())

    # Do other work...

    # Optionally wait later
    await task
```

## Semaphore (Limit Concurrency)

```python
semaphore = asyncio.Semaphore(10)  # max 10 concurrent

async def fetch_with_limit(url):
    async with semaphore:
        return await fetch(url)

async def main():
    urls = [...]  # 100 urls
    results = await asyncio.gather(*[fetch_with_limit(u) for u in urls])
```

## Queue

```python
queue = asyncio.Queue()

async def producer():
    for i in range(10):
        await queue.put(i)

async def consumer():
    while True:
        item = await queue.get()
        print(f"Processing {item}")
        queue.task_done()

async def main():
    asyncio.create_task(consumer())
    await producer()
    await queue.join()  # wait until all items processed
```

## Run Sync Function in Thread

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

def blocking_io():
    # CPU-bound or blocking I/O
    return expensive_computation()

async def main():
    loop = asyncio.get_event_loop()
    result = await loop.run_in_executor(None, blocking_io)
```

## Async Context Manager

```python
class AsyncDB:
    async def __aenter__(self):
        self.conn = await create_connection()
        return self.conn

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.conn.close()

async def main():
    async with AsyncDB() as conn:
        await conn.execute("SELECT 1")
```

## Async Iterator

```python
async def fetch_pages():
    page = 1
    while True:
        data = await fetch_page(page)
        if not data:
            break
        yield data
        page += 1

async def main():
    async for page in fetch_pages():
        process(page)
```

## aiohttp Example

```python
import aiohttp

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()
```

## Notes

- `await` only works inside `async` functions
- `asyncio.run()` is the entry point (creates event loop)
- Don't mix `time.sleep()` with async - use `asyncio.sleep()`
- Use `aiohttp` instead of `requests` for async HTTP
- `asyncio.gather()` for parallel tasks, `await` for sequential

## Docs

- https://docs.python.org/3/library/asyncio.html
