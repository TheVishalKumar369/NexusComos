# Installation

## Requirements

- Python 3.8 or higher
- pip package manager

## Basic Installation

Install the core library from PyPI:

```bash
pip install nexuscosmos
```

## Installation with Extras

NexusCosmos provides optional features through extras:

### Async Support
For parallel/async data fetching:
```bash
pip install nexuscosmos[async]
```

### Progress Bars
For progress tracking on batch operations:
```bash
pip install nexuscosmos[progress]
```

### Pandas Integration
For DataFrame export:
```bash
pip install nexuscosmos[pandas]
```

### Astropy Integration
For astronomical unit handling:
```bash
pip install nexuscosmos[astropy]
```

### All Features
Install everything:
```bash
pip install nexuscosmos[async,progress,pandas,astropy]
```

## Development Installation

For contributing or development:

```bash
# Clone the repository
git clone https://github.com/TheVishalKumar369/NexusCosmos.git
cd NexusCosmos

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install in development mode
pip install -e ".[async,progress,pandas,astropy]"

# Install test dependencies
pip install pytest pytest-cov
```

## Verifying Installation

```python
import nexuscosmos
print(nexuscosmos.__version__)
```

## Upgrading

```bash
pip install --upgrade nexuscosmos
```

## Uninstalling

```bash
pip uninstall nexuscosmos
```
