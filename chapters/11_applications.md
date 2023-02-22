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

### Forest cover mapping

```{code-cell} ipython3
dataset = ee.Image('UMD/hansen/global_forest_change_2021_v1_9')
dataset.bandNames()
```

```{code-cell} ipython3
Map = geemap.Map()
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
name = 'Tree cover (%)'
Map.addLayer(treecover, treeCoverVisParam, name)
Map.add_colorbar(treeCoverVisParam, label=name, layer_name=name)
Map
```

```{code-cell} ipython3
threshold = 10
treecover_bin = treecover.gte(threshold).selfMask()
treeVisParam = {'palette': ['green']}
Map.addLayer(treecover_bin, treeVisParam, 'Tree cover bin')
```

### Forest loss and gain mapping

```{code-cell} ipython3
Map = geemap.Map()
treeloss_year = dataset.select(['lossyear'])
treeLossVisParam = {'min': 0, 'max': 21, 'palette': ['yellow', 'red']}
layer_name = 'Tree loss year'
Map.addLayer(treeloss_year, treeLossVisParam, layer_name)
Map.add_colorbar(treeLossVisParam, label=layer_name, layer_name=layer_name)
Map
```

```{code-cell} ipython3
treeloss = dataset.select(['loss']).selfMask()
treegain = dataset.select(['gain']).selfMask()
Map.addLayer(treeloss, {'palette': 'red'}, 'Tree loss')
Map.addLayer(treegain, {'palette': 'yellow'}, 'Tree gain')
Map
```

### Zonal statistics by country

```{code-cell} ipython3
Map = geemap.Map()
countries = ee.FeatureCollection(geemap.examples.get_ee_path('countries'))
style = {'color': '#000000ff', 'fillColor': '#00000000'}
Map.addLayer(countries.style(**style), {}, 'Countries')
Map
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
    'forest_cover.csv', names='NAME', values='Class_sum', max_rows=20, height=400
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

## Summary

## References

