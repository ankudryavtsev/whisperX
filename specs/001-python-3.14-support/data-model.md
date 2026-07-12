# Data Model: Python 3.14 Support

This feature involves updating python environment package version constraints and CI workflows. There are no database entities, state machines, or data models introduced by this change.

## Configuration Schemas Modified

### `pyproject.toml`
* **Field:** `[project].requires-python`
* **Change:** `">=3.10, <3.14"` ➡️ `">=3.10, <3.15"`
