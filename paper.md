---
title: 'NexusCosmos: A Dataset-Agnostic Python Library for Astronomical Data Acquisition'
tags:
  - Python
  - astronomy
  - data acquisition
  - ephemeris
  - API client
  - async
authors:
  - name: Pancha Narayan Sahu
    orcid: 0009-0003-7618-7588
    affiliation: 1
affiliations:
  - name: Independent Researcher
    index: 1
date: 12 December 2025
bibliography: paper.bib
---

# Summary

NexusCosmos is a Python library designed to simplify astronomical data acquisition from any REST API source. Unlike existing tools that are tightly coupled to specific data providers, NexusCosmos provides a unified, dataset-agnostic interface that works with any HTTP-based astronomical data service. The library features intelligent caching, configurable rate limiting, asynchronous batch operations, progress tracking, and multiple export formats.

# Statement of Need

Astronomical research increasingly relies on programmatic access to remote data services. While excellent domain-specific tools exist—such as Astroquery [@astroquery] for accessing astronomical databases—researchers often face challenges when:

1. **Working with multiple data sources**: Each API requires learning different interfaces and handling different response formats.
2. **Processing large datasets**: Sequential requests to remote services are slow and inefficient.
3. **Managing rate limits**: Exceeding API rate limits leads to blocked requests and failed analyses.
4. **Reproducing results**: Without proper caching, re-running analyses requires re-fetching data.

NexusCosmos addresses these challenges by providing:

- **A unified interface**: One consistent API for any REST-based data source
- **Async batch operations**: Fetch hundreds of objects in parallel (3-10x faster)
- **Intelligent caching**: Automatic disk-based caching with configurable TTL
- **Rate limiting**: Built-in throttling to respect API limits
- **Progress tracking**: Real-time feedback for long-running operations
- **Export flexibility**: Save data as JSON, CSV, or pandas DataFrames

# Target Audience

NexusCosmos is designed for:

- **Astronomers and astrophysicists** who need to fetch ephemeris data, orbital parameters, or observational data programmatically
- **Graduate students** learning to work with astronomical data APIs
- **Data scientists** working on astronomical datasets who need efficient data retrieval
- **Software developers** building astronomical applications that consume external data

# Key Features

## Dataset-Agnostic Design

```python
from nexuscosmos import DataAcquisition

# Works with ANY REST API
client = DataAcquisition(
    base_url="https://any-astronomy-api.com/data",
    cache_ttl=86400  # Cache for 24 hours
)

response = client.fetch(params={"object": "Mars", "date": "2025-01-01"})
```

## Async Batch Operations

```python
import asyncio
from nexuscosmos import AsyncDataAcquisition

async def fetch_solar_system():
    client = AsyncDataAcquisition(base_url="https://api.example.com")
    
    objects = ["Mercury", "Venus", "Earth", "Mars", "Jupiter", 
               "Saturn", "Uranus", "Neptune"]
    
    # Fetch all planets in parallel
    results = await client.fetch_batch_async(
        [{"object": obj} for obj in objects],
        show_progress=True
    )
    return results

data = asyncio.run(fetch_solar_system())
```

## Built-in JPL Horizons Support

NexusCosmos includes built-in support for the JPL Horizons system [@jpl_horizons], one of the most widely used ephemeris services:

```python
from nexuscosmos.datasets.horizons import HorizonsClient

client = HorizonsClient()
ephemeris = client.get_ephemeris(
    target="499",  # Mars
    start_time="2025-01-01",
    stop_time="2025-12-31",
    step_size="1d"
)
```

# Comparison with Existing Tools

NexusCosmos complements existing tools like Astroquery [@astroquery] by providing a more flexible, dataset-agnostic approach:

| Feature | NexusCosmos | Astroquery | Requests |
|---------|-------------|------------|----------|
| Dataset-agnostic | Yes | No | Yes |
| Built-in caching | Yes | Partial | No |
| Async support | Yes | No | No |
| Rate limiting | Yes | Partial | No |
| Progress tracking | Yes | No | No |
| Astronomy-focused | Yes | Yes | No |

# Implementation

NexusCosmos is implemented in pure Python with minimal dependencies. The core library requires only the `requests` package [@requests]. Optional features are available through extras:

- `nexuscosmos[async]` - Async support via `aiohttp`
- `nexuscosmos[progress]` - Progress bars via `tqdm`
- `nexuscosmos[pandas]` - DataFrame export via `pandas`
- `nexuscosmos[astropy]` - Astropy integration [@astropy]

The architecture follows a modular design with base classes that can be extended for specific data sources.

# Availability

NexusCosmos is available on PyPI (`pip install nexuscosmos`) and GitHub (https://github.com/TheVishalKumar369/NexusComos). The library is released under the MIT License.

# Acknowledgements

We thank the astronomical community for their feedback and the developers of the JPL Horizons system [@jpl_horizons] for providing an excellent public API.

# References
