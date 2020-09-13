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

RESULTS: [ ‘measurement’, ‘station]

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



### Purpose  

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


*Difference 1 - 

*Difference 2 - 

*Difference 3 - 

## Summary

high level results and two additional queiries
