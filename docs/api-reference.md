# API Reference

Complete API documentation for NexusCosmos.

## nexuscosmos

### DataAcquisition

Main class for fetching data from any REST API.

```python
class DataAcquisition(base_url, **kwargs)
```

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `base_url` | str | Required | Base URL of the API |
| `cache_ttl` | int | 3600 | Cache time-to-live in seconds |
| `cache_dir` | str | ".nexuscosmos_cache" | Cache directory path |
| `rate_limit` | float | 1.0 | Max requests per second |
| `timeout` | int | 30 | Request timeout in seconds |
| `retry_count` | int | 3 | Number of retry attempts |
| `retry_delay` | float | 1.0 | Delay between retries |
| `headers` | dict | None | Custom HTTP headers |
| `debug` | bool | False | Enable debug mode |

**Methods:**

#### fetch()

```python
def fetch(params=None, endpoint="", method="GET", use_cache=True) -> Response
```

Fetch data from the API.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `params` | dict | None | Query parameters |
| `endpoint` | str | "" | Additional endpoint path |
| `method` | str | "GET" | HTTP method |
| `use_cache` | bool | True | Whether to use cache |

**Returns:** `Response` object

**Example:**
```python
response = client.fetch(params={"object": "Mars"})
```

#### fetch_batch()

```python
def fetch_batch(params_list, show_progress=False) -> list[Response]
```

Fetch multiple requests sequentially.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `params_list` | list[dict] | Required | List of parameter dicts |
| `show_progress` | bool | False | Show progress bar |

**Returns:** List of `Response` objects

#### cache_stats()

```python
def cache_stats() -> dict
```

Get cache statistics.

**Returns:** Dictionary with keys: `total_entries`, `size_bytes`, `hits`, `misses`, `hit_rate`

#### clear_cache()

```python
def clear_cache(key=None) -> None
```

Clear cache entries.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `key` | str | None | Specific key to clear, or all if None |

---

### AsyncDataAcquisition

Async version for parallel fetching.

```python
class AsyncDataAcquisition(base_url, **kwargs)
```

Same parameters as `DataAcquisition`.

**Methods:**

#### fetch_async()

```python
async def fetch_async(params=None, endpoint="") -> Response
```

Async fetch single request.

#### fetch_batch_async()

```python
async def fetch_batch_async(params_list, max_concurrent=10, show_progress=False) -> list[Response]
```

Fetch multiple requests in parallel.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `params_list` | list[dict] | Required | List of parameter dicts |
| `max_concurrent` | int | 10 | Max simultaneous requests |
| `show_progress` | bool | False | Show progress bar |

**Example:**
```python
import asyncio

async def main():
    client = AsyncDataAcquisition(base_url="https://api.example.com")
    results = await client.fetch_batch_async(
        [{"id": i} for i in range(100)],
        max_concurrent=5
    )
    return results

results = asyncio.run(main())
```

---

### Response

Response object returned by fetch operations.

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| `data` | dict/list | Parsed response data |
| `raw_text` | str | Raw response text |
| `status_code` | int | HTTP status code |
| `headers` | dict | Response headers |
| `cached` | bool | Whether from cache |
| `fetch_time` | float | Request duration (seconds) |
| `url` | str | Requested URL |
| `success` | bool | Whether request succeeded |
| `error` | str | Error message if failed |

---

### DataExporter

Export data to various formats.

```python
class DataExporter(data)
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `data` | list/dict | Data to export |

**Methods:**

#### to_json()

```python
def to_json(filepath, indent=2) -> None
```

Export to JSON file.

#### to_csv()

```python
def to_csv(filepath, columns=None, delimiter=",", include_header=True) -> None
```

Export to CSV file.

#### to_dataframe()

```python
def to_dataframe() -> pandas.DataFrame
```

Convert to pandas DataFrame. Requires `pandas` installed.

---

## nexuscosmos.datasets.horizons

### HorizonsClient

Client for JPL Horizons API.

```python
class HorizonsClient(**kwargs)
```

Inherits from `DataAcquisition` with Horizons-specific defaults.

**Methods:**

#### get_ephemeris()

```python
def get_ephemeris(target, start_time, stop_time, step_size="1d", 
                  center="500@10", ref_plane="ECLIPTIC") -> EphemerisResponse
```

Get ephemeris data for a target body.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `target` | str | Required | Target body ID or name |
| `start_time` | str | Required | Start date (YYYY-MM-DD) |
| `stop_time` | str | Required | End date (YYYY-MM-DD) |
| `step_size` | str | "1d" | Time step (e.g., "1h", "1d", "1mo") |
| `center` | str | "500@10" | Observer location |
| `ref_plane` | str | "ECLIPTIC" | Reference plane |

**Returns:** `EphemerisResponse` object

**Example:**
```python
from nexuscosmos.datasets.horizons import HorizonsClient

client = HorizonsClient()
eph = client.get_ephemeris(
    target="499",        # Mars
    start_time="2025-01-01",
    stop_time="2025-12-31",
    step_size="1d"
)

for row in eph.data:
    print(f"{row['date']}: RA={row['ra']}, Dec={row['dec']}")
```

#### get_vectors()

```python
def get_vectors(target, start_time, stop_time, step_size="1d",
                center="500@10", ref_frame="ICRF") -> VectorResponse
```

Get position/velocity vectors.

---

## nexuscosmos.config

### Config

Configuration management.

```python
class Config(**kwargs)
```

**Class Methods:**

#### from_file()

```python
@classmethod
def from_file(filepath) -> Config
```

Load configuration from TOML file.

#### from_env()

```python
@classmethod
def from_env() -> Config
```

Load configuration from environment variables.

---

## nexuscosmos.utils

### Exceptions

```python
class NexusCosmosError(Exception)
    """Base exception for all NexusCosmos errors."""

class ConnectionError(NexusCosmosError)
    """Network connection failed."""

class TimeoutError(NexusCosmosError)
    """Request timed out."""

class RateLimitError(NexusCosmosError)
    """API rate limit exceeded."""

class ParseError(NexusCosmosError)
    """Failed to parse response."""

class CacheError(NexusCosmosError)
    """Cache operation failed."""
```

### Utility Functions

#### ExponentialBackoff

```python
class ExponentialBackoff(base_delay=1.0, max_delay=60.0, factor=2.0)
```

Exponential backoff calculator for retries.

```python
backoff = ExponentialBackoff()
for attempt in range(5):
    delay = backoff.next_delay()
    print(f"Attempt {attempt + 1}, wait {delay}s")
```
