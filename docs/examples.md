# Examples

Real-world examples using NexusCosmos.

## Example 1: Tracking Mars for a Year

Fetch daily positions of Mars for an entire year:

```python
from nexuscosmos.datasets.horizons import HorizonsClient
from nexuscosmos import DataExporter

# Create client
client = HorizonsClient(cache_ttl=86400 * 30)  # Cache for 30 days

# Get Mars ephemeris for 2025
ephemeris = client.get_ephemeris(
    target="499",           # Mars
    start_time="2025-01-01",
    stop_time="2025-12-31",
    step_size="1d"
)

print(f"Retrieved {len(ephemeris.data)} data points")

# Export to CSV
exporter = DataExporter(ephemeris.data)
exporter.to_csv("mars_2025.csv")

# Or analyze with pandas
df = exporter.to_dataframe()
print(df.describe())
```

## Example 2: Solar System Survey

Fetch data for all planets in parallel:

```python
import asyncio
from nexuscosmos.datasets.horizons import HorizonsClient

# Planet NAIF IDs
PLANETS = {
    "Mercury": "199",
    "Venus": "299",
    "Earth": "399",
    "Mars": "499",
    "Jupiter": "599",
    "Saturn": "699",
    "Uranus": "799",
    "Neptune": "899"
}

async def survey_solar_system():
    client = HorizonsClient()
    
    results = {}
    for name, naif_id in PLANETS.items():
        print(f"Fetching {name}...")
        eph = client.get_ephemeris(
            target=naif_id,
            start_time="2025-06-01",
            stop_time="2025-06-01",
            step_size="1d"
        )
        results[name] = eph.data[0] if eph.data else None
    
    return results

# Run survey
data = asyncio.run(survey_solar_system())

# Display results
print("\nSolar System Survey - June 1, 2025")
print("-" * 50)
for planet, info in data.items():
    if info:
        print(f"{planet:12} RA: {info.get('ra', 'N/A'):>10}  Dec: {info.get('dec', 'N/A'):>10}")
```

## Example 3: Asteroid Tracking

Track multiple asteroids using batch operations:

```python
import asyncio
from nexuscosmos import AsyncDataAcquisition

# Near-Earth asteroids to track
ASTEROIDS = [
    "2000433",   # Eros
    "2000253",   # Mathilde
    "2004179",   # Toutatis
    "2025143",   # Itokawa
    "2101955",   # Bennu
]

async def track_asteroids():
    client = AsyncDataAcquisition(
        base_url="https://ssd.jpl.nasa.gov/api/horizons.api",
        rate_limit=5.0
    )
    
    # Build parameter list
    params_list = []
    for asteroid_id in ASTEROIDS:
        params_list.append({
            "format": "json",
            "COMMAND": f"'{asteroid_id}'",
            "OBJ_DATA": "YES",
            "MAKE_EPHEM": "YES",
            "EPHEM_TYPE": "OBSERVER",
            "CENTER": "'500@399'",
            "START_TIME": "'2025-01-01'",
            "STOP_TIME": "'2025-01-02'",
            "STEP_SIZE": "'1 d'",
        })
    
    # Fetch all in parallel
    results = await client.fetch_batch_async(
        params_list,
        max_concurrent=3,
        show_progress=True
    )
    
    return results

results = asyncio.run(track_asteroids())
print(f"Successfully fetched {sum(1 for r in results if r.success)} asteroids")
```

## Example 4: Custom API Integration

Use NexusCosmos with any REST API:

```python
from nexuscosmos import DataAcquisition

# Example: Open Notify API (ISS location)
class ISSTracker:
    def __init__(self):
        self.client = DataAcquisition(
            base_url="http://api.open-notify.org",
            cache_ttl=60,  # Cache for 1 minute
            rate_limit=0.5  # Max 1 request per 2 seconds
        )
    
    def get_position(self):
        """Get current ISS position."""
        response = self.client.fetch(endpoint="/iss-now.json")
        if response.success:
            pos = response.data.get("iss_position", {})
            return {
                "latitude": float(pos.get("latitude", 0)),
                "longitude": float(pos.get("longitude", 0)),
                "timestamp": response.data.get("timestamp")
            }
        return None
    
    def get_crew(self):
        """Get current ISS crew."""
        response = self.client.fetch(endpoint="/astros.json")
        if response.success:
            people = response.data.get("people", [])
            return [p["name"] for p in people if p.get("craft") == "ISS"]
        return []

# Usage
tracker = ISSTracker()
position = tracker.get_position()
print(f"ISS Location: {position['latitude']}, {position['longitude']}")

crew = tracker.get_crew()
print(f"ISS Crew: {', '.join(crew)}")
```

## Example 5: Data Analysis Pipeline

Complete pipeline from fetch to analysis:

```python
import asyncio
from nexuscosmos.datasets.horizons import HorizonsClient
from nexuscosmos import DataExporter

def analyze_planetary_motion():
    client = HorizonsClient(cache_ttl=86400 * 7)
    
    # Fetch Jupiter's moons
    moons = {
        "Io": "501",
        "Europa": "502",
        "Ganymede": "503",
        "Callisto": "504"
    }
    
    all_data = []
    
    for moon_name, moon_id in moons.items():
        print(f"Fetching {moon_name}...")
        eph = client.get_ephemeris(
            target=moon_id,
            start_time="2025-01-01",
            stop_time="2025-01-31",
            step_size="6h"  # Every 6 hours
        )
        
        # Add moon name to each row
        for row in eph.data:
            row["moon"] = moon_name
            all_data.append(row)
    
    # Export combined data
    exporter = DataExporter(all_data)
    
    # Save as JSON
    exporter.to_json("galilean_moons_jan2025.json")
    
    # Analyze with pandas
    df = exporter.to_dataframe()
    
    # Group by moon
    for moon in moons.keys():
        moon_data = df[df["moon"] == moon]
        print(f"\n{moon}:")
        print(f"  Data points: {len(moon_data)}")
        if "ra" in moon_data.columns:
            print(f"  RA range: {moon_data['ra'].min()} to {moon_data['ra'].max()}")
    
    return df

# Run analysis
df = analyze_planetary_motion()
```

## Example 6: Error Handling & Retry

Robust data fetching with error handling:

```python
import time
from nexuscosmos import DataAcquisition
from nexuscosmos.utils import NexusCosmosError, RateLimitError

def robust_fetch(client, params, max_attempts=3):
    """Fetch with comprehensive error handling."""
    for attempt in range(max_attempts):
        try:
            response = client.fetch(params=params)
            if response.success:
                return response.data
            else:
                print(f"Attempt {attempt + 1}: Request failed - {response.error}")
                
        except RateLimitError:
            wait_time = 2 ** attempt * 10  # Exponential backoff
            print(f"Rate limited. Waiting {wait_time}s...")
            time.sleep(wait_time)
            
        except NexusCosmosError as e:
            print(f"Attempt {attempt + 1}: Error - {e}")
            if attempt < max_attempts - 1:
                time.sleep(2 ** attempt)
    
    return None

# Usage
client = DataAcquisition(
    base_url="https://api.example.com",
    retry_count=0  # We handle retries manually
)

data = robust_fetch(client, {"object": "Mars"})
if data:
    print("Successfully retrieved data")
else:
    print("Failed after all attempts")
```

## Example 7: Progress Monitoring

Track progress of large batch operations:

```python
import asyncio
from nexuscosmos import AsyncDataAcquisition

async def fetch_with_progress():
    client = AsyncDataAcquisition(
        base_url="https://api.example.com",
        rate_limit=10.0
    )
    
    # Generate 100 requests
    params_list = [{"id": i, "type": "observation"} for i in range(100)]
    
    # Fetch with progress bar
    results = await client.fetch_batch_async(
        params_list,
        max_concurrent=5,
        show_progress=True  # Shows: Fetching: 100%|██████████| 100/100 [00:15<00:00]
    )
    
    # Summary
    successful = sum(1 for r in results if r.success)
    cached = sum(1 for r in results if r.cached)
    
    print(f"\nResults:")
    print(f"  Successful: {successful}/{len(results)}")
    print(f"  From cache: {cached}")
    
    return results

asyncio.run(fetch_with_progress())
```

## Example 8: Combining with Astropy

Use NexusCosmos with Astropy for unit handling:

```python
# Requires: pip install nexuscosmos[astropy]
from nexuscosmos.datasets.horizons import HorizonsClient
from astropy import units as u
from astropy.coordinates import SkyCoord

def get_skycoord(target, date):
    """Get target position as Astropy SkyCoord."""
    client = HorizonsClient()
    
    eph = client.get_ephemeris(
        target=target,
        start_time=date,
        stop_time=date,
        step_size="1d"
    )
    
    if eph.data:
        row = eph.data[0]
        return SkyCoord(
            ra=float(row["ra"]) * u.deg,
            dec=float(row["dec"]) * u.deg,
            frame="icrs"
        )
    return None

# Get Mars position
mars = get_skycoord("499", "2025-06-15")
print(f"Mars position: {mars.to_string('hmsdms')}")

# Calculate separation between planets
venus = get_skycoord("299", "2025-06-15")
separation = mars.separation(venus)
print(f"Mars-Venus separation: {separation.deg:.2f} degrees")
```
