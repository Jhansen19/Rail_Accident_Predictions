# US Freight Rail Risk Analysis

This repository contains all scripts, data processing notebooks, and outputs for predicting train accident risk across Indiana rail segments by combining train volume, historical accident, commodity hazard, and community vulnerability data. The goal is to provide the State of Indiana with actionable maps highlighting high‑risk segments—especially those transporting hazardous goods through vulnerable communities.

You can also review the full project report in the [United States Freight Rail Vulnerabilities.pdf](https://github.com/Jhansen19/Rail_Accident_Predictions/blob/main/United%20States%20Freight%20Rail%20Vulnerabilities.pdf) for more detailed context and findings.

---

## Table of Contents

* [Read below for instructions on how to generate the risk indices](#read-below-for-instructions-on-how-to-generate-the-risk-indices)
* [Report: Predicting Train Accidents Based on Train Volumes in Indiana](#report-predicting-train-accidents-based-on-train-volumes-in-indiana)
* [Introduction](#introduction)
* [Problems Faced](#problems-faced)
* [How the Model Works to Predict Train Accidents](#how-the-model-works-to-predict-train-accidents)
* [Model Verification and Testing](#model-verification-and-testing)
* [Ways to Improve the Model](#ways-to-improve-the-model)
* [Understanding the Risk Index](#understanding-the-risk-index)
* [Additional Details](#additional-details)
* [Conclusion](#conclusion)

---

## Read below for instructions on how to generate the risk indices

### Commodity Risk Data

1. **Run** `Commodity_Data_ETL.ipynb` to produce `Indiana_Rail_FAF.csv`.

   * *Note:* Passthrough volume estimation can take \~1 hour and requires the following dependencies (too large to store here):

     * [tl\_2024\_us\_county.zip (80 MB)](https://www2.census.gov/geo/tiger/TIGER2024/COUNTY/)
     * [FAF5.6.1.zip (500 MB+)](https://www.bts.gov/faf)
2. **Run** `AAR Analysis-City Estimates.ipynb` to generate `AAR_Analysis_City_Estimates.xlsx` (commodity risk indices by city).

### SVI Risk Data

1. **Run** `Indiana_svi&accidents&rail.ipynb` to generate `Indiana_SVI_Scale1-10.csv` (social vulnerability index scaled 1–10).

### Merging Commodity and SVI Risk Data

1. **Run** `SVI_Freight_Merge.ipynb` to produce `Merged_SVI_AAR_Data.csv`.

### Generating Rail Segment Accident Predictions

1. **Run** `TrainVolumes.ipynb` (outputs `train_volumes.parquet`).
2. **Run** `VolumeProjection.ipynb` (outputs `track_segment_volumes.csv`).
3. **Run** `TrainAccidentPrediction.ipynb` (requires `Indiana_Accidents_Since_2011_v2.csv`, outputs `annual_accident_probabilities.csv`).
4. **Run** `RiskIndex_Generation.ipynb` to create the interactive map (`combined_vulnerability_map.html`).

---

## Report: Predicting Train Accidents Based on Train Volumes in Indiana

### Introduction

This project predicts the likelihood of train accidents on Indiana rail segments by integrating:

* Historical accidents (2011–present)
* Annual train volumes (spatially joined to track segments)
* Community vulnerability (census‑tract SVI, income inequality)
* Commodity hazard (ranked 1–10 by toxicity/danger, 10 highest)

Designed for the State of Indiana, our analysis pinpoints segments transporting hazardous materials through the most vulnerable communities, aiding resource allocation and preventative action.

### Problems Faced

* **Data Formatting Issues:** Inconsistent date/time entries required cleaning (e.g., stripping extraneous “0:00”).
* **Spatial Mismatches:** Slight coordinate misalignments fixed by buffering accidents by \~50 m before spatial joins.
* **Model Selection Challenges:** Checked for overdispersion in accident counts to choose Poisson vs. Negative Binomial modeling.

### How the Model Works to Predict Train Accidents

1. **Data Aggregation**

   * Spatial join links accident points to track segments.
   * Accident counts aggregated per segment per year.
   * Data merged with annual train volume, track length, commodity hazard, and census‑tract vulnerability.
2. **Statistical Modeling**

   * **Model:** Poisson regression (suitable for count data).
   * **Predictors:**

     * Log of annual train volume
     * Track segment length
     * Commodity hazard score (1–10)
     * Community vulnerability index (SVI, 1–10)
3. **Prediction Process**

   * Trained on 80% of data.
   * Predicts expected accident count (risk index) for each segment.

### Model Verification and Testing

* **Test Set:** Remaining 20% of data.
* **Performance Metrics:**

  * MSE ≈ 2.77
  * RMSE ≈ 1.66
  * MAE ≈ 0.75
  * R² ≈ 0.03 (indicates additional factors beyond those included)

### Ways to Improve the Model

* **Additional Predictors:** Track condition, maintenance records, weather, human factors.
* **Temporal Analysis:** Seasonality and year‑over‑year volume trends.
* **Advanced Techniques:** Negative Binomial regression, Random Forests, Gradient Boosting.
* **Data Quality:** More precise GPS, up‑to‑date train volume data.

### Understanding the Risk Index

* **Definition:** Predicted accident count per segment.
* **Usage:**

  * **Interactive Maps:** Color‑code segments by risk.
  * **Resource Allocation:** Target high‑risk, high‑vulnerability corridors.
  * **Preventative Actions:** Prioritize inspections and maintenance where needed.

### Additional Details

* **Community Focus:** Combines train capacity, accident probability, commodity hazard, and census‑tract vulnerability to highlight critical areas.
* **Interactive Map:** Built with Folium; shows rail lines overlaid on census tracts, colored by combined vulnerability.

---

## Conclusion

By uniting volume, historical accident, commodity hazard, and community vulnerability data, this analysis provides the State of Indiana with a detailed risk landscape. While the model explains a portion of variability (R² = 0.03), it offers a foundation for targeted safety interventions on the most vulnerable and hazardous rail segments.

