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

# Making Maps with Cartoee

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

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/giswqs/geebook/blob/master/chapters/08_cartoee.ipynb)

```{code-cell} ipython3
!pip install pygis
```

```{code-cell} ipython3
import ee
import geemap
```

```{code-cell} ipython3
geemap.ee_initialize()
```

## Cartoee quickstart

```{code-cell} ipython3
%pylab inline

import ee
import geemap

# import the cartoee functionality from geemap
from geemap import cartoee
```

```{code-cell} ipython3
geemap.ee_initialize()
```

### Plotting an image

```{code-cell} ipython3
# get an image
srtm = ee.Image("CGIAR/SRTM90_V4")
```

```{code-cell} ipython3
# geospatial region in format [E,S,W,N]
region = [180, -60, -180, 85]  # define bounding box to request data
vis = {'min': 0, 'max': 3000}  # define visualization parameters for image
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# use cartoee to get a map
ax = cartoee.get_map(srtm, region=region, vis_params=vis)

# add a colorbar to the map using the visualization params we passed to the map
cartoee.add_colorbar(ax, vis, loc="bottom", label="Elevation", orientation="horizontal")

# add gridlines to the map at a specified interval
cartoee.add_gridlines(ax, interval=[60, 30], linestyle=":")

# add coastlines using the cartopy api
ax.coastlines(color="red")

show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

cmap = "gist_earth"  # colormap we want to use
# cmap = "terrain"

# use cartoee to get a map
ax = cartoee.get_map(srtm, region=region, vis_params=vis, cmap=cmap)

# add a colorbar to the map using the visualization params we passed to the map
cartoee.add_colorbar(
    ax, vis, cmap=cmap, loc="right", label="Elevation", orientation="vertical"
)

# add gridlines to the map at a specified interval
cartoee.add_gridlines(ax, interval=[60, 30], linestyle="--")

# add coastlines using the cartopy api
ax.coastlines(color="red")

ax.set_title(label='Global Elevation Map', fontsize=15)

show()
```

### Plotting an RGB image

```{code-cell} ipython3
# get a landsat image to visualize
image = ee.Image('LANDSAT/LC08/C01/T1_SR/LC08_044034_20140318')

# define the visualization parameters to view
vis = {"bands": ['B5', 'B4', 'B3'], "min": 0, "max": 5000, "gamma": 1.3}
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# use cartoee to get a map
ax = cartoee.get_map(image, vis_params=vis)

# pad the view for some visual appeal
cartoee.pad_view(ax)

# add the gridlines and specify that the xtick labels be rotated 45 degrees
cartoee.add_gridlines(ax, interval=0.5, xtick_rotation=45, linestyle=":")

# add the coastline
ax.coastlines(color="yellow")

show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# here is the bounding box of the map extent we want to use
# formatted a [E,S,W,N]
zoom_region = [-121.8025, 37.3458, -122.6265, 37.9178]

# plot the map over the region of interest
ax = cartoee.get_map(image, vis_params=vis, region=zoom_region)

# add the gridlines and specify that the xtick labels be rotated 45 degrees
cartoee.add_gridlines(ax, interval=0.15, xtick_rotation=45, linestyle=":")

# add coastline
ax.coastlines(color="yellow")

show()
```

### Adding north arrow and scale bar

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# here is the bounding box of the map extent we want to use
# formatted a [E,S,W,N]
zoom_region = [-121.8025, 37.3458, -122.6265, 37.9178]

# plot the map over the region of interest
ax = cartoee.get_map(image, vis_params=vis, region=zoom_region)

# add the gridlines and specify that the xtick labels be rotated 45 degrees
cartoee.add_gridlines(ax, interval=0.15, xtick_rotation=45, linestyle=":")

# add coastline
ax.coastlines(color="yellow")

# add north arrow
cartoee.add_north_arrow(
    ax, text="N", xy=(0.05, 0.25), text_color="white", arrow_color="white", fontsize=20
)

# add scale bar
cartoee.add_scale_bar_lite(
    ax, length=10, xy=(0.1, 0.05), fontsize=20, color="white", unit="km"
)

ax.set_title(label='Landsat False Color Composite (Band 5/4/3)', fontsize=15)

show()
```

+++

## Using custom projections

```{code-cell} ipython3
# !pip install cartopy scipy
# !pip install geemap
```

```{code-cell} ipython3
import ee
import geemap
from geemap import cartoee
import cartopy.crs as ccrs

%pylab inline
```

```{code-cell} ipython3
geemap.ee_initialize()
```

### Plotting an image on a map

```{code-cell} ipython3
# get an earth engine image of ocean data for Jan-Mar 2018
ocean = (
    ee.ImageCollection('NASA/OCEANDATA/MODIS-Terra/L3SMI')
    .filter(ee.Filter.date('2018-01-01', '2018-03-01'))
    .median()
    .select(["sst"], ["SST"])
)
```

```{code-cell} ipython3
# set parameters for plotting
# will plot the Sea Surface Temp with specific range and colormap
visualization = {'bands': "SST", 'min': -2, 'max': 30}
# specify region to focus on
bbox = [180, -88, -180, 88]
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# plot the result with cartoee using a PlateCarre projection (default)
ax = cartoee.get_map(ocean, cmap='plasma', vis_params=visualization, region=bbox)
cb = cartoee.add_colorbar(ax, vis_params=visualization, loc='right', cmap='plasma')

ax.set_title(label='Sea Surface Temperature', fontsize=15)

ax.coastlines()
plt.show()
```

### Mapping with different projections

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# create a new Mollweide projection centered on the Pacific
projection = ccrs.Mollweide(central_longitude=-180)

# plot the result with cartoee using the Mollweide projection
ax = cartoee.get_map(
    ocean, vis_params=visualization, region=bbox, cmap='plasma', proj=projection
)
cb = cartoee.add_colorbar(
    ax, vis_params=visualization, loc='bottom', cmap='plasma', orientation='horizontal'
)

ax.set_title("Mollweide projection")

ax.coastlines()
plt.show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# create a new Goode homolosine projection centered on the Pacific
projection = ccrs.Robinson(central_longitude=-180)

# plot the result with cartoee using the Goode homolosine projection
ax = cartoee.get_map(
    ocean, vis_params=visualization, region=bbox, cmap='plasma', proj=projection
)
cb = cartoee.add_colorbar(
    ax, vis_params=visualization, loc='bottom', cmap='plasma', orientation='horizontal'
)

ax.set_title("Robinson projection")

ax.coastlines()
plt.show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# create a new Goode homolosine projection centered on the Pacific
projection = ccrs.InterruptedGoodeHomolosine(central_longitude=-180)

# plot the result with cartoee using the Goode homolosine projection
ax = cartoee.get_map(
    ocean, vis_params=visualization, region=bbox, cmap='plasma', proj=projection
)
cb = cartoee.add_colorbar(
    ax, vis_params=visualization, loc='bottom', cmap='plasma', orientation='horizontal'
)

ax.set_title("Goode homolosine projection")

ax.coastlines()
plt.show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# create a new orographic projection focused on the Pacific
projection = ccrs.EqualEarth(central_longitude=-180)

# plot the result with cartoee using the orographic projection
ax = cartoee.get_map(
    ocean, vis_params=visualization, region=bbox, cmap='plasma', proj=projection
)
cb = cartoee.add_colorbar(
    ax, vis_params=visualization, loc='right', cmap='plasma', orientation='vertical'
)

ax.set_title("Equal Earth projection")

ax.coastlines()
plt.show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# create a new orographic projection focused on the Pacific
projection = ccrs.Orthographic(-130, -10)

# plot the result with cartoee using the orographic projection
ax = cartoee.get_map(
    ocean, vis_params=visualization, region=bbox, cmap='plasma', proj=projection
)
cb = cartoee.add_colorbar(
    ax, vis_params=visualization, loc='right', cmap='plasma', orientation='vertical'
)

ax.set_title("Orographic projection")

ax.coastlines()
plt.show()
```

### Warping artifacts

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# Create a new region to focus on
spole = [180, -88, -180, 0]

projection = ccrs.SouthPolarStereo()

# plot the result with cartoee focusing on the south pole
ax = cartoee.get_map(
    ocean, cmap='plasma', vis_params=visualization, region=spole, proj=projection
)
cb = cartoee.add_colorbar(ax, vis_params=visualization, loc='right', cmap='plasma')

ax.coastlines()
ax.set_title('The South Pole')
plt.show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# plot the result with cartoee focusing on the south pole
ax = cartoee.get_map(
    ocean, cmap='plasma', vis_params=visualization, region=spole, proj=projection
)
cb = cartoee.add_colorbar(ax, vis_params=visualization, loc='right', cmap='plasma')

ax.coastlines()
ax.set_title('The South Pole')

# get bounding box coordinates of a zoom area
zoom = spole
zoom[-1] = -20

# convert bbox coordinate from [W,S,E,N] to [W,E,S,N] as matplotlib expects
zoom_extent = cartoee.bbox_to_extent(zoom)

# set the extent of the map to the zoom area
ax.set_extent(zoom_extent, ccrs.PlateCarree())

plt.show()
```

+++

## Using multiple data layers

```{code-cell} ipython3
import ee
import geemap
from geemap import cartoee
import cartopy.crs as ccrs

%pylab inline
```

### Create an interactive map

```{code-cell} ipython3
Map = geemap.Map()

image = (
    ee.ImageCollection('MODIS/MCD43A4_006_NDVI')
    .filter(ee.Filter.date('2018-04-01', '2018-05-01'))
    .select("NDVI")
    .first()
)

vis_params = {
    'min': 0.0,
    'max': 1.0,
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
Map.setCenter(-7.03125, 31.0529339857, 2)
Map.addLayer(image, vis_params, 'MODIS NDVI')

countries = ee.FeatureCollection('users/giswqs/public/countries')
style = {"color": "00000088", "width": 1, "fillColor": "00000000"}
Map.addLayer(countries.style(**style), {}, "Countries")

ndvi = image.visualize(**vis_params)
blend = ndvi.blend(countries.style(**style))

Map.addLayer(blend, {}, "Blend")

Map
```

### Plot an image with the default projection

```{code-cell} ipython3
# specify region to focus on
bbox = [180, -88, -180, 88]
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# plot the result with cartoee using a PlateCarre projection (default)
ax = cartoee.get_map(blend, region=bbox)
cb = cartoee.add_colorbar(ax, vis_params=vis_params, loc='right')

ax.set_title(label='MODIS NDVI', fontsize=15)

# ax.coastlines()
plt.show()
```

### Plot an image with a different projection

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

projection = ccrs.EqualEarth(central_longitude=-180)

# plot the result with cartoee using a PlateCarre projection (default)
ax = cartoee.get_map(blend, region=bbox, proj=projection)
cb = cartoee.add_colorbar(ax, vis_params=vis_params, loc='right')

ax.set_title(label='MODIS NDVI', fontsize=15)

# ax.coastlines()
plt.show()
```

+++

## Adding a scale bar and legend

### Scale bar

```{code-cell} ipython3
import ee
import geemap
from geemap import cartoee
import matplotlib.pyplot as plt
```

```{code-cell} ipython3
geemap.ee_initialize()
```

```{code-cell} ipython3
# Get image
lon = -115.1585
lat = 36.1500
start_year = 1984
end_year = 2011

point = ee.Geometry.Point(lon, lat)
years = ee.List.sequence(start_year, end_year)


def get_best_image(year):

    start_date = ee.Date.fromYMD(year, 1, 1)
    end_date = ee.Date.fromYMD(year, 12, 31)
    image = (
        ee.ImageCollection("LANDSAT/LT05/C01/T1_SR")
        .filterBounds(point)
        .filterDate(start_date, end_date)
        .sort("CLOUD_COVER")
        .first()
    )
    return ee.Image(image)


collection = ee.ImageCollection(years.map(get_best_image))


vis_params = {"bands": ['B4', 'B3', 'B2'], "min": 0, "max": 5000}

image = ee.Image(collection.first())
```

```{code-cell} ipython3
w = 0.4
h = 0.3

region = [lon + w, lat - h, lon - w, lat + h]

fig = plt.figure(figsize=(10, 8))

# use cartoee to get a map
ax = cartoee.get_map(image, region=region, vis_params=vis_params)

# add gridlines to the map at a specified interval
cartoee.add_gridlines(ax, interval=[0.2, 0.2], linestyle=":")

# add north arrow
north_arrow_dict = {
    "text": "N",
    "xy": (0.10, 0.36),
    "arrow_length": 0.15,
    "text_color": "white",
    "arrow_color": "white",
    "fontsize": 20,
    "width": 5,
    "headwidth": 15,
    "ha": "center",
    "va": "center",
}
cartoee.add_north_arrow(ax, **north_arrow_dict)

# add scale bar
scale_bar_dict = {
    'metric_distance': 4,
    'unit': "km",
    'at_x': (0.05, 0.2),
    'at_y': (0.08, 0.11),
    'max_stripes': 5,
    'ytick_label_margins': 0.25,
    'fontsize': 8,
    'font_weight': "bold",
    'rotation': 0,
    'zorder': 999,
    'paddings': {"xmin": 0.05, "xmax": 0.05, "ymin": 1.5, "ymax": 0.5},
}

cartoee.add_scale_bar(ax, **scale_bar_dict)

ax.set_title(label='Las Vegas, NV', fontsize=15)
plt.show()
```

### Legend

```{code-cell} ipython3
# !pip install cartopy scipy
# !pip install geemap
```

```{code-cell} ipython3
%pylab inline

import ee
import geemap

# import the cartoee functionality from geemap
from geemap import cartoee
```

```{code-cell} ipython3
geemap.ee_initialize()
```

#### Plot an RGB image

```{code-cell} ipython3
# get a landsat image to visualize
image = ee.Image('LANDSAT/LC08/C01/T1_SR/LC08_044034_20140318')

# define the visualization parameters to view
vis = {"bands": ['B5', 'B4', 'B3'], "min": 0, "max": 5000, "gamma": 1.3}
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# use cartoee to get a map
ax = cartoee.get_map(image, vis_params=vis)

# pad the view for some visual appeal
cartoee.pad_view(ax)

# add the gridlines and specify that the xtick labels be rotated 45 degrees
cartoee.add_gridlines(ax, interval=0.5, xtick_rotation=0, linestyle=":")

# add the coastline
ax.coastlines(color="cyan")

show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# here is the bounding box of the map extent we want to use
# formatted a [E,S,W,N]
zoom_region = [-121.8025, 37.3458, -122.6265, 37.9178]

# plot the map over the region of interest
ax = cartoee.get_map(image, vis_params=vis, region=zoom_region)

# add the gridlines and specify that the xtick labels be rotated 45 degrees
cartoee.add_gridlines(ax, interval=0.15, xtick_rotation=0, linestyle=":")

# add coastline
ax.coastlines(color="cyan")

show()
```

#### Adding north arrow, scale bar, and legend

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# here is the bounding box of the map extent we want to use
# formatted a [E,S,W,N]
zoom_region = [-121.8025, 37.3458, -122.6265, 37.9178]

# plot the map over the region of interest
ax = cartoee.get_map(image, vis_params=vis, region=zoom_region)

# add the gridlines and specify that the xtick labels be rotated 45 degrees
cartoee.add_gridlines(ax, interval=0.15, xtick_rotation=0, linestyle=":")

# add coastline
ax.coastlines(color="cyan")

# add north arrow
cartoee.add_north_arrow(
    ax, text="N", xy=(0.05, 0.25), text_color="white", arrow_color="white", fontsize=20
)

# add scale bar
cartoee.add_scale_bar_lite(
    ax, length=10, xy=(0.1, 0.05), fontsize=20, color="white", unit="km"
)

ax.set_title(label='Landsat False Color Composite (Band 5/4/3)', fontsize=15)

# add legend
legend_elements = [
    Line2D([], [], color='#00ffff', lw=2, label='Coastline'),
    Line2D(
        [],
        [],
        marker='o',
        color='#A8321D',
        label='City',
        markerfacecolor='#A8321D',
        markersize=10,
        ls='',
    ),
]

cartoee.add_legend(ax, legend_elements, loc='lower right')

show()
```

+++

## Creating animations

+++

```{code-cell} ipython3
import os
import ee
import geemap
from geemap import cartoee

%pylab inline
```

```{code-cell} ipython3
# geemap.update_package()
```

### Create an interactive map

```{code-cell} ipython3
Map = geemap.Map()
Map
```

### Create an ImageCollection

```{code-cell} ipython3
lon = -115.1585
lat = 36.1500
start_year = 1984
end_year = 2011

point = ee.Geometry.Point(lon, lat)
years = ee.List.sequence(start_year, end_year)


def get_best_image(year):

    start_date = ee.Date.fromYMD(year, 1, 1)
    end_date = ee.Date.fromYMD(year, 12, 31)
    image = (
        ee.ImageCollection("LANDSAT/LT05/C01/T1_SR")
        .filterBounds(point)
        .filterDate(start_date, end_date)
        .sort("CLOUD_COVER")
        .first()
    )
    return ee.Image(image)


collection = ee.ImageCollection(years.map(get_best_image))
```

### Display a sample image

```{code-cell} ipython3
vis_params = {"bands": ['B4', 'B3', 'B2'], "min": 0, "max": 5000}

image = ee.Image(collection.first())
Map.addLayer(image, vis_params, 'First image')
Map.setCenter(lon, lat, 8)
Map
```

### Get a sample output image

```{code-cell} ipython3
w = 0.4
h = 0.3

region = [lon + w, lat - h, lon - w, lat + h]

fig = plt.figure(figsize=(10, 8))

# use cartoee to get a map
ax = cartoee.get_map(image, region=region, vis_params=vis_params)

# add gridlines to the map at a specified interval
cartoee.add_gridlines(ax, interval=[0.2, 0.2], linestyle=":")

# add north arrow
north_arrow_dict = {
    "text": "N",
    "xy": (0.1, 0.3),
    "arrow_length": 0.15,
    "text_color": "white",
    "arrow_color": "white",
    "fontsize": 20,
    "width": 5,
    "headwidth": 15,
    "ha": "center",
    "va": "center",
}
cartoee.add_north_arrow(ax, **north_arrow_dict)

# add scale bar
scale_bar_dict = {
    "length": 10,
    "xy": (0.1, 0.05),
    "linewidth": 3,
    "fontsize": 20,
    "color": "white",
    "unit": "km",
    "ha": "center",
    "va": "bottom",
}
cartoee.add_scale_bar_lite(ax, **scale_bar_dict)

ax.set_title(label='Las Vegas, NV', fontsize=15)

show()
```

### Create timelapse animations

```{code-cell} ipython3
cartoee.get_image_collection_gif(
    ee_ic=collection,
    out_dir=os.path.expanduser("~/Downloads/timelapse"),
    out_gif="animation.gif",
    vis_params=vis_params,
    region=region,
    fps=5,
    mp4=True,
    grid_interval=(0.2, 0.2),
    plot_title="Las Vegas, NV",
    date_format='YYYY-MM-dd',
    fig_size=(10, 8),
    dpi_plot=100,
    file_format="png",
    north_arrow_dict=north_arrow_dict,
    scale_bar_dict=scale_bar_dict,
    verbose=True,
)
```

+++

## Plotting vector data

```{code-cell} ipython3
# !pip install cartopy scipy
# !pip install geemap
```

```{code-cell} ipython3
import ee
import geemap
from geemap import cartoee
from geemap.datasets import DATA
import geemap.colormaps as cmap
import cartopy.crs as ccrs

%pylab inline
```

### Plot a simple vector

```{code-cell} ipython3
Map = geemap.Map()

features = ee.FeatureCollection(DATA.users_giswqs_public_countries)

style = {'color': '000000ff', 'width': 1, 'lineType': 'solid', 'fillColor': '0000ff40'}

Map.addLayer(features.style(**style), {}, "Polygons")
Map.setCenter(-14.77, 34.70, 2)
Map
```

```{code-cell} ipython3
# specify region to focus on
bbox = [180, -88, -180, 88]
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# plot the result with cartoee using a PlateCarre projection (default)
ax = cartoee.get_map(features, region=bbox, style=style)
ax.set_title(label='Countries', fontsize=15)
cartoee.add_gridlines(ax, interval=30)

plt.show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

projection = ccrs.EqualEarth(central_longitude=-180)
ax = cartoee.get_map(features, region=bbox, proj=projection, style=style)
ax.set_title(label='Countries', fontsize=15)

plt.show()
```

+++

### Plot a styled vector

```{code-cell} ipython3
Map = geemap.Map()

features = ee.FeatureCollection(DATA.users_giswqs_public_countries)

palette = cmap.palettes.gist_earth
features_styled = geemap.vector_styling(features, column="name", palette=palette)

Map.add_styled_vector(features, column="name", palette=palette, layer_name='Polygon')
Map.setCenter(-14.77, 34.70, 2)
Map
```

```{code-cell} ipython3
bbox = [180, -88, -180, 88]
fig = plt.figure(figsize=(15, 10))

ax = cartoee.get_map(features_styled, region=bbox)
ax.set_title(label='Countries', fontsize=15)
cartoee.add_gridlines(ax, interval=30)

plt.show()
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

projection = ccrs.EqualEarth(central_longitude=-180)
ax = cartoee.get_map(features_styled, region=bbox, proj=projection)
ax.set_title(label='Countries', fontsize=15)

plt.show()
```

## References

