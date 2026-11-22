# Source-code roadmap

The public implementation will be organized into four reproducible stages:

1. Build multi-scale AlphaEarth, POI-text, and mobility views at POI anchors.
2. Train the tri-modal contrastive projectors.
3. Apply the frozen AlphaEarth projector to regional embedding fields.
4. Run common-support linear and MLP probes for the nine downstream tasks.

The current public implementation is provided in `notebooks/`. This directory is reserved for a future modular package extracted from the validated notebook pipeline. Public modules must avoid hard-coded local paths and expose data locations and hyperparameters through configuration files.
