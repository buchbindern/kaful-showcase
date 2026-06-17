# Sample data schema

## Input

Kaful takes OEM documentation as input — PDFs, instruction books, datasheets. No structured data required.

```
input/
  oem_manual.pdf          # primary source (instruction book, service manual)
  compressed_air_manual.pdf  # supplementary reference (optional)
```

## Intermediate: extracted component schema

After the Schema Extract stage, each component is represented as a structured object:

```json
{
  "component_id": "air_filter_component",
  "component_name": "Air filter",
  "failure_modes": [
    {
      "failure_id": "choked_filter",
      "description": "Filter element becomes clogged, restricting airflow",
      "degradation_parameter": "pressure_drop_kpa",
      "threshold": 0.5,
      "source": "GA55-90C instruction book, section 8.2"
    }
  ],
  "service_interval_hours": 4000,
  "degradation_model": "linear"
}
```

## Output: RUL prediction per component

```json
{
  "machine": "Atlas Copco GA90C",
  "usage_profile": "light_intermediate",
  "predictions": {
    "air_filter_component.choked_filter": {
      "already_triggered": false,
      "rul_mean": 3294.46,
      "rul_std": 73.06,
      "rul_min": 3013.0,
      "rul_max": 3453.0,
      "beyond_horizon": false
    }
  }
}
```

### Field definitions

| Field | Type | Description |
|---|---|---|
| `already_triggered` | bool | Whether this failure mode has already occurred given the usage profile |
| `rul_mean` | float | Mean RUL estimate across the particle ensemble (hours) |
| `rul_std` | float | Standard deviation across the particle ensemble |
| `rul_min` | float | Minimum RUL across the ensemble (pessimistic bound) |
| `rul_max` | float | Maximum RUL across the ensemble (optimistic bound) |
| `beyond_horizon` | bool | True if failure is not predicted within the simulation window |

### Usage profiles

Three standard profiles model different operating intensities:

| Profile | Description | Duty cycle |
|---|---|---|
| `light_intermediate` | Low-intensity intermittent operation | ~40% |
| `moderate_commercial` | Standard commercial use | ~65% |
| `continuous_industrial` | 24/7 industrial operation | ~95% |