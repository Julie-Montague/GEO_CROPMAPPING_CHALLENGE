# GEO_CROPMAPPING_CHALLENGE
This project contains the notebooks used for the zindi crop_challenge

## REPO STRUCTURE

## How to run the code
|----data/
|       |--- test_data.csv
|       |--- train_data.csv
|
|----submissions/ (models predictions on the test data)
|---- requirements.txt
|---- Readme.md
|--- {notebook}.ipynb

## OVERVIEW
Advances in machine learning and artificial intelligence offer the potential to significantly improve cropland and land cover classification by leveraging time-series satellite imagery.
This challenge focuses on arid and semi-arid regions, where the primary difficulty lies in distinguishing cropland from pastures and steppe land. Participants are tasked with designing and evaluating methods for cropland mapping in two test regions:
   -Fergana, Uzbekistan – a densely cultivated region with complex irrigation patterns.
   -Orenburg, Russia – a semi-arid landscape where cropland is interspersed with steppe and pasture.

## OBJECTIVES
 1. Accuracy: Develop robust models that improve cropland classification compared to existing global products.

 2. Cost-effectiveness: Design methods that balance performance with computational efficiency and scalability.

 3. Regional Adaptability: Tackle the unique challenges of arid and semi-arid landscapes where cropland is hard to distinguish from pastures and steppe.

 4. Global Impact: Contribute to better agricultural monitoring and food security by enhancing global cropland mapping initiatives.

## ARCHITECTURE DIAGRAM

## ETL PROCESS
** Assumptions**
- Cropland exhibits distinct seasonal amplitude and timing, even when absolute greenness is modest in drylands.
- Training and test data come from related but not identical distributions (region/seasonal shift); mitigate with region-aware validation and simple domain controls.
- 
### Extract
 1. Train data
- The provided shp files were used to extract sentinel 1 and 2 data from google earth engine (s1 : COPERNICUS/S1_GRD, s2: COPERNICUS/S2_SR_HARMONIZED).
- The train data was extracted using the dates available on the test data
- Due to memory limitations, the data was retrieved in chunks and the netebook saves the files directly to google drive

 2. Test Data
- As the provided geolocations were masked, the test data used relied on the provided sentinel 1 and 2 data.

Sentinel 1 data : focuses on radar data
Sentinel 2 data: high spatial resolution optical mission data

  ### Transformation
  #### Cleansing
  - Drop duplicates
  - Drop coordinate columns as these columns are masked on the test data thus may not bave relevant modelling information
  - 
  #### Aggregation
  s1_columns : [['VH', 'VV','orbit', 'polarization', 'rel_orbit', 'region', 'month']
  s2_columns :
        i.) band_cols = ['B11','B12','B2','B3','B4','B5','B6','B7','B8','B8A']
        ii.) meta_cols = ['cloud_pct','solar_azimuth','solar_zenith']
  - The extracted and provided data was at a daily granular level. This was aggregated to ID-Region-Month level.
  - For S1 data, the following aggregations were taken :
        - {'VH':'mean','VV':'mean',
            'orbit':lambda x: x.mode()[0],
            'polarization': lambda x: x.mode()[0],
            'rel_orbit': lambda x: x.mode()[0]}}
  - S2 data was aggregated as follows:
        -{**{b: 'median' for b in band_cols},
           'cloud_pct': 'median',
           'solar_azimuth': 'median',
           'solar_zenith': 'median'}
- The aggregated s1 and s2 data are then combined for both the train and test data.

#### Pre-Processing
- clip outliers using the 25 and 75 quantile values
- Apply quantile mapping to reduce covariate shift between the train and test data
- Apply rebust scaling
- Encode region column

#### Feature Engineering
- Develop vegetation indices (calculated using the band columns
- Add targeted seasonal features
- Create ratio and difference features using s1 data
- The data table is transformed from a long format (ID-Region-Month) to a wide format (ID-Region). Thi creates one row per ID with columns at time series monthly level
- Indices like TVI (Triangular Vegetation Index) and TGI (Triangular Greenness Index) have wide ranges thus we clip to reduce the noise level.
- Run a correlation check to eliminate correlation features. This reduces the dataframe size from 713 to 514

### Load
The final datasets are saved as csv files :
 1. train_data.csv
 2. test_data,csv

## Data Modelling
Task : Binary classification: distinguish cropland from pasture/steppe in arid & semi-arid settings.
- Due to the difference in region characteristics, each region was modelled separately
- Pipeline steps included :
    1. Imputer - SimpleImputer(mean)
    2. Sampler - RandomOverSample (Applied on Orenburg region only)
    3. Scaler - StandardScaler
    4. Feature_selection - use selectKBest(k=60)
- Models used (at random state 42):
    1. Random forest
    2. Logistic Regression
    3. LighGBM
    4. XGBoost
       
  ### Training Process
  - StratifiedKFold used for validation
  - No tuning : The data was first trained on fixed models with default parameters and the best models with the highest out-of-fold accuracy were saved
  - Hypeparameter tuning : using randomsearchCV on declared parameter spaces
  - Thresholding : default threshold at 0.5
  - evaluation metric:mean accuracy score, regional accuracy scores
  - Model selection: pick by primary metric on OOF/val
  - complexity control : selected features capped at 60
  - output feature importance visual
  - A metadata json file is also saved shaowing the models chosen, features used, saved model path and accuracy scores
  VERSIONING : the models and metadata folder is saved using the timestamp of model run

### Final Model inference
  - The best modelling pipeline (from both untuned and tuning stage) are retrained on the whole train data set.
  - The submission files are saved as csv files.
  - The submission file from the pipeline with the highest out-of-fold-accuracy(either from the no tuning stage or the tuning stage) is the submitted
