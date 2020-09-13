# Surfs Up Challenge

## Overview of Project

Our primary customer for this project is W. Avy, an investor for our Surf ’n Shake shop in Hawaii.  W. Avy has invested in a surf shop earlier in his career, and wants to make sure that we are thinking through all details of our plan.  He is particularly interested in the weather, as he overlooked this element with his surf shop and it failed because of excessive rain.  W. Avy has tasked us with conducting weather analysis of Oahu and to include this analysis in our final report.  As part of our previous analysis, we have already looked at precipitation for the past year, the number of reporting stations across the island, max/min/avg rainfall for individual reporting station IDs and temperature observation for each station.  For our current analysis, W. Avy wants to look more specifically at temperature data for the months of June and December in Oahu to see if surf and ice cream business is sustainable year round.  

Using a combination of SQLite, SQLAlchemy, Python, and Flask, W. Avy has asked us to provide analysis based on a SQLite file containing information on [Measurements and Stations](https://github.com/MaureenFromuth/surfs_up/blob/master/hawaii.sqlite).  To conduct analysis of the file, we utilized SQLAlchemy to create an engine to read the SQLite data for Hawaii, take the existing database within the SQLite data and put into a new base/model, and then see what classes/tables that model found within our data.  Finally, to make querying of those classes/tables easier, we create a reference or variable for each:  one for ‘measurement’ and one for ‘station’.  

```
# Create an engine to connect to the data in SQLite
engine = create_engine("sqlite:///hawaii.sqlite")

# Reflect an existing database into a new model
Base = automap_base()

# Reflect the tables
Base.prepare(engine, reflect=True)

# We can view all of the classes that automap found
Base.classes.keys()

RESULTS: [‘measurement’, ‘station]

# Save references to each table
Measurement = Base.classes.measurement
Station = Base.classes.station

# Create our session (link) from Python to the DB
session = Session(engine)
```

Of note, we used Pandas to query SQL to get the structure of the two tables, Measurement and Station.  This information will be helpful in understanding all of the data available for this task and any follow-on analysis or queries.  We used the code below to identify the columns in each table.  

```
# Connect the engine above 
conn = engine.connect()  

# Create a dataframe out of your query for measurement
table_measurement = pd.read_sql('SELECT * FROM measurement', conn)
table_measurement.head()

# Create a dataframe out of your query for station
table_station = pd.read_sql('SELECT * FROM station', conn)
table_station.head()
```

Using the results from above, the measurement data contains the following information:
- id (Integer, Key Value)
- station (String, Foreign Key <- station.station)
- date (Date)
- prcp (Integer)
- tobs (Integer)

Using the results from above, the station data contains the following information:
- id (Integer, Key Value)
- station (String)
- name (String)
- latitude (Integer)
- longitude (Integer)
- elevation (Integer)



## Results

Overall the temperatures for Oahu in June and December were very comparable with similar mean temperatures as well as standard deviations and percentiles, but December, not surprisingly, did have a lower min temperature compared to June.  Specifics for each month are depicted below as well as three key differences between June and December.  


In order to extract and analyze temperatures for June and December, we first needed to import another function from SQLAlchemy called extract.  In doing so, we were then able to query the measurement table, and more specifically the tobs (temperature observation) and date columns within that table.  We specifically used the date column to filter out any rows that did not have the month ’06’ within the date using func.strftime function.  Once we returned only the rows associated with June measurements, we used list comprehension to turn our data into a list.  We used the list and Panda’s pd.DataFrame function to create a dataframe with two columns, “Date” and “June Temps”, setting “Date” as the index and sorting by the index.  Finally, using Python’s ‘describe’ function, we were able to generate a statistical summary for the June temperatures in Oahu, as depicted in the graphic below.

```
# 1. Import the sqlalchemy extract function.
from sqlalchemy import extract

# 2. Write a query that filters the Measurement table to retrieve the temperatures for the month of June. 
june_temps = session.query(Measurement.date, Measurement.tobs).filter(func.strftime("%m", Measurement.date) == "06").all()

#  3. Convert the June temperatures to a list.
june_temp_list = [r for r in june_temps]

# 4. Create a DataFrame from the list of temperatures for the month of June. 
june_temp_df = pd.DataFrame(june_temp_list, columns=['Date', 'June Temps'])
june_temp_df.set_index(june_temp_df['Date'], inplace=True)
june_temp_df = june_temp_df.sort_index() 

# 5. Calculate and print out the summary statistics for the June temperature DataFrame.
june_temp_df.describe()
```

>**Descriptive Statistics for Temperatures in June in Oahu*

![Descriptive Statistics for Temperatures in June in Oahu](https://github.com/MaureenFromuth/surfs_up/blob/master/Descriptive%20Statistics%20for%20Temperatures%20in%20June%20in%20Oahu.png)


We conducted the same approach to identify the summary statistics for December, changing the func.strftime function to filter out all dates except those with months “12”, and creating the column names as “December Temps” instead of “June Temps”.   Finally, using Python’s ‘describe’ function, we were able to generate a statistical summary for the December temperatures in Oahu, as depicted in the graphic below.

```
# 6. Write a query that filters the Measurement table to retrieve the temperatures for the month of December.
dec_temps = session.query(Measurement.date, Measurement.tobs).filter(func.strftime("%m", Measurement.date) == "12").all()

# 7. Convert the December temperatures to a list.
dec_temp_list = [r for r in dec_temps]

# 8. Create a DataFrame from the list of temperatures for the month of December. 
dec_temp_df = pd.DataFrame(dec_temp_list, columns=['Date', 'December Temps'])
dec_temp_df.set_index(dec_temp_df['Date'], inplace=True)
dec_temp_df = dec_temp_df.sort_index() 

# 9. Calculate and print out the summary statistics for the Decemeber temperature DataFrame.
dec_temp_df.describe()
```

>**Descriptive Statistics for Temperatures in December in Oahu*

![Descriptive Statistics for Temperatures in December in Oahu](https://github.com/MaureenFromuth/surfs_up/blob/master/Descriptive%20Statistics%20for%20Temperatures%20in%20June%20in%20Oahu.png)


*Difference 1 - The Minimum Temperature in December is 8 Degrees Lower Than June

Perhaps the most notable different between June and December temperatures is the minimum temperature for each month, with June being 64 degrees and December being 56 degrees.  This is not incredibly surprising considering the time of year, but the relatively small difference compared to other states in the US does highlight the overall steady temperature despite the time of year.  


*Difference 2 - The Mean Temperature in December and June are 3 Degrees Apart

While this is a relatively small difference between the two months, the mean temperature in December is about 3 degrees lower than that of June.  Of particular note, if you look at the standard deviations of both June and December, they are between 3 and 4, and the quartiles also line up very similarly with December being about 3 degrees less than June.  These numbers indicate the consistency of the temperatures around low to mid 70’s regardless of the month.  Furthermore, it reinforces the point above that, especially when compared to other states, Hawaii’s temperatures are steady despite the time of year.  


*Difference 3 - There are Approximately 200 More Observable Data Points for June Compared to December 

June has 1700 data points as compared to December in which there is 1517 data points.  This is noticeable considering the fact that December has one more calendar day than June does.  The differences in observable data points for temperature may not have any impact on the outcome of the summary statistics, but it may depending upon the cause of the difference.  For example, why aren’t the stations collecting temperatures at a regular rate - does temperature or precipitation have any impact on their ability to collect?  If it does, than this could indicate additional issues with the December data collection and thus the summary statistics.  Additional queries breaking out day of the month and year-over-year collection per day analysis should be done to determine the cause of the difference.  



## Summary

Overall, the temperatures between June and December are comparable (mean temp of this 74.9 vs. 71.0, respectively) with the exception of the minimum temperature.  The minimum temperature differences, however, are still relatively close (only 8 degrees apart), and further the assessment that the temperatures in Oahu stay fairly consistent year round.  

There is additional analysis and queries that can be done to better understand the reason for the differences in collected temperatures in December and June.  More specifically, we should use a groupby function against both December and June dataframe for the year and also the day of the month.  We could see if there are any trends that are noticeable within collection.  

```
june_temp_df[['year','month','day']] = june_temp_df['Date'].str.split('-',expand=True).apply(pd.to_numeric)
june_temp_df= june_temp_df.groupby('year').count()['Date']

dec_temp_df[['year','month','day']] = dec_temp_df['Date'].str.split('-',expand=True).apply(pd.to_numeric)
dec_temp_df= dec_temp_df.groupby('year').count()['Date']
```

Using this code and plotting it on a bar chart, you can see that a major reason for the difference is a lack of data from 2017 for the month of December.  June recorded approximately 200 collections in 2017, which is likely responsible for the difference in data points between the two months.  The lack of collection in December 2017 may be due to the date the data was delivered, i.e. they delivered it sometime between the end of June 2017 and the beginning of December 2017 and therefore the data was not collected/provided.


Another query you can do is group by day of the month rather than year.  In doing so, you can see if there are any major differences in the daily counts between the two months.  Of note, because we already know that December 2017 wasn’t recorded in this dataset, we will like see about 10-15 collections less per day in December than in June.   


```
june_temp_df_day= june_temp_df.groupby(‘day’).count()['Date']

dec_temp_df_day= dec_temp_df.groupby(‘day’).count()['Date']
```

Using this code and plotting the results on a bar chart, you immediately see the differences in daily collect of about 10-15 as we anticipated.  A unique trend, however, is clearly visible on 25 December, a federal holiday, with a noticeable decrease in collected temperatures as compared to the other days.  Seeing this leads to questions about how the temperature is collected - is it manually done or done automatically.  If it is done manually, do the federal workers have a day off and that’s why the daily collect is lower?  These questions may not be answered by the existing data, but does lead to follow-on questions for the data owners. 
