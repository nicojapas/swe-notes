---
title: SQLite
description: Python sqlite3, connections, queries
---
# SQLite Cheatsheet

## Setup

```python
import sqlite3
from datetime import datetime

def init_db():
    conn = sqlite3.connect("requests.db")
    conn.execute("""
        CREATE TABLE IF NOT EXISTS requests (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT,
            prompt TEXT,
            status TEXT,
            tool_used INTEGER
        )
    """)
    conn.commit()
    conn.close()

init_db()
```

## Insert

```python
def log_request(prompt: str, status: str, tool_used: bool):
    conn = sqlite3.connect("requests.db")
    conn.execute(
        "INSERT INTO requests (timestamp, prompt, status, tool_used) VALUES (?, ?, ?, ?)",
        (datetime.now().isoformat(), prompt, status, int(tool_used))
    )
    conn.commit()
    conn.close()
```

## Read

```python
conn = sqlite3.connect("requests.db")
cursor = conn.execute("SELECT * FROM requests")
rows = cursor.fetchall()
conn.close()
```

## Notes

- Booleans stored as INTEGER (0/1)
- Always use `?` placeholders (prevents SQL injection)
- Call `conn.commit()` after writes
- Call `conn.close()` when done
