# Quick Start Guide

Get up and running with NexusCosmos in 5 minutes.

## Basic Usage

### 1. Import and Create a Client

```python
from nexuscosmos import DataAcquisition

# Create a client for any REST API
client = DataAcquisition(
    base_url="https://your-astronomy-api.com/endpoint",
    cache_ttl=3600,      # Cache responses for 1 hour
    rate_limit=1.0       # Max 1 request per second
)
```

### 2. Fetch Data

```python
# Simple GET request
response = client.fetch(params={
    "object": "Mars",
    "date": "2025-01-01"
})

# Access the data
print(response.data)
print(response.status_code)
print(response.cached)  # True if served from cache
```

### 3. Batch Operations

```python
# Fetch multiple objects
objects = ["Mercury", "Venus", "Earth", "Mars"]
results = client.fetch_batch(
    [{"object": obj} for obj in objects],
    show_progress=True  # Requires nexuscosmos[progress]
)

for obj, result in zip(objects, results):
    print(f"{obj}: {result.data}")
```

## Using JPL Horizons

NexusCosmos includes built-in support for JPL Horizons:

```python
from nexuscosmos.datasets.horizons import HorizonsClient

# Create Horizons client
client = HorizonsClient()

# Get ephemeris data for Mars
ephemeris = client.get_ephemeris(
    target="499",           # Mars NAIF ID
    start_time="2025-01-01",
    stop_time="2025-01-31",
    step_size="1d"
)

# Access parsed data
print(f"Rows: {len(ephemeris.data)}")
print(f"Columns: {ephemeris.data[0].keys()}")
```

## Async Operations

For faster parallel fetching:

```python
import asyncio
from nexuscosmos import AsyncDataAcquisition

async def main():
    client = AsyncDataAcquisition(
        base_url="https://api.example.com",
        cache_ttl=3600
    )
    
    # Fetch 100 objects in parallel
    params_list = [{"id": i} for i in range(100)]
    results = await client.fetch_batch_async(
        params_list,
        max_concurrent=10,  # Limit concurrent requests
        show_progress=True
    )
    
    return results

# Run async code
results = asyncio.run(main())
```

## Exporting Data

```python
from nexuscosmos import DataExporter

# Export to different formats
exporter = DataExporter(results)

# Save as JSON
exporter.to_json("output.json")

# Save as CSV
exporter.to_csv("output.csv")

# Convert to DataFrame (requires pandas)
df = exporter.to_dataframe()
```

## Caching

Caching is automatic. Control it with:

```python
# Create client with caching
client = DataAcquisition(
    base_url="https://api.example.com",
    cache_ttl=86400,              # 24 hours
    cache_dir=".my_cache"         # Custom cache directory
)

# Check cache statistics
stats = client.cache_stats()
print(f"Cache hits: {stats['hits']}")
print(f"Cache misses: {stats['misses']}")

# Clear cache
client.clear_cache()
```

## Error Handling

```python
from nexuscosmos import DataAcquisition
from nexuscosmos.utils import NexusCosmosError

client = DataAcquisition(base_url="https://api.example.com")

try:
    response = client.fetch(params={"object": "invalid"})
except NexusCosmosError as e:
    print(f"Error: {e}")
    # Access details
    print(f"Status code: {e.status_code}")
    print(f"Response: {e.response}")
```

## Next Steps

- Read the [User Guide](user-guide.md) for detailed features
- Check [API Reference](api-reference.md) for all methods
- See [Examples](examples.md) for real-world use cases
