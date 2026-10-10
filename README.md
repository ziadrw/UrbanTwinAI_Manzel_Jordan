# Menzal منزل UrbanTwin AI PoC

This project contains a Jupyter notebook that demonstrates a reproducible Earth Observation (EO) → AI → urban-planning workflow using Sentinel-2 and Landsat-9 data.

## Installation

A `requirements.txt` file is provided for convenience, but for this project it is **not always the most reliable installation method**.

This notebook depends on the geospatial stack (`GDAL`, `GeoPandas`, `Fiona`, `PyProj`, `Shapely`), and these packages can be difficult to install consistently with plain `pip`, especially across operating systems.

### Recommended approach

Use **Conda/Mamba** with `conda-forge`:

```bash
mamba create -n menzal-poc -c conda-forge \
  python=3.11 gdal geopandas scikit-learn matplotlib pandas numpy jupyterlab notebook ipykernel
mamba activate menzal-poc
```

### Alternative approach

A `requirements.txt` file is included and may work in environments where GDAL and its native dependencies are already available:

```bash
pip install -r requirements.txt
```

If the pip installation fails, prefer the Conda/Mamba setup above.

## 1. Environment and Configuration

Defines the Python libraries, input paths, output folders, study-area settings, clustering parameters, Random Forest settings, and validation configuration.

Main libraries:

- GDAL
- GeoPandas
- NumPy / Pandas
- scikit-learn
- Matplotlib

## 2. Geospatial Helpers

Utility functions for:

- reading and writing GeoTIFFs
- reprojection and resampling
- clipping rasters to the study area
- handling NoData values
- creating the regular planning grid
- rasterizing vector features

## 3. Data Inventory

Summarizes the EO products used:

- **Sentinel-2C Level-2A** for surface reflectance, spectral indices, and clustering.
- **Landsat-9 Collection 2 Level-2** for observed Land Surface Temperature (LST).

The supplied Sentinel-1 scene is excluded because it does not overlap the UAE study area.

## 4. Study Context and Planning Grid

Uses the supplied AOI as a geographic reference and creates a larger study area around it.

A **250 m planning grid** is generated to summarize model outputs into geographic planning units.

## 5. Sentinel-2 Preprocessing

Prepares Sentinel-2 by:

- loading B02, B03, B04, B08, B11, and B12
- converting digital numbers to surface reflectance
- resampling bands to a common 10 m grid
- using the Scene Classification Layer (SCL) to mask clouds, shadows, and invalid pixels

## 6. Feature Engineering

Creates the features used by the models:

- Sentinel-2 reflectance bands
- NDVI
- NDBI
- MNDWI
- BSI

These are treated as spectral features, not confirmed land-cover labels.

## 7. Surface-Regime Clustering

Uses `StandardScaler` and `MiniBatchKMeans` to group Sentinel-2 pixels into recurring spectral surface regimes.

The result is a 10 m cluster map. Since no independent land-cover labels are available, clusters are not treated as validated semantic classes.

## 8. Landsat Observed LST

Loads Landsat `ST_B10`, applies QA masking, reads the product scale and offset, and converts values to degrees Celsius.

The notebook uses the existing Level-2 LST product instead of recalculating LST from raw thermal radiance.

## 9. Visual Context

Displays:

- Sentinel-2 true-color imagery
- observed Landsat LST

This provides a quick visual check of the selected study area.

## 10. Sentinel-2 / Landsat Grid Alignment

Aggregates Sentinel-2 features from 10 m to the Landsat 30 m grid.

This creates a common modeling grid for comparing Sentinel-2 features with Landsat LST.

## 11. Random Forest LST Regression

Trains a `RandomForestRegressor` to predict Landsat LST from Sentinel-2 features.

Validation uses spatial blocks instead of a random pixel split to reduce spatial leakage.

Reported metrics include:

- MAE
- RMSE
- R²
- bias
- train/test sample counts
- train/test spatial-group counts

## 12. Predicted LST and Residuals

Produces:

- observed LST
- predicted LST
- residual map (`observed - predicted`)

Observed and predicted LST use the same temperature scale for direct comparison.

## 13. Surface Regimes and Thermal Behavior

Aggregates cluster fractions to the Landsat grid and identifies which surface regime is most associated with higher observed LST.

## 14. 250 m Urban-Planning Indicators

Summarizes each 250 m grid cell using indicators such as:

- mean and P90 LST
- hot-area fraction
- thermal-pressure percentile
- vegetation-like fraction
- dominant cluster
- predicted LST
- model residual
- heat-associated surface fraction

A prototype planning-priority score combines thermal pressure, vegetation deficit, and heat-associated surface coverage.

## 15. Main Outputs

The main PoC outputs are:

1. NDVI map
2. Sentinel-2 surface-regime clustering map
3. observed LST map
4. predicted LST map
5. LST residual map
6. 250 m planning-priority map

The notebook also exports GeoTIFF, GeoJSON, CSV, model-metric, cluster-profile, and provenance files.

## 16. Data Accuracy and Limitations

Documents:

- Sentinel-2 and Landsat product provenance
- gIQ as the platform used to access the challenge data
- the spatial validation setup
- the lack of independent temperature and land-cover ground truth
- the exclusion of Sentinel-1
- thermal-resolution limitations
- the distinction between LST and human heat exposure
- that the planning-priority score is a PoC heuristic, not a validated risk score
