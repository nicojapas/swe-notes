---
title: SQLAlchemy
description: ORM setup, models, queries, relationships
---
# SQLAlchemy Cheatsheet

## Setup (2.0 Style)

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey, DateTime
from sqlalchemy.orm import declarative_base, sessionmaker, relationship
from datetime import datetime

engine = create_engine("postgresql://user:pass@localhost/db")
Session = sessionmaker(bind=engine)
Base = declarative_base()
```

## Models

```python
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    email = Column(String, unique=True, nullable=False)
    name = Column(String)
    created_at = Column(DateTime, default=datetime.utcnow)

    orders = relationship("Order", back_populates="user")

class Order(Base):
    __tablename__ = "orders"

    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    total = Column(Integer)
    status = Column(String, default="pending")

    user = relationship("User", back_populates="orders")
```

## Create Tables

```python
Base.metadata.create_all(engine)
```

## Session Pattern

```python
with Session() as session:
    # queries here
    session.commit()
```

## Insert

```python
with Session() as session:
    user = User(email="alice@example.com", name="Alice")
    session.add(user)
    session.commit()
    print(user.id)  # auto-populated after commit
```

## Query

```python
from sqlalchemy import select

with Session() as session:
    # Get by ID
    user = session.get(User, 1)

    # Filter
    stmt = select(User).where(User.email == "alice@example.com")
    user = session.execute(stmt).scalar_one_or_none()

    # Multiple results
    stmt = select(User).where(User.name.like("%alice%"))
    users = session.execute(stmt).scalars().all()
```

## Update

```python
with Session() as session:
    user = session.get(User, 1)
    user.name = "Bob"
    session.commit()
```

## Delete

```python
with Session() as session:
    user = session.get(User, 1)
    session.delete(user)
    session.commit()
```

## Filtering

```python
from sqlalchemy import select, and_, or_

# Multiple conditions
stmt = select(Order).where(
    and_(
        Order.status == "pending",
        Order.total > 100
    )
)

# OR
stmt = select(User).where(
    or_(
        User.name == "Alice",
        User.name == "Bob"
    )
)

# IN
stmt = select(User).where(User.id.in_([1, 2, 3]))

# LIKE
stmt = select(User).where(User.email.like("%@gmail.com"))

# NULL
stmt = select(User).where(User.name.is_(None))
```

## Order, Limit, Offset

```python
stmt = (
    select(User)
    .order_by(User.created_at.desc())
    .limit(10)
    .offset(20)
)
```

## Joins

```python
# Implicit join via relationship
stmt = select(Order).join(Order.user).where(User.email == "alice@example.com")

# Explicit join
stmt = select(User, Order).join(Order, User.id == Order.user_id)
```

## Aggregations

```python
from sqlalchemy import func

# Count
stmt = select(func.count(User.id))
count = session.execute(stmt).scalar()

# Group by
stmt = (
    select(Order.status, func.count(Order.id))
    .group_by(Order.status)
)
results = session.execute(stmt).all()
```

## Async SQLAlchemy

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession)

async def get_user(user_id: int):
    async with AsyncSessionLocal() as session:
        return await session.get(User, user_id)
```

## FastAPI Dependency

```python
from fastapi import Depends

def get_db():
    db = Session()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/{user_id}")
def get_user(user_id: int, db: Session = Depends(get_db)):
    return db.get(User, user_id)
```

## Notes

- Always use `session.commit()` after writes
- Use `session.rollback()` on errors
- `scalar_one_or_none()` returns None if not found, raises if multiple
- `scalars().all()` for list of model objects
- Use `select()` instead of legacy `session.query()`

## Docs

- https://docs.sqlalchemy.org/en/20/
