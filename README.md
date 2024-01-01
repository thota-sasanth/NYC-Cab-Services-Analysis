# NYC-Cab-Services-Analysis
A Comprehensive Analysis of Cab Services in NYC and Fare Prediction

## Problem Statement & Background

To conduct a comprehensive analysis of Cab services (Yellow taxis and High Volume For-Hire-Vehicles such as Uber, Lyft, etc) in NYC for 2021-23 and find out patterns in the usage of these services. Furthermore, to develop predictive models for accurately estimating trip fares. <br>
<br>

In the bustling metropolis of New York City (NYC), taxi rides serve as a crucial component of urban transportation, embodying the dynamic heartbeat of the city that never sleeps. Understanding the intricate usage patterns of these taxis is a practical necessity for optimizing the efficiency and reliability of the city's transportation network. Various factors, such as weather conditions, events, and demographic shifts, significantly impact the demand and supply of taxi services, making it imperative for transportation planners and stakeholders to discern these influences. Deciphering how these factors interplay with the ebb and flow of taxi rides not only ensures smoother transportation experiences for riders but also lays the foundation for informed decision-making in the ever-evolving landscape of New York City's vibrant urban environment. Additionally, forecasting fare trends becomes a crucial aspect of urban transportation planning. Accurate fare predictions empower both passengers and drivers with valuable information, fostering a more transparent and equitable system. It helps passengers to plan and budget their transportation costs in advance and for cab services to optimize their operations and improve customer satisfaction. 

## Data Collection
I have collected data from the following sources - 
* NYC Trip data (Yellow taxis & High Volume For-Hire-Vehicles (HV-FHV)) from [TLC](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page#:~:text=,NYC%20Taxi%20and%20Limousine) 
* Weather API ([Visual Crossing](https://www.visualcrossing.com/weather-api))<br>
* 2021 ACS 5-year NTA (Neighborhood Tabulation Areas) Level [Data](https://www.nyc.gov/site/planning/planning-level/nyc-population/american-community-survey.page)
* Calendar / Events Data ([American Holidays](https://www.usa.gov/holidays), [Farmers’ Almanac Seasons](https://www.farmersalmanac.com/the-seasons))

The NYC TLC (Taxi & Limousine Commission) is a government agency responsible for regulating and overseeing the taxi and for-hire vehicle industry in New York City. This is the most popular dataset used for cab trip analyses in general. For weather data, I created a script that calls the Visual Crossing Weather API and fetches hourly borough level weather data for a given location within a time range. For Census / Demographics data, I was able to collect NTA level data (as taxi zones are based on NYC DCP’s NTAs). For Calender data, I consolidated the American holidays list and seasons date ranges.

## Data Preprocessing
* __Handling Unnecessary & Missing data__: I cleaned the dataset by dropping unnecessary columns such as dispatching_base_num. I also dropped columns that had more than 60% of the data as NaN, duplicate rows, and rows with null values.
* __Data Imputation & Outlier Handling__: I applied mean/median imputations wherever it was appropriate. I removed outliers such as rows with a trip duration of 0 seconds, speeds > 200 mph, and fare < 0.
* __Feature Engineering__: I applied label encoding and one hot encoding for categorical features such as trip_length_category. I engineered new features such as trip_speed, waiting_time, total_fare, trip_profit, etc.
* __Merging & Filtering__: I filtered data within 2021-23 timelines. I merged taxi data, hourly weather data, and calendar data based on appropriate columns to get a single dataset. I aggregate the results from this dataset and combine them with census data whenever needed to get final outputs.


## Experimental Setup:
Considering that the total data has __0.5 Billion+__ NYC taxi trips (>90 GB size), I experimented with different __BIG DATA__ setups. 
* __Dask + Coiled Servers + GCP Buckets__: Although the Dask framework performs well with Coiled servers (since a compatible ecosystem is being provided by the same company), I encountered reliability issues while loading large datasets, especially with data stored in GCP buckets and Azure Blob Storage.
* __Databricks (with PySpark) + Azure Blob Storage__: I opted for Databricks due to its robust support for the PySpark framework, and initially chose Azure Blob Storage for data storage. However, after extensive testing, I found that Azure Blob Storage was not sufficiently efficient for reading large datasets.
<br>
Finally, I was able to sucessfully work on the data using the following setup -

* __Databricks (with PySpark) + Azure Data Lake Storage__: I configured separate Databricks clusters with distributed and scalable architecture, ranging from 4 to 8 worker nodes with one master node. Each node is powered by Photon acceleration, and delta cache optimization and is equipped with 16GB of RAM and 4 cores.

## Analysis:

### Geospatial Analysis

The below choropleth map shows the distribution of ride pickups (left) and dropoffs (right) across different taxi zones -
<br>

<p align="center">
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/pickup.png" width="400" height="300"> <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/dropoff.png" width="400" height="300">
</p>

<br>
We can see that the spread of pickup and drop-offs are similar which conveys that the majority of cab rides are between a particular subset of taxi zones. The top 5 zones for pickups and dropoffs are the same (LaGuardia Airport, JFK Airport, Crown Heights North, East Village, Bushwick South) but in a different ranking order.

<br>
<br>
The below arc diagram shows the pickup frequency proportions among NYC boroughs - 

<p align="center">
  <br>
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/borough_trips.png" width="500" height="400"> 
</p>

The coloured arcs represent the rides having pickup locations starting from the borough of the same colour & the thickness of the arc represents the number of rides. The size of each borough circle represents the proportion of total rides that happened in NYC. We can see that Manhattan has the highest share (40%) followed by Brooklyn. Staten Island has very less rides which aligns with it being the least populated borough. Also, the figure is roughly symmetrical (horizontally) reflecting a balanced transportation flow between these boroughs.


### Temporal Analysis
The below figures show the distribution of rides across different days of the week and hours of the day - 

<br>

<p align="center">
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/time_of_day_ride_freq.png" width="400" height="300"> <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/time_of_day.png" width="400" height="300">
</p>

<br>

We can see for weekends that there is a high frequency of rides during late nights. The reasons might be due to holidays and the popularity of nightlife events in NYC (Fridays/Saturdays after 6 PM - next day 1 AM). For weekdays, there are significantly more rides during 8AMs and 4PMs (mainly because of people commuting). We can also see that the lowest number of rides occur at 4 AM. It may be because the cab shift change occurs around this time. 

The below line plot illustrates the variability in taxi travel times from Manhattan to JFK Airport - 
<p align="center">
  <br>
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/trip_time_var.png" width="600" height="400"> 
</p>

The median travel time, represented by the blue line, reveals that the longest durations are in the afternoon, likely due to peak traffic, while the shortest is late at night when roads are clearer. There's a wider spread in travel times during the day, suggesting inconsistent trip durations possibly due to fluctuating traffic conditions or diversions. 

### Weather Impact

The below bar chart compares the average fare price per mile for High Volume For-Hire Vehicles (HVFHV) and Yellow Cabs across different weather conditions - 
<p align="center">
  <br>
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/weather_bar.png" width="800" height="400"> 
</p>

It shows that HVFHV consistently charges more per mile than Yellow Cabs in all different weather conditions with the highest average fare per mile for both services occurring during Ice.

The below histogram shows the relationship between the temperature and the number of rides -  
<p align="center">
  <br>
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/temperature_impact.png" width="600" height="400"> 
</p>

There is a noticeable decline in ride counts as temperatures exceed 75 degrees. This suggests that very high temperatures (above 75) and very low temperatures (below 35) may deter people from taking rides.

### Census Impact

The below figures show the correlations between mean travel time in a taxi zone, average vehicles per household, and total rides - 

<br>

<p align="center">
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/mean%20travel%20time.png" width="400" height="300"> <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/vehicles.png" width="400" height="300">
</p>

In both cases, there is a negative correlation conveying that as average commute time increases, the total cab rides tend to decrease. This could suggest that longer travel times may discourage the use of cabs, possibly due to higher costs or longer waiting times, leading commuters to seek alternative modes of transportation. Similarly, as the average number of vehicles per household increases the total number of cab rides decreases, suggesting that households with more access to personal vehicles are less likely to use cabs for transportation.

<br>

Additionally, the below scatterplot shows that a majority of the low average household income zones have lower total number of cab rides compared to their counterparts.

<p align="center">
  <br>
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/income_scatterplot.png" width="700" height="400"> 
</p>

### Calendar / Events Impact

The below sunburst chart shows the trips variation across different seasons - 
<p align="center">
  <br>
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/season.png" width="400" height="400"> 
</p>

There is a significant increase (around 22%) in taxi revenue from 2021 to 2022, which would probably indicate the effect of the COVID-19 pandemic. Apart from that, the highest revenue comes from the Fall season which could be related to various factors such as pleasant weather, the beginning of the school year, etc.

Also, the below chart shows the revenue generated from different days (NYC holidays, weekends, weekdays) - 

<p align="center">
  <br>
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/holiday_trips.png" width="700" height="400"> 
</p>

We can observe that cabs make more revenue during a weekend over weekdays, this shows that people use cabs more frequently on weekends, and less on weekdays. Apart from that, we could also see that fewer people use cabs on holidays (NYC holidays).

### Tipping Behavior

The below packed bubble chart shows the top tipped locations in each NYC borough - 

<p align="center">
  <br>
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/tips.png" width="650" height="650"> 
</p>

The color intensities are derived based on the average tipping percentages in that zone. We can easily see zones that do significantly better within their borough in terms of ride tips (ex. Freshkills Park, Greenwood Cemetery, etc.)


## Modeling & Results
I have trained various models for predicting the base cab fare using ride related features, weather features, holiday, and season features. 
The below are the 3 baseline models I've used -
* Mean Model: The mean ‘base_passenger_fare’ from the training set was used to predict a constant value for the test dataset.
* Linear Regression (LR):  I used linear regression as it is used to model the relationship between a dependent variable (in our case the fare) and one or more independent variables/features by fitting a linear equation to the observed data. This model seemed to perform significantly better when compared to the previous ‘Mean Model’.
* Linear Regression with ElasticNet Regularization: I added ElasticNet regularizer to the Standard LR model in order to overcome some of the limitations such as overfitting and help improve the model’s generalization to the new data. I used a α value of 0.5 setting the strength of both L1 & L2 regularization terms to be equal.

Other advanced models are - 
* Random Forest Regression: Random Forests are good at capturing interactions between different features. They can automatically consider feature interactions without the need for explicit feature engineering. This is an ensemble approach which would often lead to more stable and reliable predictions compared to individual models (such as Decision Trees). It showed slightly better results compared to the previous baseline models.
* Gradient Boosting Regression: Gradient Boosting Regression, especially in the form of algorithms like Gradient Boosted Trees (GBT), tends to provide high predictive accuracy for numerical prediction models. Apart from this, it is robust to overfitting and is an ensemble model that builds decision trees sequentially. It performed similar to Random Forest Regression.
* Custom Ensemble Model: A custom ensemble model made out of the previously defined linear regression, random forest and gradient boosting models was defined. I have assigned weights for each model to be the inverse of their RMSE error on their predictions. This model provides a good amount of improvement on overall predictions.

|                  | RMSE  | MAE  | MAPE (%) | R2                |
| ---------------- | ----- | ---- | -------- | ----------------- |
| Mean Model       | 13.05 | 9.92 | 64.25    | -1.42 x 10^-14    | 
| Linear Regression| 6.40  | 4.16 | 24.73    | 0.76              | 
| Linear Regression + ElasticNet Regularization | 6.45 | 4.18 | 24.94 | 0.75 |
| Linear Regression| 6.38  | 4.07 | 23.64    | 0.76              | 
| Linear Regression| 6.38  | 3.93 | 21.77    | 0.76              | 
| Linear Regression| 6.19  | 3.56 | 20.96    | 0.73              |

<br>
