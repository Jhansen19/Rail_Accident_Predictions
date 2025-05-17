# Read below for intructions on how to generate the risk indices 

Commodity Risk Data
1. Run Commodity_Data_ETL.ipynb to generate Indiana_Rail_FAF.csv - generating passthrough volume estimates is time intensive (approximately 1 hr)
	CANNOT BE RUN FROM GITHUB WITHOUT THE BELOW DEPENDENCIES
	Two of the dependencies were too large to upload to github, but can be accessed here:
		tl_2024_us_county.zip - https://www2.census.gov/geo/tiger/TIGER2024/COUNTY/ (80MB)
		FAF5.6.1.zip(.csv) - Regoinal Database - https://www.bts.gov/faf (500MB+)
2. Run AAR Analysis-City Estimates to generate Commodity Risk Index output, AAR_Analysis_City_Estimates.xlsx

SVI Risk Data
1. Run Indiana_svi&accidents&rail.ipynb to generate Indiana_SVI_Scale1-10.csv

Merging Commdity and SVI Risk data
1. Run SVI_Freight_Merge.ipynb to generate Merged_SVI_AAR_Data.csv

Generating Rail Segment Accident Predictions
1. Run TrainVolumes.ipynb, data dependencies available in the data folder, outputs train_volumes.parquet to the data folder
2. Run VolumeProjection.ipynb to track_segment_volumes.csv in the data folder
3. Run TrainAccidentPrediction.ipynb, dependent on Indiana_accidents_since_2011_v2.csv, produces annual_accident_probabilities.csv

RiskIndex Generations
1.Run RiskIndex_Generation.ipynb to generate combined_vulnerability_map.html

# Report: Predicting Train Accidents Based on Train Volumes in Indiana

1. # **Introduction**

The goal of this project is to predict train accidents in Indiana by analyzing train traffic volumes and historical accident data. Understanding where accidents are more likely to occur can help improve safety measures and reduce future incidents.

2. # **Decisions and Assumptions Made**

   1. ## **Data Selection**

* Accident Data: Historical records of train accidents in Indiana were used, including details such as date, time, location (latitude and longitude), and accident numbers.  
* Train Volume Data: Data on train track segments in Indiana were utilized, providing information about each segment's location, length, and the estimated number of trains passing through annually.

  2. ## **Data Preparation**

* Spatial Association: Accidents were linked to specific track segments based on geographic locations by performing a spatial join.  
* Time Frame: The analysis considered accidents from various years without focusing on temporal trends.

  3. ## **Assumptions**

* Accurate Locations: It was assumed that the location data for both accidents and track segments were accurate and shared the same coordinate system.  
* Influence Factors: The number of accidents on a track segment was assumed to be influenced by the train volume (annual number of trains) and the length of the segment.  
* Consistency: It was assumed that train volumes remained relatively constant over the analyzed period.

3. # **Problems Faced**

   1. ## **Data Formatting Issues**

* Date and Time Parsing: The accident data had inconsistent date and time formats, including extraneous text like "0:00" in the date column. Cleaning and standardizing this information was necessary to accurately extract the year, month, and day of each accident.

  2. ## **Spatial Mismatches**

* Accident and Track Alignment: Some accidents did not perfectly align with the track segments due to slight inaccuracies in location data. To address this, the accident points were buffered by approximately 50 meters to ensure they matched with the correct track segments.

  3. ## **Model Selection Challenges**

* Overdispersion Check: When using statistical models for count data (such as accident counts), it is important to check for overdispersion (when variance exceeds the mean). This verification was essential to choose the appropriate model.

4. # **How the Model Works to Predict Train Accidents**

   1. ## **Data Aggregation**

1. Linking Accidents to Tracks: A spatial join connected each accident to the corresponding track segment based on geographic location.  
2. Counting Accidents: The number of accidents per track segment per year was calculated.  
3. Merging Data: Accident counts were combined with the train volume data for each track segment.

   2. ## **Statistical Modeling**

* Model Used: A Poisson regression model was chosen, which is suitable for predicting counts of events like accidents.  
* Rationale for Poisson Regression: This model assumes that the event count is influenced by certain predictors (in this case, train volume and track length) and that events occur independently.  
* Predictors Included:  
  * Log of Annual Train Volume: The logarithm of the annual train volume was used to account for the wide range of train counts.  
  * Track Length: Including the length of the track segment accounted for the possibility that longer tracks might experience more accidents simply due to covering more area.

  3. ## **Prediction Process**

* Training the Model: The model was trained using 80% of the aggregated data.  
* Making Predictions: The trained model estimated the expected number of accidents for each track segment based on the predictors.  
* Risk Index: The predicted accident counts serve as a risk index, indicating the likelihood of accidents on each segment.

5. # **Model Verification and Testing**

   1. ## **Evaluating Accuracy**

* Testing Data: The remaining 20% of the data not used for training was used to test the model's predictions.  
* Performance Metrics:  
  * Mean Squared Error (MSE): Measures the average squared difference between predicted and actual accident counts. The MSE was approximately 2.77.  
  * Root Mean Squared Error (RMSE): The square root of MSE, providing an error measure in the same units as the data. The RMSE was about 1.66.  
  * Mean Absolute Error (MAE): The average absolute difference between predicted and actual counts, which was around 0.75.  
  * R-squared Value: Indicates how much of the variance in the data is explained by the model. An R-squared of 0.03 suggests the model explains about 3% of the variability, which is low.

  2. ## **Interpreting the Results**

* Low R-squared: This indicates that while the model captures some relationship between train volume, track length, and accidents, there are likely other important factors not included in the model.  
* Model Fit: The model may not perfectly predict accident counts but provides a foundational assessment for identifying higher-risk areas.

6. # **Ways to Improve the Model**

1. Include More Predictors:  
   * Track Conditions: Incorporating information about track quality, maintenance schedules, and age could improve predictions.  
   * Environmental Factors: Considering weather conditions, such as rain or snow, which might impact accident rates.  
   * Human Factors: Including data on human error, staff training levels, or staffing numbers.  
2. Time-Based Analysis:  
   * Seasonality: Accounting for seasonal trends, such as more accidents occurring in winter months.  
   * Yearly Changes: Incorporating changes in train volume and accident rates over time.  
3. Advanced Modeling Techniques:  
   * Negative Binomial Regression: If overdispersion is significant, this model can handle it better than Poisson regression.  
   * Machine Learning Models: Techniques like Random Forests or Gradient Boosting can capture complex patterns and interactions.  
4. Data Quality Enhancement:  
   * Improved Location Accuracy: Utilizing more precise GPS data for accidents and track segments.  
   * Up-to-Date Train Volumes: Ensuring train volume data reflects the same time periods as the accident data for better alignment.

7. # **Understanding the Risk Index**

* Definition: The risk index is the predicted number of accidents for each track segment, as estimated by the model.  
* Usage:  
  * Visualization: By mapping the risk index, high-risk areas can be visually identified on the map.  
  * Resource Allocation: Helps railway companies prioritize safety measures where they are needed most.  
  * Preventative Actions: Authorities can focus on segments with higher risk for inspections, maintenance, or installing safety features.

8. # **Additional Details**

   1. ## **Visualization with Folium**

* Interactive Maps: The Folium library was used to create interactive maps displaying the track segments colored according to their risk index.  
* Color Scale: A color gradient enhances the ability to quickly identify segments with higher predicted accident counts.  
* Adjustments for Clarity: By increasing the line weight and selecting appropriate color scales, the train lines and high-risk areas are made clearly visible on the map.

  2. ## **Benefits of the Approach**

* Data-Driven Insights: The model provides an evidence-based method to assess risk on train tracks.  
* Scalability: The methodology can be applied to other regions or expanded with additional data.  
* Cost-Effective: Helps in efficiently allocating limited resources by focusing on the most critical areas.

9. # **Conclusion**

By combining historical accident data with train volume information, a model was developed to predict the likelihood of train accidents on different track segments in Indiana. While the model has limitations and room for improvement, it serves as a valuable tool for identifying high-risk areas and enhancing railway safety efforts.

