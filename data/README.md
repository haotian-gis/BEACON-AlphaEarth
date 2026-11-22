# Data organization

BEACON uses multiple data sources with different access and redistribution conditions. This directory stores documentation and small, non-sensitive examples only. Do not commit raw or provider-restricted data.

## Expected input groups

- AlphaEarth annual embeddings for the Houston metropolitan area
- POI attributes used to construct textual descriptions
- Hourly and monthly POI visitation patterns for 2024
- Downstream labels for socioeconomic, health, activity, environmental, land-cover, and land-use tasks
- Spatial boundaries or sampling geometries for each downstream support

## Recommended local layout

```text
data/
├── raw/          immutable source files; ignored by Git
├── interim/      harmonized spatial and temporal inputs; ignored by Git
├── processed/    model-ready tensors and benchmark tables; ignored by Git
└── examples/     optional synthetic or redistributable samples
```

Document the provider, version, access date, license, spatial coverage, temporal coverage, coordinate reference system, and preprocessing history for every source before public release.
