# NexusCosmos Documentation

Welcome to NexusCosmos - a powerful, dataset-agnostic Python library for astronomical data acquisition.

## Table of Contents

1. [Installation](installation.md)
2. [Quick Start](quickstart.md)
3. [User Guide](user-guide.md)
4. [API Reference](api-reference.md)
5. [Examples](examples.md)
6. [Contributing](../CONTRIBUTING.md)

## Overview

NexusCosmos provides a unified interface to fetch astronomical data from any REST API with:

- **Intelligent Caching** - Avoid redundant requests
- **Rate Limiting** - Respect API limits automatically
- **Async Support** - Fetch data in parallel
- **Progress Tracking** - Monitor long operations
- **Multiple Export Formats** - JSON, CSV, DataFrame

## Quick Example

```python
from nexuscosmos import DataAcquisition

# Create a client for any API
client = DataAcquisition(
    base_url="https://astronomy-api.example.com",
    cache_ttl=3600
)

# Fetch data
response = client.fetch(params={"object": "Mars"})
print(response.data)
```

## License

MIT License - see [LICENSE](../LICENSE) for details.
