# Mocking

Mocking is identifying external dependencies (APIs, databases) and replacing them with controlled "fake" objects during tests.

## `unittest.mock`

Standard library module for mocking.

```python
from unittest.mock import Mock, patch

# Basic Mock
m = Mock()
m.some_method.return_value = 42

print(m.some_method()) # 42
m.some_method.assert_called_once()
```

## `patch`
Replaces an object in a module with a mock.

### File to test (`my_module.py`)
```python
import requests

def get_json(url):
    response = requests.get(url)
    return response.json()
```

### Test
```python
from unittest.mock import patch
from my_module import get_json

# 'my_module.requests' is the target to patch!
# NOT 'requests' (where it is defined), but where it is LOOKED UP.
@patch('my_module.requests')
def test_get_json(mock_requests):
    # Setup mock behavior
    mock_response = Mock()
    mock_response.json.return_value = {"id": 1}
    mock_requests.get.return_value = mock_response

    result = get_json("http://fake.com")
    
    assert result == {"id": 1}
    mock_requests.get.assert_called_with("http://fake.com")
```
