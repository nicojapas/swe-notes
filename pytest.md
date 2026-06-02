# Pytest Cheatsheet

## Run Tests

```bash
pytest                          # all tests
pytest tests/test_api.py        # specific file
pytest -k "test_login"          # match name pattern
pytest -x                       # stop on first failure
pytest -v                       # verbose
pytest --tb=short               # shorter traceback
pytest -s                       # show print statements
```

## Basic Test

```python
def test_addition():
    assert 1 + 1 == 2
    assert "hello" in "hello world"
    assert [1, 2] == [1, 2]
```

## Fixtures

```python
import pytest

@pytest.fixture
def client():
    """Runs before each test that uses it."""
    from app import create_app
    app = create_app()
    return app.test_client()

def test_health(client):
    response = client.get("/health")
    assert response.status_code == 200
```

## Fixture Scopes

```python
@pytest.fixture(scope="function")  # default - each test
@pytest.fixture(scope="class")     # once per class
@pytest.fixture(scope="module")    # once per file
@pytest.fixture(scope="session")   # once per test run
def db_connection():
    conn = create_connection()
    yield conn
    conn.close()  # teardown after scope ends
```

## Parametrize

```python
@pytest.mark.parametrize("input,expected", [
    ("hello", 5),
    ("world", 5),
    ("", 0),
])
def test_length(input, expected):
    assert len(input) == expected
```

## Mocking

```python
from unittest.mock import patch, MagicMock

def test_api_call():
    with patch("myapp.client.fetch") as mock_fetch:
        mock_fetch.return_value = {"data": "test"}

        result = my_function()

        mock_fetch.assert_called_once_with("https://api.example.com")
        assert result == {"data": "test"}
```

## Mock as Decorator

```python
@patch("myapp.services.external_api")
def test_something(mock_api):
    mock_api.return_value = {"status": "ok"}
    # test code here
```

## Async Tests

```python
import pytest

@pytest.mark.asyncio
async def test_async_function():
    result = await my_async_function()
    assert result == expected
```

Requires: `pip install pytest-asyncio`

```python
# conftest.py
pytest_plugins = ('pytest-asyncio',)
```

## Expecting Exceptions

```python
def test_raises():
    with pytest.raises(ValueError) as exc_info:
        raise ValueError("invalid input")

    assert "invalid" in str(exc_info.value)
```

## conftest.py

```python
# tests/conftest.py - fixtures available to all tests in directory

import pytest

@pytest.fixture
def api_key():
    return "test-key-123"
```

## Markers

```python
@pytest.mark.slow
def test_slow_operation():
    pass

@pytest.mark.skip(reason="not implemented")
def test_future_feature():
    pass

@pytest.mark.skipif(sys.platform == "win32", reason="Unix only")
def test_unix_feature():
    pass
```

```bash
pytest -m "not slow"  # skip slow tests
```

## Notes

- Test files must start with `test_` or end with `_test.py`
- Test functions must start with `test_`
- Use `yield` in fixtures for teardown logic
- `conftest.py` fixtures are auto-discovered

## Docs

- https://docs.pytest.org/
