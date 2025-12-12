# User Guide

This guide covers all features of NexusCosmos in detail.

## Core Concepts

### DataAcquisition Client

The `DataAcquisition` class is the main interface for fetching data:

```python
from nexuscosmos import DataAcquisition

client = DataAcquisition(
    base_url="https://api.example.com",  # Required: API base URL
    cache_ttl=3600,                       # Cache time-to-live in seconds
    cache_dir=".nexuscosmos_cache",       # Cache directory
    rate_limit=1.0,                       # Requests per second limit
    timeout=30,                           # Request timeout in seconds
    retry_count=3,                        # Number of retries on failure
    retry_delay=1.0                       # Delay between retries
)
```

### Response Objects

All fetch operations return a `Response` object:

```python
response = client.fetch(params={"key": "value"})

# Available attributes
response.data          # Parsed response data (dict or list)
response.raw_text      # Raw response text
response.status_code   # HTTP status code
response.headers       # Response headers
response.cached        # True if served from cache
response.fetch_time    # Time taken to fetch (seconds)
response.url           # Final URL requested
```

## Caching System

### How Caching Works

NexusCosmos automatically caches responses to disk:

1. A hash is computed from the request URL and parameters
2. If a cached response exists and hasn't expired, it's returned
3. Otherwise, a new request is made and cached

### Cache Configuration

```python
client = DataAcquisition(
    base_url="https://api.example.com",
    cache_ttl=86400,                    # 24 hours
    cache_dir="/path/to/cache"          # Custom location
)
```

### Cache Management

```python
# Get cache statistics
stats = client.cache_stats()
print(f"Total entries: {stats['total_entries']}")
print(f"Cache size: {stats['size_bytes']} bytes")
print(f"Hit rate: {stats['hit_rate']:.2%}")

# Clear all cache
client.clear_cache()

# Clear cache for specific key
client.clear_cache(key="specific_request_key")
```

### Bypassing Cache

```python
# Force fresh request
response = client.fetch(
    params={"object": "Mars"},
    use_cache=False
)
```

## Rate Limiting

### Built-in Rate Limiting

```python
client = DataAcquisition(
    base_url="https://api.example.com",
    rate_limit=2.0  # Max 2 requests per second
)
```

### Adaptive Rate Limiting

The client automatically backs off when receiving 429 (Too Many Requests) responses.

## Batch Operations

### Synchronous Batch

```python
params_list = [
    {"object": "Mars"},
    {"object": "Venus"},
    {"object": "Jupiter"}
]

results = client.fetch_batch(
    params_list,
    show_progress=True  # Show progress bar
)

for params, result in zip(params_list, results):
    if result.success:
        print(f"{params['object']}: {result.data}")
    else:
        print(f"{params['object']}: Error - {result.error}")
```

### Async Batch (Faster)

```python
import asyncio
from nexuscosmos import AsyncDataAcquisition

async def fetch_planets():
    client = AsyncDataAcquisition(
        base_url="https://api.example.com",
        rate_limit=10.0  # Higher limit for async
    )
    
    params_list = [{"object": p} for p in [
        "Mercury", "Venus", "Earth", "Mars",
        "Jupiter", "Saturn", "Uranus", "Neptune"
    ]]
    
    results = await client.fetch_batch_async(
        params_list,
        max_concurrent=5,    # Max 5 simultaneous requests
        show_progress=True
    )
    
    return results

results = asyncio.run(fetch_planets())
```

## Error Handling & Recovery

### Exception Types

```python
from nexuscosmos.utils import (
    NexusCosmosError,      # Base exception
    ConnectionError,        # Network issues
    TimeoutError,          # Request timeout
    RateLimitError,        # Rate limit exceeded
    ParseError             # Response parsing failed
)
```

### Error Recovery

```python
client = DataAcquisition(
    base_url="https://api.example.com",
    retry_count=3,         # Retry failed requests
    retry_delay=1.0,       # Wait between retries
    fallback_to_cache=True # Use stale cache on failure
)
```

### Custom Error Handling

```python
try:
    response = client.fetch(params={"object": "Mars"})
except RateLimitError:
    print("Rate limited - waiting...")
    time.sleep(60)
    response = client.fetch(params={"object": "Mars"})
except TimeoutError:
    print("Request timed out")
except NexusCosmosError as e:
    print(f"Error: {e}")
```

## Data Export

### JSON Export

```python
from nexuscosmos import DataExporter

exporter = DataExporter(results)
exporter.to_json("output.json", indent=2)
```

### CSV Export

```python
exporter.to_csv("output.csv")

# With custom options
exporter.to_csv(
    "output.csv",
    columns=["date", "ra", "dec", "magnitude"],
    delimiter=";",
    include_header=True
)
```

### Pandas DataFrame

```python
# Requires: pip install nexuscosmos[pandas]
df = exporter.to_dataframe()

# Now use pandas features
print(df.describe())
df.to_excel("output.xlsx")
```

## Logging & Debugging

### Enable Logging

```python
import logging

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

# Or just for nexuscosmos
logger = logging.getLogger("nexuscosmos")
logger.setLevel(logging.DEBUG)
```

### Request Inspection

```python
# Enable request debugging
client = DataAcquisition(
    base_url="https://api.example.com",
    debug=True
)

response = client.fetch(params={"object": "Mars"})

# Inspect the request
print(f"URL: {response.url}")
print(f"Time: {response.fetch_time}s")
print(f"Cached: {response.cached}")
```

## Configuration

### Environment Variables

```bash
export NEXUSCOSMOS_CACHE_DIR="/path/to/cache"
export NEXUSCOSMOS_CACHE_TTL="86400"
export NEXUSCOSMOS_RATE_LIMIT="1.0"
```

### Configuration File

Create `nexuscosmos.toml` in your project:

```toml
[cache]
directory = ".cache"
ttl = 86400

[requests]
timeout = 30
retry_count = 3
rate_limit = 1.0

[logging]
level = "INFO"
```

Load configuration:

```python
from nexuscosmos.config import Config

config = Config.from_file("nexuscosmos.toml")
client = DataAcquisition(
    base_url="https://api.example.com",
    config=config
)
```

## Extending NexusCosmos

### Custom Data Source

```python
from nexuscosmos.acquisition_base import BaseAcquisition

class MyCustomClient(BaseAcquisition):
    def __init__(self):
        super().__init__(
            base_url="https://my-api.com",
            cache_ttl=3600
        )
    
    def get_object(self, name: str):
        """Fetch object by name."""
        return self.fetch(params={"name": name})
    
    def parse_response(self, response):
        """Custom response parsing."""
        data = response.json()
        return self._transform_data(data)
```

### Custom Parser

```python
from nexuscosmos.parsers_base import BaseTextParser

class MyParser(BaseTextParser):
    def parse(self, text: str) -> dict:
        """Parse custom text format."""
        lines = text.strip().split("\n")
        # Custom parsing logic
        return {"rows": lines}
```
