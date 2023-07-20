---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.4
kernelspec:
  display_name: geo
  language: python
  name: python3
---

# Creating Timelapse Animations

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

## The map function

```{code-cell} ipython3
myList = ee.List.sequence(1, 10)
myList
```

```{code-cell} ipython3
def computeSquares(number):
    return ee.Number(number).pow(2)


squares = myList.map(computeSquares)
squares
```

```{code-cell} ipython3
squares = myList.map(lambda number: ee.Number(number).pow(2))
squares
```

+++

## Creating cloud-free composites

```{code-cell} ipython3
Map = geemap.Map()
fc = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017').filter(
    ee.Filter.eq('country_na', 'Netherlands')
)

Map.addLayer(fc, {'color': 'ff000000'}, "Netherlands")
Map.centerObject(fc)
Map
```

```{code-cell} ipython3
years = ee.List.sequence(2013, 2022)
```

```{code-cell} ipython3
def yearly_image(year):

    start_date = ee.Date.fromYMD(year, 1, 1)
    end_date = start_date.advance(1, "year")

    collection = (
        ee.ImageCollection('LANDSAT/LC08/C02/T1')
        .filterDate(start_date, end_date)
        .filterBounds(fc)
    )

    image = ee.Algorithms.Landsat.simpleComposite(collection).clipToCollection(fc)

    return image
```

```{code-cell} ipython3
images = years.map(yearly_image)
```

```{code-cell} ipython3
vis_params = {'bands': ['B5', 'B4', 'B3'], 'max': 128}
for index in range(0, 10):
    image = ee.Image(images.get(index))
    layer_name = "Year " + str(index + 2013)
    Map.addLayer(image, vis_params, layer_name)
Map
```

+++

## Creating time series

```{code-cell} ipython3
collection = ee.ImageCollection("COPERNICUS/S2_HARMONIZED").filterMetadata(
    'CLOUDY_PIXEL_PERCENTAGE', 'less_than', 10
)
```

```{code-cell} ipython3
start_date = '2016-01-01'
end_date = '2022-12-31'
region = ee.Geometry.BBox(-122.5549, 37.6968, -122.3446, 37.8111)
```

```{code-cell} ipython3
images = geemap.create_timeseries(
    collection, start_date, end_date, region, frequency='year', reducer='median'
)
images
```

```{code-cell} ipython3
Map = geemap.Map()

vis_params = {"min": 0, "max": 4000, "bands": ["B8", "B4", "B3"]}
labels = [str(y) for y in range(2016, 2023)]

Map.addLayer(images, vis_params, "Sentinel-2", False)
Map.add_time_slider(images, vis_params, time_interval=2, labels=labels)
Map.centerObject(region)
Map
```

+++

+++

## NAIP timelapse

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
Map
```

```{code-cell} ipython3
roi = Map.user_roi
if roi is None:
    roi = ee.Geometry.BBox(-99.1019, 47.1274, -99.0334, 47.1562)
    Map.addLayer(roi)
    Map.centerObject(roi)
```

```{code-cell} ipython3
collection = geemap.naip_timeseries(roi, start_year=2009, end_year=2022, RGBN=True)
```

```{code-cell} ipython3
years = geemap.image_dates(collection, date_format='YYYY').getInfo()
print(years)
```

```{code-cell} ipython3
size = len(years)
images = collection.toList(size)
for i in range(size):
    image = ee.Image(images.get(i))
    Map.addLayer(image, {'bands': ['N', 'R', 'G']}, years[i])
Map
```

```{code-cell} ipython3
timelapse = geemap.naip_timelapse(
    roi,
    out_gif="naip.gif",
    bands=['N', 'R', 'G'],
    frames_per_second=3,
    title='NAIP Timelapse',
)
geemap.show_image(timelapse)
```

## Landsat timelapse

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
roi = Map.user_roi
if roi is None:
    roi = ee.Geometry.BBox(-74.7222, -8.5867, -74.1596, -8.2824)
    Map.addLayer(roi)
    Map.centerObject(roi)
```

```{code-cell} ipython3
timelapse = geemap.landsat_timelapse(
    roi,
    out_gif='landsat.gif',
    start_year=1984,
    end_year=2022,
    start_date='01-01',
    end_date='12-31',
    bands=['SWIR1', 'NIR', 'Red'],
    frames_per_second=5,
    title='Landsat Timelapse',
    progress_bar_color='blue',
    mp4=True,
)
geemap.show_image(timelapse)
```

```{code-cell} ipython3
Map = geemap.Map()
roi = ee.Geometry.BBox(-115.5541, 35.8044, -113.9035, 36.5581)
Map.addLayer(roi)
Map.centerObject(roi)
Map
```

```{code-cell} ipython3
timelapse = geemap.landsat_timelapse(
    roi,
    out_gif='las_vegas.gif',
    start_year=1984,
    end_year=2022,
    bands=['NIR', 'Red', 'Green'],
    frames_per_second=5,
    title='Las Vegas, NV',
    font_color='blue',
)
geemap.show_image(timelapse)
```

```{code-cell} ipython3
Map = geemap.Map()
roi = ee.Geometry.BBox(113.8252, 22.1988, 114.0851, 22.3497)
Map.addLayer(roi)
Map.centerObject(roi)
Map
```

```{code-cell} ipython3
timelapse = geemap.landsat_timelapse(
    roi,
    out_gif='hong_kong.gif',
    start_year=1990,
    end_year=2022,
    start_date='01-01',
    end_date='12-31',
    bands=['SWIR1', 'NIR', 'Red'],
    frames_per_second=3,
    title='Hong Kong',
)
geemap.show_image(timelapse)
```

## Sentinel-1 timelapse

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
roi = Map.user_roi
if roi is None:
    roi = ee.Geometry.BBox(117.1132, 3.5227, 117.2214, 3.5843)
    Map.addLayer(roi)
    Map.centerObject(roi)
```

```{code-cell} ipython3
timelapse = geemap.sentinel1_timelapse(
    roi,
    out_gif='sentinel1.gif',
    start_year=2019,
    end_year=2019,
    start_date='04-01',
    end_date='08-01',
    bands=['VV'],
    frequency='day',
    vis_params={"min": -30, "max": 0},
    palette="Greys",
    frames_per_second=3,
    title='Sentinel-1 Timelapse',
    add_colorbar=True,
    colorbar_bg_color='gray',
)
geemap.show_image(timelapse)
```

## Sentinel-2 timelapse

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
roi = Map.user_roi
if roi is None:
    roi = ee.Geometry.BBox(-74.7222, -8.5867, -74.1596, -8.2824)
    Map.addLayer(roi)
    Map.centerObject(roi)
```

```{code-cell} ipython3
timelapse = geemap.sentinel2_timelapse(
    roi,
    out_gif='sentinel2.gif',
    start_year=2016,
    end_year=2021,
    start_date='01-01',
    end_date='12-31',
    frequency='year',
    bands=['SWIR1', 'NIR', 'Red'],
    frames_per_second=3,
    title='Sentinel-2 Timelapse',
)
geemap.show_image(timelapse)
```

## MODIS timelapse

### MODIS vegetation indices

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
roi = Map.user_roi
if roi is None:
    roi = ee.Geometry.BBox(-18.6983, -36.1630, 52.2293, 38.1446)
    Map.addLayer(roi)
    Map.centerObject(roi)
```

```{code-cell} ipython3
timelapse = geemap.modis_ndvi_timelapse(
    roi,
    out_gif='ndvi.gif',
    data='Terra',
    band='NDVI',
    start_date='2000-01-01',
    end_date='2022-12-31',
    frames_per_second=3,
    title='MODIS NDVI Timelapse',
    overlay_data='countries',
)
geemap.show_image(timelapse)
```

### MODIS temperature

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
roi = Map.user_roi
if roi is None:
    roi = ee.Geometry.BBox(-171.21, -57.13, 177.53, 79.99)
    Map.addLayer(roi)
    Map.centerObject(roi)
```

```{code-cell} ipython3
timelapse = geemap.modis_ocean_color_timelapse(
    satellite='Aqua',
    start_date='2018-01-01',
    end_date='2020-12-31',
    roi=roi,
    frequency='month',
    out_gif='temperature.gif',
    overlay_data='continents',
    overlay_color='yellow',
    overlay_opacity=0.5,
)
geemap.show_image(timelapse)
```

## GOES timelapse

```{code-cell} ipython3
roi = ee.Geometry.BBox(167.1898, -28.5757, 202.6258, -12.4411)
start_date = "2022-01-15T03:00:00"
end_date = "2022-01-15T07:00:00"
data = "GOES-17"
scan = "full_disk"
```

```{code-cell} ipython3
timelapse = geemap.goes_timelapse(
    roi, "goes.gif", start_date, end_date, data, scan, framesPerSecond=5
)
geemap.show_image(timelapse)
```

```{code-cell} ipython3
roi = ee.Geometry.BBox(-159.5954, 24.5178, -114.2438, 60.4088)
start_date = "2021-10-24T14:00:00"
end_date = "2021-10-25T01:00:00"
data = "GOES-17"
scan = "full_disk"
```

```{code-cell} ipython3
timelapse = geemap.goes_timelapse(
    roi, "hurricane.gif", start_date, end_date, data, scan, framesPerSecond=5
)
geemap.show_image(timelapse)
```

```{code-cell} ipython3
roi = ee.Geometry.BBox(-121.0034, 36.8488, -117.9052, 39.0490)
start_date = "2020-09-05T15:00:00"
end_date = "2020-09-06T02:00:00"
data = "GOES-17"
scan = "full_disk"
```

```{code-cell} ipython3
timelapse = geemap.goes_fire_timelapse(
    roi, "fire.gif", start_date, end_date, data, scan, framesPerSecond=5
)
geemap.show_image(timelapse)
```

## Fading effects

```{code-cell} ipython3
in_gif = "https://i.imgur.com/ZWSZC5z.gif"
geemap.show_image(in_gif)
```

```{code-cell} ipython3
out_gif = "gif_fading.gif"
geemap.gif_fading(in_gif, out_gif, verbose=False)
geemap.show_image(out_gif)
```

```{code-cell} ipython3
roi = ee.Geometry.BBox(-69.3154, -22.8371, -69.1900, -22.7614)
timelapse = geemap.landsat_timelapse(
    roi,
    out_gif='mines.gif',
    start_year=2004,
    end_year=2010,
    frames_per_second=1,
    title='Copper mines, Chile',
    fading=True,
)
geemap.show_image(timelapse)
```

+++

## Adding text to timelapse

```{code-cell} ipython3
url = 'https://i.imgur.com/Rx0wjSw.gif'
in_gif = 'animation.gif'
geemap.download_file(url, in_gif)
geemap.show_image(in_gif)
```

```{code-cell} ipython3
out_gif = 'las_vegas.gif'
geemap.add_text_to_gif(
    in_gif,
    out_gif,
    xy=('3%', '5%'),
    text_sequence=1984,
    font_size=30,
    font_color='#0000ff',
    add_progress_bar=True,
    progress_bar_color='#ffffff',
    progress_bar_height=5,
    duration=100,
    loop=0,
)
geemap.show_image(out_gif)
```

```{code-cell} ipython3
geemap.add_text_to_gif(
    out_gif, out_gif, xy=('45%', '90%'), text_sequence="Las Vegas", font_color='black'
)
geemap.show_image(out_gif)
```

+++

## Adding image and colorbar to timelapse

### Preparing data

```{code-cell} ipython3
aoi = ee.Geometry.Polygon(
    [[[-179.0, 78.0], [-179.0, -58.0], [179.0, -58.0], [179.0, 78.0]]], None, False
)

collection = (
    ee.ImageCollection('NOAA/GFS0P25')
    .filterDate('2018-12-22', '2018-12-23')
    .limit(24)
    .select('temperature_2m_above_ground')
)

video_args = {
    'dimensions': 768,
    'region': aoi,
    'framesPerSecond': 10,
    'crs': 'EPSG:3857',
    'min': -35.0,
    'max': 35.0,
    'palette': ['blue', 'purple', 'cyan', 'green', 'yellow', 'red'],
}

saved_gif = 'temperature.gif'
geemap.download_ee_video(collection, video_args, saved_gif)
geemap.show_image(saved_gif)
```

### Adding animated text

```{code-cell} ipython3
text = [str(n).zfill(2) + ":00" for n in range(0, 24)]
out_gif = 'temperature_v2.gif'
geemap.add_text_to_gif(
    saved_gif,
    out_gif,
    xy=('3%', '5%'),
    text_sequence=text,
    font_size=30,
    font_color='#ffffff',
)

geemap.add_text_to_gif(
    out_gif,
    out_gif,
    xy=('32%', '92%'),
    text_sequence='NOAA GFS Hourly Temperature',
    font_color='white',
)
geemap.show_image(out_gif)
```

### Adding logo

```{code-cell} ipython3
noaa_logo = 'https://i.imgur.com/gZ6BYZB.png'
ee_logo = 'https://i.imgur.com/Qbvacvm.jpg'

geemap.add_image_to_gif(
    out_gif, out_gif, in_image=noaa_logo, xy=('2%', '80%'), image_size=(80, 80)
)

geemap.add_image_to_gif(
    out_gif, out_gif, in_image=ee_logo, xy=('13%', '79%'), image_size=(85, 85)
)
```

### Adding colorbar

```{code-cell} ipython3
palette = ['blue', 'purple', 'cyan', 'green', 'yellow', 'red']
colorbar = geemap.save_colorbar(width=2.5, height=0.3, vmin=-35, vmax=35, palette=palette, transparent=True)
geemap.show_image(colorbar)
```

```{code-cell} ipython3
geemap.add_image_to_gif(
    out_gif, out_gif, in_image=colorbar, xy=('72%', '85%'), image_size=(250, 250)
)
geemap.show_image(out_gif)
```

+++

## Summary

