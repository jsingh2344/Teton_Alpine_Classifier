# Alpine Terrain Random Forest Project Log

## Project Aim

Build a first random forest classifier for alpine terrain in a region that matters locally: the Tetons. The goal is learning-first: understand the workflow, write the code myself, inspect the results critically, and iterate toward a more trustworthy land-cover map.

## Research Inspiration

Primary paper:

- Yang Qichi et al. (2023), "A novel alpine land cover classification strategy based on a deep convolutional neural network and multi-source remote sensing data in Google Earth Engine." https://www.tandfonline.com/doi/full/10.1080/15481603.2023.2233756

Ideas to borrow:

- Use Google Earth Engine for cloud-scale remote sensing workflows.
- Combine multiple data sources rather than relying on optical imagery alone.
- Bootstrap training samples from existing land-cover products, then improve labels over time.
- Evaluate the final map with accuracy metrics and by ecological reasonableness.
- Analyze alpine land-cover patterns along elevation gradients.

Differences in this project:

- Use Random Forest instead of a deep convolutional neural network.
- Start with a small, familiar study area around Grand Teton National Park.
- Use Dynamic World as weak labels for the first pass, then move toward hand-checked labels.

## Current Workflow

Notebook: `random_forest_tetons.ipynb`

First-pass model:

- Study area: rectangular AOI around Grand Teton alpine terrain.
- Season: summer 2023, from June 15 to September 20.
- Predictors: Sentinel-2 bands, NDVI, NDWI, NDSI, BSI, Sentinel-1 VV/VH, elevation, slope, aspect.
- Labels: high-confidence Dynamic World classes.
- Model: `ee.Classifier.smileRandomForest` with 100 trees.
- Evaluation: 70/30 train-validation split, confusion matrix, overall accuracy, kappa.

## Setup Notes

- Local venv exists at `.venv`.
- Installed packages already found: `earthengine-api`, `geemap`, `sklearn`, `pandas`, `numpy`.
- Earth Engine still needs user authentication and possibly a Cloud project in the notebook.
- Global `jupyter` was not found in the shell, so open the notebook through an editor or install/use Jupyter in the venv.

## Key Assumptions To Revisit

- `ALPINE_MIN_ELEV_M = 2200` is only a starter threshold.
- Dynamic World weak labels may be wrong in alpine terrain, especially for sparse vegetation, rock, snow, shadow, and mixed pixels.
- A summer median composite may hide phenology that would help distinguish meadow, shrub, forest, snow, and barren terrain.
- The AOI rectangle includes some non-target terrain; a more precise boundary may improve interpretation.

## Experiment Log

| Date | Run | Change | Validation Accuracy | Notes |
| --- | --- | --- | --- | --- |
| 2026-06-23 | 001 | Created starter GEE Random Forest notebook using Dynamic World weak labels and multi-source predictors. | TBD | Run after Earth Engine authentication. |

## Questions While Learning

- Which classes are actually useful for the Tetons: snow/ice, bare rock, alpine meadow, shrub/scrub, forest, water?
- Should low-elevation forest be included as context, or masked out?
- Do Sentinel-1 VV/VH bands help separate shrub, meadow, and forest here?
- Which predictor bands rank as most important, and does that make ecological sense?
- Where does the model fail visually, and are those failures label problems, predictor problems, or class-definition problems?

## Next Steps

1. Authenticate Earth Engine and run the notebook top to bottom.
2. Save the first confusion matrix and class histogram in this log.
3. Inspect the map near familiar places and write down obvious errors.
4. Adjust the AOI and alpine elevation threshold.
5. Create a small set of hand-labeled validation points or polygons.
6. Compare three models: Sentinel-2 only, Sentinel-2 plus terrain, Sentinel-2 plus Sentinel-1 plus terrain.
