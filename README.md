# GEO_CROPMAPPING_CHALLENGE
This project contains the notebooks used for the zindi crop_challenge

#NOTEBOOKS
## CODE NOTEBOOKS RUN ON LOCAL MACHINE
Due to colab limitations, the following notebooks were run on the local machine

1. Train_data_retrieval.ipynb
- The notebook retrieved sentinel 1 and 2 data for the train data. Due to google colab limits, the notebook was run on my local computer. The data is automatically saved on google drive. You can then download from the google drive to merge the data chunks.
- This file takes aapproximately 15 minutes to run
EXPECTED OUTPUT : [EE_EXPORTS_S1,EE_EXPORTS_S2] - two folders containing chunked data

2. Train_Data_Chunk_Merge.ipynb
- After downloading the train data folders from google drive, this notebook is used to combine the chunks into one csv file.
- runtime : less than 1 minute
EXPECTED OUTPUT : [S1_train.csv, s2_train.csv]

3. S1_S2_Data_Merge_(TRAIN&TEST).ipynb
- This notebook combines s1 and s2 data for the train and test data.
- the output data is aggregate at ID-month level
- runtime : less than 2 minutes
EXPECTED OUTPUT : [train_data_combined.csv,test_data_combined.csv]

We then upload the datasets from notebook 3 to google colab for use on the modelling notebook

## CODE NOTEBOOKS RUN ON GOOGLE COLAB
4. Model_tuning.ipynb
- This notebook ingests train_data_combined and test_data combined, performs exploratory data analysis, preprocessing and feature engineering.
The processed datasets are then train on different models and hyperparameters and the best models saved for submission file generation.
- runtime : Approximately 50 minutes
EXPECTED OUPUT : Best pkl model files and a metadata json file containing model paths and model details

5. best_models.ipynb
The file loads the metadata json and pkl files for the best models per region and uses this to generate the test data predictions.
- runtime : less than 1 minute
EXPECTED OUTPUT : final_submission.csv
