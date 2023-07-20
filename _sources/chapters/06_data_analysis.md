---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Analyzing Geospatial Data

```{contents}
:local:
:depth: 2
```

## Introduction

## Technical requirements

```bash
conda create -n gee python
conda activate gee
conda install -c conda-forge mamba
mamba install -c conda-forge pygis
```

```bash
jupyter lab
```

```{code-cell} ipython3
# %pip install pygis
```

```{code-cell} ipython3
import ee
import geemap
```

```{code-cell} ipython3
geemap.ee_initialize()
```

## Earth Engine data reductions

### List reductions

```{code-cell} ipython3
values = ee.List.sequence(1, 10)
print(values.getInfo())
```

```{code-cell} ipython3
count = values.reduce(ee.Reducer.count())
print(count.getInfo())  # 10
```

```{code-cell} ipython3
min_value = values.reduce(ee.Reducer.min())
print(min_value.getInfo())  # 1
```

```{code-cell} ipython3
max_value = values.reduce(ee.Reducer.max())
print(max_value.getInfo())  # 10
```

```{code-cell} ipython3
min_max_value = values.reduce(ee.Reducer.minMax())
print(min_max_value.getInfo())
```

```{code-cell} ipython3
mean_value = values.reduce(ee.Reducer.mean())
print(mean_value.getInfo())  # 5.5
```

```{code-cell} ipython3
median_value = values.reduce(ee.Reducer.median())
print(median_value.getInfo())  # 5.5
```

```{code-cell} ipython3
sum_value = values.reduce(ee.Reducer.sum())
print(sum_value.getInfo())  # 55
```

```{code-cell} ipython3
std_value = values.reduce(ee.Reducer.stdDev())
print(std_value.getInfo())  # 2.8723
```

### ImageCollection reductions

```{code-cell} ipython3
Map = geemap.Map()

# Load an image collection, filtered so it's not too much data.
collection = (
    ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA')
    .filterDate('2021-01-01', '2021-12-31')
    .filter(ee.Filter.eq('WRS_PATH', 44))
    .filter(ee.Filter.eq('WRS_ROW', 34))
)

# Compute the median in each band, each pixel.
# Band names are B1_median, B2_median, etc.
median = collection.reduce(ee.Reducer.median())

# The output is an Image.  Add it to the map.
vis_param = {'bands': ['B5_median', 'B4_median', 'B3_median'], 'gamma': 2}
Map.setCenter(-122.3355, 37.7924, 8)
Map.addLayer(median, vis_param)
Map
```

```{code-cell} ipython3
median = collection.median()
print(median.bandNames().getInfo())
```

### Image reductions

```{code-cell} ipython3
Map = geemap.Map()
image = ee.Image('LANDSAT/LC08/C01/T1/LC08_044034_20140318').select(['B4', 'B3', 'B2'])
maxValue = image.reduce(ee.Reducer.max())
Map.centerObject(image, 8)
Map.addLayer(image, {}, 'Original image')
Map.addLayer(maxValue, {'max': 13000}, 'Maximum value image')
Map
```

### FeatureCollection reductions

```{code-cell} ipython3
Map = geemap.Map()
census = ee.FeatureCollection('TIGER/2010/Blocks')
benton = census.filter(
    ee.Filter.And(ee.Filter.eq('statefp10', '41'), ee.Filter.eq('countyfp10', '003'))
)
Map.setCenter(-123.27, 44.57, 13)
Map.addLayer(benton)
Map
```

```{code-cell} ipython3
# Compute sums of the specified properties.
properties = ['pop10', 'housing10']
sums = benton.filter(ee.Filter.notNull(properties)).reduceColumns(
    **{'reducer': ee.Reducer.sum().repeat(2), 'selectors': properties}
)
sums
```

```{code-cell} ipython3
print(benton.aggregate_sum('pop10'))  # 85579
print(benton.aggregate_sum('housing10'))  # 36245
```

```{code-cell} ipython3
benton.aggregate_stats('pop10')
```

## Image descriptive statistics

```{code-cell} ipython3
Map = geemap.Map()

centroid = ee.Geometry.Point([-122.4439, 37.7538])
image = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR').filterBounds(centroid).first()
vis = {'min': 0, 'max': 3000, 'bands': ['B5', 'B4', 'B3']}

Map.centerObject(centroid, 8)
Map.addLayer(image, vis, "Landsat-8")
Map
```

```{code-cell} ipython3
image.propertyNames()
```

```{code-cell} ipython3
image.get('CLOUD_COVER')  # 0.05
```

```{code-cell} ipython3
props = geemap.image_props(image)
props
```

```{code-cell} ipython3
stats = geemap.image_stats(image, scale=30)
stats
```

## Zonal statistics with Earth Engine

### Zonal statistics

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)

# Add NASA SRTM
dem = ee.Image('USGS/SRTMGL1_003')
dem_vis = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}
Map.addLayer(dem, dem_vis, 'SRTM DEM')

# Add 5-year Landsat TOA composite
landsat = ee.Image('LANDSAT/LE7_TOA_5YEAR/1999_2003')
landsat_vis = {'bands': ['B4', 'B3', 'B2'], 'gamma': 1.4}
Map.addLayer(landsat, landsat_vis, "Landsat", False)

# Add US Census States
states = ee.FeatureCollection("TIGER/2018/States")
style = {'fillColor': '00000000'}
Map.addLayer(states.style(**style), {}, 'US States')
Map
```

```{code-cell} ipython3
out_dem_stats = 'dem_stats.csv'
geemap.zonal_stats(
    dem, states, out_dem_stats, statistics_type='MEAN', scale=1000, return_fc=False
)
```

```{code-cell} ipython3
out_landsat_stats = 'landsat_stats.csv'
geemap.zonal_stats(
    landsat,
    states,
    out_landsat_stats,
    statistics_type='MEAN',
    scale=1000,
    return_fc=False,
)
```

### Zonal statistics by group

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)

# Add NLCD data
dataset = ee.Image('USGS/NLCD_RELEASES/2019_REL/NLCD/2019')
landcover = dataset.select('landcover')
Map.addLayer(landcover, {}, 'NLCD 2019')

# Add US census states
states = ee.FeatureCollection("TIGER/2018/States")
style = {'fillColor': '00000000'}
Map.addLayer(states.style(**style), {}, 'US States')

# Add NLCD legend
Map.add_legend(title='NLCD Land Cover', builtin_legend='NLCD')
Map
```

```{code-cell} ipython3
nlcd_stats = 'nlcd_stats.csv'

geemap.zonal_stats_by_group(
    landcover,
    states,
    nlcd_stats,
    statistics_type='SUM',
    denominator=1e6,
    decimal_places=2,
)
```

```{code-cell} ipython3
nlcd_stats = 'nlcd_stats_pct.csv'

geemap.zonal_stats_by_group(
    landcover,
    states,
    nlcd_stats,
    statistics_type='PERCENTAGE',
    denominator=1e6,
    decimal_places=2,
)
```

### Zonal statistics with two images

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
dem = ee.Image('USGS/3DEP/10m')
vis = {'min': 0, 'max': 4000, 'palette': 'terrain'}
Map.addLayer(dem, vis, 'DEM')
Map
```

```{code-cell} ipython3
landcover = ee.Image("USGS/NLCD_RELEASES/2019_REL/NLCD/2019").select('landcover')
Map.addLayer(landcover, {}, 'NLCD 2019')
Map.add_legend(title='NLCD Land Cover Classification', builtin_legend='NLCD')
```

```{code-cell} ipython3
stats = geemap.image_stats_by_zone(dem, landcover, reducer='MEAN')
stats
```

```{code-cell} ipython3
stats.to_csv('mean.csv', index=False)
```

```{code-cell} ipython3
geemap.image_stats_by_zone(dem, landcover, out_csv="std.csv", reducer='STD')
```

## Coordinate grids and fishnets

### Creating coordinate grids

```{code-cell} ipython3
lat_grid = geemap.latitude_grid(step=5.0, west=-180, east=180, south=-85, north=85)
```

```{code-cell} ipython3
Map = geemap.Map()
style = {'fillColor': '00000000'}
Map.addLayer(lat_grid.style(**style), {}, 'Latitude Grid')
Map
```

```{code-cell} ipython3
df = geemap.ee_to_df(lat_grid)
df
```

```{code-cell} ipython3
lon_grid = geemap.longitude_grid(step=5.0, west=-180, east=180, south=-85, north=85)
```

```{code-cell} ipython3
Map = geemap.Map()
style = {'fillColor': '00000000'}
Map.addLayer(lon_grid.style(**style), {}, 'Longitude Grid')
Map
```

```{code-cell} ipython3
grid = geemap.latlon_grid(
    lat_step=10, lon_step=10, west=-180, east=180, south=-85, north=85
)
```

```{code-cell} ipython3
Map = geemap.Map()
style = {'fillColor': '00000000'}
Map.addLayer(grid.style(**style), {}, 'Coordinate Grid')
Map
```

### Creating fishnets

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
roi = Map.user_roi

if roi is None:
    roi = ee.Geometry.BBox(-112.8089, 33.7306, -88.5951, 46.6244)
    Map.addLayer(roi, {}, 'ROI')
    Map.user_roi = None

Map.centerObject(roi)
```

```{code-cell} ipython3
fishnet = geemap.fishnet(roi, h_interval=2.0, v_interval=2.0, delta=1)
style = {'color': 'blue', 'fillColor': '00000000'}
Map.addLayer(fishnet.style(**style), {}, 'Fishnet')
```

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
roi = Map.user_roi

if roi is None:
    roi = ee.Geometry.Polygon(
        [
            [
                [-64.602356, -1.127399],
                [-68.821106, -12.625598],
                [-60.647278, -22.498601],
                [-47.815247, -21.111406],
                [-43.860168, -8.913564],
                [-54.582825, -0.775886],
                [-60.823059, 0.454555],
                [-64.602356, -1.127399],
            ]
        ]
    )
    Map.addLayer(roi, {}, 'ROI')

Map.centerObject(roi)
Map
```

```{code-cell} ipython3
fishnet = geemap.fishnet(roi, rows=6, cols=8, delta=1)
style = {'color': 'blue', 'fillColor': '00000000'}
Map.addLayer(fishnet.style(**style), {}, 'Fishnet')
```

## Extracting pixel values

### Extracting values to points

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)

dem = ee.Image('USGS/SRTMGL1_003')
landsat7 = ee.Image('LANDSAT/LE7_TOA_5YEAR/1999_2003')

vis_params = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}

Map.addLayer(
    landsat7,
    {'bands': ['B4', 'B3', 'B2'], 'min': 20, 'max': 200, 'gamma': 2},
    'Landsat 7',
)
Map.addLayer(dem, vis_params, 'SRTM DEM', True, 1)
Map
```

```{code-cell} ipython3
in_shp = 'us_cities.shp'
url = 'https://github.com/giswqs/data/raw/main/us/us_cities.zip'
geemap.download_file(url)
```

```{code-cell} ipython3
in_fc = geemap.shp_to_ee(in_shp)
Map.addLayer(in_fc, {}, 'Cities')
```

```{code-cell} ipython3
geemap.extract_values_to_points(in_fc, dem, out_fc="dem.shp")
```

```{code-cell} ipython3
geemap.shp_to_gdf("dem.shp")
```

```{code-cell} ipython3
geemap.extract_values_to_points(in_fc, landsat7, 'landsat.csv')
```

```{code-cell} ipython3
geemap.csv_to_df('landsat.csv')
```

### Extracting pixel values along a transect

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
Map.add_basemap("TERRAIN")

image = ee.Image('USGS/SRTMGL1_003')
vis_params = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}
Map.addLayer(image, vis_params, 'SRTM DEM', True, 0.5)
Map
```

```{code-cell} ipython3
line = Map.user_roi
if line is None:
    line = ee.Geometry.LineString(
        [[-120.2232, 36.3148], [-118.9269, 36.7121], [-117.2022, 36.7562]]
    )
    Map.addLayer(line, {}, "ROI")
Map.centerObject(line)
```

```{code-cell} ipython3
reducer = 'mean'
transect = geemap.extract_transect(
    image, line, n_segments=100, reducer=reducer, to_pandas=True
)
transect
```

```{code-cell} ipython3
geemap.line_chart(
    data=transect,
    x='distance',
    y='mean',
    markers=True,
    x_label='Distance (m)',
    y_label='Elevation (m)',
    height=400,
)
```

```{code-cell} ipython3
transect.to_csv('transect.csv')
```

### Interactive region reduction

```{code-cell} ipython3
Map = geemap.Map()

collection = (
    ee.ImageCollection('MODIS/061/MOD13A2')
    .filterDate('2015-01-01', '2019-12-31')
    .select('NDVI')
)

image = collection.toBands()

ndvi_vis = {
    'min': 0.0,
    'max': 9000.0,
    'palette': 'ndvi',
}

Map.addLayer(image, {}, 'MODIS NDVI Time-series')
Map.addLayer(image.select(0), ndvi_vis, 'First image')

Map
```

```{code-cell} ipython3
dates = geemap.image_dates(collection).getInfo()
dates
```

```{code-cell} ipython3
len(dates)
```

```{code-cell} ipython3
Map.set_plot_options(add_marker_cluster=True)
Map.roi_reducer = ee.Reducer.mean()
Map
```

```{code-cell} ipython3
Map.extract_values_to_points('ndvi.csv')
```

## Mapping available image count

```{code-cell} ipython3
collection = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
image = geemap.image_count(
    collection, region=None, start_date='2021-01-01', end_date='2022-01-01', clip=False
)
```

```{code-cell} ipython3
Map = geemap.Map()
vis = {'min': 0, 'max': 60, 'palette': 'coolwarm'}
Map.addLayer(image, vis, 'Image Count')
Map.add_colorbar(vis, label='Landsat 8 Image Count')

countries = ee.FeatureCollection(geemap.examples.get_ee_path('countries'))
style = {"color": "00000088", "width": 1, "fillColor": "00000000"}
Map.addLayer(countries.style(**style), {}, "Countries")
Map
```

## Cloud-free composites

```{code-cell} ipython3
Map = geemap.Map()

collection = ee.ImageCollection('LANDSAT/LC08/C02/T1').filterDate(
    '2021-01-01', '2022-01-01'
)

composite = ee.Algorithms.Landsat.simpleComposite(collection)

vis_params = {'bands': ['B5', 'B4', 'B3'], 'max': 128}

Map.setCenter(-122.3578, 37.7726, 10)
Map.addLayer(composite, vis_params, 'TOA composite')
Map
```

```{code-cell} ipython3
customComposite = ee.Algorithms.Landsat.simpleComposite(
    **{'collection': collection, 'percentile': 30, 'cloudScoreRange': 5}
)

Map.addLayer(customComposite, vis_params, 'Custom TOA composite')
Map.setCenter(-105.4317, 52.5536, 11)
```

```{code-cell} ipython3
vis_params = [
    {'bands': ['B4', 'B3', 'B2'], 'min': 0, 'max': 128},
    {'bands': ['B5', 'B4', 'B3'], 'min': 0, 'max': 128},
    {'bands': ['B7', 'B6', 'B4'], 'min': 0, 'max': 128},
    {'bands': ['B6', 'B5', 'B2'], 'min': 0, 'max': 128},
]

labels = [
    'Natural Color (4, 3, 2)',
    'Color Infrared (5, 4, 3)',
    'Short-Wave Infrared (7, 6 4)',
    'Agriculture (6, 5, 2)',
]
```

```{code-cell} ipython3
geemap.linked_maps(
    rows=2,
    cols=2,
    height="300px",
    center=[37.7726, -122.1578],
    zoom=9,
    ee_objects=[composite],
    vis_params=vis_params,
    labels=labels,
    label_position="topright",
)
```

## Quality mosaicking

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
countries = ee.FeatureCollection(geemap.examples.get_ee_path('countries'))
roi = countries.filter(ee.Filter.eq('ISO_A3', 'USA'))
Map.addLayer(roi, {}, 'roi')
Map
```

```{code-cell} ipython3
start_date = '2020-01-01'
end_date = '2021-01-01'
collection = (
    ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA')
    .filterBounds(roi)
    .filterDate(start_date, end_date)
)
```

```{code-cell} ipython3
median = collection.median()
vis_rgb = {
    'bands': ['B4', 'B3', 'B2'],
    'min': 0,
    'max': 0.4,
}
Map.addLayer(median, vis_rgb, 'Median')
Map
```

```{code-cell} ipython3
def add_ndvi(image):
    ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI')
    return image.addBands(ndvi)
```

```{code-cell} ipython3
def add_time(image):
    date = ee.Date(image.date())

    img_date = ee.Number.parse(date.format('YYYYMMdd'))
    image = image.addBands(ee.Image(img_date).rename('date').toInt())

    img_month = ee.Number.parse(date.format('M'))
    image = image.addBands(ee.Image(img_month).rename('month').toInt())

    img_doy = ee.Number.parse(date.format('D'))
    image = image.addBands(ee.Image(img_doy).rename('doy').toInt())

    return image
```

```{code-cell} ipython3
images = collection.map(add_ndvi).map(add_time)
```

```{code-cell} ipython3
greenest = images.qualityMosaic('NDVI')
```

```{code-cell} ipython3
greenest.bandNames()
```

```{code-cell} ipython3
ndvi = greenest.select('NDVI')
vis_ndvi = {'min': 0, 'max': 1, 'palette': 'ndvi'}
Map.addLayer(ndvi, vis_ndvi, 'NDVI')
Map.add_colorbar(vis_ndvi, label='NDVI', layer_name='NDVI')
Map
```

```{code-cell} ipython3
Map.addLayer(greenest, vis_rgb, 'Greenest pixel')
```

```{code-cell} ipython3
vis_month = {'palette': ['red', 'blue'], 'min': 1, 'max': 12}
Map.addLayer(greenest.select('month'), vis_month, 'Greenest month')
Map.add_colorbar(vis_month, label='Month', layer_name='Greenest month')
```

```{code-cell} ipython3
vis_doy = {'palette': ['brown', 'green'], 'min': 1, 'max': 365}
Map.addLayer(greenest.select('doy'), vis_doy, 'Greenest doy')
Map.add_colorbar(vis_doy, label='Day of year', layer_name='Greenest doy')
```

## Interactive charts

### Chart Overview

### Data table charts

```{code-cell} ipython3
data = geemap.examples.get_path('countries.geojson')
df = geemap.geojson_to_df(data)
df.head()
```

```{code-cell} ipython3
geemap.bar_chart(
    data=df,
    x='NAME',
    y='POP_EST',
    x_label='Country',
    y_label='Population',
    descending=True,
    max_rows=30,
    title='World Population',
    height=500,
    layout_args={'title_x': 0.5, 'title_y': 0.85},
)
```

```{code-cell} ipython3
geemap.pie_chart(
    data=df,
    names='NAME',
    values='POP_EST',
    max_rows=30,
    height=600,
    title='World Population',
    legend_title='Country',
    layout_args={'title_x': 0.47, 'title_y': 0.87},
)
```

```{code-cell} ipython3
data = geemap.examples.get_path('life_exp.csv')
df = geemap.csv_to_df(data)
df = df[df['continent'] == 'Oceania']
df.head()
```

```{code-cell} ipython3
geemap.line_chart(
    df,
    x='year',
    y='lifeExp',
    color='country',
    x_label='Year',
    y_label='Life expectancy',
    legend_title='Country',
    height=400,
    markers=True,
)
```

### Earth Engine object charts

```{code-cell} ipython3
import geemap.chart as chart
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
collection = ee.FeatureCollection('projects/google/charts_feature_example')
Map.addLayer(collection, {}, "Ecoregions")
Map
```

#### Chart by feature

```{code-cell} ipython3
features = collection.select('[0-9][0-9]_tmean|label')
df = geemap.ee_to_df(features, sort_columns=True)
df
```

```{code-cell} ipython3
xProperty = "label"
yProperties = df.columns[:12]

labels = [
    'Jan',
    'Feb',
    'Mar',
    'Apr',
    'May',
    'Jun',
    'Jul',
    'Aug',
    'Sep',
    'Oct',
    'Nov',
    'Dec',
]
colors = [
    '#604791',
    '#1d6b99',
    '#39a8a7',
    '#0f8755',
    '#76b349',
    '#f0af07',
    '#e37d05',
    '#cf513e',
    '#96356f',
    '#724173',
    '#9c4f97',
    '#696969',
]
title = "Average Monthly Temperature by Ecoregion"
xlabel = "Ecoregion"
ylabel = "Temperature"
```

```{code-cell} ipython3
options = {
    "labels": labels,
    "colors": colors,
    "title": title,
    "xlabel": xlabel,
    "ylabel": ylabel,
    "legend_location": "top-left",
    "height": "500px",
}
```

```{code-cell} ipython3
chart.feature_byFeature(features, xProperty, yProperties, **options)
```

#### Chart by property

```{code-cell} ipython3
features = collection.select('[0-9][0-9]_ppt|label')
df = geemap.ee_to_df(features, sort_columns=True)
df
```

```{code-cell} ipython3
keys = df.columns[:12]
values = [
    'Jan',
    'Feb',
    'Mar',
    'Apr',
    'May',
    'Jun',
    'Jul',
    'Aug',
    'Sep',
    'Oct',
    'Nov',
    'Dec',
]
xProperties = dict(zip(keys, values))
seriesProperty = "label"
```

```{code-cell} ipython3
options = {
    'title': "Average Ecoregion Precipitation by Month",
    'colors': ['#f0af07', '#0f8755', '#76b349'],
    'xlabel': "Month",
    'ylabel': "Precipitation (mm)",
    'legend_location': "top-left",
    "height": "500px",
}
```

```{code-cell} ipython3
chart.feature_byProperty(features, xProperties, seriesProperty, **options)
```

#### Feature histograms

```{code-cell} ipython3
source = ee.ImageCollection('OREGONSTATE/PRISM/Norm81m').toBands()
region = ee.Geometry.Rectangle(-123.41, 40.43, -116.38, 45.14)
samples = source.sample(region, 5000)
prop = '07_ppt'
```

```{code-cell} ipython3
options = {
    "title": 'July Precipitation Distribution for NW USA',
    "xlabel": 'Precipitation (mm)',
    "ylabel": 'Pixel count',
    "colors": ['#1d6b99'],
}
```

```{code-cell} ipython3
chart.feature_histogram(samples, prop, **options)
```

```{code-cell} ipython3
chart.feature_histogram(samples, prop, maxBuckets=30, **options)
```

```{code-cell} ipython3
chart.feature_histogram(samples, prop, minBucketWidth=0.5, **options)
```

```{code-cell} ipython3
chart.feature_histogram(samples, prop, minBucketWidth=3, maxBuckets=30, **options)
```

## Unsupervised classification

```{code-cell} ipython3
Map = geemap.Map()

point = ee.Geometry.Point([-88.0664, 41.9411])

image = (
    ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
    .filterBounds(point)
    .filterDate('2022-01-01', '2022-12-31')
    .sort('CLOUD_COVER')
    .first()
    .select('SR_B[1-7]')
)

region = image.geometry()
image = image.multiply(0.0000275).add(-0.2).set(image.toDictionary())
vis_params = {'min': 0, 'max': 0.3, 'bands': ['SR_B5', 'SR_B4', 'SR_B3']}

Map.centerObject(region, 8)
Map.addLayer(image, vis_params, "Landsat-9")
Map
```

```{code-cell} ipython3
geemap.get_info(image)
```

```{code-cell} ipython3
image.get('DATE_ACQUIRED').getInfo()
```

```{code-cell} ipython3
image.get('CLOUD_COVER').getInfo()
```

```{code-cell} ipython3
training = image.sample(
    **{
        # "region": region,
        'scale': 30,
        'numPixels': 5000,
        'seed': 0,
        'geometries': True,  # Set this to False to ignore geometries
    }
)

Map.addLayer(training, {}, 'Training samples')
Map
```

```{code-cell} ipython3
geemap.ee_to_df(training.limit(5))
```

```{code-cell} ipython3
n_clusters = 5
clusterer = ee.Clusterer.wekaKMeans(n_clusters).train(training)
```

```{code-cell} ipython3
result = image.cluster(clusterer)
Map.addLayer(result.randomVisualizer(), {}, 'clusters')
Map
```

```{code-cell} ipython3
legend_dict = {
    'Open Water': '#466b9f',
    'Developed, High Intensity': '#ab0000',
    'Developed, Low Intensity': '#d99282',
    'Forest': '#1c5f2c',
    'Cropland': '#ab6c28'

}

palette = list(legend_dict.values())

Map.addLayer(
    result, {'min': 0, 'max': 4, 'palette': palette}, 'Labelled clusters'
)
Map.add_legend(title='Land Cover Type',legend_dict=legend_dict , position='bottomright')
Map
```

```{code-cell} ipython3
geemap.download_ee_image(image, filename='unsupervised.tif', region=region, scale=90)
```

## Supervised classification

```{code-cell} ipython3
Map = geemap.Map()
point = ee.Geometry.Point([-122.4439, 37.7538])

image = (
    ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
    .filterBounds(point)
    .filterDate('2019-01-01', '2020-01-01')
    .sort('CLOUD_COVER')
    .first()
    .select('SR_B[1-7]')
)

image = image.multiply(0.0000275).add(-0.2).set(image.toDictionary())
vis_params = {'min': 0, 'max': 0.3, 'bands': ['SR_B5', 'SR_B4', 'SR_B3']}

Map.centerObject(point, 8)
Map.addLayer(image, vis_params, "Landsat-8")
Map
```

```{code-cell} ipython3
geemap.get_info(image)
```

```{code-cell} ipython3
image.get('DATE_ACQUIRED').getInfo()
```

```{code-cell} ipython3
image.get('CLOUD_COVER').getInfo()
```

```{code-cell} ipython3
nlcd = ee.Image('USGS/NLCD_RELEASES/2019_REL/NLCD/2019')
landcover = nlcd.select('landcover').clip(image.geometry())
Map.addLayer(landcover, {}, 'NLCD Landcover')
Map
```

```{code-cell} ipython3
points = landcover.sample(
    **{
        'region': image.geometry(),
        'scale': 30,
        'numPixels': 5000,
        'seed': 0,
        'geometries': True,
    }
)

Map.addLayer(points, {}, 'training', False)
```

```{code-cell} ipython3
print(points.size().getInfo())
```

```{code-cell} ipython3
bands = ['SR_B1', 'SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7']
label = 'landcover'
features = image.select(bands).sampleRegions(
    **{'collection': points, 'properties': [label], 'scale': 30}
)
```

```{code-cell} ipython3
geemap.ee_to_df(features.limit(5))
```

```{code-cell} ipython3
params = {

    'features': features,
    'classProperty': label,
    'inputProperties': bands,

}
classifier = ee.Classifier.smileCart(maxNodes=None).train(**params)
```

```{code-cell} ipython3
classified = image.select(bands).classify(classifier).rename('landcover')
Map.addLayer(classified.randomVisualizer(), {}, 'Classified')
Map
```

```{code-cell} ipython3
geemap.get_info(nlcd)
```

```{code-cell} ipython3
class_values = nlcd.get('landcover_class_values')
class_palette = nlcd.get('landcover_class_palette')
classified = classified.set({
    'landcover_class_values': class_values,
    'landcover_class_palette': class_palette
})
```

```{code-cell} ipython3
Map.addLayer(classified, {}, 'Land cover')
Map.add_legend(title="Land cover type", builtin_legend='NLCD')
Map
```

```{code-cell} ipython3
geemap.download_ee_image(
    landcover,
    filename='supervised.tif',
    region=image.geometry(),
    scale=30
    )
```

## Accuracy assessment

```{code-cell} ipython3
Map = geemap.Map()
point = ee.Geometry.Point([-122.4439, 37.7538])

img = (
    ee.ImageCollection('COPERNICUS/S2_SR')
    .filterBounds(point)
    .filterDate('2020-01-01', '2021-01-01')
    .sort('CLOUDY_PIXEL_PERCENTAGE')
    .first()
    .select('B.*')
)

vis_params = {'min': 100, 'max': 3500, 'bands': ['B11',  'B8',  'B3']}

Map.centerObject(point, 9)
Map.addLayer(img, vis_params, "Sentinel-2")
Map
```

```{code-cell} ipython3
lc = ee.Image('ESA/WorldCover/v100/2020')
classValues = [10, 20, 30, 40, 50, 60, 70, 80, 90, 95, 100]
remapValues = ee.List.sequence(0, 10)
label = 'lc'
lc = lc.remap(classValues, remapValues).rename(label).toByte()
```

```{code-cell} ipython3
sample = img.addBands(lc).stratifiedSample(**{
  'numPoints': 100,
  'classBand': label,
  'region': img.geometry(),
  'scale': 10,
  'geometries': True
})
```

```{code-cell} ipython3
sample = sample.randomColumn()
trainingSample = sample.filter('random <= 0.8')
validationSample = sample.filter('random > 0.8')
```

```{code-cell} ipython3
trainedClassifier = ee.Classifier.smileRandomForest(numberOfTrees=10).train(**{
  'features': trainingSample,
  'classProperty': label,
  'inputProperties': img.bandNames()
})
```

```{code-cell} ipython3
print('Results of trained classifier', trainedClassifier.explain().getInfo())
```

```{code-cell} ipython3
trainAccuracy = trainedClassifier.confusionMatrix()
trainAccuracy.getInfo()
```

```{code-cell} ipython3
trainAccuracy.accuracy().getInfo()
```

```{code-cell} ipython3
trainAccuracy.kappa().getInfo()
```

```{code-cell} ipython3
validationSample = validationSample.classify(trainedClassifier)
validationAccuracy = validationSample.errorMatrix(label, 'classification')
validationAccuracy.getInfo()
```

```{code-cell} ipython3
validationAccuracy.accuracy().getInfo()
```

```{code-cell} ipython3
validationAccuracy.producersAccuracy().getInfo()
```

```{code-cell} ipython3
validationAccuracy.consumersAccuracy().getInfo()
```

```{code-cell} ipython3
import csv

with open("training.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(trainAccuracy.getInfo())

with open("validation.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(validationAccuracy.getInfo())
```

```{code-cell} ipython3
imgClassified = img.classify(trainedClassifier)
```

```{code-cell} ipython3
classVis = {
  'min': 0,
  'max': 10,
  'palette': ['006400' ,'ffbb22', 'ffff4c', 'f096ff', 'fa0000', 'b4b4b4',
            'f0f0f0', '0064c8', '0096a0', '00cf75', 'fae6a0']
}
Map.addLayer(lc, classVis, 'ESA Land Cover', False)
Map.addLayer(imgClassified, classVis, 'Classified')
Map.addLayer(trainingSample, {'color': 'black'}, 'Training sample')
Map.addLayer(validationSample, {'color': 'white'}, 'Validation sample')
Map.add_legend(title='Land Cover Type', builtin_legend='ESA_WorldCover')
Map.centerObject(img)
Map
```

## Using locally trained machine learning models

```{code-cell} ipython3
import pandas as pd
from geemap import ml
from sklearn import ensemble
```

### Train a model locally using scikit-learn

```{code-cell} ipython3
url = "https://raw.githubusercontent.com/gee-community/geemap/master/examples/data/rf_example.csv"
df = pd.read_csv(url)
df
```

```{code-cell} ipython3
feature_names = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7']
label = "landcover"
```

```{code-cell} ipython3
X = df[feature_names]
y = df[label]
n_trees = 10
rf = ensemble.RandomForestClassifier(n_trees).fit(X, y)
```

### Convert a sklearn classifier object to a list of strings

```{code-cell} ipython3
trees = ml.rf_to_strings(rf, feature_names)
```

```{code-cell} ipython3
print(len(trees))
```

```{code-cell} ipython3
print(trees[0])
```

### Convert sklearn classifier to GEE classifier

```{code-cell} ipython3
ee_classifier = ml.strings_to_classifier(trees)
ee_classifier.getInfo()
```

### Classify image using GEE classifier

```{code-cell} ipython3
# Make a cloud-free Landsat 8 TOA composite (from raw imagery).
l8 = ee.ImageCollection('LANDSAT/LC08/C01/T1')

image = ee.Algorithms.Landsat.simpleComposite(
    collection=l8.filterDate('2018-01-01', '2018-12-31'), asFloat=True
)
```

```{code-cell} ipython3
classified = image.select(feature_names).classify(ee_classifier)
```

```{code-cell} ipython3
Map = geemap.Map(center=(37.75, -122.25), zoom=11)

Map.addLayer(
    image,
    {"bands": ['B7', 'B5', 'B3'], "min": 0.05, "max": 0.55, "gamma": 1.5},
    'image',
)
Map.addLayer(
    classified,
    {"min": 0, "max": 2, "palette": ['red', 'green', 'blue']},
    'classification',
)
Map
```

### Save trees to the cloud

```{code-cell} ipython3
user_id = geemap.ee_user_id()
asset_id = user_id + "/random_forest_strings_test"
asset_id
```

```{code-cell} ipython3
ml.export_trees_to_fc(trees, asset_id)
```

```{code-cell} ipython3
rf_fc = ee.FeatureCollection(asset_id)
another_classifier = ml.fc_to_classifier(rf_fc)
classified = image.select(feature_names).classify(another_classifier)
```

### Save trees locally

```{code-cell} ipython3
out_csv = "trees.csv"
ml.trees_to_csv(trees, out_csv)
another_classifier = ml.csv_to_classifier(out_csv)
classified = image.select(feature_names).classify(another_classifier)
```

## Sankey diagrams

```{code-cell} ipython3
import sankee

sankee.datasets.LCMS_LC.sankify(
    years=[1990, 2000, 2010, 2020],
    region=ee.Geometry.Point([-122.192688, 46.25917]).buffer(2000),
    max_classes=3,
    title="Mount St. Helens Recovery",
)
```

```{code-cell} ipython3
Map = geemap.Map(height=650)
Map
```

## Summary

