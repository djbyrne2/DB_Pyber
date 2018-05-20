
# Pyber Ride Sharing
## Analysis

### 1) Nearly 97% of drivers operate in a city (77.8%) or suburban areas (19%).
### 2) Rural areas had the fewest rides but on the whole had highest average fares per ride.
### 3)Urban areas have the low average fares, but high ridership -- implying a strong correlation between average fare price and the total number of rides.


```python
# Import Dependencies
import os
import csv
import pandas as pd
import matplotlib.pyplot as plt
```


```python
# Import file using os module. Verify file path exists

city_csv_filepath = os.path.join('raw_data', 'city_data.csv')
ride_csv_filepath = os.path.join('raw_data', 'ride_data.csv')

print(os.path.exists(city_csv_filepath))
print(os.path.exists(ride_csv_filepath))
```

    True
    True



```python
# Read file with pandas. Create dataframe. Call .head() method

city_data_df = pd.read_csv(city_csv_filepath)

# city_data_df.head()
```


```python
# Read file with pandas. Create dataframe. Call .head() method

ride_data_df = pd.read_csv(ride_csv_filepath)

# ride_data_df.head()
```


```python
#merge

merged_rideshare_df = city_data_df.merge(ride_data_df, on = 'city')

# merged_rideshare_df.head()
```


```python
# Group ride_data_df on city 

grouped_city_ride_data = merged_rideshare_df.groupby("city")

# grouped_city_ride_data.head()
```


```python
# Find Average Fare ($) Per City

ave_fare_city = grouped_city_ride_data['fare'].mean()

# ave_fare_city
```


```python
# Find Total Number of Rides Per City

ride_count_city = grouped_city_ride_data['ride_id'].count()

# ride_count_city
```


```python
# Find Total Number of Drivers Per City
number_drivers_city = grouped_city_ride_data['driver_count'].mean()

# number_drivers_city
```


```python
#Join ito New DataFrame. Issues merging. Workaround solved, but possible issue still..

city_analysis = pd.DataFrame.join(city_data_df, ride_count_city, on="city")
# city_analysis

city_analysis = pd.DataFrame.join(city_analysis, ave_fare_city, on="city")


city_analysis = pd.DataFrame.join(city_analysis, number_drivers_city, on="city", how='left', lsuffix='_left', rsuffix='_right')
# city_analysis

city_analysis_cleaned = city_analysis.drop(['driver_count_right'], axis=1)

city_analysis_cleaned.head()
```


```python
#Relabel Column Names

city_analysis_cleaned.columns = ["City", "Driver Count", "Type of City", "Ride Count", "Average Fare"]
```


```python
city_analysis_cleaned.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Driver Count</th>
      <th>Type of City</th>
      <th>Ride Count</th>
      <th>Average Fare</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Kelseyland</td>
      <td>63</td>
      <td>Urban</td>
      <td>28</td>
      <td>21.806429</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Nguyenbury</td>
      <td>8</td>
      <td>Urban</td>
      <td>26</td>
      <td>25.899615</td>
    </tr>
    <tr>
      <th>2</th>
      <td>East Douglas</td>
      <td>12</td>
      <td>Urban</td>
      <td>22</td>
      <td>26.169091</td>
    </tr>
    <tr>
      <th>3</th>
      <td>West Dawnfurt</td>
      <td>34</td>
      <td>Urban</td>
      <td>29</td>
      <td>22.330345</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Rodriguezburgh</td>
      <td>52</td>
      <td>Urban</td>
      <td>23</td>
      <td>21.332609</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Create dataframes by city type

urban = city_analysis_cleaned[(city_analysis_cleaned['Type of City'] == "Urban")]
```


```python
suburban = city_analysis_cleaned[(city_analysis_cleaned['Type of City'] == "Suburban")]                       
```


```python
rural = city_analysis_cleaned[(city_analysis_cleaned['Type of City'] == "Rural")]
```


```python
# Set Plot figure size
plt.figure(figsize=(10,8))
plt.figtext(.95, .5, "Note: \nCircle size corelates with driver count per city")
```




    Text(0.95,0.5,'Note: \nCircle size corelates with driver count per city')




    <matplotlib.figure.Figure at 0x115febba8>



```python
# Set plot figure size
plt.figure(figsize=(10,8))
plt.figtext(.95, .5, "Note: \nCircle size corelates with driver count per city")

# Plot by type
plt.scatter(rural["Ride Count"], rural["Average Fare"], marker="o", facecolors="gold", edgecolors="black", label="Rural", s=rural["Driver Count"]*10, alpha=0.75)
plt.scatter(suburban["Ride Count"], suburban["Average Fare"], marker="o", facecolors="lightskyblue", edgecolors="black", label="Suburban", s=suburban["Driver Count"]*10, alpha=0.75)
plt.scatter(urban["Ride Count"], urban["Average Fare"], marker="o", facecolors="lightcoral", edgecolors="black", label="Urban", s=urban["Driver Count"]*10, alpha=0.75)

# Add labels to the x and y axes 
plt.title("Pyber Ride Sharing Data (2016)")
plt.xlabel("Total Number of Rides (per City)")
plt.ylabel("Average Fare ($)")

# Set your x and y limits
plt.xlim(0, 40)
plt.ylim(15, 60)


# Set a grid on plot and format legend# Set a  
plt.grid()
lgnd= plt.legend(loc="upper right", scatterpoints=1, fontsize=10, title="City Types")
lgnd.legendHandles[0]._sizes = [75]
lgnd.legendHandles[1]._sizes = [75]
lgnd.legendHandles[2]._sizes = [75]
```


![png](output_17_0.png)



```python
by_city_type = merged_rideshare_df.groupby('type')['fare', 'ride_id', 'driver_count']
```


```python
by_city_fare = by_city_type.sum()['fare']

labels = ["Rural", "Suburban", "Urban"]
colors = ["gold", "lightskyblue", "lightcoral"]
explode = [0, 0, .1]

plt.pie(by_city_fare, startangle = 110, colors = colors, labels = labels, explode=explode, autopct = "%1.1f%%", shadow = True, wedgeprops = {'linewidth': .5, 'edgecolor': 'black'})

#pie chart display
plt.title('% of Total Fares by City Type')
plt.axis('equal')
plt.show()
```


![png](output_19_0.png)



```python
by_city_ride = by_city_type.sum()['ride_id']

labels = ["Rural", "Suburban", "Urban"]
colors = ["gold", "lightskyblue", "lightcoral"]
explode = [0 , 0 , .1]

plt.pie(by_city_ride, startangle = 110, colors = colors, labels = labels, explode = explode, autopct = "%1.2f%%", shadow = True, wedgeprops = {'linewidth': .5, 'edgecolor': 'black'})

#pie chart display
plt.title('% of Total Rides by City Type')
plt.axis('equal')
plt.show()


```


![png](output_20_0.png)



```python
by_city_driver = city_data_df.groupby('type').sum()['driver_count']

labels = ["Rural", "Suburban", "Urban"]
colors = ["gold", "lightskyblue", "lightcoral"]
explode = [0 , 0, .1]

plt.pie(by_city_driver, startangle = 110 , colors = colors, labels = labels, explode = explode,  autopct = "%1.2f%%", shadow = True, wedgeprops = {'linewidth': .5, 'edgecolor': 'black'})

#pie chart display
plt.title('% of Total Drivers by City Type')
plt.axis('equal')
plt.show()

```


![png](output_21_0.png)

