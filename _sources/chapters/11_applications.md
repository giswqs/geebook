---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.4
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Earth Engine Applications

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
mamba install -c conda-forge geemap pygis
```

```bash
jupyter lab
```

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/giswqs/geebook/blob/master/chapters/11_applications.ipynb)

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

## Analyzing surface water dynamics

### Surface water occurrence

```{code-cell} ipython3
dataset = ee.Image('JRC/GSW1_4/GlobalSurfaceWater')
dataset.bandNames()
```

```{code-cell} ipython3
Map = geemap.Map()
Map.add_basemap('HYBRID')
Map
```

```{code-cell} ipython3
image = dataset.select(['occurrence'])
region = Map.user_roi # Draw a polygon on the map
if region is None:
    region = ee.Geometry.BBox(-99.957, 46.8947, -99.278, 47.1531)
vis_params = {'min': 0.0, 'max': 100.0, 'palette': ['ffffff', 'ffbbbb', '0000ff']}
Map.addLayer(image, vis_params, 'Occurrence')
Map.addLayer(region, {}, 'ROI', True, 0.5)
Map.centerObject(region)
Map.add_colorbar(vis_params, label='Water occurrence (%)', layer_name='Occurrence')
```

```{code-cell} ipython3
df = geemap.image_histogram(
    image,
    region,
    scale=30,
    return_df=True,
)
df
```

```{code-cell} ipython3
hist = geemap.image_histogram(
    image,
    region,
    scale=30,
    x_label='Water Occurrence (%)',
    y_label='Pixel Count',
    title='Surface Water Occurrence',
    layout_args={'title': dict(x=0.5)},
    return_df=False,
)
hist
```

```{code-cell} ipython3
# !pip install -U kaleido
```

```{code-cell} ipython3
hist.update_layout(
    autosize=False,
    width=800,
    height=400,
    margin=dict(l=30, r=20, b=10, t=50, pad=4)
)
hist.write_image('water_occurrence.jpg', scale=2)
```

### Surface water monthly history

```{code-cell} ipython3
dataset = ee.ImageCollection('JRC/GSW1_4/MonthlyHistory')
dataset.size()
```

```{code-cell} ipython3
dataset.aggregate_array("system:index")
```

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
image = dataset.filterDate('2020-08-01', '2020-09-01').first()
region = Map.user_roi # Draw a polygon on the map
if region is None:
    region = ee.Geometry.BBox(-99.957, 46.8947, -99.278, 47.1531)
vis_params = {'min': 0.0, 'max': 2.0, 'palette': ['ffffff', 'fffcb8', '0905ff']}

Map.addLayer(image, vis_params, 'Water')
Map.addLayer(region, {}, 'ROI', True, 0.5)
Map.centerObject(region)
```

```{code-cell} ipython3
geemap.jrc_hist_monthly_history(
    region=region, scale=30, frequency='month', denominator=1e4, y_label='Area (ha)'
)
```

```{code-cell} ipython3
geemap.jrc_hist_monthly_history(
    region=region,
    start_month=6,
    end_month=9,
    scale=30,
    frequency='month',
    denominator=1e4,
    y_label='Area (ha)',
)
```

```{code-cell} ipython3
geemap.jrc_hist_monthly_history(
    region=region,
    start_month=6,
    end_month=9,
    scale=30,
    frequency='year',
    reducer='mean',
    denominator=1e4,
    y_label='Area (ha)',
)
```

```{code-cell} ipython3
geemap.jrc_hist_monthly_history(
    region=region,
    start_month=6,
    end_month=9,
    scale=30,
    frequency='year',
    reducer='max',
    denominator=1e4,
    y_label='Area (ha)',
)
```

## Mapping flood extent

### Create an interactive map

```{code-cell} ipython3
Map = geemap.Map(center=[29.3055, 68.9062], zoom=6)
Map
```

### Search datasets

```{code-cell} ipython3
country_name = 'Pakistan'
pre_flood_start_date = '2021-08-01'
pre_flood_end_date = '2021-09-30'
flood_start_date = '2022-08-01'
flood_end_date = '2022-09-30'
```

### Visualize datasets

```{code-cell} ipython3
country = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017').filter(
    ee.Filter.eq('country_na', country_name)
)
style = {'color': 'black', 'fillColor': '00000000'}
Map.addLayer(country.style(**style), {}, country_name)
Map.centerObject(country)
Map
```

### Create Landsat composites

```{code-cell} ipython3
landsat_col_2021 = (
    ee.ImageCollection('LANDSAT/LC08/C02/T1')
    .filterDate(pre_flood_start_date, pre_flood_end_date)
    .filterBounds(country)
)
landsat_2021 = ee.Algorithms.Landsat.simpleComposite(landsat_col_2021).clipToCollection(
    country
)
vis_params = {'bands': ['B6', 'B5', 'B4'], 'max': 128}
Map.addLayer(landsat_2021, vis_params, 'Landsat 2021')
```

```{code-cell} ipython3
landsat_col_2022 = (
    ee.ImageCollection('LANDSAT/LC08/C02/T1')
    .filterDate(flood_start_date, flood_end_date)
    .filterBounds(country)
)
landsat_2022 = ee.Algorithms.Landsat.simpleComposite(landsat_col_2022).clipToCollection(
    country
)
Map.addLayer(landsat_2022, vis_params, 'Landsat 2022')
Map.centerObject(country)
Map
```

### Compare Landsat composites side by side

```{code-cell} ipython3
Map = geemap.Map()
Map.setCenter(68.4338, 26.4213, 7)

left_layer = geemap.ee_tile_layer(landsat_2021, vis_params, 'Landsat 2021')
right_layer = geemap.ee_tile_layer(landsat_2022, vis_params, 'Landsat 2022')

Map.split_map(
    left_layer, right_layer, left_label='Landsat 2021', right_label='Landsat 2022'
)
Map.addLayer(country.style(**style), {}, country_name)
Map
```

### Compute Normalized Difference Water Index (NDWI)

```{code-cell} ipython3
ndwi_2021 = landsat_2021.normalizedDifference(['B3', 'B5']).rename('NDWI')
ndwi_2022 = landsat_2022.normalizedDifference(['B3', 'B5']).rename('NDWI')
```

```{code-cell} ipython3
Map = geemap.Map()
Map.setCenter(68.4338, 26.4213, 7)
ndwi_vis = {'min': -1, 'max': 1, 'palette': 'ndwi'}

left_layer = geemap.ee_tile_layer(ndwi_2021, ndwi_vis, 'NDWI 2021')
right_layer = geemap.ee_tile_layer(ndwi_2022, ndwi_vis, 'NDWI 2022')

Map.split_map(left_layer, right_layer, left_label='NDWI 2021', right_label='NDWI 2022')
Map.addLayer(country.style(**style), {}, country_name)
Map
```

### Extract Landsat water extent

```{code-cell} ipython3
threshold = 0.1
water_2021 = ndwi_2021.gt(threshold).selfMask()
water_2022 = ndwi_2022.gt(threshold).selfMask()
```

```{code-cell} ipython3
Map = geemap.Map()
Map.setCenter(68.4338, 26.4213, 7)

Map.addLayer(landsat_2021, vis_params, 'Landsat 2021', False)
Map.addLayer(landsat_2022, vis_params, 'Landsat 2022', False)

left_layer = geemap.ee_tile_layer(
    water_2021, {'palette': 'blue'}, 'Water 2021'
)
right_layer = geemap.ee_tile_layer(
    water_2022, {'palette': 'red'}, 'Water 2022'
)

Map.split_map(
    left_layer, right_layer, left_label='Water 2021', right_label='Water 2022'
)
Map.addLayer(country.style(**style), {}, country_name)
Map
```

### Extract Landsat flood extent

```{code-cell} ipython3
flood_extent = water_2022.unmask().subtract(water_2021.unmask()).gt(0).selfMask()
```

```{code-cell} ipython3
Map = geemap.Map()
Map.setCenter(68.4338, 26.4213, 7)

Map.addLayer(landsat_2021, vis_params, 'Landsat 2021', False)
Map.addLayer(landsat_2022, vis_params, 'Landsat 2022', False)

left_layer = geemap.ee_tile_layer(
    water_2021, {'palette': 'blue'}, 'Water 2021'
)
right_layer = geemap.ee_tile_layer(
    water_2022, {'palette': 'red'}, 'Water 2022'
)

Map.split_map(
    left_layer, right_layer, left_label='Water 2021', right_label='Water 2022'
)

Map.addLayer(flood_extent, {'palette': 'cyan'}, 'Flood Extent')
Map.addLayer(country.style(**style), {}, country_name)
Map
```

### Calculate Landsat flood area

```{code-cell} ipython3
area_2021 = geemap.zonal_stats(
    water_2021, country, scale=1000, statistics_type='SUM', return_fc=True
)
geemap.ee_to_df(area_2021)
```

```{code-cell} ipython3
area_2022 = geemap.zonal_stats(
    water_2022, country, scale=1000, statistics_type='SUM', return_fc=True
)
geemap.ee_to_df(area_2022)
```

```{code-cell} ipython3
flood_area = geemap.zonal_stats(
    flood_extent, country, scale=1000, statistics_type='SUM', return_fc=True
)
geemap.ee_to_df(flood_area)
```

+++

### Create Sentinel-1 SAR composites

```{code-cell} ipython3
s1_col_2021 = (
    ee.ImageCollection('COPERNICUS/S1_GRD')
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
    .filter(ee.Filter.eq('instrumentMode', 'IW'))
    .filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'))
    .filterDate(pre_flood_start_date, pre_flood_end_date)
    .filterBounds(country)
    .select('VV')
)
s1_col_2021
```

```{code-cell} ipython3
s1_col_2022 = (
    ee.ImageCollection('COPERNICUS/S1_GRD')
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
    .filter(ee.Filter.eq('instrumentMode', 'IW'))
    .filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'))
    .filterDate(flood_start_date, flood_end_date)
    .filterBounds(country)
    .select('VV')
)
s1_col_2022
```

```{code-cell} ipython3
Map = geemap.Map()
Map.add_basemap('HYBRID')
sar_2021 = s1_col_2021.reduce(ee.Reducer.percentile([20])).clipToCollection(country)
sar_2022 = s1_col_2022.reduce(ee.Reducer.percentile([20])).clipToCollection(country)
Map.addLayer(sar_2021, {'min': -25, 'max': -5}, 'SAR 2021')
Map.addLayer(sar_2022, {'min': -25, 'max': -5}, 'SAR 2022')
Map.centerObject(country)
Map
```

### Apply speckle filtering

```{code-cell} ipython3
col_2021 = s1_col_2021.map(lambda img: img.focal_median(100, 'circle', 'meters'))
col_2022 = s1_col_2022.map(lambda img: img.focal_median(100, 'circle', 'meters'))

Map = geemap.Map()
Map.add_basemap('HYBRID')
sar_2021 = col_2021.reduce(ee.Reducer.percentile([20])).clipToCollection(country)
sar_2022 = col_2022.reduce(ee.Reducer.percentile([20])).clipToCollection(country)
Map.addLayer(sar_2021, {'min': -25, 'max': -5}, 'SAR 2021')
Map.addLayer(sar_2022, {'min': -25, 'max': -5}, 'SAR 2022')
Map.centerObject(country)
Map
```

### Compare Sentinel-1 SAR composites side by side

```{code-cell} ipython3
Map = geemap.Map()
Map.setCenter(68.4338, 26.4213, 7)

left_layer = geemap.ee_tile_layer(sar_2021, {'min': -25, 'max': -5}, 'SAR 2021')
right_layer = geemap.ee_tile_layer(sar_2022, {'min': -25, 'max': -5}, 'SAR 2022')

Map.split_map(
    left_layer, right_layer, left_label='Sentinel-1 2021', right_label='Sentinel-1 2022'
)
Map.addLayer(country.style(**style), {}, country_name)
Map
```

### Extract SAR water extent

```{code-cell} ipython3
threshold = -18
water_2021 = sar_2021.lt(threshold)
water_2022 = sar_2022.lt(threshold)
```

```{code-cell} ipython3
Map = geemap.Map()
Map.setCenter(68.4338, 26.4213, 7)

Map.addLayer(sar_2021, {'min': -25, 'max': -5}, 'SAR 2021')
Map.addLayer(sar_2022, {'min': -25, 'max': -5}, 'SAR 2022')

left_layer = geemap.ee_tile_layer(
    water_2021.selfMask(), {'palette': 'blue'}, 'Water 2021'
)
right_layer = geemap.ee_tile_layer(
    water_2022.selfMask(), {'palette': 'red'}, 'Water 2022'
)

Map.split_map(
    left_layer, right_layer, left_label='Water 2021', right_label='Water 2022'
)
Map.addLayer(country.style(**style), {}, country_name)
Map
```

### Extract SAR flood extent

```{code-cell} ipython3
flood_extent = water_2022.unmask().subtract(water_2021.unmask()).gt(0).selfMask()
```

```{code-cell} ipython3
Map = geemap.Map()
Map.setCenter(68.4338, 26.4213, 7)

Map.addLayer(sar_2021, {'min': -25, 'max': -5}, 'SAR 2021')
Map.addLayer(sar_2022, {'min': -25, 'max': -5}, 'SAR 2022')

left_layer = geemap.ee_tile_layer(
    water_2021, {'palette': 'blue'}, 'Water 2021'
)
right_layer = geemap.ee_tile_layer(
    water_2022, {'palette': 'red'}, 'Water 2022'
)

Map.split_map(
    left_layer, right_layer, left_label='Water 2021', right_label='Water 2022'
)

Map.addLayer(flood_extent, {'palette': 'cyan'}, 'Flood Extent')
Map.addLayer(country.style(**style), {}, country_name)
Map
```

### Calculate SAR flood area

```{code-cell} ipython3
area_2021 = geemap.zonal_stats(
    water_2021, country, scale=1000, statistics_type='SUM', return_fc=True
)
geemap.ee_to_df(area_2021)
```

```{code-cell} ipython3
area_2022 = geemap.zonal_stats(
    water_2022, country, scale=1000, statistics_type='SUM', return_fc=True
)
geemap.ee_to_df(area_2022)
```

```{code-cell} ipython3
flood_area = geemap.zonal_stats(
    flood_extent, country, scale=1000, statistics_type='SUM', return_fc=True
)
geemap.ee_to_df(flood_area)
```

## Forest cover change analysis

```{code-cell} ipython3
Map = geemap.Map()
Map.add_basemap('HYBRID')
Map
```

```{code-cell} ipython3
dataset = ee.Image('UMD/hansen/global_forest_change_2021_v1_9')
```

```{code-cell} ipython3
dataset.bandNames().getInfo()
```

```{code-cell} ipython3
first_bands = ['first_b50', 'first_b40', 'first_b30']
first_image = dataset.select(first_bands)
Map.addLayer(first_image, {'bands': first_bands, 'gamma': 1.5}, 'Year 2000 Bands 5/4/3')
```

```{code-cell} ipython3
last_bands = ['last_b50', 'last_b40', 'last_b30']
last_image = dataset.select(last_bands)
Map.addLayer(last_image, {'bands': last_bands, 'gamma': 1.5}, 'Year 2021 Bands 5/4/3')
```

```{code-cell} ipython3
treecover = dataset.select(['treecover2000'])

treeCoverVisParam = {'min': 0, 'max': 100, 'palette': ['black', 'green']}

name1 = 'Tree cover (%)'
Map.addLayer(treecover, treeCoverVisParam, name1)
Map.add_colorbar(treeCoverVisParam, label=name1, layer_name=name1)
Map
```

```{code-cell} ipython3
threshold = 10
treecover_bin = treecover.gte(threshold).selfMask()
treeVisParam = {'palette': ['green']}
Map.addLayer(treecover_bin, treeVisParam, 'Tree cover bin')
```

```{code-cell} ipython3
treeloss_year = dataset.select(['lossyear'])

treeLossVisParam = {'min': 0, 'max': 21, 'palette': ['yellow', 'red']}

layer_name = 'Tree loss year'
Map.addLayer(treeloss_year, treeLossVisParam, layer_name)
Map.add_colorbar(treeLossVisParam, label=layer_name, layer_name=layer_name)
```

```{code-cell} ipython3
treeloss = dataset.select(['loss']).selfMask()
Map.addLayer(treeloss, {'palette': 'red'}, 'Tree loss')
Map
```

```{code-cell} ipython3
treegain = dataset.select(['gain']).selfMask()
Map.addLayer(treegain, {'palette': 'yellow'}, 'Tree gain')
Map
```

```{code-cell} ipython3
countries = ee.FeatureCollection(geemap.examples.get_ee_path('countries'))
```

```{code-cell} ipython3
geemap.ee_to_df(countries)
```

```{code-cell} ipython3
style = {'color': '#ffff0088', 'fillColor': '#00000000'}
Map.addLayer(countries.style(**style), {}, 'Countries')
```

```{code-cell} ipython3
geemap.zonal_stats_by_group(
    treecover_bin,
    countries,
    'forest_cover.csv',
    statistics_type='SUM',
    denominator=1e6,
    scale=1000,
)
```

```{code-cell} ipython3
geemap.pie_chart(
    'forest_cover.csv', names='NAME', values='Class_sum', max_rows=20, height=600
)
```

```{code-cell} ipython3
geemap.bar_chart(
    'forest_cover.csv',
    x='NAME',
    y='Class_sum',
    max_rows=20,
    x_label='Country',
    y_label='Forest area (km2)',
)
```

```{code-cell} ipython3
geemap.zonal_stats_by_group(
    treeloss,
    countries,
    'treeloss.csv',
    statistics_type='SUM',
    denominator=1e6,
    scale=1000,
)
```

```{code-cell} ipython3
geemap.pie_chart(
    'treeloss.csv', names='NAME', values='Class_sum', max_rows=20, height=600
)
```

```{code-cell} ipython3
geemap.bar_chart(
    'treeloss.csv',
    x='NAME',
    y='Class_sum',
    max_rows=20,
    x_label='Country',
    y_label='Forest loss area (km2)',
)
```

## Land cover change analysis

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

## Water app

```{code-cell} ipython3
col = ee.FeatureCollection(geemap.examples.get_ee_path('countries'))
```

```{code-cell} ipython3
geemap.examples.get_ee_path('countries')
```

```{code-cell} ipython3
col.size().getInfo()
```

```{code-cell} ipython3
# Check geemap installation
import subprocess

try:
    import geemap
except ImportError:
    print('geemap package is not installed. Installing ...')
    subprocess.check_call(["python", '-m', 'pip', 'install', 'geemap'])
```

```{code-cell} ipython3
# Import libraries
import os
import ee
import geemap
import ipywidgets as widgets
from bqplot import pyplot as plt
from ipyleaflet import WidgetControl
```

```{code-cell} ipython3
# Create an interactive map
Map = geemap.Map(center=[40, -100], zoom=4, add_google_map=False)
Map.add_basemap('HYBRID')
Map.add_basemap('ROADMAP')

# Add Earth Engine data
fc = ee.FeatureCollection('TIGER/2018/Counties')
Map.addLayer(fc, {}, 'US Counties')

states = ee.FeatureCollection('TIGER/2018/States')
# Map.addLayer(states, {}, 'US States')

Map
```

```{code-cell} ipython3
# Designe interactive widgets

style = {'description_width': 'initial'}

output_widget = widgets.Output(layout={'border': '1px solid black'})
output_control = WidgetControl(widget=output_widget, position='bottomright')
Map.add_control(output_control)

admin1_widget = widgets.Text(
    description='State:', value='Tennessee', width=200, style=style
)

admin2_widget = widgets.Text(
    description='County:', value='Knox', width=300, style=style
)

aoi_widget = widgets.Checkbox(
    value=False, description='Use user-drawn AOI', style=style
)

download_widget = widgets.Checkbox(
    value=False, description='Download chart data', style=style
)


def aoi_change(change):
    Map.layers = Map.layers[:4]
    Map.user_roi = None
    Map.user_rois = None
    Map.draw_count = 0
    admin1_widget.value = ''
    admin2_widget.value = ''
    output_widget.clear_output()


aoi_widget.observe(aoi_change, names='value')

band_combo = widgets.Dropdown(
    description='Band combo:',
    options=[
        'Red/Green/Blue',
        'NIR/Red/Green',
        'SWIR2/SWIR1/NIR',
        'NIR/SWIR1/Red',
        'SWIR2/NIR/Red',
        'SWIR2/SWIR1/Red',
        'SWIR1/NIR/Blue',
        'NIR/SWIR1/Blue',
        'SWIR2/NIR/Green',
        'SWIR1/NIR/Red',
    ],
    value='NIR/Red/Green',
    style=style,
)

year_widget = widgets.IntSlider(
    min=1984, max=2020, value=2010, description='Selected year:', width=400, style=style
)

fmask_widget = widgets.Checkbox(
    value=True, description='Apply fmask(remove cloud, shadow, snow)', style=style
)


# Normalized Satellite Indices: https://www.usna.edu/Users/oceano/pguth/md_help/html/norm_sat.htm

nd_options = [
    'Vegetation Index (NDVI)',
    'Water Index (NDWI)',
    'Modified Water Index (MNDWI)',
    'Snow Index (NDSI)',
    'Soil Index (NDSI)',
    'Burn Ratio (NBR)',
    'Customized',
]
nd_indices = widgets.Dropdown(
    options=nd_options,
    value='Modified Water Index (MNDWI)',
    description='Normalized Difference Index:',
    style=style,
)

first_band = widgets.Dropdown(
    description='1st band:',
    options=['Blue', 'Green', 'Red', 'NIR', 'SWIR1', 'SWIR2'],
    value='Green',
    style=style,
)

second_band = widgets.Dropdown(
    description='2nd band:',
    options=['Blue', 'Green', 'Red', 'NIR', 'SWIR1', 'SWIR2'],
    value='SWIR1',
    style=style,
)

nd_threshold = widgets.FloatSlider(
    value=0,
    min=-1,
    max=1,
    step=0.01,
    description='Threshold:',
    orientation='horizontal',
    style=style,
)

nd_color = widgets.ColorPicker(
    concise=False, description='Color:', value='blue', style=style
)


def nd_index_change(change):
    if nd_indices.value == 'Vegetation Index (NDVI)':
        first_band.value = 'NIR'
        second_band.value = 'Red'
    elif nd_indices.value == 'Water Index (NDWI)':
        first_band.value = 'NIR'
        second_band.value = 'SWIR1'
    elif nd_indices.value == 'Modified Water Index (MNDWI)':
        first_band.value = 'Green'
        second_band.value = 'SWIR1'
    elif nd_indices.value == 'Snow Index (NDSI)':
        first_band.value = 'Green'
        second_band.value = 'SWIR1'
    elif nd_indices.value == 'Soil Index (NDSI)':
        first_band.value = 'SWIR1'
        second_band.value = 'NIR'
    elif nd_indices.value == 'Burn Ratio (NBR)':
        first_band.value = 'NIR'
        second_band.value = 'SWIR2'
    elif nd_indices.value == 'Customized':
        first_band.value = None
        second_band.value = None


nd_indices.observe(nd_index_change, names='value')

submit = widgets.Button(
    description='Submit', button_style='primary', tooltip='Click me', style=style
)

full_widget = widgets.VBox(
    [
        widgets.HBox([admin1_widget, admin2_widget, aoi_widget, download_widget]),
        widgets.HBox([band_combo, year_widget, fmask_widget]),
        widgets.HBox([nd_indices, first_band, second_band, nd_threshold, nd_color]),
        submit,
    ]
)

full_widget
```

```{code-cell} ipython3
# Capture user interaction with the map


def handle_interaction(**kwargs):
    latlon = kwargs.get('coordinates')
    if kwargs.get('type') == 'click' and not aoi_widget.value:
        Map.default_style = {'cursor': 'wait'}
        xy = ee.Geometry.Point(latlon[::-1])
        selected_fc = fc.filterBounds(xy)

        with output_widget:
            output_widget.clear_output()

            try:
                feature = selected_fc.first()
                admin2_id = feature.get('NAME').getInfo()
                statefp = feature.get('STATEFP')
                admin1_fc = ee.Feature(
                    states.filter(ee.Filter.eq('STATEFP', statefp)).first()
                )
                admin1_id = admin1_fc.get('NAME').getInfo()
                admin1_widget.value = admin1_id
                admin2_widget.value = admin2_id
                Map.layers = Map.layers[:4]
                geom = selected_fc.geometry()
                layer_name = admin1_id + '-' + admin2_id
                Map.addLayer(
                    ee.Image().paint(geom, 0, 2), {'palette': 'red'}, layer_name
                )
                print(layer_name)
            except:
                print('No feature could be found')
                Map.layers = Map.layers[:4]

        Map.default_style = {'cursor': 'pointer'}
    else:
        Map.draw_count = 0


Map.on_interaction(handle_interaction)
```

```{code-cell} ipython3
# Click event handler


def submit_clicked(b):

    with output_widget:
        output_widget.clear_output()
        print('Computing...')
        Map.default_style = {'cursor': 'wait'}

        try:
            admin1_id = admin1_widget.value
            admin2_id = admin2_widget.value
            band1 = first_band.value
            band2 = second_band.value
            selected_year = year_widget.value
            threshold = nd_threshold.value
            bands = band_combo.value.split('/')
            apply_fmask = fmask_widget.value
            palette = nd_color.value
            use_aoi = aoi_widget.value
            download = download_widget.value

            if use_aoi:
                if Map.user_roi is not None:
                    roi = Map.user_roi
                    layer_name = 'User drawn AOI'
                    geom = roi
                else:
                    output_widget.clear_output()
                    print('No user AOI could be found.')
                    return
            else:

                statefp = ee.Feature(
                    states.filter(ee.Filter.eq('NAME', admin1_id)).first()
                ).get('STATEFP')
                roi = fc.filter(
                    ee.Filter.And(
                        ee.Filter.eq('NAME', admin2_id),
                        ee.Filter.eq('STATEFP', statefp),
                    )
                )
                layer_name = admin1_id + '-' + admin2_id
                geom = roi.geometry()

            Map.layers = Map.layers[:4]
            Map.addLayer(ee.Image().paint(geom, 0, 2), {'palette': 'red'}, layer_name)

            images = geemap.landsat_timeseries(
                roi=roi,
                start_year=1984,
                end_year=2020,
                start_date='01-01',
                end_date='12-31',
                apply_fmask=apply_fmask,
            )
            nd_images = images.map(lambda img: img.normalizedDifference([band1, band2]))
            result_images = nd_images.map(lambda img: img.gt(threshold))

            selected_image = ee.Image(
                images.toList(images.size()).get(selected_year - 1984)
            )
            selected_result_image = ee.Image(
                result_images.toList(result_images.size()).get(selected_year - 1984)
            ).selfMask()

            vis_params = {'bands': bands, 'min': 0, 'max': 3000}

            Map.addLayer(selected_image, vis_params, 'Landsat ' + str(selected_year))
            Map.addLayer(
                selected_result_image,
                {'palette': palette},
                'Result ' + str(selected_year),
            )

            def cal_area(img):
                pixel_area = img.multiply(ee.Image.pixelArea()).divide(1e4)
                img_area = pixel_area.reduceRegion(
                    **{
                        'geometry': geom,
                        'reducer': ee.Reducer.sum(),
                        'scale': 1000,
                        'maxPixels': 1e12,
                        'bestEffort': True,
                    }
                )
                return img.set({'area': img_area})

            areas = result_images.map(cal_area)
            stats = areas.aggregate_array('area').getInfo()
            x = list(range(1984, 2021))
            y = [item.get('nd') for item in stats]

            fig = plt.figure(1)
            fig.layout.height = '270px'
            plt.clear()
            plt.plot(x, y)
            plt.title('Temporal trend (1984-2020)')
            plt.xlabel('Year')
            plt.ylabel('Area (ha)')

            output_widget.clear_output()

            plt.show()

            if download:
                out_dir = os.path.join(os.path.expanduser('~'), 'Downloads')
                out_name = 'chart_' + geemap.random_string() + '.csv'
                out_csv = os.path.join(out_dir, out_name)
                if not os.path.exists(out_dir):
                    os.makedirs(out_dir)
                with open(out_csv, 'w') as f:
                    f.write('year, area (ha)\n')
                    for index, item in enumerate(x):
                        line = '{},{:.2f}\n'.format(item, y[index])
                        f.write(line)
                link = geemap.create_download_link(
                    out_csv, title="Click here to download the chart data: "
                )
                display(link)

        except Exception as e:
            print(e)
            print('An error occurred during computation.')

        Map.default_style = {'cursor': 'default'}


submit.on_click(submit_clicked)
```

## Summary

