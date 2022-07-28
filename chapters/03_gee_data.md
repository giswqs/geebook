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

# Using Earth Engine Data

```{contents}
:local:
:depth: 3
```

## Introduction

+++

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

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/giswqs/geebook/blob/master/chapters/02_maps.ipynb)

```{code-cell} ipython3
# pip install pygis
```

```{code-cell} ipython3
import ee
import geemap
```

```{code-cell} ipython3
geemap.ee_initialize()
```

## Earth Engine data types

### Image

#### Loading Earth Engine images

```{code-cell} ipython3
image = ee.Image('USGS/SRTMGL1_003')
```

#### Visualizing Earth Engine images

```{code-cell} ipython3
Map = geemap.Map(center=[21.79, 70.87], zoom=3)
image = ee.Image('USGS/SRTMGL1_003')
vis_params = {
    'min': 0,
    'max': 6000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}
Map.addLayer(image, vis_params, 'SRTM')
Map
```

#### Loading Cloud GeoTIFFs

```{code-cell} ipython3
Map = geemap.Map()
URL = 'https://bit.ly/3aSZ0fH'
image = geemap.load_GeoTIFF(URL)
vis = {
    "min": 3000,
    "max": 13500,
    "bands": ["B3", "B2", "B1"],
}
Map.addLayer(image, vis, 'Cloud GeoTIFF')
Map.centerObject(image)
Map
```

```{code-cell} ipython3
B3 = 'gs://gcp-public-data-landsat/LC08/01/044/034/LC08_L1TP_044034_20131228_20170307_01_T1/LC08_L1TP_044034_20131228_20170307_01_T1_B3.TIF'
B4 = 'gs://gcp-public-data-landsat/LC08/01/044/034/LC08_L1TP_044034_20131228_20170307_01_T1/LC08_L1TP_044034_20131228_20170307_01_T1_B4.TIF'
B5 = 'gs://gcp-public-data-landsat/LC08/01/044/034/LC08_L1TP_044034_20131228_20170307_01_T1/LC08_L1TP_044034_20131228_20170307_01_T1_B5.TIF'
```

```{code-cell} ipython3
URLs = [B3, B4, B5]
collection = geemap.load_GeoTIFFs(URLs)
image = collection.toBands().rename(['Green', 'Red', 'NIR']).selfMask()
```

```{code-cell} ipython3
Map = geemap.Map()
vis = {'bands': ['NIR', 'Red', 'Green'], 'min': 100, 'max': 12000, 'gamma': 0.8}
Map.addLayer(image, vis, 'Image')
Map.centerObject(image.geometry(), 8)
Map
```

### ImageCollection

#### Loading image collections

```{code-cell} ipython3
collection = ee.ImageCollection('COPERNICUS/S2_SR')
```

#### Visualizing image collections

```{code-cell} ipython3
Map = geemap.Map()
collection = ee.ImageCollection('COPERNICUS/S2_SR')
image = collection.median()

vis = {
    'min': 0.0,
    'max': 3000,
    'bands': ['B4', 'B3', 'B2'],
}

Map.setCenter(83.277, 17.7009, 12)
Map.addLayer(image, vis, 'Sentinel-2')
Map
```

#### Filtering image collections

```{code-cell} ipython3
Map = geemap.Map()
collection = (
    ee.ImageCollection('COPERNICUS/S2_SR')
    .filterDate('2021-01-01', '2022-01-01')
    .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 5))
)
image = collection.median()

vis = {
    'min': 0.0,
    'max': 3000,
    'bands': ['B4', 'B3', 'B2'],
}

Map.setCenter(83.277, 17.7009, 12)
Map.addLayer(image, vis, 'Sentinel-2')
Map
```

### Geometry

#### Geometry types

#### Creating Geometry objects

```{code-cell} ipython3
Map = geemap.Map()

point = ee.Geometry.Point([1.5, 1.5])

lineString = ee.Geometry.LineString([[-35, -10], [35, -10], [35, 10], [-35, 10]])

linearRing = ee.Geometry.LinearRing(
    [[-35, -10], [35, -10], [35, 10], [-35, 10], [-35, -10]]
)

rectangle = ee.Geometry.Rectangle([-40, -20, 40, 20])

polygon = ee.Geometry.Polygon([[[-5, 40], [65, 40], [65, 60], [-5, 60], [-5, 60]]])

Map.addLayer(point, {}, 'Point')
Map.addLayer(lineString, {}, 'LineString')
Map.addLayer(linearRing, {}, 'LinearRing')
Map.addLayer(rectangle, {}, 'Rectangle')
Map.addLayer(polygon, {}, 'Polygon')
Map
```

```{code-cell} ipython3
Map = geemap.Map()

point = ee.Geometry.Point([1.5, 1.5])

lineString = ee.Geometry.LineString(
    [[-35, -10], [35, -10], [35, 10], [-35, 10]], None, False
)

linearRing = ee.Geometry.LinearRing(
    [[-35, -10], [35, -10], [35, 10], [-35, 10], [-35, -10]], None, False
)

rectangle = ee.Geometry.Rectangle([-40, -20, 40, 20], None, False)

polygon = ee.Geometry.Polygon(
    [[[-5, 40], [65, 40], [65, 60], [-5, 60], [-5, 60]]], None, False
)

Map.addLayer(point, {}, 'Point')
Map.addLayer(lineString, {}, 'LineString')
Map.addLayer(linearRing, {}, 'LinearRing')
Map.addLayer(rectangle, {}, 'Rectangle')
Map.addLayer(polygon, {}, 'Polygon')
Map
```

#### Using drawing tools

```{code-cell} ipython3
if Map.user_roi is not None:
    print(Map.user_roi.getInfo())
```

### Feature

#### Creating Feature objects

```{code-cell} ipython3
# Create an ee.Geometry.
polygon = ee.Geometry.Polygon(
    [[[-35, -10], [35, -10], [35, 10], [-35, 10], [-35, -10]]], None, False
)

# Create a Feature from the Geometry.
polyFeature = ee.Feature(polygon, {'foo': 42, 'bar': 'tart'})
```

```{code-cell} ipython3
polyFeature.getInfo()
```

```{code-cell} ipython3
Map = geemap.Map()
Map.addLayer(polyFeature, {}, 'feature')
Map
```

```{code-cell} ipython3
# Create a dictionary of properties, some of which may be computed values.
props = {'foo': ee.Number(8).add(88), 'bar': 'nihao'}

# Create a None geometry feature with the dictionary of properties.
nowhereFeature = ee.Feature(None, props)

nowhereFeature.getInfo()
```

#### Setting Feature properties

```{code-cell} ipython3
# Make a feature and set some properties.
feature = (
    ee.Feature(ee.Geometry.Point([-122.22599, 37.17605]))
    .set('genus', 'Sequoia')
    .set('species', 'sempervirens')
)

# Overwrite the old properties with a new dictionary.
newDict = {'genus': 'Brachyramphus', 'presence': 1, 'species': 'marmoratus'}
feature = feature.set(newDict)

# Check the result.
feature.getInfo()
```

#### Getting Feature properties

```{code-cell} ipython3
prop = feature.get('species')
prop.getInfo()
```

```{code-cell} ipython3
props = feature.toDictionary()
props.getInfo()
```

### FeatureCollection

#### Loading feature collections

```{code-cell} ipython3
Map = geemap.Map()
fc = ee.FeatureCollection('TIGER/2016/Roads')
Map.setCenter(-73.9596, 40.7688, 12)
Map.addLayer(fc, {}, 'Census roads')
Map
```

#### Creating feature collections

```{code-cell} ipython3
# Make a list of Features.
features = [
    ee.Feature(ee.Geometry.Rectangle(30.01, 59.80, 30.59, 60.15), {'name': 'Voronoi'}),
    ee.Feature(ee.Geometry.Point(-73.96, 40.781), {'name': 'Thiessen'}),
    ee.Feature(ee.Geometry.Point(6.4806, 50.8012), {'name': 'Dirichlet'}),
]

# Create a FeatureCollection from the list and print it.
fromList = ee.FeatureCollection(features)
```

```{code-cell} ipython3
fromList.getInfo()
```

#### Filtering feature collections

```{code-cell} ipython3
Map = geemap.Map()
states = ee.FeatureCollection('TIGER/2018/States')
feat = states.filter(ee.Filter.eq('NAME', 'Texas'))
Map.addLayer(feat, {}, 'Texas')
Map.centerObject(feat)
Map
```

```{code-cell} ipython3
texas = feat.first()
texas.toDictionary().getInfo()
```

```{code-cell} ipython3
Map = geemap.Map()
states = ee.FeatureCollection('TIGER/2018/States')
fc = states.filter(ee.Filter.inList('NAME', ['California', 'Oregon', 'Washington']))
Map.addLayer(fc, {}, 'West Coast')
Map.centerObject(fc)
Map
```

```{code-cell} ipython3
region = Map.user_roi
if region is None:
    region = ee.Geometry.BBox(-88.40, 29.88, -77.90, 35.39)

fc = ee.FeatureCollection('TIGER/2018/States').filterBounds(region)
Map.addLayer(fc, {}, 'Southeastern U.S.')
Map.centerObject(fc)
```

#### Visualizing feature collections

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
states = ee.FeatureCollection("TIGER/2018/States")
Map.addLayer(states, {}, "US States")
Map
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
states = ee.FeatureCollection("TIGER/2018/States")
image = ee.Image().paint(states, 0, 3)
Map.addLayer(image, {'palette': 'red'}, "US States")
Map
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
states = ee.FeatureCollection("TIGER/2018/States")
style = {'color': '0000ffff', 'width': 2, 'lineType': 'solid', 'fillColor': 'FF000080'}
Map.addLayer(states.style(**style), {}, "US States")
Map
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
states = ee.FeatureCollection("TIGER/2018/States")
vis_params = {
    'color': '000000',
    'colorOpacity': 1,
    'pointSize': 3,
    'pointShape': 'circle',
    'width': 2,
    'lineType': 'solid',
    'fillColorOpacity': 0.66,
}
palette = ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5']
Map.add_styled_vector(
    states, column="NAME", palette=palette, layer_name="Styled vector", **vis_params
)
Map
```

#### Styling by attribute

```{code-cell} ipython3
Map = geemap.Map(center=[28.00142, -81.7424], zoom=13)
Map.add_basemap('HYBRID')
```

```{code-cell} ipython3
types = [
    "Freshwater Forested/Shrub Wetland",
    "Freshwater Emergent Wetland",
    "Freshwater Pond",
    "Estuarine and Marine Wetland",
    "Riverine",
    "Lake",
    "Estuarine and Marine Deepwater",
    "Other",
]

colors = [
    "#008837",
    "#7FC31C",
    "#688CC0",
    "#66C2A5",
    "#0190BF",
    "#13007C",
    "#007C88",
    "#B28653",
]

fillColor = [c + "A8" for c in colors]
```

```{code-cell} ipython3
fc = ee.FeatureCollection("projects/sat-io/open-datasets/NWI/wetlands/FL_Wetlands")
styled_fc = geemap.ee_vector_style(
    fc, column='WETLAND_TY', labels=types, fillColor=fillColor, color='00000000'
)
```

```{code-cell} ipython3
Map.addLayer(styled_fc, {}, 'NWI')
Map.add_legend(title='Wetland Type', labels=types, colors=colors)
Map
```

```{code-cell} ipython3
fuels = [
    'Coal',
    'Oil',
    'Gas',
    'Hydro',
    'Nuclear',
    'Solar',
    'Waste',
    'Wind',
    'Geothermal',
    'Biomass',
]
colors = [
    '000000',
    '593704',
    'BC80BD',
    '0565A6',
    'E31A1C',
    'FF7F00',
    '6A3D9A',
    '5CA2D1',
    'FDBF6F',
    '229A00',
]
```

```{code-cell} ipython3
fc = ee.FeatureCollection("WRI/GPPD/power_plants").filter(
    ee.Filter.inList('fuel1', fuels)
)
styled_fc = geemap.ee_vector_style(fc, column="fuel1", labels=fuels, color=colors)
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
Map.addLayer(styled_fc, {}, 'Power Plants')
Map.add_legend(title="Power Plant Fuel Type", labels=fuels, colors=colors)
Map
```

```{code-cell} ipython3
types = ['I', 'U', 'S', 'M', 'C', 'O']
labels = ['Interstate', 'U.S.', 'State recognized', 'Common Name', 'County', 'Other']
colors = ['E31A1C', 'FF7F00', '6A3D9A', '000000', 'FDBF6F', '229A00']
width = [8, 5, 4, 2, 1, 1]
```

```{code-cell} ipython3
fc = ee.FeatureCollection('TIGER/2016/Roads')
styled_fc = geemap.ee_vector_style(
    fc, column='rttyp', labels=types, color=colors, width=width
)
```

```{code-cell} ipython3
Map = geemap.Map(center=[40.7424, -73.9724], zoom=13)
Map.addLayer(styled_fc, {}, 'Census Roads')
Map.add_legend(title='Route Type', labels=labels, colors=colors)
Map
```

## Earth Engine Data Catalog

### Searching for datasets

```{code-cell} ipython3
dataset_xyz = ee.Image('USGS/SRTMGL1_003')
Map.addLayer(dataset_xyz, {}, "USGS/SRTMGL1_003")
```

```{code-cell} ipython3
Map = geemap.Map()
dem = ee.Image('USGS/SRTMGL1_003')
vis_params = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}
Map.addLayer(dem, vis_params, 'SRTM DEM')
Map
```

### Using the datasets module

```{code-cell} ipython3
from geemap.datasets import DATA
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
dataset = ee.Image(DATA.USGS_GAP_CONUS_2011)
Map.addLayer(dataset, {}, 'GAP CONUS')
Map
```

```{code-cell} ipython3
from geemap.datasets import get_metadata

get_metadata(DATA.USGS_GAP_CONUS_2011)
```

+++

## Getting image metadata

```{code-cell} ipython3
image = ee.Image('LANDSAT/LC09/C02/T1_L2/LC09_044034_20220503')
```

```{code-cell} ipython3
image.bandNames().getInfo()
```

```{code-cell} ipython3
image.select('SR_B1').projection().getInfo()
```

```{code-cell} ipython3
image.select('SR_B1').projection().nominalScale().getInfo()
```

```{code-cell} ipython3
image.propertyNames().getInfo()
```

```{code-cell} ipython3
image.get('CLOUD_COVER').getInfo()
```

```{code-cell} ipython3
image.get('DATE_ACQUIRED').getInfo()
```

```{code-cell} ipython3
image.get('system:time_start').getInfo()
```

```{code-cell} ipython3
date = ee.Date(image.get('system:time_start'))
date.format('YYYY-MM-dd').getInfo()
```

```{code-cell} ipython3
image.toDictionary().getInfo()
```

```{code-cell} ipython3
props = geemap.image_props(image)
props.getInfo()
```

## Calculating descriptive statistics

```{code-cell} ipython3
image = ee.Image('LANDSAT/LC09/C02/T1_L2/LC09_044034_20220503')
geemap.image_min_value(image).getInfo()
```

```{code-cell} ipython3
geemap.image_max_value(image).getInfo()
```

```{code-cell} ipython3
geemap.image_mean_value(image).getInfo()
```

```{code-cell} ipython3
geemap.image_stats(image).getInfo()
```

## Using the inspector tool

```{code-cell} ipython3
# Create an interactive map
Map = geemap.Map(center=(40, -100), zoom=4)

# Add Earth Engine datasets
dem = ee.Image('USGS/SRTMGL1_003')
landsat7 = ee.Image('LANDSAT/LE7_TOA_5YEAR/1999_2003').select(
    ['B1', 'B2', 'B3', 'B4', 'B5', 'B7']
)
states = ee.FeatureCollection("TIGER/2018/States")

# Set visualization parameters.
vis_params = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}

# Add Earth Engine layers to the map
Map.addLayer(dem, vis_params, 'SRTM DEM')
Map.addLayer(
    landsat7,
    {'bands': ['B4', 'B3', 'B2'], 'min': 20, 'max': 200, 'gamma': 2.0},
    'Landsat 7',
)
Map.addLayer(states, {}, "US States")

# Display the map
Map
```

+++

## Converting JavaScript to Python

### Interactive conversion

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
# Load an image.
image = ee.Image('LANDSAT/LC08/C02/T1_TOA/LC08_044034_20140318')

# Define the visualization parameters.
vizParams = {'bands': ['B5', 'B4', 'B3'], 'min': 0, 'max': 0.5, 'gamma': [0.95, 1.1, 1]}

# Center the map and display the image.
Map.setCenter(-122.1899, 37.5010, 10)
# San Francisco Bay
Map.addLayer(image, vizParams, 'False color composite')
```

### Batch conversion

```{code-cell} ipython3
snippet = """
// Load an image.
var image = ee.Image('LANDSAT/LC08/C02/T1_TOA/LC08_044034_20140318');

// Create an NDWI image, define visualization parameters and display.
var ndwi = image.normalizedDifference(['B3', 'B5']);
var ndwiViz = {min: 0.5, max: 1, palette: ['00FFFF', '0000FF']};
Map.addLayer(ndwi, ndwiViz, 'NDWI');
Map.centerObject(image)
"""

geemap.js_snippet_to_py(snippet)
```

```{code-cell} ipython3
import ee
import geemap

Map = geemap.Map()

# Load an image.
image = ee.Image('LANDSAT/LC08/C02/T1_TOA/LC08_044034_20140318')

# Create an NDWI image, define visualization parameters and display.
ndwi = image.normalizedDifference(['B3', 'B5'])
ndwiViz = {'min': 0.5, 'max': 1, 'palette': ['00FFFF', '0000FF']}
Map.addLayer(ndwi, ndwiViz, 'NDWI')
Map.centerObject(image)
Map
```

```{code-cell} ipython3
import os
from geemap.conversion import *

# Set the output directory
out_dir = os.getcwd()

# Set the input directory
js_dir = get_js_examples(out_dir)

# Convert Earth Engine JavaScripts Python
js_to_python_dir(in_dir=js_dir, out_dir=out_dir, use_qgis=False)

# Convert Python scripts to Jupyter notebooks
py_to_ipynb_dir(js_dir)
```

## Calling JavaScript functions from Python

```{code-cell} ipython3
pip install oeel
```

```{code-cell} ipython3
import oeel
```

```{code-cell} ipython3
oeel = geemap.requireJS()
```

```{code-cell} ipython3
ic = ee.ImageCollection("COPERNICUS/S2_SR")

icSize = (
    oeel.Algorithms.Sentinel2.cloudfree(maxCloud=20, S2Collection=ic)
    .filterDate('2020-01-01', '2020-01-02')
    .size()
)
print('Cloud free imagery: ', icSize.getInfo())
```

```{code-cell} ipython3
var generateRasterGrid = function(origin, dx, dy, proj) {
    var coords = origin.transform(proj).coordinates();
    origin = ee.Image.constant(coords.get(0)).addBands(ee.Image.constant(coords.get(1)));

    var pixelCoords = ee.Image.pixelCoordinates(proj);

    var grid = pixelCoords
       .subtract(origin)
       .divide([dx, dy]).floor()
       .toInt().reduce(ee.Reducer.sum()).bitwiseAnd(1).rename('grid');

    var xy = pixelCoords.reproject(proj.translate(coords.get(0), coords.get(1)).scale(dx, dy));

    var id = xy.multiply(ee.Image.constant([1, 1000000])).reduce(ee.Reducer.sum()).rename('id');

    return grid
      .addBands(id)
      .addBands(xy);
  }

exports.grid_test = grid_test;
```

```{code-cell} ipython3
url = 'https://tinyurl.com/27xy4oh9'
lib = geemap.requireJS(lib_path=url)
```

```{code-cell} ipython3
lib.availability
```

```{code-cell} ipython3
grid = lib.generateGrid(-180, -70, 180, 70, 10, 10, 0, 0)
grid.first().getInfo()
```

```{code-cell} ipython3
Map = geemap.Map()
style = {'fillColor': '00000000'}
Map.addLayer(grid.style(**style), {}, 'Grid')
Map
```

```{code-cell} ipython3
var grid_test = function() {

    var gridRaster = generateRasterGrid(ee.Geometry.Point(0, 0), 10, 10, ee.Projection('EPSG:4326'))
    Map.addLayer(gridRaster.select('id').randomVisualizer(), {}, 'Grid raster')

    var gridVector = generateGrid(-180, -70, 180, 70, 10, 10, 0, 0)
    Map.addLayer(gridVector, {}, 'Grid vector')
}
```

```{code-cell} ipython3
Map = geemap.Map()
lib = geemap.requireJS(lib_path='grid.js', Map=Map)
```

```{code-cell} ipython3
lib.grid_test()
Map
```

```{code-cell} ipython3
lib = geemap.requireJS('users/gena/packages:grid')
```

```{code-cell} ipython3
grid = lib.generateGrid(-180, -70, 180, 70, 10, 10, 0, 0)
```

```{code-cell} ipython3
Map = geemap.Map()
style = {'fillColor': '00000000'}
Map.addLayer(grid.style(**style), {}, 'Grid')
Map
```

## Summary

## References

- https://geemap.org/notebooks/03_inspector_tool/
- https://geemap.org/notebooks/08_ee_js_to_ipynb/
- https://geemap.org/notebooks/15_convert_js_to_py/
- https://geemap.org/notebooks/19_search_places_and_datasets/
- https://geemap.org/notebooks/22_import_assets
- https://geemap.org/notebooks/30_image_props_stats/
- https://geemap.org/notebooks/38_cloud_geotiff/
- https://geemap.org/notebooks/44_cog_stac/
- https://geemap.org/notebooks/64_data_catalog/
- https://geemap.org/notebooks/120_javascript/
- https://developers.google.com/earth-engine/guides/image_overview
- https://developers.google.com/earth-engine/guides/ic_creating
- https://developers.google.com/earth-engine/guides/ic_filtering
- https://developers.google.com/earth-engine/guides/features
- https://developers.google.com/earth-engine/guides/feature_collections
