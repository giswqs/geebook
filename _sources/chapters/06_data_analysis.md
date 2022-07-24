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

## Introduction

Click the **Open in Colab** button below to open this notebook in Google Colab:

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/giswqs/geebook/blob/master/chapters/06_data_analysis.ipynb)

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
import ee
import geemap
```

```{code-cell} ipython3
geemap.ee_initialize()
```

## Reducer

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
collection = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA') \
  .filterDate('2021-01-01', '2021-12-31') \
  .filter(ee.Filter.eq('WRS_PATH', 44)) \
  .filter(ee.Filter.eq('WRS_ROW', 34))

# Compute the median in each band, each pixel.
# Band names are B1_median, B2_median, etc.
median = collection.reduce(ee.Reducer.median())

# The output is an Image.  Add it to the map.
vis_param = {'bands': ['B5_median',  'B4_median',  'B3_median'], 'gamma': 2}
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
import geemap

Map = geemap.Map()

# Load an image and select some bands of interest.
image = ee.Image('LANDSAT/LC08/C01/T1/LC08_044034_20140318') \
    .select(['B4', 'B3', 'B2'])

# Reduce the image to get a one-band maximum value image.
maxValue = image.reduce(ee.Reducer.max())

# Display the result.
Map.centerObject(image, 8)
Map.addLayer(image, {}, 'Original image')
Map.addLayer(maxValue, {'max': 13000}, 'Maximum value image')
Map
```

### FeatureCollection reductions

```{code-cell} ipython3
Map = geemap.Map()

# Load US cenus data as a FeatureCollection.
census = ee.FeatureCollection('TIGER/2010/Blocks')

# Filter the collection to include only Benton County, OR.
benton = census.filter(
  ee.Filter.And(
    ee.Filter.eq('statefp10', '41'),
    ee.Filter.eq('countyfp10', '003')
  )
)

# Display Benton County cenus blocks.
Map.setCenter(-123.27, 44.57, 13)
Map.addLayer(benton)
Map
```

```{code-cell} ipython3
# Compute sums of the specified properties.
properties = ['pop10', 'housing10']
sums = benton \
    .filter(ee.Filter.notNull(properties)) \
    .reduceColumns(**{
      'reducer': ee.Reducer.sum().repeat(2),
      'selectors': properties
    })

# Print the resultant Dictionary.
print(sums.getInfo())
```

```{code-cell} ipython3
print(benton.aggregate_sum('pop10').getInfo())  # 85579
print(benton.aggregate_sum('housing10').getInfo())  #36245
```

```{code-cell} ipython3
benton.aggregate_stats('pop10').getInfo()
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
image.propertyNames().getInfo()
```

```{code-cell} ipython3
image.get('CLOUD_COVER').getInfo()  # 0.05
```

```{code-cell} ipython3
props = geemap.image_props(image)
props.getInfo()
```

```{code-cell} ipython3
stats = geemap.image_stats(image, scale=30)
stats.getInfo()
```

## Zonal statistics

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
geemap.zonal_stats(dem, states, out_dem_stats, statistics_type='MEAN', scale=1000, return_fc=False)
```

```{code-cell} ipython3
out_landsat_stats = 'landsat_stats.csv'
geemap.zonal_stats(
    landsat, states, out_landsat_stats, statistics_type='MEAN', scale=1000, return_fc=False
)
```

## Zonal statistics by group

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
    denominator=1000000,
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
    denominator=1000000,
    decimal_places=2,
)
```

## Zonal statistics with two images

```{code-cell} ipython3
import ee
import geemap
import geemap.colormaps as cm
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
Map
```

```{code-cell} ipython3
dem = ee.Image('USGS/3DEP/10m')
vis = {'min': 0, 'max': 4000, 'palette': cm.palettes.dem}
```

```{code-cell} ipython3
Map.addLayer(dem, vis, 'DEM')
```

```{code-cell} ipython3
landcover = ee.Image("USGS/NLCD_RELEASES/2019_REL/NLCD/2019").select('landcover')
```

```{code-cell} ipython3
Map.addLayer(landcover, {}, 'NLCD 2019')
Map.add_legend(builtin_legend='NLCD')
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

## Creating coordinate grids

```{code-cell} ipython3
Map = geemap.Map()
```

```{code-cell} ipython3
lat_grid = geemap.latitude_grid(step=5.0, west=-180, east=180, south=-85, north=85)
```

```{code-cell} ipython3
Map.addLayer(lat_grid, {}, 'Latitude Grid')
```

```{code-cell} ipython3
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
Map.addLayer(lon_grid, {}, 'Longitude Grid')
Map
```

```{code-cell} ipython3
grid = geemap.latlon_grid(
    lat_step=10, lon_step=10, west=-180, east=180, south=-85, north=85
)
```

```{code-cell} ipython3
Map = geemap.Map()
Map.addLayer(grid, {}, 'Coordinate Grid')
Map
```

## Creating fishnets

```{code-cell} ipython3
Map = geemap.Map()
Map

```

```{code-cell} ipython3
data = Map.user_roi

if data is None:
    data = ee.Geometry.BBox(-112.8089, 33.7306, -88.5951, 46.6244)
    Map.addLayer(data, {}, 'ROI')
    Map.user_roi = None

Map.centerObject(data)
```

```{code-cell} ipython3
fishnet = geemap.fishnet(data, h_interval=2.0, v_interval=2.0, delta=1)
```

```{code-cell} ipython3
Map.addLayer(fishnet, {}, 'Fishnet 1')
```

```{code-cell} ipython3
data = Map.user_roi

if data is None:
    data = ee.Geometry.Polygon(
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
    Map.addLayer(data, {}, 'ROI2')

Map.centerObject(data)
Map
```

```{code-cell} ipython3
fishnet = geemap.fishnet(data, rows=6, cols=8, delta=1)
```

```{code-cell} ipython3
Map.addLayer(fishnet, {}, 'Fishnet 2')
```

## Sankey diagrams

```{code-cell} ipython3
import sankee
sankee.datasets.LCMS_LC.sankify(
  years=[1990, 2000, 2010, 2020],
  region=ee.Geometry.Point([-122.192688, 46.25917]).buffer(2000),
  max_classes=3,
  title="Mount St. Helens Recovery"
)
```

```{code-cell} ipython3
Map = geemap.Map(height=650)
Map
```

## Mapping available imagery

```{code-cell} ipython3
import geemap.colormaps as cm
```

```{code-cell} ipython3
collection = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
image = geemap.image_count(
    collection, region=None, start_date='2021-01-01', end_date='2022-01-01', clip=False
)
```

```{code-cell} ipython3
Map = geemap.Map()
vis = {'min': 0, 'max': 60, 'palette': cm.palettes.coolwarm}
Map.addLayer(image, vis, 'Landsat 8 Image Count')

countries = ee.FeatureCollection('users/giswqs/public/countries')
style = {"color": "00000088", "width": 1, "fillColor": "00000000"}
Map.addLayer(countries.style(**style), {}, "Countries")
Map.add_colorbar(vis, label='Landsat 8 Image Count')

Map
```

## Using Landsat 9

```{code-cell} ipython3
# !pip install geemap
```

```{code-cell} ipython3
import ee
import geemap
```

```{code-cell} ipython3
Map = geemap.Map()
```

```{code-cell} ipython3
collection = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
print(collection.size().getInfo())
```

```{code-cell} ipython3
median = collection.median()
```

```{code-cell} ipython3
def apply_scale_factors(image):
    opticalBands = image.select('SR_B.').multiply(0.0000275).add(-0.2)
    thermalBands = image.select('ST_B.*').multiply(0.00341802).add(149.0)
    return image.addBands(opticalBands, None, True).addBands(thermalBands, None, True)
```

```{code-cell} ipython3
:tags: []

dataset = apply_scale_factors(median)
```

```{code-cell} ipython3
vis_natural = {
    'bands': ['SR_B4', 'SR_B3', 'SR_B2'],
    'min': 0.0,
    'max': 0.3,
}

vis_nir = {
    'bands': ['SR_B5', 'SR_B4', 'SR_B3'],
    'min': 0.0,
    'max': 0.3,
}
```

```{code-cell} ipython3
Map.addLayer(dataset, vis_natural, 'True color (432)')
Map.addLayer(dataset, vis_nir, 'Color infrared (543)')
Map
```

```{code-cell} ipython3
vis_params = [
    {'bands': ['SR_B4', 'SR_B3', 'SR_B2'], 'min': 0, 'max': 0.3},
    {'bands': ['SR_B5', 'SR_B4', 'SR_B3'], 'min': 0, 'max': 0.3},
    {'bands': ['SR_B7', 'SR_B6', 'SR_B4'], 'min': 0, 'max': 0.3},
    {'bands': ['SR_B6', 'SR_B5', 'SR_B2'], 'min': 0, 'max': 0.3},
]
```

```{code-cell} ipython3
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
    height="400px",
    center=[40, -100],
    zoom=4,
    ee_objects=[dataset],
    vis_params=vis_params,
    labels=labels,
    label_position="topright",
)
```

```{code-cell} ipython3
landsat8 = ee.Image('LANDSAT/LC08/C02/T1_L2/LC08_015043_20130402')
landsat9 = ee.Image('LANDSAT/LC09/C02/T1_L2/LC09_015043_20211231')
```

```{code-cell} ipython3
landsat8 = apply_scale_factors(landsat8)
landsat9 = apply_scale_factors(landsat9)
```

```{code-cell} ipython3
left_layer = geemap.ee_tile_layer(landsat8, vis_natural, 'Landsat 8')
right_layer = geemap.ee_tile_layer(landsat9, vis_natural, 'Landsat 9')
```

```{code-cell} ipython3
Map = geemap.Map()
Map.split_map(left_layer, right_layer)
Map
```

## Interactive region reduction

### Import libraries

```{code-cell} ipython3
import os
import ee
import geemap
```

### Create an interactive map

```{code-cell} ipython3
m = geemap.Map()
```

### Add add to the map

```{code-cell} ipython3
collection = (
    ee.ImageCollection('MODIS/006/MOD13A2')
    .filterDate('2015-01-01', '2019-12-31')
    .select('NDVI')
)

# Convert the image collection to an image.
image = collection.toBands()

ndvi_vis = {
    'min': 0.0,
    'max': 9000.0,
    'palette': [
        'FFFFFF',
        'CE7E45',
        'DF923D',
        'F1B555',
        'FCD163',
        '99B718',
        '74A901',
        '66A000',
        '529400',
        '3E8601',
        '207401',
        '056201',
        '004C00',
        '023B01',
        '012E01',
        '011D01',
        '011301',
    ],
}

m.addLayer(image, {}, 'MODIS NDVI Time-series')
m.addLayer(image.select(0), ndvi_vis, 'MODIS NDVI VIS')

m
```

### Set reducer

```{code-cell} ipython3
m.set_plot_options(add_marker_cluster=True, marker=None)
m.roi_reducer = ee.Reducer.mean()
```

### Export data

```{code-cell} ipython3
out_dir = os.path.join(os.path.expanduser('~'), 'Downloads')
# out_csv = os.path.join(out_dir, 'points.csv')
out_shp = os.path.join(out_dir, 'ndvi.shp')
m.extract_values_to_points(out_shp)
```

## Extracting values to points

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
# Add Earth Engine dataset
dem = ee.Image('USGS/SRTMGL1_003')
landsat7 = ee.Image('LANDSAT/LE7_TOA_5YEAR/1999_2003')

# Set visualization parameters.
vis_params = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}

# Add Earth Engine layers to Map
Map.addLayer(
    landsat7, {'bands': ['B4', 'B3', 'B2'], 'min': 20, 'max': 200}, 'Landsat 7'
)
Map.addLayer(dem, vis_params, 'SRTM DEM', True, 1)
```

```{code-cell} ipython3
work_dir = os.path.expanduser('~/Downloads')
in_shp = os.path.join(work_dir, 'us_cities.shp')
if not os.path.exists(in_shp):
    data_url = 'https://github.com/giswqs/data/raw/main/us/us_cities.zip'
    geemap.download_from_url(data_url, out_dir=work_dir)
```

```{code-cell} ipython3
in_fc = geemap.shp_to_ee(in_shp)
Map.addLayer(in_fc, {}, 'Cities')
```

```{code-cell} ipython3
out_shp = os.path.join(work_dir, 'dem.shp')
geemap.extract_values_to_points(in_fc, dem, out_shp)
```

```{code-cell} ipython3
out_csv = os.path.join(work_dir, 'landsat.csv')
geemap.extract_values_to_points(in_fc, landsat7, out_csv)
```

## Extracting pixels along transect

```{code-cell} ipython3
from bqplot import pyplot as plt
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
Map.add_basemap("TERRAIN")
Map
```

```{code-cell} ipython3
image = ee.Image('USGS/SRTMGL1_003')
vis_params = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}
Map.addLayer(image, vis_params, 'SRTM DEM', True, 0.5)
```

```{code-cell} ipython3
# Use the drawing tool to draw any line on the map.
line = Map.user_roi
if line is None:
    line = ee.Geometry.LineString(
        [[-120.223279, 36.314849], [-118.926969, 36.712192], [-117.202217, 36.756215]]
    )
    Map.addLayer(line, {}, "ROI")
Map.centerObject(line)
```

```{code-cell} ipython3
line.getInfo()
```

```{code-cell} ipython3
reducer = 'mean'  # Any ee.Reducer, e.g., mean, median, min, max, stdDev
transect = geemap.extract_transect(
    image, line, n_segments=100, reducer=reducer, to_pandas=True
)
```

```{code-cell} ipython3
transect
```

```{code-cell} ipython3
transect.to_csv('transect.csv')
```

```{code-cell} ipython3
fig = plt.figure()
plt.plot(transect['distance'], transect[reducer])
plt.xlabel('Distance')
plt.ylabel("Elevation")
plt.show()
```

## Unsupervised classification

### Unsupervised classification algorithms available in Earth Engine

### Step-by-step tutorial

#### Import libraries

```{code-cell} ipython3
import ee
import geemap
```

#### Create an interactive map

```{code-cell} ipython3
Map = geemap.Map()
Map
```

#### Add data to the map

```{code-cell} ipython3
# point = ee.Geometry.Point([-122.4439, 37.7538])
point = ee.Geometry.Point([-87.7719, 41.8799])

image = (
    ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(point)
    .filterDate('2019-01-01', '2019-12-31')
    .sort('CLOUD_COVER')
    .first()
    .select('B[1-7]')
)

vis_params = {'min': 0, 'max': 3000, 'bands': ['B5', 'B4', 'B3']}

Map.centerObject(point, 8)
Map.addLayer(image, vis_params, "Landsat-8")
```

#### Check image properties

```{code-cell} ipython3
props = geemap.image_props(image)
props.getInfo()
```

```{code-cell} ipython3
props.get('IMAGE_DATE').getInfo()
```

```{code-cell} ipython3
props.get('CLOUD_COVER').getInfo()
```

#### Make training dataset

```{code-cell} ipython3
# region = Map.user_roi
# region = ee.Geometry.Rectangle([-122.6003, 37.4831, -121.8036, 37.8288])
# region = ee.Geometry.Point([-122.4439, 37.7538]).buffer(10000)
```

```{code-cell} ipython3
# Make the training dataset.
training = image.sample(
    **{
        #     'region': region,
        'scale': 30,
        'numPixels': 5000,
        'seed': 0,
        'geometries': True,  # Set this to False to ignore geometries
    }
)

Map.addLayer(training, {}, 'training', False)
Map
```

#### Train the clusterer

```{code-cell} ipython3
# Instantiate the clusterer and train it.
n_clusters = 5
clusterer = ee.Clusterer.wekaKMeans(n_clusters).train(training)
```

#### Classify the image

```{code-cell} ipython3
# Cluster the input using the trained clusterer.
result = image.cluster(clusterer)

# # Display the clusters with random colors.
Map.addLayer(result.randomVisualizer(), {}, 'clusters')
Map
```

#### Label the clusters

```{code-cell} ipython3
legend_keys = ['One', 'Two', 'Three', 'Four', 'ect']
legend_colors = ['#8DD3C7', '#FFFFB3', '#BEBADA', '#FB8072', '#80B1D3']

# Reclassify the map
result = result.remap([0, 1, 2, 3, 4], [1, 2, 3, 4, 5])

Map.addLayer(
    result, {'min': 1, 'max': 5, 'palette': legend_colors}, 'Labelled clusters'
)
Map.add_legend(
    legend_keys=legend_keys, legend_colors=legend_colors, position='bottomright'
)
Map
```

#### Visualize the result

```{code-cell} ipython3
print('Change layer opacity:')
cluster_layer = Map.layers[-1]
cluster_layer.interact(opacity=(0, 1, 0.1))
```

#### Export the result

```{code-cell} ipython3
import os

out_dir = os.path.join(os.path.expanduser('~'), 'Downloads')
out_file = os.path.join(out_dir, 'cluster.tif')
```

```{code-cell} ipython3
geemap.ee_export_image(result, filename=out_file, scale=90)
```

```{code-cell} ipython3
geemap.ee_export_image_to_drive(
    result, description='clusters', folder='export', scale=90
)
```

## Supervised classification

### Supervised classification algorithms available in Earth Engine

### Step-by-step tutorial

#### Import libraries

```{code-cell} ipython3
import ee
import geemap
```

#### Create an interactive map

```{code-cell} ipython3
Map = geemap.Map()
Map
```

#### Add data to the map

```{code-cell} ipython3
point = ee.Geometry.Point([-122.4439, 37.7538])
# point = ee.Geometry.Point([-87.7719, 41.8799])

image = (
    ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(point)
    .filterDate('2016-01-01', '2016-12-31')
    .sort('CLOUD_COVER')
    .first()
    .select('B[1-7]')
)

vis_params = {'min': 0, 'max': 3000, 'bands': ['B5', 'B4', 'B3']}

Map.centerObject(point, 8)
Map.addLayer(image, vis_params, "Landsat-8")
```

#### Check image properties

```{code-cell} ipython3
ee.Date(image.get('system:time_start')).format('YYYY-MM-dd').getInfo()
```

```{code-cell} ipython3
image.get('CLOUD_COVER').getInfo()
```

#### Make training dataset

```{code-cell} ipython3
# region = Map.user_roi
# region = ee.Geometry.Rectangle([-122.6003, 37.4831, -121.8036, 37.8288])
# region = ee.Geometry.Point([-122.4439, 37.7538]).buffer(10000)
```

```{code-cell} ipython3
nlcd = ee.Image('USGS/NLCD/NLCD2016').select('landcover').clip(image.geometry())
Map.addLayer(nlcd, {}, 'NLCD')
Map
```

```{code-cell} ipython3
# Make the training dataset.
points = nlcd.sample(
    **{
        'region': image.geometry(),
        'scale': 30,
        'numPixels': 5000,
        'seed': 0,
        'geometries': True,  # Set this to False to ignore geometries
    }
)

Map.addLayer(points, {}, 'training', False)
```

```{code-cell} ipython3
print(points.size().getInfo())
```

```{code-cell} ipython3
print(points.first().getInfo())
```

#### Train the classifier

```{code-cell} ipython3
# Use these bands for prediction.
bands = ['B1', 'B2', 'B3', 'B4', 'B5', 'B6', 'B7']


# This property of the table stores the land cover labels.
label = 'landcover'

# Overlay the points on the imagery to get training.
training = image.select(bands).sampleRegions(
    **{'collection': points, 'properties': [label], 'scale': 30}
)

# Train a CART classifier with default parameters.
trained = ee.Classifier.smileCart().train(training, label, bands)
```

```{code-cell} ipython3
print(training.first().getInfo())
```

#### Classify the image

```{code-cell} ipython3
# Classify the image with the same bands used for training.
result = image.select(bands).classify(trained)

# # Display the clusters with random colors.
Map.addLayer(result.randomVisualizer(), {}, 'classified')
Map
```

#### Render categorical map

```{code-cell} ipython3
class_values = nlcd.get('landcover_class_values').getInfo()
class_values
```

```{code-cell} ipython3
class_palette = nlcd.get('landcover_class_palette').getInfo()
class_palette
```

```{code-cell} ipython3
landcover = result.set('classification_class_values', class_values)
landcover = landcover.set('classification_class_palette', class_palette)
```

```{code-cell} ipython3
Map.addLayer(landcover, {}, 'Land cover')
Map
```

#### Visualize the result

```{code-cell} ipython3
print('Change layer opacity:')
cluster_layer = Map.layers[-1]
cluster_layer.interact(opacity=(0, 1, 0.1))
```

#### Add a legend to the map

```{code-cell} ipython3
Map.add_legend(builtin_legend='NLCD')
Map
```

#### Export the result

```{code-cell} ipython3
import os

out_dir = os.path.join(os.path.expanduser('~'), 'Downloads')
out_file = os.path.join(out_dir, 'landcover.tif')
```

```{code-cell} ipython3
geemap.ee_export_image(landcover, filename=out_file, scale=900)
```

```{code-cell} ipython3
geemap.ee_export_image_to_drive(
    landcover, description='landcover', folder='export', scale=900
)
```

## Accuracy assessment

### Supervised classification algorithms available in Earth Engine

### Step-by-step tutorial

#### Import libraries

```{code-cell} ipython3
import ee
import geemap
```

#### Create an interactive map

```{code-cell} ipython3
Map = geemap.Map()
Map
```

#### Add data to the map

```{code-cell} ipython3
NLCD2016 = ee.Image('USGS/NLCD/NLCD2016').select('landcover')
Map.addLayer(NLCD2016, {}, 'NLCD 2016')
```

```{code-cell} ipython3
NLCD_metadata = ee.FeatureCollection("users/giswqs/landcover/NLCD2016_metadata")
Map.addLayer(NLCD_metadata, {}, 'NLCD Metadata')
```

```{code-cell} ipython3
# point = ee.Geometry.Point([-122.4439, 37.7538])  # Sanfrancisco, CA
# point = ee.Geometry.Point([-83.9293, 36.0526])   # Knoxville, TN
point = ee.Geometry.Point([-88.3070, 41.7471])  # Chicago, IL
```

```{code-cell} ipython3
metadata = NLCD_metadata.filterBounds(point).first()
region = metadata.geometry()
```

```{code-cell} ipython3
metadata.get('2016on_bas').getInfo()
```

```{code-cell} ipython3
doy = metadata.get('2016on_bas').getInfo().replace('LC08_', '')
doy
```

```{code-cell} ipython3
ee.Date.parse('YYYYDDD', doy).format('YYYY-MM-dd').getInfo()
```

```{code-cell} ipython3
start_date = ee.Date.parse('YYYYDDD', doy)
end_date = start_date.advance(1, 'day')
```

```{code-cell} ipython3
image = (
    ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(point)
    .filterDate(start_date, end_date)
    .first()
    .select('B[1-7]')
    .clip(region)
)

vis_params = {'min': 0, 'max': 3000, 'bands': ['B5', 'B4', 'B3']}

Map.centerObject(point, 8)
Map.addLayer(image, vis_params, "Landsat-8")
Map
```

```{code-cell} ipython3
nlcd_raw = NLCD2016.clip(region)
Map.addLayer(nlcd_raw, {}, 'NLCD')
```

#### Prepare for consecutive class labels

```{code-cell} ipython3
raw_class_values = nlcd_raw.get('landcover_class_values').getInfo()
print(raw_class_values)
```

```{code-cell} ipython3
n_classes = len(raw_class_values)
new_class_values = list(range(0, n_classes))
new_class_values
```

```{code-cell} ipython3
class_palette = nlcd_raw.get('landcover_class_palette').getInfo()
print(class_palette)
```

```{code-cell} ipython3
nlcd = nlcd_raw.remap(raw_class_values, new_class_values).select(
    ['remapped'], ['landcover']
)
nlcd = nlcd.set('landcover_class_values', new_class_values)
nlcd = nlcd.set('landcover_class_palette', class_palette)
```

```{code-cell} ipython3
Map.addLayer(nlcd, {}, 'NLCD')
Map
```

#### Make training data

```{code-cell} ipython3
# Make the training dataset.
points = nlcd.sample(
    **{
        'region': region,
        'scale': 30,
        'numPixels': 5000,
        'seed': 0,
        'geometries': True,  # Set this to False to ignore geometries
    }
)

Map.addLayer(points, {}, 'training', False)
```

```{code-cell} ipython3
print(points.size().getInfo())
```

```{code-cell} ipython3
print(points.first().getInfo())
```

#### Split training and testing

```{code-cell} ipython3
# Use these bands for prediction.
bands = ['B1', 'B2', 'B3', 'B4', 'B5', 'B6', 'B7']

# This property of the table stores the land cover labels.
label = 'landcover'

# Overlay the points on the imagery to get training.
sample = image.select(bands).sampleRegions(
    **{'collection': points, 'properties': [label], 'scale': 30}
)

# Adds a column of deterministic pseudorandom numbers.
sample = sample.randomColumn()

split = 0.7

training = sample.filter(ee.Filter.lt('random', split))
validation = sample.filter(ee.Filter.gte('random', split))
```

```{code-cell} ipython3
training.first().getInfo()
```

```{code-cell} ipython3
validation.first().getInfo()
```

#### Train the classifier

```{code-cell} ipython3
classifier = ee.Classifier.smileRandomForest(10).train(training, label, bands)
```

#### Classify the image

```{code-cell} ipython3
# Classify the image with the same bands used for training.
result = image.select(bands).classify(classifier)

# # Display the clusters with random colors.
Map.addLayer(result.randomVisualizer(), {}, 'classfied')
Map
```

#### Render categorical map

```{code-cell} ipython3
class_values = nlcd.get('landcover_class_values').getInfo()
print(class_values)
```

```{code-cell} ipython3
class_palette = nlcd.get('landcover_class_palette').getInfo()
print(class_palette)
```

```{code-cell} ipython3
landcover = result.set('classification_class_values', class_values)
landcover = landcover.set('classification_class_palette', class_palette)
```

```{code-cell} ipython3
Map.addLayer(landcover, {}, 'Land cover')
Map
```

#### Visualize the result

```{code-cell} ipython3
print('Change layer opacity:')
cluster_layer = Map.layers[-1]
cluster_layer.interact(opacity=(0, 1, 0.1))
```

#### Add a legend to the map

```{code-cell} ipython3
Map.add_legend(builtin_legend='NLCD')
Map
```

#### Accuracy assessment

##### Training dataset

```{code-cell} ipython3
train_accuracy = classifier.confusionMatrix()
```

```{code-cell} ipython3
train_accuracy.getInfo()
```

```{code-cell} ipython3
train_accuracy.accuracy().getInfo()
```

```{code-cell} ipython3
train_accuracy.kappa().getInfo()
```

```{code-cell} ipython3
train_accuracy.producersAccuracy().getInfo()
```

```{code-cell} ipython3
train_accuracy.consumersAccuracy().getInfo()
```

##### Validation dataset

```{code-cell} ipython3
validated = validation.classify(classifier)
```

```{code-cell} ipython3
validated.first().getInfo()
```

```{code-cell} ipython3
test_accuracy = validated.errorMatrix('landcover', 'classification')
```

```{code-cell} ipython3
test_accuracy.getInfo()
```

```{code-cell} ipython3
test_accuracy.accuracy().getInfo()
```

```{code-cell} ipython3
test_accuracy.kappa().getInfo()
```

```{code-cell} ipython3
test_accuracy.producersAccuracy().getInfo()
```

```{code-cell} ipython3
test_accuracy.consumersAccuracy().getInfo()
```

#### Download confusion matrix

```{code-cell} ipython3
import csv
import os

out_dir = os.path.join(os.path.expanduser('~'), 'Downloads')
training_csv = os.path.join(out_dir, 'train_accuracy.csv')
testing_csv = os.path.join(out_dir, 'test_accuracy.csv')

with open(training_csv, "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(train_accuracy.getInfo())

with open(testing_csv, "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(test_accuracy.getInfo())
```

#### Reclassify land cover map

```{code-cell} ipython3
landcover = landcover.remap(new_class_values, raw_class_values).select(
    ['remapped'], ['classification']
)
```

```{code-cell} ipython3
landcover = landcover.set('classification_class_values', raw_class_values)
landcover = landcover.set('classification_class_palette', class_palette)
```

```{code-cell} ipython3
Map.addLayer(landcover, {}, 'Final land cover')
Map
```

#### Export the result

```{code-cell} ipython3
import os

out_dir = os.path.join(os.path.expanduser('~'), 'Downloads')
out_file = os.path.join(out_dir, 'landcover.tif')
```

```{code-cell} ipython3
geemap.ee_export_image(landcover, filename=out_file, scale=900)
```

```{code-cell} ipython3
geemap.ee_export_image_to_drive(
    landcover, description='landcover', folder='export', scale=900
)
```

## Crearting training samples

```{code-cell} ipython3
import ee
import geemap
```

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
if Map.user_rois is not None:
    training_samples = Map.user_rois
    print(training_samples.getInfo())
```

## Using locally trained machine learning models

```{code-cell} ipython3
import ee
import geemap
import pandas as pd

from geemap import ml
from sklearn import ensemble
```

```{code-cell} ipython3
geemap.ee_initialize()
```

### Train a model locally using scikit-learn

```{code-cell} ipython3
# read the feature table to train our RandomForest model
# data taken from ee.FeatureCollection('GOOGLE/EE/DEMOS/demo_landcover_labels')

url = "https://raw.githubusercontent.com/giswqs/geemap/master/examples/data/rf_example.csv"
df = pd.read_csv(url)
```

```{code-cell} ipython3
df
```

```{code-cell} ipython3
# specify the names of the features (i.e. band names) and label
# feature names used to extract out features and define what bands

feature_names = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7']
label = "landcover"
```

```{code-cell} ipython3
# get the features and labels into separate variables
X = df[feature_names]
y = df[label]
```

```{code-cell} ipython3
# create a classifier and fit
n_trees = 10
rf = ensemble.RandomForestClassifier(n_trees).fit(X, y)
```

### Convert a sklearn classifier object to a list of strings

```{code-cell} ipython3
# convert the estimator into a list of strings
# this function also works with the ensemble.ExtraTrees estimator
trees = ml.rf_to_strings(rf, feature_names)
```

```{code-cell} ipython3
# print the first tree to see the result
print(trees[0])
```

```{code-cell} ipython3
print(trees[1])
```

```{code-cell} ipython3
# number of trees we converted should equal the number of trees we defined for the model
len(trees) == n_trees
```

### Convert sklearn classifier to GEE classifier

```{code-cell} ipython3
# create a ee classifier to use with ee objects from the trees
ee_classifier = ml.strings_to_classifier(trees)
```

```{code-cell} ipython3
# ee_classifier.getInfo()
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
# classify the image using the classifier we created from the local training
# note: here we select the feature_names from the image that way the classifier knows which bands to use
classified = image.select(feature_names).classify(ee_classifier)
```

```{code-cell} ipython3
# display results
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
user_id
```

```{code-cell} ipython3
# specify asset id where to save trees
# be sure to change <user_name> to your ee user name
asset_id = user_id + "/random_forest_strings_test"
asset_id
```

```{code-cell} ipython3
# kick off an export process so it will be saved to the ee asset
ml.export_trees_to_fc(trees, asset_id)

# this will kick off an export task, so wait a few minutes before moving on
```

```{code-cell} ipython3
# read the exported tree feature collection
rf_fc = ee.FeatureCollection(asset_id)

# convert it to a classifier, very similar to the `ml.trees_to_classifier` function
another_classifier = ml.fc_to_classifier(rf_fc)

# classify the image again but with the classifier from the persisted trees
classified = image.select(feature_names).classify(another_classifier)
```

```{code-cell} ipython3
# display results
# we should get the exact same results as before
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

### Save trees locally

```{code-cell} ipython3
import os

out_csv = os.path.expanduser("~/Downloads/trees.csv")
```

```{code-cell} ipython3
ml.trees_to_csv(trees, out_csv)
```

```{code-cell} ipython3
another_classifier = ml.csv_to_classifier(out_csv)
```

```{code-cell} ipython3
classified = image.select(feature_names).classify(another_classifier)
```

```{code-cell} ipython3
# display results
# we should get the exact same results as before
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

## Timeseries analysis

### Quality mosaic

### Import libraries

```{code-cell} ipython3
import ee
import geemap
```

### Create an interactive map

```{code-cell} ipython3
Map = geemap.Map()
Map
```

### Define a region of interest (ROI)

```{code-cell} ipython3
countries = ee.FeatureCollection('users/giswqs/public/countries')
Map.addLayer(countries, {}, 'coutries')
```

```{code-cell} ipython3
roi = countries.filter(ee.Filter.eq('id', 'USA'))
Map.addLayer(roi, {}, 'roi')
```

### Filter ImageCollection

```{code-cell} ipython3
start_date = '2019-01-01'
end_date = '2019-12-31'

l8 = (
    ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA')
    .filterBounds(roi)
    .filterDate(start_date, end_date)
)
```

```{code-cell} ipython3
# print(l8.size().getInfo())
```

### Create a median composite

```{code-cell} ipython3
median = l8.median()

visParams = {
    'bands': ['B4', 'B3', 'B2'],
    'min': 0,
    'max': 0.4,
}

Map.addLayer(median, visParams, 'Median')
```

### Define functions to add time bands

```{code-cell} ipython3
def addNDVI(image):
    ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI')
    return image.addBands(ndvi)
```

```{code-cell} ipython3
def addDate(image):
    img_date = ee.Date(image.date())
    img_date = ee.Number.parse(img_date.format('YYYYMMdd'))
    return image.addBands(ee.Image(img_date).rename('date').toInt())
```

```{code-cell} ipython3
def addMonth(image):
    img_date = ee.Date(image.date())
    img_doy = ee.Number.parse(img_date.format('M'))
    return image.addBands(ee.Image(img_doy).rename('month').toInt())
```

```{code-cell} ipython3
def addDOY(image):
    img_date = ee.Date(image.date())
    img_doy = ee.Number.parse(img_date.format('D'))
    return image.addBands(ee.Image(img_doy).rename('doy').toInt())
```

### Map over an ImageCollection

```{code-cell} ipython3
withNDVI = l8.map(addNDVI).map(addDate).map(addMonth).map(addDOY)
```

### Create a quality mosaic

```{code-cell} ipython3
greenest = withNDVI.qualityMosaic('NDVI')
```

```{code-cell} ipython3
greenest.bandNames().getInfo()
```

### Display the max value band

```{code-cell} ipython3
ndvi = greenest.select('NDVI')
palette = [
    '#d73027',
    '#f46d43',
    '#fdae61',
    '#fee08b',
    '#d9ef8b',
    '#a6d96a',
    '#66bd63',
    '#1a9850',
]
Map.addLayer(ndvi, {'palette': palette}, 'NDVI')
```

```{code-cell} ipython3
Map.addLayer(greenest, visParams, 'Greenest pixel')
Map
```

### Display time bands

```{code-cell} ipython3
Map.addLayer(
    greenest.select('month'),
    {'palette': ['red', 'blue'], 'min': 1, 'max': 12},
    'Greenest month',
)
```

```{code-cell} ipython3
Map.addLayer(
    greenest.select('doy'),
    {'palette': ['brown', 'green'], 'min': 1, 'max': 365},
    'Greenest doy',
)
```

## Interactive charts

```{code-cell} ipython3
import ee
import geemap
import geemap.chart as chart
```

```{code-cell} ipython3
# geemap.update_package()
```

### Creating a chart from ee.FeatureCollection by feature

```{code-cell} ipython3
Map = geemap.Map()

features = ee.FeatureCollection('projects/google/charts_feature_example').select(
    '[0-9][0-9]_tmean|label'
)

Map.addLayer(features, {}, "Ecoregions")
Map
```

```{code-cell} ipython3
df = geemap.ee_to_pandas(features)
df
```

```{code-cell} ipython3
xProperty = "label"
yProperties = [str(x).zfill(2) + "_tmean" for x in range(1, 13)]

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

### Creating a chart from ee.FeatureCollection by property

```{code-cell} ipython3
Map = geemap.Map()

features = ee.FeatureCollection('projects/google/charts_feature_example').select(
    '[0-9][0-9]_ppt|label'
)

Map.addLayer(features, {}, 'Features')
Map
```

```{code-cell} ipython3
df = geemap.ee_to_pandas(features)
df
```

```{code-cell} ipython3
keys = [str(x).zfill(2) + "_ppt" for x in range(1, 13)]
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
```

```{code-cell} ipython3
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

### Histogram

```{code-cell} ipython3
import ee
import geemap
import geemap.chart as chart
```

```{code-cell} ipython3
geemap.ee_initialize()
```

```{code-cell} ipython3
source = ee.ImageCollection('OREGONSTATE/PRISM/Norm81m').toBands()
region = ee.Geometry.Rectangle(-123.41, 40.43, -116.38, 45.14)
my_sample = source.sample(region, 5000)
property = '07_ppt'
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
chart.feature_histogram(my_sample, property, **options)
```

```{code-cell} ipython3
chart.feature_histogram(my_sample, property, maxBuckets=30, **options)
```

```{code-cell} ipython3
chart.feature_histogram(my_sample, property, minBucketWidth=0.5, **options)
```

```{code-cell} ipython3
chart.feature_histogram(my_sample, property, minBucketWidth=3, maxBuckets=30, **options)
```

## Global land cover area

```{code-cell} ipython3
Map = geemap.Map()
dataset = ee.ImageCollection("ESA/WorldCover/v100").first()
Map.addLayer(dataset, {'bands': ['Map']}, 'ESA Land Cover')
Map.add_legend(builtin_legend='ESA_WorldCover')
Map
```

```{code-cell} ipython3
df = geemap.image_area_by_group(
    dataset, scale=1000, denominator=1e6, decimal_places=4, verbose=True
)
df
```

```{code-cell} ipython3
df.to_csv('esa_area.csv')
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
Map.add_basemap('HYBRID')

nlcd = ee.Image('USGS/NLCD_RELEASES/2019_REL/NLCD/2019')
landcover = nlcd.select('landcover')

Map.addLayer(landcover, {}, 'NLCD Land Cover 2019')
Map.add_legend(
    title="NLCD Land Cover Classification", builtin_legend='NLCD', height='465px'
)
Map
```

```{code-cell} ipython3
df = geemap.image_area_by_group(
    landcover, scale=1000, denominator=1e6, decimal_places=4, verbose=True
)
df
```

```{code-cell} ipython3
df.to_csv('nlcd_area.csv')
```

## Whitebox-Tools

```{code-cell} ipython3
import os
import geemap
import whiteboxgui
```

```{code-cell} ipython3
out_dir = os.path.expanduser('~/Downloads')
dem = os.path.join(out_dir, 'dem.tif')

if not os.path.exists(dem):
    dem_url = 'https://drive.google.com/file/d/1vRkAWQYsLWCi6vcTMk8vLxoXMFbdMFn8/view?usp=sharing'
    geemap.download_from_gdrive(dem_url, 'dem.tif', out_dir, unzip=False)
```

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
whiteboxgui.show()
```

```{code-cell} ipython3
whiteboxgui.show(tree=True)
```

## Summary

## References

