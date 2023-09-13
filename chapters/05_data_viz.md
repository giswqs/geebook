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

# Visualizing Geospatial Data

```{contents}
:local:
:depth: 2
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

## Using the plotting tool

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)

landsat7 = ee.Image('LANDSAT/LE7_TOA_5YEAR/1999_2003').select(
    ['B1', 'B2', 'B3', 'B4', 'B5', 'B7']
)

landsat_vis = {'bands': ['B4', 'B3', 'B2'], 'gamma': 1.4}
Map.addLayer(landsat7, landsat_vis, "Landsat")

hyperion = ee.ImageCollection('EO1/HYPERION').filter(
    ee.Filter.date('2016-01-01', '2017-03-01')
)

hyperion_vis = {
    'min': 1000.0,
    'max': 14000.0,
    'gamma': 2.5,
}
Map.addLayer(hyperion, hyperion_vis, 'Hyperion')
Map
```

```{code-cell} ipython3
Map.set_plot_options(add_marker_cluster=True, overlay=True)
```

+++

## Changing layer opacity

```{code-cell} ipython3
Map = geemap.Map(center=(40, -100), zoom=4)

dem = ee.Image('USGS/SRTMGL1_003')
states = ee.FeatureCollection("TIGER/2018/States")

vis_params = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}

Map.addLayer(dem, vis_params, 'SRTM DEM', True, 1)
Map.addLayer(states, {}, "US States", True)

Map
```

+++

## Visualizing raster data

### Single-band images

```{code-cell} ipython3
Map = geemap.Map(center=[12, 69], zoom=3)
dem = ee.Image('USGS/SRTMGL1_003')
vis_params = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}
Map.addLayer(dem, vis_params, 'SRTM DEM')
Map
```

```{code-cell} ipython3
vis_params = {
    'bands': ['elevation'],
    'palette': ['333399', ' 00b2b2', ' 99eb85', ' ccbe7d', ' 997c76', ' ffffff'],
    'min': 0.0,
    'max': 6000.0,
    'opacity': 1.0,
    'gamma': 1.0,
}
```

### Multi-band images

```{code-cell} ipython3
Map = geemap.Map()
landsat7 = ee.Image('LANDSAT/LE7_TOA_5YEAR/1999_2003')
vis_params = {
    'min': 20,
    'max': 200,
    'gamma': 2,
    'bands': ['B4', 'B3', 'B2'],
}
Map.addLayer(landsat7, vis_params, 'Landsat 7')
Map
```

+++

## Visualizing vector data

```{code-cell} ipython3
Map = geemap.Map()
states = ee.FeatureCollection("TIGER/2018/States")
Map.addLayer(states, {}, "US States")
Map
```

```{code-cell} ipython3
vis_params = {
    'color': 'ff0000ff',
    'width': 2,
    'lineType': 'solid',
    'fillColor': '00000000',
}
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
states = ee.FeatureCollection("TIGER/2018/States")
Map.addLayer(states.style(**vis_params), {}, "US States")
Map
```

## Creating legends

### Built-in legends

```{code-cell} ipython3
from geemap.legends import builtin_legends

for legend in builtin_legends:
    print(legend)
```

```{code-cell} ipython3
Map.add_legend(builtin_legend='NLCD')
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

### Custom legends

```{code-cell} ipython3
Map = geemap.Map(add_google_map=False)

labels = ['One', 'Two', 'Three', 'Four', 'etc']

# colors can be defined using either hex code or RGB (0-255, 0-255, 0-255)
colors = ['#8DD3C7', '#FFFFB3', '#BEBADA', '#FB8072', '#80B1D3']
# legend_colors = [(255, 0, 0), (127, 255, 0), (127, 18, 25), (36, 70, 180), (96, 68 123)]

Map.add_legend(
    labels=labels, colors=colors, position='bottomright'
)
Map
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)

legend_dict = {
    '11 Open Water': '466b9f',
    '12 Perennial Ice/Snow': 'd1def8',
    '21 Developed, Open Space': 'dec5c5',
    '22 Developed, Low Intensity': 'd99282',
    '23 Developed, Medium Intensity': 'eb0000',
    '24 Developed High Intensity': 'ab0000',
    '31 Barren Land (Rock/Sand/Clay)': 'b3ac9f',
    '41 Deciduous Forest': '68ab5f',
    '42 Evergreen Forest': '1c5f2c',
    '43 Mixed Forest': 'b5c58f',
    '51 Dwarf Scrub': 'af963c',
    '52 Shrub/Scrub': 'ccb879',
    '71 Grassland/Herbaceous': 'dfdfc2',
    '72 Sedge/Herbaceous': 'd1d182',
    '73 Lichens': 'a3cc51',
    '74 Moss': '82ba9e',
    '81 Pasture/Hay': 'dcd939',
    '82 Cultivated Crops': 'ab6c28',
    '90 Woody Wetlands': 'b8d9eb',
    '95 Emergent Herbaceous Wetlands': '6c9fb8',
}

nlcd = ee.Image('USGS/NLCD_RELEASES/2019_REL/NLCD/2019')
landcover = nlcd.select('landcover')

Map.addLayer(landcover, {}, 'NLCD Land Cover 2019')
Map.add_legend(title="NLCD Land Cover Classification", legend_dict=legend_dict)
Map
```

### Earth Engine class table

```{code-cell} ipython3
Map = geemap.Map()

dataset = ee.ImageCollection("ESA/WorldCover/v100").first()
Map.addLayer(dataset, {'bands': ['Map']}, "Landcover")

ee_class_table = """
Value	Color	Description
10	006400	Trees
20	ffbb22	Shrubland
30	ffff4c	Grassland
40	f096ff	Cropland
50	fa0000	Built-up
60	b4b4b4	Barren / sparse vegetation
70	f0f0f0	Snow and ice
80	0064c8	Open water
90	0096a0	Herbaceous wetland
95	00cf75	Mangroves
100	fae6a0	Moss and lichen
"""

legend_dict = geemap.legend_from_ee(ee_class_table)
Map.add_legend(title="ESA Land Cover", legend_dict=legend_dict)
Map
```

## Creating color bars

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

```{code-cell} ipython3
Map.add_colorbar(vis_params, label="Elevation (m)", layer_name="SRTM DEM")
Map
```

```{code-cell} ipython3
Map.add_colorbar(
    vis_params, label="Elevation (m)", layer_name="SRTM DEM", orientation="vertical"
)
```

```{code-cell} ipython3
Map.add_colorbar(
    vis_params,
    label="Elevation (m)",
    layer_name="SRTM DEM",
    orientation="vertical",
    transparent_bg=True,
)
```

## Displaying labels

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4, add_google_map=False)
states = ee.FeatureCollection("TIGER/2018/States")
style = {'color': 'black', 'fillColor': "00000000"}
Map.addLayer(states.style(**style), {}, "US States")
Map
```

```{code-cell} ipython3
Map.add_labels(
    data=states,
    column="STUSPS",
    font_size="12pt",
    font_color="blue",
    font_family="arial",
    font_weight="bold",
    draggable=True,
)
```

```{code-cell} ipython3
Map.remove_labels()
```

```{code-cell} ipython3
centroids = geemap.vector_centroids(states)
df = geemap.ee_to_df(centroids)
df
```

```{code-cell} ipython3
Map.add_labels(
    data=df,
    column="STUSPS",
    font_size="12pt",
    font_color="blue",
    font_family="arial",
    font_weight="bold",
    x='longitude',
    y='latitude',
)
Map
```

## Image overlay

```{code-cell} ipython3
Map = geemap.Map(center=(25, -115), zoom=5)
url = 'https://i.imgur.com/06Q1fSz.png'
image = geemap.ImageOverlay(url=url, bounds=((13, -130), (32, -100)))
Map.add_layer(image)
Map
```

```{code-cell} ipython3
image.url = 'https://i.imgur.com/U0axit9.png'
Map
```

```{code-cell} ipython3
url = 'https://i.imgur.com/06Q1fSz.png'
filename = 'hurricane.png'
geemap.download_file(url, filename)
```

```{code-cell} ipython3
Map = geemap.Map(center=(25, -115), zoom=5)
image = geemap.ImageOverlay(url=filename, bounds=((13, -130), (32, -100)))
Map.add_layer(image)
Map
```

## Video overlay

```{code-cell} ipython3
Map = geemap.Map(center=(25, -115), zoom=5)
url = 'https://labs.mapbox.com/bites/00188/patricia_nasa.webm'
bounds = ((13, -130), (32, -100))
Map.video_overlay(url, bounds)
Map
```

## Split-panel maps

```{code-cell} ipython3
Map = geemap.Map()
Map.split_map(left_layer='HYBRID', right_layer='TERRAIN')
Map
```

```{code-cell} ipython3
list(geemap.basemaps.keys())
```

```{code-cell} ipython3
Map = geemap.Map(center=(40, -100), zoom=4, height=600)

nlcd_2001 = ee.Image('USGS/NLCD_RELEASES/2019_REL/NLCD/2001').select('landcover')
nlcd_2019 = ee.Image('USGS/NLCD_RELEASES/2019_REL/NLCD/2019').select('landcover')

left_layer = geemap.ee_tile_layer(nlcd_2001, {}, 'NLCD 2001')
right_layer = geemap.ee_tile_layer(nlcd_2019, {}, 'NLCD 2019')

Map.split_map(left_layer, right_layer, add_close_button=True)
Map
```

+++

## Linked maps

```{code-cell} ipython3
image = (
    ee.ImageCollection('COPERNICUS/S2')
    .filterDate('2018-09-01', '2018-09-30')
    .map(lambda img: img.divide(10000))
    .median()
)

vis_params = [
    {'bands': ['B4', 'B3', 'B2'], 'min': 0, 'max': 0.3, 'gamma': 1.3},
    {'bands': ['B8', 'B11', 'B4'], 'min': 0, 'max': 0.3, 'gamma': 1.3},
    {'bands': ['B8', 'B4', 'B3'], 'min': 0, 'max': 0.3, 'gamma': 1.3},
    {'bands': ['B12', 'B12', 'B4'], 'min': 0, 'max': 0.3, 'gamma': 1.3},
]

labels = [
    'Natural Color (B4/B3/B2)',
    'Land/Water (B8/B11/B4)',
    'Color Infrared (B8/B4/B3)',
    'Vegetation (B12/B11/B4)',
]

geemap.linked_maps(
    rows=2,
    cols=2,
    height="300px",
    center=[38.4151, 21.2712],
    zoom=12,
    ee_objects=[image],
    vis_params=vis_params,
    labels=labels,
    label_position="topright",
)
```

+++

## Timeseries inspector

### Visualizing image collections

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
collection = ee.ImageCollection('USGS/NLCD_RELEASES/2019_REL/NLCD').select('landcover')
vis_params = {'bands': ['landcover']}
years = collection.aggregate_array('system:index').getInfo()
years
```

```{code-cell} ipython3
Map.ts_inspector(
    left_ts=collection,
    right_ts=collection,
    left_names=years,
    right_names=years,
    left_vis=vis_params,
    right_vis=vis_params,
    width='80px',
)
Map
```

### Visualizing planet.com imagery

```{code-cell} ipython3
import os

os.environ["PLANET_API_KEY"] = "your-api-key"
```

```{code-cell} ipython3
monthly_tiles = geemap.planet_monthly_tiles()
geemap.ts_inspector(monthly_tiles)
```

```{code-cell} ipython3
quarterly_tiles = geemap.planet_quarterly_tiles()
geemap.ts_inspector(quarterly_tiles)
```

```{code-cell} ipython3
tiles = geemap.planet_tiles()
geemap.ts_inspector(tiles)
```

```{code-cell} ipython3
Map = geemap.Map()
Map
```

## Time slider

### Visualizing vegetation data

```{code-cell} ipython3
Map = geemap.Map()

collection = (
    ee.ImageCollection('MODIS/MCD43A4_006_NDVI')
    .filter(ee.Filter.date('2018-06-01', '2018-07-01'))
    .select("NDVI")
)
vis_params = {
    'min': 0.0,
    'max': 1.0,
    'palette': 'ndvi',
}

Map.add_time_slider(collection, vis_params, time_interval=2)
Map
```

### Visualizing weather data

```{code-cell} ipython3
Map = geemap.Map()

collection = (
    ee.ImageCollection('NOAA/GFS0P25')
    .filterDate('2018-12-22', '2018-12-23')
    .limit(24)
    .select('temperature_2m_above_ground')
)

vis_params = {
    'min': -40.0,
    'max': 35.0,
    'palette': ['blue', 'purple', 'cyan', 'green', 'yellow', 'red'],
}

labels = [str(n).zfill(2) + ":00" for n in range(0, 24)]
Map.add_time_slider(collection, vis_params, labels=labels, time_interval=1, opacity=0.8)
Map
```

### Visualizing Sentinel-2 imagery

```{code-cell} ipython3
Map = geemap.Map(center=[37.75, -122.45], zoom=12)

collection = (
    ee.ImageCollection('COPERNICUS/S2_SR')
    .filterBounds(ee.Geometry.Point([-122.45, 37.75]))
    .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than', 10)
)

vis_params = {"min": 0, "max": 4000, "bands": ["B8", "B4", "B3"]}

Map.add_time_slider(collection, vis_params)
Map
```

+++

## Shaded relief maps

```{code-cell} ipython3
import geemap.colormaps as cm

Map = geemap.Map()

dem = ee.Image("USGS/SRTMGL1_003")
hillshade = ee.Terrain.hillshade(dem)

vis = {'min': 0, 'max': 6000, 'palette': cm.palettes.terrain}
blend = geemap.blend(top_layer=dem, top_vis=vis)

Map.addLayer(hillshade, {}, 'Hillshade')
Map.addLayer(blend, {}, 'Shaded relief')

Map.add_colorbar(vis, label='Elevation (m)')
Map.setCenter(91.4206, 27.3225, zoom=9)
Map
```

```{code-cell} ipython3
left_layer = geemap.ee_tile_layer(blend, {}, "Shaded relief")
right_layer = geemap.ee_tile_layer(hillshade, {}, "Hillshade")
Map.split_map(left_layer, right_layer)
```

```{code-cell} ipython3
Map = geemap.Map()
nlcd = ee.Image("USGS/NLCD_RELEASES/2019_REL/NLCD/2019").select('landcover')
nlcd_vis = {'bands': ['landcover']}
blend = geemap.blend(nlcd, dem, top_vis=nlcd_vis, expression='a*b')
Map.addLayer(blend, {}, 'Blend NLCD')
Map.add_legend(builtin_legend='NLCD', title='NLCD Land Cover')
Map.setCenter(-118.1310, 35.6816, 10)
Map
```

## Elevation contours

```{code-cell} ipython3
import geemap.colormaps as cm
```

```{code-cell} ipython3
Map = geemap.Map()
image = ee.Image("USGS/SRTMGL1_003")
hillshade = ee.Terrain.hillshade(image)
Map.addLayer(hillshade, {}, "Hillshade")
Map
```

```{code-cell} ipython3
vis_params = {'min': 0, "max": 5000, "palette": cm.palettes.dem}
Map.addLayer(image, vis_params, "dem", True, 0.5)
Map.add_colorbar(vis_params, label='Elevation (m)')
```

```{code-cell} ipython3
contours = geemap.create_contours(image, 0, 5000, 100, region=None)
Map.addLayer(contours, {'palette': 'black'}, 'contours')
Map.setCenter(-119.3678, 37.1671, 12)
```

## Visualizing NetCDF data

```{code-cell} ipython3
url = 'https://github.com/gee-community/geemap/raw/master/examples/data/wind_global.nc'
filename = 'wind_global.nc'
geemap.download_file(url, output=filename)
```

```{code-cell} ipython3
data = geemap.read_netcdf(filename)
data
```

```{code-cell} ipython3
Map = geemap.Map(layers_control=True)
Map.add_netcdf(
    filename,
    variables=['v_wind'],
    palette='coolwarm',
    shift_lon=True,
    layer_name='v_wind',
)

geojson = 'https://github.com/gee-community/geemap/raw/master/examples/data/countries.geojson'
Map.add_geojson(geojson, layer_name='Countries')
Map
```

```{code-cell} ipython3
Map = geemap.Map(layers_control=True)
Map.add_basemap('CartoDB.DarkMatter')
Map.add_velocity(filename, zonal_speed='u_wind', meridional_speed='v_wind')
Map
```

+++

## Visualizing LiDAR data

```{code-cell} ipython3
%pip install "geemap[lidar]"
```

```{code-cell} ipython3
import os

url = (
    'https://drive.google.com/file/d/1H_X1190vL63BoFYa_cVBDxtIa8rG-Usb/view?usp=sharing'
)
filename = 'madison.las'

if not os.path.exists(filename):
    geemap.download_file(url, 'madison.zip', unzip=True)
```

```{code-cell} ipython3
las = geemap.read_lidar(filename)
```

```{code-cell} ipython3
las.header
```

```{code-cell} ipython3
las.header.point_count
```

```{code-cell} ipython3
list(las.point_format.dimension_names)
```

```{code-cell} ipython3
las.X
```

```{code-cell} ipython3
las.intensity
```

```{code-cell} ipython3
geemap.view_lidar(filename, cmap='terrain', backend='pyvista', background='gray')
```

```{code-cell} ipython3
geemap.view_lidar(filename, backend='ipygany', background='white')
```

## Visualizing raster data in 3D

```{code-cell} ipython3
url = 'https://github.com/giswqs/data/raw/main/raster/srtm90.tif'
image = 'srtm90.tif'
if not os.path.exists(image):
    geemap.download_file(url, image)
```

```{code-cell} ipython3
geemap.plot_raster(image, cmap='terrain', figsize=(15, 10))
```

```{code-cell} ipython3
geemap.plot_raster_3d('srtm90.tif', factor=2, cmap='terrain', background='gray')
```

## Creating choropleth maps

```{code-cell} ipython3
data = geemap.examples.datasets.countries_geojson
```

```{code-cell} ipython3
Map = geemap.Map()
Map.add_data(
    data, column='POP_EST', scheme='Quantiles', cmap='Blues', legend_title='Population'
)
Map
```

```{code-cell} ipython3
Map = geemap.Map()
Map.add_data(
    data,
    column='POP_EST',
    scheme='EqualInterval',
    cmap='Blues',
    legend_title='Population',
)
Map
```

```{code-cell} ipython3
Map = geemap.Map()
Map.add_data(
    data,
    column='POP_EST',
    scheme='FisherJenks',
    cmap='Blues',
    legend_title='Population',
)
Map
```

```{code-cell} ipython3
Map = geemap.Map()
Map.add_data(
    data,
    column='POP_EST',
    scheme='JenksCaspall',
    cmap='Blues',
    legend_title='Population',
)
Map
```

## Summary

