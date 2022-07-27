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

# Creating Timelapses

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

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/giswqs/geebook/blob/master/chapters/09_timelapse.ipynb)

```{code-cell} ipython3
pip install pygis
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
print(myList.getInfo())
```

```{code-cell} ipython3
def computeSquares(number):
    return ee.Number(number).pow(2)

squares = myList.map(computeSquares)
print(squares.getInfo())
```

```{code-cell} ipython3
squares = myList.map(lambda number: ee.Number(number).pow(2))
print(squares.getInfo())
```

+++

## Creating cloud-free composites

```{code-cell} ipython3
Map = geemap.Map()
fc = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017') \
    .filter(ee.Filter.eq('country_na', 'Netherlands'))

Map.addLayer(fc, {'color': 'ff000000'}, "Netherlands")
Map.centerObject(fc)
Map
```

```{code-cell} ipython3
years = ee.List.sequence(2013, 2021)
```

```{code-cell} ipython3
def yearly_image(year):

    start_date = ee.Date.fromYMD(year, 1, 1)
    end_date = start_date.advance(1, "year")

    collection = (
        ee.ImageCollection('LANDSAT/LC08/C01/T1')
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
for index in range(0, 9):
    image = ee.Image(images.get(index))
    layer_name = "Year " + str(index + 2013)
    Map.addLayer(image, vis_params, layer_name)
```

## Creating image timeseries

```{code-cell} ipython3
collection = ee.ImageCollection("COPERNICUS/S2_HARMONIZED")
```

```{code-cell} ipython3
region = ee.Geometry.BBox(-122.5549, 37.6968, -122.3446, 37.8111)
```

```{code-cell} ipython3
start_date = '2016-01-01'
end_date = '2021-12-31'
```

```{code-cell} ipython3
images = geemap.create_timeseries(collection, start_date, end_date, Map.user_roi, frequency='year', reducer='median')
```

```{code-cell} ipython3
images.getInfo()
```

```{code-cell} ipython3
Map = geemap.Map()

# S2 = (
#     ee.ImageCollection('COPERNICUS/S2_SR')
#     .filterBounds(ee.Geometry.Point([-122.45, 37.75]))
#     .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than', 10)
# )

vis_params = {"min": 0, "max": 4000, "bands": ["B8", "B4", "B3"]}

Map.addLayer(images, {}, "Sentinel-2", False)
Map.add_time_slider(images, vis_params, time_interval=2)
Map.centerObject(region)
Map
```

+++

## Creating timelapses

### NAIP

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
Map
```

```{code-cell} ipython3
region = Map.user_roi
if region is None:
    region = ee.Geometry.BBox(-99.1019, 47.1274, -99.0334, 47.1562)
    Map.addLayer(region)
    Map.centerObject(region)
```

```{code-cell} ipython3
collection = geemap.naip_timeseries(region, start_year=2009, end_year=2020, RGBN=True)
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
geemap.naip_timelapse(
    region, out_gif="naip.gif", bands=['N', 'R', 'G'], frames_per_second=3, title='NAIP Timelapse'
)
```

## Creating Landsat timelapses

### Adding animated text

```{code-cell} ipython3
import geemap
import os
```

```{code-cell} ipython3
geemap.show_youtube('fDnDVuM_Ke4')
```

### Update the geemap package

```{code-cell} ipython3
# geemap.update_package()
```

### Add animated text to an existing GIF

```{code-cell} ipython3
in_gif = os.path.abspath('../data/animation.gif')
out_dir = os.path.join(os.path.expanduser('~'), 'Downloads')
out_gif = os.path.join(out_dir, 'output.gif')
if not os.path.exists(out_dir):
    os.makedirs(out_dir)
```

```{code-cell} ipython3
geemap.show_image(in_gif)
```

#### Add animated text to GIF

```{code-cell} ipython3
geemap.add_text_to_gif(
    in_gif,
    out_gif,
    xy=('5%', '5%'),
    text_sequence=1984,
    font_size=30,
    font_color='#0000ff',
    duration=100,
)
```

```{code-cell} ipython3
geemap.show_image(out_gif)
```

#### Add place name

```{code-cell} ipython3
geemap.add_text_to_gif(
    out_gif, out_gif, xy=('30%', '85%'), text_sequence="Las Vegas", font_color='black'
)
```

```{code-cell} ipython3
geemap.show_image(out_gif)
```

#### Change font type

```{code-cell} ipython3
geemap.system_fonts()
```

```{code-cell} ipython3
geemap.add_text_to_gif(
    in_gif,
    out_gif,
    xy=('5%', '5%'),
    text_sequence=1984,
    font_size=30,
    font_color='#0000ff',
    duration=100,
)
geemap.add_text_to_gif(
    out_gif,
    out_gif,
    xy=('30%', '85%'),
    text_sequence="Las Vegas",
    font_type="timesbd.ttf",
    font_size=30,
    font_color='black',
)
geemap.show_image(out_gif)
```

### Create GIF from Earth Engine data

+++

#### Prepare for an ImageCollection

```{code-cell} ipython3
import ee
import geemap

ee.Initialize()

# Define an area of interest geometry with a global non-polar extent.
aoi = ee.Geometry.Polygon(
    [[[-179.0, 78.0], [-179.0, -58.0], [179.0, -58.0], [179.0, 78.0]]], None, False
)

# Import hourly predicted temperature image collection for northern winter
# solstice. Note that predictions extend for 384 hours; limit the collection
# to the first 24 hours.
tempCol = (
    ee.ImageCollection('NOAA/GFS0P25')
    .filterDate('2018-12-22', '2018-12-23')
    .limit(24)
    .select('temperature_2m_above_ground')
)

# Define arguments for animation function parameters.
videoArgs = {
    'dimensions': 768,
    'region': aoi,
    'framesPerSecond': 10,
    'crs': 'EPSG:3857',
    'min': -40.0,
    'max': 35.0,
    'palette': ['blue', 'purple', 'cyan', 'green', 'yellow', 'red'],
}
```

#### Save the GIF to local drive

```{code-cell} ipython3
saved_gif = os.path.join(os.path.expanduser('~'), 'Downloads/temperature.gif')
geemap.download_ee_video(tempCol, videoArgs, saved_gif)
```

```{code-cell} ipython3
geemap.show_image(saved_gif)
```

#### Generate an hourly text sequence

```{code-cell} ipython3
text = [str(n).zfill(2) + ":00" for n in range(0, 24)]
print(text)
```

#### Add text to GIF

```{code-cell} ipython3
out_gif = os.path.join(os.path.expanduser('~'), 'Downloads/output2.gif')
```

```{code-cell} ipython3
geemap.add_text_to_gif(
    saved_gif,
    out_gif,
    xy=('3%', '5%'),
    text_sequence=text,
    font_size=30,
    font_color='#ffffff',
)
```

```{code-cell} ipython3
geemap.add_text_to_gif(
    out_gif,
    out_gif,
    xy=('32%', '92%'),
    text_sequence='NOAA GFS Hourly Temperature',
    font_color='white',
)
```

```{code-cell} ipython3
geemap.show_image(out_gif)
```

### Adding colorbar to an image

```{code-cell} ipython3
import geemap
import os
```

```{code-cell} ipython3
# geemap.update_package()
```

#### Download a GIF

```{code-cell} ipython3
from geemap import *
```

```{code-cell} ipython3
url = 'https://i.imgur.com/MSde1om.gif'
out_dir = os.path.join(os.path.expanduser('~'), 'Downloads')
if not os.path.exists(out_dir):
    os.makedirs(out_dir)
download_from_url(url, out_file_name='temp.gif', out_dir=out_dir)
```

```{code-cell} ipython3
in_gif = os.path.join(out_dir, 'temp.gif')
show_image(in_gif)
```

#### Get image URLs

```{code-cell} ipython3
noaa_logo = 'https://bit.ly/3ahJoMq'
ee_logo = 'https://i.imgur.com/Qbvacvm.jpg'
```

#### Set output GIF path

```{code-cell} ipython3
out_gif = os.path.join(out_dir, 'output.gif')
```

#### Add images to GIF

```{code-cell} ipython3
add_image_to_gif(
    in_gif, out_gif, in_image=noaa_logo, xy=('2%', '80%'), image_size=(80, 80)
)
```

```{code-cell} ipython3
add_image_to_gif(
    out_gif, out_gif, in_image=ee_logo, xy=('13%', '79%'), image_size=(85, 85)
)
```

#### Display output GIF

```{code-cell} ipython3
show_image(out_gif)
```

#### Create a colorbar

```{code-cell} ipython3
width = 250
height = 30
palette = ['blue', 'purple', 'cyan', 'green', 'yellow', 'red']
labels = [-40, 35]
colorbar = create_colorbar(
    width=width,
    height=height,
    palette=palette,
    vertical=False,
    add_labels=True,
    font_size=20,
    labels=labels,
)
```

```{code-cell} ipython3
show_image(colorbar)
```

#### Add colorbar to GIF

```{code-cell} ipython3
add_image_to_gif(
    out_gif, out_gif, in_image=colorbar, xy=('69%', '89%'), image_size=(250, 250)
)
```

```{code-cell} ipython3
show_image(out_gif)
```

### Timelapse fading

```{code-cell} ipython3
import ee
import geemap
```

```{code-cell} ipython3
url = "https://i.imgur.com/ZWSZC5z.gif"
```

```{code-cell} ipython3
geemap.show_image(url, verbose=False)
```

```{code-cell} ipython3
out_gif = "gif_fading.gif"
```

```{code-cell} ipython3
geemap.gif_fading(url, out_gif, verbose=False)
```

```{code-cell} ipython3
geemap.show_image(out_gif)
```

+++

```{code-cell} ipython3
Map = geemap.Map()
Map.add_basemap("HYBRID")
Map
```

```{code-cell} ipython3
roi = ee.Geometry.Polygon(
    [
        [
            [-69.315491, -22.837104],
            [-69.315491, -22.751488],
            [-69.190006, -22.751488],
            [-69.190006, -22.837104],
            [-69.315491, -22.837104],
        ]
    ]
)
```

```{code-cell} ipython3
Map.addLayer(roi, {}, "ROI")
Map.centerObject(roi)
```

```{code-cell} ipython3
title = "Sierra Gorda copper mines, Chile"
out_gif = "timelapse.gif"
```

```{code-cell} ipython3
geemap.landsat_timelapse(
    roi,
    out_gif,
    start_year=2004,
    end_year=2010,
    frames_per_second=1,
    title=title,
    fading=False,
)
```

```{code-cell} ipython3
geemap.show_image(out_gif)
```

```{code-cell} ipython3
out_fading_gif = "timelapse_fading.gif"
```

```{code-cell} ipython3
geemap.landsat_timelapse(
    roi,
    out_fading_gif,
    start_year=2004,
    end_year=2010,
    frames_per_second=1,
    title=title,
    fading=True,
)
```

```{code-cell} ipython3
geemap.show_image(out_fading_gif)
```

+++

## Creating GOES timelapses

```{code-cell} ipython3
import os
import ee
import geemap
```

```{code-cell} ipython3
# geemap.update_package()
```

```{code-cell} ipython3
m = geemap.ee_initialize()
```

```{code-cell} ipython3
region = ee.Geometry.Polygon(
    [
        [
            [-159.5954379282731, 60.40883060191719],
            [-159.5954379282731, 24.517881970830725],
            [-114.2438754282731, 24.517881970830725],
            [-114.2438754282731, 60.40883060191719],
        ]
    ],
    None,
    False,
)
```

```{code-cell} ipython3
start_date = "2021-10-24T14:00:00"
end_date = "2021-10-25T01:00:00"
data = "GOES-17"
scan = "full_disk"
```

```{code-cell} ipython3
col = geemap.goes_timeseries(start_date, end_date, data, scan, region)
```

```{code-cell} ipython3
visParams = {
    'bands': ['CMI_C02', 'CMI_GREEN', 'CMI_C01'],
    'min': 0,
    'max': 0.8,
    'dimensions': 700,
    'framesPerSecond': 9,
    'region': region,
    'crs': col.first().projection(),
}
```

```{code-cell} ipython3
out_dir = os.path.expanduser("~/Downloads")
out_gif = os.path.join(out_dir, "goes_timelapse.gif")
if not os.path.exists(out_dir):
    os.makedirs(out_dir)
```

```{code-cell} ipython3
geemap.download_ee_video(col, visParams, out_gif)
```

```{code-cell} ipython3
timestamps = geemap.image_dates(col, date_format='YYYY-MM-dd HH:mm').getInfo()
```

```{code-cell} ipython3
geemap.add_text_to_gif(
    out_gif,
    out_gif,
    xy=('3%', '3%'),
    text_sequence=timestamps,
    font_size=20,
    font_color='#ffffff',
)
```

```{code-cell} ipython3
geemap.goes_timelapse(
    out_gif, start_date, end_date, data, scan, region, framesPerSecond=10
)
```

## Creating MODIS timelapses

+++

## Summary

## References

- https://geemap.org/notebooks/16_add_animated_text/
- https://geemap.org/notebooks/17_add_colorbar_to_gif/
- https://geemap.org/notebooks/18_create_landsat_timelapse/
- https://geemap.org/notebooks/39_timelapse/
- https://geemap.org/notebooks/71_timelapse/
- https://geemap.org/notebooks/72_time_slider_gui/
- https://geemap.org/notebooks/81_goes_timelapse/
- https://geemap.org/notebooks/90_naip_timelapse/
- https://geemap.org/notebooks/98_timelapse_fading/
