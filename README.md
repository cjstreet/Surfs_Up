# Surfs_Up
SQLite, SQLAlchemy, and Flask to use SQL database structures and querying methods.  Visualizations in python using jupyter NB.


## Overview:
Prepare and analyze the Hawaii dataset to describe key differences in weather between June and December in Oahu. Give recommendations for further analysis and determine if a surf business is sustainable year-round.

## Results: 
- The average temperature in Oahu in June is 75 degrees fahrenheit.
- The average temperature in Oahu in December is 71 degrees fahrenheit.
- The temperatures for both months flunctuate around 3%
- The minimum temperature in December is about 8 degrees less than in June. 
- The maximum temperature in June is 85 and December 83.

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>June Temperature</th>
      <th>December Temperature</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1700.000000</td>
       <td>1517.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>74.944118</td>
       <td>71.041529</td>
    </tr>
    <tr>
      <th>std</th>
      <td>3.257417</td>
       <td>3.745920</td>
    </tr>
    <tr>
      <th>min</th>
      <td>64.000000</td>
      <td>56.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>73.000000</td>
      <td>69.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>75.000000</td>
      <td>71.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>77.000000</td>
      <td>74.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>85.000000</td>
      <td>83.000000</td>
    </tr>
  </tbody>
</table>
</div>

## Summary:

The temperatures in both June and December are very similar. The average temperatures for December is only 4 degrees less than June. December's min temperature is 56 and max is 81, so the range is 29. June's min temperature is 64 and max is 85, so the range is 21. These temperatures and ranges are pretty close. For both the months the temperature only flunctuates around plus or minus 3 degrees. Again, the temperatures in both June and December are very similar and they are great temperatures for surfing and ice cream. So, I would recommend Oahu as a good place for surfing and ice cream.

Further analysis should be done on precipitation for June and December. A summary of precipitation statistics to compare for these two months is recommended. In addition, wind speed might be a factor in whether there are waves to surf or not. A summary of wind statistics for these two month is also recommended. The added information will help to provide a better picture of weather as a whole durings these months. 

## Notebook:

```python
# Dependencies
import numpy as np

# Python SQL toolkit and Object Relational Mapper
import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func, inspect
import datetime as dt
import pandas as pd
```

```python
engine = create_engine("sqlite:///hawaii.sqlite")

# reflect an existing database into a new model
Base = automap_base()
# reflect the tables
Base.prepare(engine, reflect=True)

# Save references to each table
Measurement = Base.classes.measurement
Station = Base.classes.station
```


```python
# Create our session (link) from Python to the DB
session = Session(engine)
```

## Explore Database


```python
# Use the Inspector to explore the database and print the table names
inspector = inspect(engine)
inspector.get_table_names()
```

    ['measurement', 'station']

```python
# Use Inspector to print the column names and types
columns = inspector.get_columns('measurement')
for c in columns:
    print(c['name'], c["type"])
```

    id INTEGER
    station TEXT
    date TEXT
    prcp FLOAT
    tobs FLOAT



```python
# Use `engine.execute` to select and display the first 10 rows from the measurement table
engine.execute('SELECT * FROM measurement LIMIT 10').fetchall()
```

    [(1, 'USC00519397', '2010-01-01', 0.08, 65.0),
     (2, 'USC00519397', '2010-01-02', 0.0, 63.0),
     (3, 'USC00519397', '2010-01-03', 0.0, 74.0),
     (4, 'USC00519397', '2010-01-04', 0.0, 76.0),
     (5, 'USC00519397', '2010-01-06', None, 73.0),
     (6, 'USC00519397', '2010-01-07', 0.06, 70.0),
     (7, 'USC00519397', '2010-01-08', 0.0, 64.0),
     (8, 'USC00519397', '2010-01-09', 0.0, 68.0),
     (9, 'USC00519397', '2010-01-10', 0.0, 73.0),
     (10, 'USC00519397', '2010-01-11', 0.01, 64.0)]



## D1: Determine the Summary Statistics for June


```python
# Import the sqlalchemy extract function.
from sqlalchemy import extract

# 1. Write a query that filters the Measurement table to retrieve the temperatures for the month of June. 
sel = [Measurement.date, 
       Measurement.tobs]
june_temp = session.query(*sel).\
    filter(func.strftime("%m", Measurement.date) == "06").\
    order_by(Measurement.date).all()
june_temp[0:10]
```

    [('2010-06-01', 78.0),
     ('2010-06-01', 74.0),
     ('2010-06-01', 73.0),
     ('2010-06-01', 77.0),
     ('2010-06-01', 69.0),
     ('2010-06-01', 76.0),
     ('2010-06-01', 71.0),
     ('2010-06-01', 74.0),
     ('2010-06-01', 70.0),
     ('2010-06-02', 76.0)]

```python
# 2. Convert the June temperatures to a list. Print the first 50 temps.
temp = [temp[1] for temp in june_temp[:]]
print(temp[::50])
```

    [78.0, 79.0, 76.0, 77.0, 71.0, 78.0, 72.0, 73.0, 73.0, 71.0, 72.0, 70.0, 71.0, 71.0, 78.0, 75.0, 76.0, 71.0, 78.0, 75.0, 77.0, 74.0, 72.0, 78.0, 69.0, 80.0, 78.0, 72.0, 81.0, 79.0, 80.0, 80.0, 77.0, 78.0]


```python
# 3. Create a DataFrame from the list of temperatures for the month of June. 
df = pd.DataFrame(june_temp, columns=['Date','June Temperature'])
df.set_index(df['Date'], inplace=True)
print(df[0:10].to_string(index=False))
```

          Date  June Temperature
    2010-06-01              78.0
    2010-06-01              74.0
    2010-06-01              73.0
    2010-06-01              77.0
    2010-06-01              69.0
    2010-06-01              76.0
    2010-06-01              71.0
    2010-06-01              74.0
    2010-06-01              70.0
    2010-06-02              76.0

```python
# 4. Calculate and print out the summary statistics for the June temperature DataFrame.
df.describe()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>June Temperature</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1700.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>74.944118</td>
    </tr>
    <tr>
      <th>std</th>
      <td>3.257417</td>
    </tr>
    <tr>
      <th>min</th>
      <td>64.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>73.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>75.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>77.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>85.000000</td>
    </tr>
  </tbody>
</table>
</div>

```python

```

## D2: Determine the Summary Statistics for December


```python
# 6. Write a query that filters the Measurement table to retrieve the temperatures for the month of December.
sel = [Measurement.date, 
       Measurement.tobs]
dec_temp = session.query(*sel).\
    filter(func.strftime("%m", Measurement.date) == "12").\
    order_by(Measurement.date).all()
dec_temp[:10]
```

    [('2010-12-01', 76.0),
     ('2010-12-01', 73.0),
     ('2010-12-01', 73.0),
     ('2010-12-01', 72.0),
     ('2010-12-01', 78.0),
     ('2010-12-01', 72.0),
     ('2010-12-01', 70.0),
     ('2010-12-01', 71.0),
     ('2010-12-02', 72.0),
     ('2010-12-02', 73.0)]


```python
# 7. Convert the December temperatures to a list.
temp = [temp[1] for temp in dec_temp[:]]
print(temp[::50])

```
    [76.0, 66.0, 70.0, 70.0, 74.0, 73.0, 70.0, 71.0, 73.0, 73.0, 71.0, 73.0, 71.0, 65.0, 73.0, 70.0, 73.0, 64.0, 72.0, 68.0, 68.0, 71.0, 65.0, 75.0, 72.0, 68.0, 76.0, 67.0, 70.0, 73.0, 73.0]

```python
# 8. Create a DataFrame from the list of temperatures for the month of December. 
df = pd.DataFrame(dec_temp, columns=['Date','Dec Temperature'])
df.set_index(df['Date'], inplace=True)
print(df[0:10].to_string(index=False))
```
          Date  Dec Temperature
    2010-12-01             76.0
    2010-12-01             73.0
    2010-12-01             73.0
    2010-12-01             72.0
    2010-12-01             78.0
    2010-12-01             72.0
    2010-12-01             70.0
    2010-12-01             71.0
    2010-12-02             72.0
    2010-12-02             73.0

```python
# 9. Calculate and print out the summary statistics for the Decemeber temperature DataFrame.
df.describe()
```

<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Dec Temperature</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1517.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>71.041529</td>
    </tr>
    <tr>
      <th>std</th>
      <td>3.745920</td>
    </tr>
    <tr>
      <th>min</th>
      <td>56.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>69.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>71.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>74.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>83.000000</td>
    </tr>
  </tbody>
</table>
</div>

```python

```
