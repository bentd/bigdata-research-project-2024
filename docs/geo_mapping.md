Creating a heat map and bubble map on a map of the US by state can be done using libraries like `pandas`, `geopandas`, `folium`, and `plotly`. Here's a step-by-step guide to achieve both visualizations:

### 1. Heat Map by State

First, ensure you have the necessary libraries installed:

```bash
pip install pandas geopandas folium
```

Then, create the heat map using the following code:

```python
import pandas as pd
import geopandas as gpd
import folium

# Example DataFrame for heat map
data = {
    'state': ['California', 'Texas', 'New York', 'Florida', 'Illinois'],
    'value': [100, 90, 80, 70, 60]
}
df = pd.DataFrame(data)

# Load the US states shapefile
gdf = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))
gdf = gdf[gdf['continent'] == 'North America']
gdf = gdf[gdf['name'] == 'United States']
states = gpd.read_file('https://raw.githubusercontent.com/johan/world.geo.json/master/countries/USA.geo.json')

# Merge the DataFrame with the GeoDataFrame
states = states.rename(columns={'name': 'state'})
merged = states.set_index('state').join(df.set_index('state'))

# Initialize the map centered on the US
m = folium.Map(location=[37.0902, -95.7129], zoom_start=5)

# Create a choropleth map
folium.Choropleth(
    geo_data=states,
    name='choropleth',
    data=merged,
    columns=[merged.index, 'value'],
    key_on='feature.properties.state',
    fill_color='YlOrRd',
    fill_opacity=0.7,
    line_opacity=0.2,
    legend_name='Value by State'
).add_to(m)

# Save the map as an HTML file
m.save('heat_map.html')
```

### 2. Bubble Map by State

For creating a bubble map, you can use `plotly`. Install it using:

```bash
pip install plotly
```

Then, create the bubble map:

```python
import pandas as pd
import plotly.express as px

# Example DataFrame for bubble map
bubble_data = {
    'state': ['California', 'Texas', 'New York', 'Florida', 'Illinois'],
    'bubble_value': [50, 40, 30, 20, 10]
}
df_bubble = pd.DataFrame(bubble_data)

# Get state centroid coordinates
state_coordinates = {
    'California': [36.7783, -119.4179],
    'Texas': [31.9686, -99.9018],
    'New York': [40.7128, -74.0060],
    'Florida': [27.9944, -81.7603],
    'Illinois': [40.6331, -89.3985]
}
df_bubble['lat'] = df_bubble['state'].map(lambda x: state_coordinates[x][0])
df_bubble['lon'] = df_bubble['state'].map(lambda x: state_coordinates[x][1])

# Create bubble map using Plotly
fig = px.scatter_geo(
    df_bubble,
    lat='lat',
    lon='lon',
    text='state',
    size='bubble_value',
    size_max=50,
    title='Bubble Map by State'
)

fig.update_geos(
    scope='usa',
    projection_type='albers usa'
)

fig.show()
```

### Explanation

1. **Heat Map**:
   - **pandas**: Used to handle data.
   - **geopandas**: Used to handle geospatial data and merge it with the pandas DataFrame.
   - **folium**: Used to create the map and the choropleth layer.

2. **Bubble Map**:
   - **pandas**: Used to handle data.
   - **plotly.express**: Used to create the bubble map, leveraging state centroid coordinates for plotting.

Save the generated HTML file for the heat map and visualize the bubble map inline using Plotly.

Feel free to adjust the datasets and parameters according to your specific needs.
