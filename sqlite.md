---
title: SQLite
description: Python sqlite3, connections, queries
---
## Setup

```python
import sqlite3
from datetime import datetime

def init_db():
    conn = sqlite3.connect("example.db")
    conn.execute("""
        CREATE TABLE IF NOT EXISTS requests (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT,
            ...
        )
    """)
    conn.commit()
    conn.close()

init_db()
```

## Insert

```python
def insert_entry(...):
    conn = sqlite3.connect("example.db")
    conn.execute(
        "INSERT INTO example (timestamp, ...) VALUES (?, ...)",
        (datetime.now().isoformat(), ...)
    )
    conn.commit()
    conn.close()
```

## Read

```python
conn = sqlite3.connect("example.db")
cursor = conn.execute("SELECT * FROM example")
rows = cursor.fetchall()
conn.close()
```

## Notes

- Booleans stored as INTEGER (0/1)
- Always use `?` placeholders (prevents SQL injection)
- Call `conn.commit()` after writes
- Call `conn.close()` when done
