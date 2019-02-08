# IBM Applied Data Science Capstone

## Introduction
In the current real estate market, it’s estimated that almost half of the current housing market are millennials with 90% of those being renters. ([1](https://www.appfolio.com/blog/2014/08/10-things-you-need-to-know-about-the-millennial-renter-infographic)) When it comes to renting or buying a house, there are many factors that need to be considered before a decision can be taken. These issues are compounded when one has only online data and reviews from other people - reviews which might not reflect an entirely unbiased and objective evaluation and further complicated if the move is from a different city, state or even country.

Two factors that have a significant influence are the location of essential services and businesses in the vicinity of the property and the general safety of the neighborhood. Typically, websites listing properties for rent only display features confined to the property itself - rental price, carpet area, number of bedrooms, bathrooms, unit ameneties, etc. While these are primary motivators for renters, any additional information about the neighborhood and its offerings can only be learned by searching mutiple websites. Again, this information would likely still be missing safety information, and is typically restricted to intuition-based summaries and anecdotes. 

This project attempts to unify the two major secondary motivators - neighborhood amenities and safety information to provide a potential renter with all the pertinent tools to make an informed decision. The focuss of this project is on the city of Los Angeles, California. The Foursquare API will be our primary source for the former, to obtain a list of popular venues in every neighborhood in a city including restaurants, bars, and entertainment venues. The safety information will be obtained through crime records from open sourced data provided by the city. 

The neighborhood information will be displayed on an interactive map overlayed with recorded criminal activity around the venues. The integration of rental property listings as well as public transit information will be pursued as future avenues of work.

## Data
There are two separate datasets that we need to separately acquire and process before we can merge them to achieve our goal. The first will be the crime records from the Los Angeles Police Department (LAPD) for all the areas under its jurisdiction. The second will be the venue information for each neighborhood which can will be obtained via the Foursquare API.

### Crime Data
The LAPD administers 21 Geographic Areas within the department. The location of each precinct, the division or area name and the location of each Community Police Station was obtained from the [Los Angeles Open Data catalog](http://data-lahub.opendata.arcgis.com/datasets/031d488e158144d0b3aecaa9c888b7b3_0). The JSON object was parsed to extract the relevant information. 

The crime data for the areas administered by the Los Angeles Police Department was obtained from the [Los Angeles Open Data catalog](https://data.lacity.org/A-Safe-City/Crime-Data-from-2010-to-Present/y8tr-7khq). The dataset contains crime data dating back from 2010 to January 2019. This data is transcribed from original crime reports.

The format of the dataset is shown below. The full dataset contained approximately 1.9 million rows. To keep computing costs relatively low, only the crimes from 2018 were chosen. This reduced the working dataset to approximtely 225000 rows.

| Column Name | Description | Type |
|-------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|
| DR Number | Division of Records Number: Official file number made up of a 2 digit year, area ID, and 5 digits | Plain Text |
| Date Reported | MM/DD/YYYY | Date & Time |
| Date Occurred | MM/DD/YYYY | Date & Time |
| Time Occurred | In 24 hour military time | Plain Text |
| Area ID | The LAPD has 21 Community Police Stations referred to as Geographic Areas within the department. These Geographic Areas are sequentially numbered from 1-21 | Plain Text |
| Area Name | The 21 Geographic Areas or Patrol Divisions are also given a name designation that references a landmark or the surrounding community that it is responsible for | Plain Text |
| Reporting District | A four-digit code that represents a sub-area within a Geographic Area. All crime records reference the "RD" that it occurred in for statistical comparisons | Plain Text |
| Crime Code | Indicates the crime committed (Same as Crime Code 1) | Plain Text |
| Crime Code Description | Defines the Crime Code provided | Plain Text |
| MO Codes | Modus Operandi: Activities associated with the suspect in commission of the crime | Plain Text |
| Victim Age | Two character numeric | Plain Text |
| Victim Sex | F - Female M - Male X - Unknown | Plain Text |
| Victim Descent | Descent Code: A - Other Asian B - Black C - Chinese D - Cambodian F - Filipino G - Guamanian H - Hispanic/Latin/Mexican I - American Indian/Alaskan Native J - Japanese K - Korean L - Laotian O - Other P - Pacific Islander S - Samoan U - Hawaiian V - Vietnamese W - White X - Unknown Z - Asian Indian | Plain Text |
| Premise Code | The type of structure, vehicle, or location where the crime took place | Plain Text |
| Premise Description | Defines the Premise Code provided | Plain Text |
| Weapon Used Code | The type of weapon used in the crime | Plain Text |
| Weapon Used Description | Defines the Weapon Used Code provided | Plain Text |
| Status Code | Status of the case (IC is the default) | Plain Text |
| Status Description | Defines the Status Code provided | Plain Text |
| Crime Code 1 | Indicates the crime committed. Crime Code 1 is the primary and most serious one. Crime Code 2, 3, and 4 are respectively less serious offenses. Lower crime class numbers are more serious | Plain Text |
| Crime Code 2 | May contain a code for an additional crime, less serious than Crime Code 1 | Plain Text |
| Crime Code 3 | May contain a code for an additional crime, less serious than Crime Code 1 | Plain Text |
| Crime Code 4 | May contain a code for an additional crime, less serious than Crime Code 1 | Plain Text |
| Address | Street address of crime incident rounded to the nearest hundred block to maintain anonymity | Plain Text |
| Cross Street | Cross Street of rounded Address | Plain Text |
| Location | The location where the crime incident occurred. Actual address is omitted for confidentiality. XY coordinates reflect the nearest 100 block | Location |

### Cleaning the LAPD Crime Dataset

First, only the pertinent columns from the dataset were read into a dataframe. These columns were:
- DR Number
- Date Occurred
- Time Occurred
- Area ID
- Area Name
- Crime Code Description
- Location

The next step was to clean the data. This involved:
- Removing whitespace from column names
- Converting column names into lower case
- Splitting location data into latitude and longitude
- Dropping rows that contained NaN
- Typecasting columns
- Extracting additional information from Time and Date Occurred such as hour, minute, day of the week, date, month
- Finally, extracting only the crime records corresponding to the year 2018

### Visualization

We can now visualize the crimes that were committed in 2018 based on various parameters such as number of crimes per:

- Type of crime
The top 25 crimes from 2018 were plotted on a bar chart. It was not very surprising to see there were a lot of theft and assault related crimes. The large number of Identity Theft cases was a surprise - perhaps we can thank Equifax for that?

![](images/6_crime_code.png)

- Per month
The crimes from 2018 were plotted on a bar chart as number of crimes per month. The number of crimes stayed consistent with a dip in February and an increase in the summer months. This can be attributed to a lower number of days in that month.

![](images/7_month.png)

- Per day of the week
The crimes from 2018 were plotted on a bar chart as number of crimes per day of the week. The number of crimes stayed consistent with a slight increase on Fridays (day 5).

![](images/8_day_of_week.png)

- Per day of the month
The crimes from 2018 were plotted on a bar chart as number of crimes per day of the month. The number of crimes stayed consistent throughout the month typically. The drop in number of crimes on the 31st is because not every month has 31 days. Similarly with reduced contributions from February (29th and 30th). It could potentially be an issue and the data would likely need a weighted average of some kind to scale it uniformly. 

The spike in crimes on the 1st day of the month is surprising. It could perhaps be attributed to pay days and the need for money to cover bills. Also, a careful look at the data revealed a significant percentage of the crimes on the 1st was on 1/1. NYE sees a lot of revellers out late and it could be a big driver in crimes on that date.

![](images/9_day_of_month.png)

- Per time of day
The crimes from 2018 were plotted on a bar chart as number of crimes per time of day. The drop after midnight is not surprising. The spike at noon is though while the spikes later in the evening are as expected.

![](images/10_time_of_day.png)

- Per area (name)
The crimes from 2018 were plotted on a bar chart as number of crimes per area of occurrence. It is clear which areas are the most dangerous - 77th Street, Central and Southwest are the best neighborhoods in the city either. 

![](images/11_area.png)

### More Cool Visualizations
The next section has some cool visualiztions that show the crime distributions in the city using choropleths, heat maps and cluster maps.

![](images/1_la_map.png)
![](images/2_la_cluster_map.png)
![](images/3_la_choropleth_map.png)
![](images/4_la_heat_map.png)
![](images/5_la_crime_by_date.png)

### Crime Data Modeling
The 2018 crime data frame was modeled using a Random Forest algorithm. The classification of crimes was based on location, time and date.
- The first step was to classify dangerous crimes such as assault and theft related crimes and assign them a value 1. All other crimes were classified with a 0.
- The next step was applying One-Hot Encoding on the hour of day, day of month, month, latitude and longitude where the crime occurred.

The RF algorithm was chosen to model and classify the dataset. This algorithm searches for the best feature among a random subset of features while splitting a node. Another important advantage of this algorithm is the ability to visualize the relative importance of different features. The dataset was split into train and test datasets. The hyperparameter *n_estimators* was tuned over a range of values from 5 to 500. The accuracy looked to be topping out at *n = 200*. The other hyperparameters were left to be at their default values.

![](images/12_rf_eval.png)

The important features were visualized and as expected, it is the location (latitude & longitude) feature which dominates over any other feature. 

![](images/13_rf_features.png)
