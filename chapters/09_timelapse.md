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
fc = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017').filter(
    ee.Filter.eq('country_na', 'Netherlands')
)

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

## Creating timeseries

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
images.size().getInfo()
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

![](https://i.imgur.com/Me6D78k.gif)

+++

+++

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

![](https://i.imgur.com/VL1V1Y4.gif)

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

![](https://i.imgur.com/LzWyyZW.gif)

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

![](https://i.imgur.com/0nLRj22.gif)

+++

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
:tags: []

timelapse = geemap.sentinel1_timelapse(
    roi,
    out_gif='sentinel1.gif',
    start_year=2019,
    end_year=2019,
    start_date='04-01',
    end_date='08-01',
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

![](https://i.imgur.com/yTSBwnB.gif)

+++

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

![](https://i.imgur.com/BmQdo9j.gif)

+++

## MODIS timelapse

### MODIS NDVI

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
    end_date='2021-12-31',
    frames_per_second=3,
    title='MODIS NDVI Timelapse',
    overlay_data='countries',
)
geemap.show_image(timelapse)
```

![](https://i.imgur.com/KZwf8c2.gif)

+++

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

![](https://i.imgur.com/ELVn5jq.gif)

+++

## GOES timelapse

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
roi = Map.user_roi
if roi is None:
    roi = ee.Geometry.BBox(167.1898, -28.5757, 202.6258, -12.4411)
    Map.addLayer(roi)
    Map.centerObject(roi)
```

```{code-cell} ipython3
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

![](https://i.imgur.com/l67i6Pj.gif)

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

![](https://i.imgur.com/zYt2b8d.gif)

```{code-cell} ipython3
Map = geemap.Map()
roi = ee.Geometry.BBox(-121.0034, 36.8488, -117.9052, 39.0490)
Map.addLayer(roi)
Map.centerObject(roi)
Map
```

```{code-cell} ipython3
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

![](https://i.imgur.com/pgPjuLS.gif)

+++

## Fading effects

+++

```{code-cell} ipython3
in_gif = "https://i.imgur.com/ZWSZC5z.gif"
```

```{code-cell} ipython3
geemap.show_image(in_gif)
```

```{code-cell} ipython3
out_gif = "gif_fading.gif"
geemap.gif_fading(in_gif, out_gif, verbose=False)
geemap.show_image(out_gif)
```

![](https://i.imgur.com/PbK89b6.gif)

+++

```{code-cell} ipython3
roi = ee.Geometry.BBox(-69.3154, -22.8371, -69.1900, -22.7614)
```

```{code-cell} ipython3
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

![](https://i.imgur.com/1ZbWs66.gif)

+++

## Adding animated text

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
