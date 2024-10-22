# <font size=5>Exploratory Data Analysis for Zomato - A Food Delivery Company 📈</font>

```python
# This Python 3 environment comes with many helpful analytics libraries installed
# It is defined by the kaggle/python Docker image: https://github.com/kaggle/docker-python
# For example, here's several helpful packages to load

import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)

# Input data files are available in the read-only "../input/" directory
# For example, running this (by clicking run or pressing Shift+Enter) will list all files under the input directory

import os
for dirname, _, filenames in os.walk('/kaggle/input'):
    for filename in filenames:
        print(os.path.join(dirname, filename))

# You can write up to 20GB to the current directory (/kaggle/working/) that gets preserved as output when you create a version using "Save & Run All" 
# You can also write temporary files to /kaggle/temp/, but they won't be saved outside of the current session
```
![image](https://github.com/user-attachments/assets/e562182f-cef3-46bc-9d4d-dd80b67a797a)
# About the Project

<u>Project Summary:</u>

This project aimed to analyze a dataset containing information about restaurants worldwide. By conducting a thorough EDA, we sought to understand the relationship between various factors, such as restaurant location, price, online ordering availability, and customer ratings.

<u>Key Exploration:</u>

 <a href='#1'>Geographical Analysis</a>
 
 <a href='#2'>Price Analysis.</a>
 
 <a href='#3'>Restaurant Performance.</a>
 
# Working on the Data 🔧
## <font color=#660000>Load Dependencies</font>
```python
import numpy as np
import pandas as pd


import matplotlib.pyplot as plt
import seaborn as sns
import plotly.express as px
import plotly.graph_objs as go
import plotly.subplots as sp

import folium
from folium.plugins import HeatMap

import warnings
warnings.simplefilter(action='ignore', category=FutureWarning)
```
## <font color=#660000> Load the data </font>
```python
# Load the zomato dataset (data related to restaurants on the zomato online food delivery platform)
data_zmt = pd.read_csv('/kaggle/input/zomato-restaurants-data/zomato.csv', encoding='latin-1')
#data_zmt.head()
```
```python
# Load the country codes dataset
data_ctry = pd.read_excel('/kaggle/input/zomato-restaurants-data/Country-Code.xlsx')
#data_ctry.head()
```
## <font color=#660000> Understanding the data</font>
```python
# Dimensionality of the dataFrame
data_zmt.shape
```
![image](https://github.com/user-attachments/assets/71e8adb6-68ee-46ab-9b83-af3a2015799c)

```python
# Dimensionality of the dataFrame
data_ctry.shape
```
![image](https://github.com/user-attachments/assets/4db9179b-8ebb-41d0-b230-13bd41876235)

```python
# Give a summary of the dataset
data_zmt.info()
```
![image](https://github.com/user-attachments/assets/e0152da9-2f05-4aa0-a091-2b2f2af3c0f9)
```python
# Give a summary of the dataset
data_ctry.info()
```
![image](https://github.com/user-attachments/assets/607bfadb-83a6-4917-83b2-1d5548b8a069)

## <font color=#660000> Data Cleaning 
```python
# Join the 2 datasets
data = pd.merge(data_zmt, data_ctry, on='Country Code')
```
```python
# Rename field names
## Convert all field names in data to lowercase and replace whitespace with underscore
data.columns = data.columns.str.lower().str.replace(' ', '_')
data.columns
```
![image](https://github.com/user-attachments/assets/175298c3-ecec-4845-a4cb-ef1f2ed56e86)
## <font color=#660000> <a id='1'>Handling Missing values</font>
```python
# Return the columns that have missing values
print([cols for cols in data.columns if data[cols].isnull().sum() > 0])
```
![image](https://github.com/user-attachments/assets/57a64031-f30e-4bfc-8088-e99ff06eb114)
```python
# Return the count of all null values in `Cuisines`
print(data['cuisines'].isnull().sum())
```
![image](https://github.com/user-attachments/assets/87d08567-6aad-411f-b848-d943763199c4)

```python
cuisines_missing_perc = data['cuisines'].isnull().sum()/data.shape[0]
print('{:.2f}%'.format(cuisines_missing_perc))
```
![image](https://github.com/user-attachments/assets/604cff10-870b-4160-907f-b243458a1f8a)
```python
## Fill the missing values with value
data['cuisines'] = data['cuisines'].fillna(value='Other')
```
# Exploratory Data Analysis 🔍 
```python
df = data.copy()
```
## <font color=#660000> <a id='1'>Geographical Analysis</font>
### <font color=#ff3d3d>What is the country with the highest ratings?</font>
```python
df_country = df.groupby(['country']).size().reset_index(name='counts').sort_values(by='counts', ascending=False)
df_country['restaurant_%'] = round((df_country['counts']/df_country['counts'].sum())*100, 2)
df_country.loc[df_country['restaurant_%'] < 3, 'country'] = 'Other'
df_ctr= df_country.groupby('country')[['counts', 'restaurant_%']].sum().reset_index()
df_ctr
```
![image](https://github.com/user-attachments/assets/4d5ea3b5-84e8-425c-8049-eb955008b166)
<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;">Global Restaurant Distribution</h4>
</div>

```python
fig = px.pie(df_ctr, values='counts', names='country', hole=0.5)
fig.update_layout(
    autosize=False,
    title=dict(
        text='Proportion of Total Restaurants by Country',
        x=0.5,
        y=0.93
    ),
    legend=dict(
        title='Country',
        x=0.8,
        y=0.9
    )
)
fig.show()
```
![image](https://github.com/user-attachments/assets/b6a6133f-b7f7-4296-a564-fc6499d0ef9f)

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;">Countries with the Most Excellent-Rated Restaurants</h4>
</div>

```python
df_excellent_ratings = df[df['rating_text'] == 'Excellent']
df_ctry_excellent_counts = df_excellent_ratings['country'].value_counts().reset_index()
df_ctry_excellent_counts.columns = ['country','count']
df_ctry_excellent_counts['ratings_perc_total'] = round(df_ctry_excellent_counts ['count'] /df_ctry_excellent_counts ['count'].sum(),2)
#df_ctry_excellent_counts
```
```python
basemap = folium.Map(location=[0, 0], zoom_start=2, prefer_canvas=True)


fig = px.choropleth(df_ctry_excellent_counts, locations='country',
                    color='count', # column to use to set color
                    hover_name='country', # column to add to hover information
                    hover_data=['ratings_perc_total', 'count'], # update hover data
                    color_continuous_scale=px.colors.sequential.YlOrRd, # set color scale
                    locationmode='country names') # specify the field to match the country name
fig.update_traces(
    hovertemplate='<b>%{hovertext}</b><br>Ratings Per Total: %{customdata[0]:.0%}<br>Nr. of Ratings: %{customdata[1]}')
    #customdata is specified in hover_data, it follows the sequence when assigning


fig.update_layout(
    title=dict(
        text= 'Global Distribution of Top-Rated Restaurants Across Countries',
        x = 0.5,
        y = 0.93
    ),
    coloraxis_colorbar=dict(
        title="Nr. of Ratings",
        x=0.86 # Adjust this value to move the colorbar
    )
)

fig.show()
```
![image](https://github.com/user-attachments/assets/eb680b51-e3b7-4795-bbe3-2d072746c4cb)

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;">Average Aggregate Rating of Restaurants by Country</h4>
</div>

```python
df_ctry_rating = df.groupby(['country'])['aggregate_rating'].mean().reset_index().sort_values(by='aggregate_rating', ascending=False)

## Create a bar chart
fig=px.bar(df_ctry_rating , x='country', y='aggregate_rating',  color='country')

## Update the hover template for all traces
fig.update_traces(
    hovertemplate='%{x}<br>Average Overall Restaurant Rating: %{y:.2f}'
)
## Modify layout
fig.update_layout(
    showlegend=False,
    title=dict(
        text='Comparision of Average Aggregate Ratings Across Countries',
        x=0.5,
        y=0.93
    ),
    xaxis_title='Country',
    yaxis_title='Average Aggregate Rating'
)
fig.show()
```
![image](https://github.com/user-attachments/assets/6051c83b-5b7e-4874-b662-e6e76777b49e)

#### ☝<font color=#700000><u><b>Observations:</b></u></font>

>India accounts for 90.6% of all restaurants in the dataset and therefore has the most number of top ratings.

>The Philippines holds the top position in terms of overall average rating amongst all the countries.

⚡<font size=4 color=#700000><b>When considering the quality of ratings, the Philippines has the highest ranking.</b>

### <font color=#ff3d3d>What are the most successful restaurants in India?</font>

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;">Indian Cities with the Most Restaurants </h4>
</div>

```python
df_india = df[df['country'] == 'India']

df_cty_restaurants = df_india.groupby(['city']).size().reset_index(name='nr_of_restaurants')
df_cty_restaurants.sort_values(by='nr_of_restaurants', ascending=False, inplace=True)
#df_cty_restaurants
```
```python
fig = px.bar(df_cty_restaurants, x='city', y='nr_of_restaurants', color='city')

fig.update_layout(
    title=dict(
        text= 'Distribution of Restaurants Across Indian Cities',
        x=0.5,
        y=0.93
    ),
    xaxis_title = 'City',
    yaxis_title = 'Nr. of Restaurants',
    legend = dict(
        title='City'
    )
)
fig.show()
```

![image](https://github.com/user-attachments/assets/5faa5158-d6c8-486f-8738-0797adb90355)

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;"> Cities with the Most Successful Restaurants in India (High Rating & Votes) </h4>
</div>

```python
## Define success as having a high aggregate rating and a high number of votes
df_successful = df_india[(df_india['aggregate_rating']> 4) & (df_india['votes']> 100)]

df_cty_successful = df_successful.groupby(['city']).size().reset_index(name='nr_of_restaurants')
df_cty_successful.sort_values(by='nr_of_restaurants', ascending=False, inplace=True)
```
```python
fig = px.bar(df_cty_successful, x='city', y='nr_of_restaurants', color='city')

fig.update_layout(
    title=dict(
        text= 'Distribution of Highly-Rated & Popular Restaurants Across Indian Cities',
        x=0.5,
        y=0.93
    ),
    xaxis_title = 'City',
    yaxis_title = 'Nr. of Restaurants',
    legend = dict(
        title='City'
    )
)
fig.show()
```
![image](https://github.com/user-attachments/assets/77d47c41-3795-4d35-be59-83e6ace14605)

#### ☝<font color=#700000><u><b>Observations:</b></u></font>

>New Delhi , Gurgaon, Noida are the top three cities that have the most restaurants, are top-rated and most popular out of all the cities in India.

⚡<font size=4 color=#700000><b>New Delhi is the city with the most successful restaurants in India</b>

## <font color=#660000><a id='2'>Price Analysis</font>
### <font color=#ff3d3d>Do more expensive restaurants receive more votes?</font>   

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;"> Vote Distribution by Restaurant Price Range</h4>
</div>

```python
# Find the total number of votes for each price range
df_price_votes = df.groupby(['price_range'])['votes'].sum().reset_index()

# Define categories for each price range and replace the original values

mapping = { 1: 'Economical',
            2: 'Mid-Range',
            3: 'Upscale',
            4: 'Fine Dining'}

df_price_votes['price_range'] = df_price_votes['price_range'].map(mapping)
```
```python
# Find the total number of votes for each price range
df_price_votes = df.groupby(['price_range'])['votes'].sum().reset_index()

# Define categories for each price range and replace the original values

mapping = { 1: 'Economical',
            2: 'Mid-Range',
            3: 'Upscale',
            4: 'Fine Dining'}

df_price_votes['price_range'] = df_price_votes['price_range'].map(mapping)
```
![image](https://github.com/user-attachments/assets/9957edb8-ff08-4f57-a267-16feecc5bdc6)

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;"> Breakdown of Restaurants by Price Category </h4>
</div>

```python
df_price_rest = df.groupby(['price_range']).size().reset_index()
df_price_rest.columns = ['price_range', 'count']

mapping = {
    1: 'Economical Eats',
    2: 'Mid-Range Dining',
    3: 'Upscale Cuisine',
    4: 'Fine Dining'
}

df_price_rest['price_range'] = df_price_rest['price_range'].map({1: 'Economical Eats',
                                                                 2: 'Mid-Range Dining',
                                                                 3: 'Upscale Cuisine',
                                                                4: 'Fine Dining'})
#df_price_rest
```
```python

fig = px.pie(df_price_rest , values='count',names='price_range', hole=0.5,
             #color_discrete_sequence=['indigo', 'magenta', 'purple','orchid'],
             hover_name = 'price_range',
             hover_data=['count'])


fig.update_traces(
    hovertemplate='<b>%{hovertext}</b><br>Total Restaurants: %{customdata[0]}'

)

fig.update_layout(
    autosize=False,
    title = dict(
        text = 'Distribution of Restaurants Across Different Price Ranges',
        x = 0.4,
        y=0.93
    ),
    legend = dict(
        title ='Price Category',
        x=0.8,
        y=0.9
    )
)

fig.show()
```
![image](https://github.com/user-attachments/assets/cf6bf7da-3a29-4bb7-acf0-32e42aa6bcf6)

#### ☝<font color=#700000><u><b>Observations:</b></u></font>

>In the process of analyzing the vote count across different price ranges, it’s crucial to consider the total number of restaurants within each category. When we adjust for the number of restaurants, it becomes evident that restaurants in the 3rd (Upscale Cuisine) and 4th (Fine Dining) price ranges, despite constituting a smaller fraction of the total number of restaurants, garner a higher number of votes.

⚡<font size=4 color=#700000><b>Restaurants with higher prices tend to attract more votes</b>

### <font color=#ff3d3d>Which of the top 10 restaurants with the most outlets has the biggest fluctuation in prices?</font> 
<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;"> Top 10 Restaurant Chains in India by Number of Outlets </h4>
</div>

```python
## Only restaurants in India
df_india = df[df['country'] == 'India']

# Filter the data for the required columns
df_india_ra = df_india[['restaurant_name', 'average_cost_for_two']]

## Find the nr. of outlets for each restaurant in India
df_india_outlets = df_india.groupby('restaurant_name').size().reset_index(name='nr_of_outlets')
## Sort by ascending nr of outlets
df_india_outlets.sort_values(by='nr_of_outlets', ascending=False, inplace=True)
## Find top 10 restaurants with the most outlets
top_10_indian_outlets = df_india_outlets.reset_index(drop=True).head(10)
top_10_indian_outlets
```
![image](https://github.com/user-attachments/assets/99f06450-e534-4fc5-bee0-b040c04073c0)

```python
fig = px.bar(top_10_indian_outlets, x='nr_of_outlets', y='restaurant_name', orientation='h', color='restaurant_name')

fig.update_traces(
    hovertemplate='Total Outlets: %{x}')

fig.update_layout(
    autosize=False,
    title=dict(
        text='Dominating Restaurant Chains in India: Top 10 by Outlet Count',
        x=0.5,
        y=0.93
    ),
    legend=dict(
        title='Restaurant'
    ),
    xaxis_title = 'Number of Outlets',
    yaxis_title='Restaurants'

)

fig.show()
```
![image](https://github.com/user-attachments/assets/7590ccaa-f6fd-4378-b9ef-5ff3e966b03f)

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;">Average Cost for Two at Top Indian Restaurant Chains (By Most Outlets)</h4>
</div>

```python
# Find the top 10 restaurants with the most outlets
top_10_restaurants =df_india_ra['restaurant_name'].value_counts().nlargest(10).index
top_10_restaurants


## Filter the data for these top 10 restaurants
df_top_10 = df_india_ra[df_india_ra['restaurant_name'].isin(top_10_restaurants)]


fig = px.box(df_top_10, x='restaurant_name', y='average_cost_for_two', color='restaurant_name')
fig.update_layout(
    title=dict(
        text='Comparison of Average Cost for Two at Top Indian Restaurant Chains',
        x=0.5,
        y=0.93
    ),
    legend=dict(
        title='Restaurant'
    ),
    xaxis_title=('Restaurant'),
    yaxis_title=('Average Cost for Two')
)
fig.show()
fig.write_html('my_plot_9.html')
```
![image](https://github.com/user-attachments/assets/2db7d822-ce5c-4aa4-b878-fbfc2a9cb570)

#### ☝<font color=#700000><u><b>Observations:</b></u></font>

>Pizza Hut  shows some variation in prices, but to a lesser extent than Barbeque Nation.
    
>McDonald's, Subway, Domino's Pizza, Green Chick Chop, Cafe Coffee Day, Giani, Keventers, Baskin Robbins: These restaurants are represented by single markers indicating specific average costs. This suggests that there are no noticeable fluctuations in their prices.

⚡<font size=4 color=#700000><b>Barbecue Nation exhibits the greatest price variation among the top 10 Indian restaurants. This indicates that the average cost for two at Barbecue Nation can vary more widely than at other comparable restaurants.</b>

## <font color=#660000><a id='3'>Restaurant Performance</font>
### <font color=#ff3d3d>How does online ordering availability impact restaurant ratings and popularity?</font>   

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;"> Breakdown of Restaurants by Online Delivery Availability</h4>
</div>

```python
# Count the unique values of 'has_online_delivery'
df_d = df.groupby(['has_online_delivery']).size().reset_index(name='count')
```
```python
fig = px.pie(df_d, values='count', names='has_online_delivery', hole=0.5)
fig.update_layout(
    autosize=False,
    title=dict(
        text='Comparison of Restaurants with and Without Online Delivery',
        x=0.5,
        y=0.93
    ),
    legend=dict(
        title='Online Delivery Availability',
        x=0.8,
        y=0.9
    )
)
fig.show()
```
![image](https://github.com/user-attachments/assets/d661f28d-8c75-44b6-87b0-f22d1185e2bc)

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;"> Impact of Online Delivery on Restaurant Customer Votes </h4>
</div>

```python
df['votes'].describe()
```
![image](https://github.com/user-attachments/assets/599c4093-dc56-4e61-8775-fc9bdb67a25d)

```python
fig = px.box(df, x='has_online_delivery', y='votes', color='has_online_delivery')
fig.update_layout(
    autosize=False,
     title=dict(
        text='Comparison of Customer Votes Based on Online Delivery Availability ',
        x=0.5,
        y=0.93
    ),
    legend=dict(
        title='Online Delivery Availability',

    ),
   xaxis_title='Online Delivery Availability',
   yaxis_title='Aggregate Rating'
)
fig.update_yaxes(range=[0, 1000])
fig.show()
```
![image](https://github.com/user-attachments/assets/64c15507-1081-458c-9bcf-7feeb49366c7)

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;"> Impact of Online Delivery on Restaurant Aggregate Ratings </h4>
</div>

```python
fig = px.box(df, x='has_online_delivery', y='aggregate_rating', color='has_online_delivery')
fig.update_layout(
    autosize=False,
     title=dict(
        text='Comparison of Aggregate Ratings Based on Online Delivery Availability',
        x=0.5,
        y=0.93
    ),
    legend=dict(
        title='Online Delivery Availability',

    ),
   xaxis_title='Online Delivery Availability',
   yaxis_title='Aggregate Rating'
)
fig.show()
```
![image](https://github.com/user-attachments/assets/341c0cfe-05bf-47db-aff6-b4cad017c588)

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;">Exploring the Relationship Between Aggregate Rating, Average Votes, and Online Delivery</h4>
</div>

```python
# Show the average votes for a combination of online availability and aggregate rating
df_d1 = df.groupby(['has_online_delivery', 'aggregate_rating'])['votes'].mean().reset_index(name='avg_votes_per_rating')
```
```python
fig = px.scatter(df_d1, x='aggregate_rating', y='avg_votes_per_rating', color='has_online_delivery')

fig.update_traces(
    hovertemplate='<b>Rating:%{x}</b><br>Average Customer Votes: %{y}'

)
fig.update_layout(
   autosize=False,
   title=dict(
       text='Impact of Online Delivery on Restaurant Ratings and Customer Votes',
       x=0.5,
       y=0.93
   ),
   legend=dict(
       title='Online Delivery'
   ),
   xaxis_title=('Aggregate Rating'),
   yaxis_title=('Average Number of Votes')
)

fig.show()
```
![image](https://github.com/user-attachments/assets/c9874c50-18e7-4197-bc2f-9db2b828ab30)

<div style="background-color:#FFA9A9; padding: 3px;">
    <h4 style="text-align:  font-size: 3px;">Correlation Between Aggregate Ratings and Votes for Restaurant Categories</h4>
</div>

```python
# Calculate correlation for all restaurants
correlation_all = df['aggregate_rating'].corr(df['votes'])

# Calculate correlation for restaurants with online delivery
correlation_delivery = df[df['has_online_delivery'] == 'Yes']['aggregate_rating'].corr(df['votes'])

# Calculate correlation for restaurants without online delivery
correlation_no_delivery = df[df['has_online_delivery'] == 'No']['aggregate_rating'].corr(df['votes'])

#print("Correlation (All): ", correlation_all)
#print("Correlation (With Delivery): ", correlation_delivery)
#print("Correlation (Without Delivery): ", correlation_no_delivery)
```
![image](https://github.com/user-attachments/assets/0ecb7392-8c6c-4a4a-9cb5-f83f7a14eec4)

#### ☝<font color=#700000><u><b>Observations:</b></u></font>

>Restaurants that offer online delivery received a significantly higher number of votes, despite the fact that the majority of restaurants do not offer online delivery.
    
>Restaurants that offer online delivery tends to have higher aggregate ratings compared to those that do not. The variability in non-delivery restaurants is higher, which can be attributed to the majority of restaurants not offering online delivery.

>Restaurants with online delivery service typically get higher ratings and, on average, accumulate more votes compared to those without the service.Availability of online delivery could play a crucial role in a restaurant's appeal and customer satisfaction.
    
>There is a weak positive correlation between aggregate ratings and the number of votes for all restaurants, regardless of online delivery availability. This suggests that restaurants with higher ratings tend to attract more votes, but the relationship is not very strong.

⚡<font size=4 color=#700000><b>Restaurants with higher ratings tend to have more votes, regardless of whether they offer online delivery or not. However,the availability of online delivery could be a significant factor in a restaurant's popularity and customer satisfaction. It seems to influence both the number of votes a restaurant receives and its aggregate rating</b>
























