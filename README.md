# NYC-Cab-Services-Analysis
A Comprehensive Analysis of Cab Services in NYC and Fare Prediction

## Problem Statement & Background

To conduct a comprehensive analysis of Cab services (Yellow taxis and High Volume For-Hire-Vehicles such as Uber, Lyft, etc) in NYC for 2021-23 and find out patterns in the usage of these services. Furthermore, to develop predictive models for accurately estimating trip fares. <br>
<br>

In the bustling metropolis of New York City (NYC), taxi rides serve as a crucial component of urban transportation, embodying the dynamic heartbeat of the city that never sleeps. Understanding the intricate usage patterns of these taxis is a practical necessity for optimizing the efficiency and reliability of the city's transportation network. Various factors, such as weather conditions, events, and demographic shifts, significantly impact the demand and supply of taxi services, making it imperative for transportation planners and stakeholders to discern these influences. Deciphering how these factors interplay with the ebb and flow of taxi rides not only ensures smoother transportation experiences for riders but also lays the foundation for informed decision-making in the ever-evolving landscape of New York City's vibrant urban environment. Additionally, forecasting fare trends becomes a crucial aspect of urban transportation planning. Accurate fare predictions empower both passengers and drivers with valuable information, fostering a more transparent and equitable system. It helps passengers to plan and budget their transportation costs in advance and for cab services to optimize their operations and improve customer satisfaction. 

## Simulated Annealing

In such large search spaces, simulated annealing (a heuristic-based optimization algorithm) could be very
useful in finding a reasonably good solution within a limited time. This algorithm is inspired by the metallurgical practice of
annealing, where metals are heated to a high temperature and then gradually cooled. This controlled
process allows metal atoms to rearrange themselves into a more stable crystalline structure, thus
reducing defects and achieving a lower-energy state. Similarly, simulated annealing in optimization
allows for a deliberate and probabilistic exploration of the solution space, which helps in finding a
globally optimal solution by avoiding being trapped in local optima.

<br>

<p align="center">
  <img src="https://github.com/thota-sasanth/Optimizing-Tourist-Itineraries-Using-Simulated-Annealing/blob/main/SA_algorithm1.png" width="400" height="300"> <img src="https://github.com/thota-sasanth/Optimizing-Tourist-Itineraries-Using-Simulated-Annealing/blob/main/SA_1.png" width="400" height="300">
</p>

<br>

The algorithm starts at a high temperature with a random initial solution and gradually finds an optimal solution after cooling down based on a neighborhood exploration strategy. The model flow chart looks like - 
<br>
<br>
<p align="center">
  <img src="https://github.com/thota-sasanth/Optimizing-Tourist-Itineraries-Using-Simulated-Annealing/blob/main/SA_flowchart.png" width="450" height="700">
</p>

## Data Collection
I have collected data from the following sources - 
* NYC Trip data (Yellow taxis & High Volume For-Hire-Vehicles (HV-FHV)) from [TLC](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page#:~:text=,NYC%20Taxi%20and%20Limousine) 
* Weather API ([Visual Crossing](https://www.visualcrossing.com/weather-api))<br>
* 2021 ACS 5-year NTA (Neighborhood Tabulation Areas) Level [Data](https://www.nyc.gov/site/planning/planning-level/nyc-population/american-community-survey.page)
* Calendar / Events Data ([American Holidays](https://www.usa.gov/holidays), [Farmers’ Almanac Seasons](https://www.farmersalmanac.com/the-seasons))

The NYC TLC (Taxi & Limousine Commission) is a government agency responsible for regulating and overseeing the taxi and for-hire vehicle industry in New York City. This is the most popular dataset used for cab trip analyses in general. For weather data, I created a script that calls the Visual Crossing Weather API and fetches hourly borough level weather data for a given location within a time range. For Census / Demographics data, I was able to collect NTA level data (as taxi zones are based on NYC DCP’s NTAs). For Calender data, I consolidated the American holidays list and seasons date ranges.

## Data Preprocessing
* Handling Unnecessary & Missing data: I cleaned the dataset by dropping unnecessary columns such as dispatching_base_num. I also dropped columns that had more than 60% of the data as NaN, duplicate rows, and rows with null values.
* Data Imputation & Outlier Handling: I applied mean/median imputations wherever it was appropriate. I removed outliers such as rows with a trip duration of 0 seconds, speeds > 200 mph, and fare < 0.
* Feature Engineering: I applied label encoding and one hot encoding for categorical features such as trip_length_category. I engineered new features such as trip_speed, waiting_time, total_fare, trip_profit, etc.
* Merging & Filtering: I filtered data within 2021-23 timelines. I merged taxi data, hourly weather data, and calendar data based on appropriate columns to get a single dataset. I aggregate the results from this dataset and combine them with census data whenever needed to get final outputs.


## Experimental Setup:
Considering that the total data has 0.5 Billion NYC taxi trips (>90 GB size), I experimented with different BIG DATA setups. 
* Dask + Coiled Servers + GCP Buckets: Although the Dask framework performs well with Coiled servers (since a compatible ecosystem is being provided by the same company), I encountered reliability issues while loading large datasets, especially with data stored in GCP buckets and Azure Blob Storage.
* Databricks (with PySpark) + Azure Blob Storage: I opted for Databricks due to its robust support for the PySpark framework, and initially chose Azure Blob Storage for data storage. However, after extensive testing, I found that Azure Blob Storage was not sufficiently efficient for reading large datasets.
Finally, I was able to sucessfully work on the data using the following setup -
Databricks (with PySpark) + Azure Data Lake Storage: I configured separate Databricks clusters with distributed and scalable architecture, ranging from 4 to 8 worker nodes with one master node. Each node is powered by Photon acceleration, and delta cache optimization and is equipped with 16GB of RAM and 4 cores.

## Analysis:

### Geospatial Analysis

The below choropleth map shows the distribution of ride pickups (left) and dropoffs (right) across different taxi zones -
<br>

<p align="center">
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/pickup.png" width="400" height="300"> <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/dropoff.png" width="400" height="300">
</p>

<br>
We can see that the spread of pickup and drop-offs are similar which conveys that the majority of cab rides are between a particular subset of taxi zones. The top 5 zones for pickups and dropoffs are the same (LaGuardia Airport, JFK Airport, Crown Heights North, East Village, Bushwick South) but in a different ranking order.

The below arc diagram shows the pickup frequency proportions among NYC boroughs - 

<p align="center">
  <br>
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/borough_trips.png" width="800" height="400"> 
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
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/trip_time_var.png" width="800" height="400"> 
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
  <img src="https://github.com/thota-sasanth/NYC-Cab-Services-Analysis/blob/main/temperature_impact.png" width="800" height="400"> 
</p>

### Census Impact
### season/holiday/calnedar/event Impact


## Results
The below plots show the results for the best run for closed TSP scenario for 15 tourist attractions - 

<p align="center">
  Initial Solution having a total distance of 36,294.76 kms
  <br>
  <img src="https://github.com/thota-sasanth/Optimizing-Tourist-Itineraries-Using-Simulated-Annealing/blob/main/initial_sol.png" width="800" height="400"> 
</p>
<p align="center">
  Final Solution having a total distance of 11,696.17 kms (around 68% reduction) 
  <br>
  <img src="https://github.com/thota-sasanth/Optimizing-Tourist-Itineraries-Using-Simulated-Annealing/blob/main/SA_sol.png" width="800" height="400">
</p>

The below plot shows how the total travel distance change w.r.t change in temperature - 

<p align="center">
  Starting Temperature 10000, 
  Initial Distance 49,585.69 kms <br>
  Final Distance 19,561.03 kms,  
  Reduction 60.55% <br>
  
  <br>
  <img src="https://github.com/thota-sasanth/Optimizing-Tourist-Itineraries-Using-Simulated-Annealing/blob/main/Temperature_variation.png" width="550" height="400"> 
</p>

Overall, the application of simulated annealing to itinerary optimization has proven to be a robust
approach, capable of handling the complexity of the TSP with a high degree of efficiency. This project could be used as a valuable tool for tourists exploring
a new country or anyone seeking assistance in crafting personalized and efficient travel itineraries.
<br>
<br>

