# BEACON: Behavioral and Semantic Enrichment of AlphaEarth

**Tri-modal contrastive learning for human-contextualized geospatial embeddings**

[![Paper](https://img.shields.io/badge/ACM%20SIGSPATIAL%202026-10.1145%2F3841645.3843383-blue)](https://doi.org/10.1145/3841645.3843383)
[![Status](https://img.shields.io/badge/status-active-brightgreen)](#repository-status)
[![License: MIT](https://img.shields.io/badge/code%20license-MIT-lightgrey.svg)](LICENSE)

## Overview

Geospatial foundation models such as AlphaEarth produce compact, globally consistent representations of the Earth's surface. Because they learn mainly from Earth-observation imagery, their embeddings capture physical and spectral characteristics more strongly than human activity or urban function.

BEACON addresses this limitation with a tri-modal contrastive learning framework that aligns three complementary views of urban space:

- **Physical:** multi-scale AlphaEarth embeddings
- **Semantic:** point-of-interest (POI) description embeddings
- **Behavioral:** hourly and seasonal POI visitation patterns

POI text and mobility provide supervision during training. After training, BEACON generates a 128-dimensional representation from AlphaEarth inputs alone, so deployment does not require POI or mobility data.

<p align="center">
  <img src="docs/images/beacon_framework.jpg" width="100%" alt="BEACON tri-modal contrastive learning framework and downstream evaluation workflow">
</p>

<p align="center"><em>Figure 1. BEACON aligns physical, semantic, and behavioral views during pretraining, then uses AlphaEarth alone for downstream inference.</em></p>

## Study design

The study uses the Houston-The Woodlands-Sugar Land Metropolitan Statistical Area in 2024.

- **Pretraining anchors:** 160,944 POIs with complete mobility records
- **AlphaEarth input:** 64-dimensional annual embeddings at 10 m resolution
- **Physical views:** mean-pooled 9 x 9 local and 19 x 19 context windows
- **Semantic view:** 384-dimensional embeddings of POI category, name, and NAICS descriptions
- **Behavioral view:** 184-dimensional profiles combining hour-of-week, monthly patterns, and four activity descriptors
- **Training objective:** four symmetric InfoNCE alignment terms
- **Evaluation:** seven regression and two classification tasks, using frozen linear and MLP probes over five seeds
- **Baselines:** coordinates, Space2Vec, SatCLIP, TESSERA, Clay, and AlphaEarth

## Method

BEACON constructs three views for every POI anchor:

- **Physical view:** a 64-dimensional AlphaEarth vector at the POI, plus mean-pooled 9 x 9 and 19 x 19 neighborhoods. A shared projector maps all three scales to a 128-dimensional space.
- **Semantic view:** POI name, category, top category, subcategory, and NAICS descriptions are encoded into a 384-dimensional text representation.
- **Behavioral view:** a 184-dimensional vector combines 168 hour-of-week visit counts, 12 monthly counts, and four activity descriptors.

The training objective is a weighted sum of four symmetric InfoNCE losses:

| Alignment pair | Weight |
|---|---:|
| AlphaEarth ↔ POI semantics | 0.35 |
| AlphaEarth ↔ mobility behavior | 0.35 |
| POI semantics ↔ mobility behavior | 0.15 |
| Local ↔ contextual AlphaEarth | 0.15 |

Training uses temperature 0.07, AdamW with a 1e-3 learning rate and 1e-4 weight decay, cosine scheduling, batch size 512, and 100 epochs. The POIs are divided into 80/10/10 train, validation, and test splits. At inference time, only the AlphaEarth branch is retained.

## Downstream benchmarks

| Task | Spatial support | Samples | Type |
|---|---|---:|---|
| Median household income | Census block group | 4,174 | Regression |
| Obesity prevalence | Census tract | 1,613 | Regression |
| Poor mental health | Census tract | 1,613 | Regression |
| Nighttime lights | 500 m grid | 139,186 | Regression |
| Daytime land-surface temperature | 1 km grid | 27,424 | Regression |
| Nighttime land-surface temperature | 1 km grid | 27,424 | Regression |
| PM2.5 concentration | 0.01 degree grid | 24,338 | Regression |
| Land cover | 10 m samples | 298,944 | Classification |
| Functional land use | Parcels | 255,431 | Classification |

## Training dynamics

<p align="center">
  <img src="docs/images/training_dynamics.jpg" width="100%" alt="BEACON training losses, cross-modal retrieval, and similarity matrices">
</p>

<p align="center"><em>Figure 2. Training and validation losses, relative pairwise losses, cross-modal retrieval, and similarity structure before and after training.</em></p>

The model converges stably, with the best validation checkpoint at epoch 95. Local-context consistency and AlphaEarth-mobility alignment are the strongest relative alignment terms. Mobility retrieval is stronger than POI-semantic retrieval, and the post-training similarity matrices show substantially clearer cross-modal correspondence.

## Main results

Under a frozen linear probe, BEACON improves human-centered prediction while remaining competitive on physical and environmental tasks.

### Regression

| Downstream target | AlphaEarth R² | BEACON R² | Relative change |
|---|---:|---:|---:|
| Median household income | 0.379 | **0.463** | **+22%** |
| Obesity prevalence | 0.431 | **0.614** | **+43%** |
| Poor mental health | 0.393 | **0.528** | **+34%** |
| Nighttime lights | 0.663 | **0.680** | +3% |
| Daytime land-surface temperature | 0.840 | **0.843** | +0.4% |
| Nighttime land-surface temperature | **0.792** | 0.787 | -0.6% |
| PM2.5 concentration | 0.845 | **0.856** | +1.3% |

### Classification

| Downstream target | AlphaEarth Macro-F1 | BEACON Macro-F1 |
|---|---:|---:|
| Land cover | **0.584** | 0.567 |
| Functional land use | 0.456 | **0.471** |

### Ablation study

The ablation results summarize performance across all downstream tasks. The human-centered score aggregates income, obesity, mental health, and functional land use; the physical/environmental score aggregates nighttime lights, land-surface temperature, PM2.5, and land cover.

| Representation | Human-centered score | Physical/environmental score |
|---|---:|---:|
| Mobility only | 0.011 | 0.015 |
| POI semantics only | 0.288 | 0.199 |
| Raw feature concatenation | 0.294 | 0.644 |
| AlphaEarth | 0.401 | 0.743 |
| AlphaEarth + mobility | 0.475 | 0.736 |
| AlphaEarth + POI semantics | 0.494 | 0.750 |
| **BEACON** | **0.535** | **0.759** |

BEACON delivers its largest gains on human-centered targets while preserving AlphaEarth's strength on physical and environmental prediction. The ablations show that POI semantics and mobility behavior provide complementary supervision; contrastive alignment is more effective than raw feature concatenation.

## Repository status

This repository is the public research companion for the ACM SIGSPATIAL 2026 paper. It includes the cleaned, English-language research pipeline as three notebooks. The notebooks contain no outputs, machine-specific paths, credentials, or provider-restricted data.

## Pipeline notebooks

1. [`01_data_preparation.ipynb`](notebooks/01_data_preparation.ipynb) builds the POI-level tri-modal training table and exports regional AlphaEarth tiles.
2. [`02_model_training.ipynb`](notebooks/02_model_training.ipynb) trains BEACON, evaluates cross-modal alignment, and projects AlphaEarth tiles to 128-dimensional BEACON embeddings.
3. [`03_downstream_tasks.ipynb`](notebooks/03_downstream_tasks.ipynb) aggregates embeddings to nine downstream tasks and evaluates frozen linear and MLP probes.

## Installation

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Set the project root before running the notebooks:

```bash
export BEACON_PROJECT_DIR=/path/to/beacon-data
```

Optional data-access steps use environment variables rather than embedded credentials or account identifiers:

```bash
export MONGODB_URI=mongodb://localhost:27017/
export OPENAI_API_KEY=your_key
export OPENAI_BASE_URL=https://api.openai.com/v1       # optional
export GOOGLE_CLOUD_PROJECT=your_google_cloud_project
export BEACON_GEE_POI_ASSET=projects/PROJECT/assets/ASSET
```

External-service and cloud-export sections are disabled by default. Review the configuration cell in each notebook before enabling them.

## Repository structure

```text
.
├── configs/            experiment and dataset configuration
├── data/               data-access notes; no restricted data
├── docs/               method documentation and paper figures
│   └── images/         framework and training-dynamics figures
├── notebooks/          complete data, training, and evaluation pipeline
├── src/                training, inference, probes, and evaluation
├── CITATION.cff
├── LICENSE
├── requirements.txt
└── README.md
```

## Data availability

This repository will not redistribute licensed POI visitation records or other restricted source data. The `data/` directory documents expected inputs, spatial supports, and access conditions. Public downstream datasets should be downloaded from their original providers and cited according to their licenses.

Do not commit raw data, credentials, API keys, trained checkpoints, or derived files that retain provider-restricted information.

## Paper

Hao Tian, Heng Cai, and Yifan Yang. 2026. **BEACON: Behavioral and Semantic Enrichment of AlphaEarth Embeddings through Tri-Modal Contrastive Learning.** In *The 34th ACM International Conference on Advances in Geographic Information Systems (SIGSPATIAL '26)*, November 3-6, 2026, Riverside, CA, USA. ACM, New York, NY, USA, 4 pages. <https://doi.org/10.1145/3841645.3843383>

```bibtex
@inproceedings{tian2026beacon,
  title     = {BEACON: Behavioral and Semantic Enrichment of AlphaEarth Embeddings through Tri-Modal Contrastive Learning},
  author    = {Tian, Hao and Cai, Heng and Yang, Yifan},
  booktitle = {Proceedings of the 34th ACM International Conference on Advances in Geographic Information Systems},
  year      = {2026},
  doi       = {10.1145/3841645.3843383}
}
```

## License

Code in this repository is released under the [MIT License](LICENSE). The paper, figures, presentation materials, datasets, and third-party assets retain their original licenses or copyright terms unless a file states otherwise.

## Contact

Hao Tian  
Department of Geography, Texas A&M University  
<haotian@tamu.edu>
