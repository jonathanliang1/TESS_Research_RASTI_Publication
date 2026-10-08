# TESS Light-Curve Provenance and Variable-Star Classification

This repository contains the analysis notebooks, derived data products, and saved results supporting:

**Jonathan Liang, "Quantifying the Impact of TESS Light-Curve Provenance on Random Forest Variable-Star Classification"**  
Prepared for submission to *RAS Techniques and Instruments (RASTI)*.

## Overview

This study tests whether alternative TESS light-curve representations of the same variable stars produce different downstream Random Forest classification results. The primary controlled experiments compare:

- **506 matched SPOC–TESSCut stars**
- **1,355 matched QLP–TESSCut stars**

Within each matched comparison, stellar identity, class labels, predictor definitions, Random Forest settings, and star-level train/test assignments are held fixed. The main repeated analysis uses 100 stratified 70/30 train/test splits and a common 39-predictor feature set.

The repository also contains sensitivity and mechanism-oriented controls involving:

- ablation of six provenance-associated observational features;
- per-class paired error analysis;
- repeated permutation-importance analysis;
- conditional TESSCut detrending;
- iterative 5-sigma clipping; and
- Gaia-based crowding and TESSCut aperture diagnostics.

## Repository structure

```text
.
├── data/
│   ├── external_cache/
│   │   └── gaia/
│   │       ├── qlp/
│   │       │   └── gaia_neighbors.parquet
│   │       └── spoc/
│   │           └── gaia_neighbors.parquet
│   ├── processed/
│   │   ├── QLP_TESSCut_features_QC.parquet
│   │   ├── QLP_TESSCut_Metadata_Consolidated_QC.parquet
│   │   ├── SPOC_TESSCut_features_QC.parquet
│   │   ├── SPOC_TESSCut_Metadata_Consolidated_QC.parquet
│   │   ├── TESSAugmented.parquet
│   │   └── TESS_features.parquet
│   └── sample/
│       ├── TESSAugmented_QC.parquet
│       ├── VSXMetadata.parquet
│       └── VSX_TESS_MATCH_Metadata.parquet
├── notebooks/
│   ├── 01_data_pipeline.ipynb
│   ├── 02_feature_extraction.ipynb
│   ├── 03_analyze_comprehensive.ipynb
│   ├── 04_controlled_analysis_monte_carlo.ipynb
│   ├── 05_controlled_detrending.ipynb
│   ├── 06_tesscut_sigma_clipping_controlled.ipynb
│   ├── 07_spoc_tess_aperture_crowding.ipynb
│   └── 08_qlp_tesscut_aperture_crowding.ipynb
└── results/
    ├── controlled_analysis/
    ├── crowding/
    │   ├── qlp/
    │   └── spoc/
    ├── detrending/
    │   ├── QLP/
    │   └── SPOC/
    └── sigma_clipping/
```

## Notebook guide

| Notebook | Purpose |
|---|---|
| `01_data_pipeline.ipynb` | VSX sample construction, VSX–TIC association, TESS light-curve acquisition, QC/standardization, provenance metadata, attrition, and catalogue-validation diagnostics |
| `02_feature_extraction.ipynb` | Common feature extraction from the light curves, including time-domain, Lomb–Scargle, and phase-folded morphology features |
| `03_analyze_comprehensive.ipynb` | Main mixed-provenance analysis, exploratory provenance-specific models, representative matched-star comparisons, and supporting diagnostics |
| `04_controlled_analysis_monte_carlo.ipynb` | Primary 100-split matched-star analysis, observational-feature ablation, per-class paired errors, and repeated feature-importance analysis |
| `05_controlled_detrending.ipynb` | Controlled conditional-detrending experiment for the matched TESSCut representations |
| `06_tesscut_sigma_clipping_controlled.ipynb` | Controlled 5-sigma clipping experiment |
| `07_spoc_tess_aperture_crowding.ipynb` | SPOC–TESSCut Gaia crowding and aperture control |
| `08_qlp_tesscut_aperture_crowding.ipynb` | QLP–TESSCut Gaia crowding and aperture control |

## Principal saved inputs

The repository includes compact derived data products so that the statistical and machine-learning analyses can be inspected without committing the full collection of raw and intermediate FITS light curves.

Important tables include:

- `data/sample/VSXMetadata.parquet`
- `data/sample/VSX_TESS_MATCH_Metadata.parquet`
- `data/sample/TESSAugmented_QC.parquet`
- `data/processed/TESSAugmented.parquet`
- `data/processed/TESS_features.parquet`
- `data/processed/SPOC_TESSCut_Metadata_Consolidated_QC.parquet`
- `data/processed/QLP_TESSCut_Metadata_Consolidated_QC.parquet`
- `data/processed/SPOC_TESSCut_features_QC.parquet`
- `data/processed/QLP_TESSCut_features_QC.parquet`

The two `gaia_neighbors.parquet` files preserve the external Gaia DR3 neighborhood-query results used in the crowding controls.

## Saved controlled-analysis results

`results/controlled_analysis/` preserves the split assignments, per-split performance, paired predictions, feature importance, ablation results, per-class results, and run manifests underlying the principal controlled analyses.

The headline repeated-split results are:

| Matched comparison | Official-product mean accuracy | TESSCut mean accuracy | Mean paired gap |
|---|---:|---:|---:|
| SPOC–TESSCut | 0.765 | 0.628 | 0.137 |
| QLP–TESSCut | 0.676 | 0.525 | 0.151 |

Accuracy, balanced accuracy, and macro F1 favor the official product in all 100 evaluated splits for both matched comparisons.

After removing six provenance-associated observational predictors, the mean accuracy gaps are 0.102 for SPOC–TESSCut and 0.153 for QLP–TESSCut.

## Mechanism and sensitivity controls

### Conditional detrending

The detrending experiment applies the fixed conditional drift-detection/correction procedure only to the matched TESSCut representations. The final detrended feature tables are preserved under:

```text
results/detrending/SPOC/TESSCut_conditional_detrended_features.parquet
results/detrending/QLP/TESSCut_conditional_detrended_features.parquet
```

The tested procedure does not narrow either provenance gap on average.

### Sigma clipping

The sigma-clipping experiment applies iterative 5-sigma clipping to the matched TESSCut light curves. It produces small, directionally inconsistent classification changes and does not account for a substantial fraction of the provenance gaps.

### Crowding and aperture diagnostics

The crowding controls use Gaia DR3 neighbor flux as an approximate crowding proxy. The analyses identify some crowding-associated prediction or feature differences, but neither matched sample shows a consistent monotonic increase in classification gap across crowding quartiles.

## Execution environment

The analyses were run with **Python 3.13**. Versions explicitly recorded for key astronomy packages are:

- `astropy 7.2.0`
- `lightkurve 2.5.1`

`requirements.txt` and `environment.yml` list the Python packages required by the notebooks. Exact versions for packages not recorded by the saved analysis are intentionally not fabricated.

## `Project_Root` configuration

The notebooks define a global `Project_Root` from the environment, with the original research location as the fallback:

```python
Project_Root = os.environ.get(
    "Project_Root",
    "/data/projects/TESS-research"
)
```

Some notebooks use `pathlib.Path` around the same value.

### Important note about the publication layout

The notebooks preserve the **original research execution layout** below `Project_Root`, including paths such as:

```text
data_pipeline/
feature_extraction/
ml_analytics/
summary/
detrend_tesscut/
```

For publication, the compact artifacts have been reorganized into the cleaner `data/` and `results/` structure shown above. Therefore, the repository is a reproducibility/archive package of the exact notebooks and derived artifacts, but the reorganized publication tree is **not yet a drop-in replacement for every original notebook path**.

To rerun the notebooks end-to-end without editing their configured paths, reconstruct the original working layout beneath `Project_Root`, or remap the notebook path configuration to the corresponding files under `data/` and `results/`.

The saved result tables allow the principal reported statistical results to be audited without repeating the external catalogue queries or the full raw-light-curve acquisition.

## Network-dependent stages

The data-pipeline notebook keeps the two expensive catalogue stages independently opt-in:

```python
RUN_VSX_CATALOG_QUERY = False
RUN_TESS_MATCH = False
```

They remain disabled by default. Set the corresponding flag to `True` only when intentionally repeating that external query stage.

The crowding notebooks also contain Gaia-query logic. The repository includes the saved Gaia neighborhood-query results used for the reported analysis.

## Raw TESS light curves

The full raw/intermediate FITS collection is not included in this Git repository because of its size. The source TESS data are publicly available through MAST, and the acquisition/extraction procedure is documented in the data-pipeline notebook.

The compact metadata, feature tables, split assignments, predictions, and result summaries required to inspect the reported analyses are included.

## Suggested analysis order

For understanding the workflow, read the notebooks in numerical order.

For reproducing the principal paper results from saved derived data, the most important notebooks are:

1. `03_analyze_comprehensive.ipynb`
2. `04_controlled_analysis_monte_carlo.ipynb`
3. `05_controlled_detrending.ipynb`
4. `06_tesscut_sigma_clipping_controlled.ipynb`
5. `07_spoc_tess_aperture_crowding.ipynb`
6. `08_qlp_tesscut_aperture_crowding.ipynb`

## External data sources

The study uses public third-party data from:

- AAVSO International Variable Star Index (VSX)
- TESS holdings at the Mikulski Archive for Space Telescopes (MAST)
- TESS Input Catalog (TIC)
- SIMBAD
- Gaia DR3

Users of those data should follow the citation and usage requirements of the corresponding services and catalogues.

## Citation

If you use this repository, please cite the associated manuscript. Citation metadata are also provided in `CITATION.cff`.

## License

The original code and repository-authored documentation are released under the MIT License; see `LICENSE`.

Third-party astronomical data retain their original terms, provenance, and citation requirements. The MIT License does not relicense third-party catalogue or mission data.
