---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3 (ipykernel)
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
mamba install -c conda-forge geemap pygis
mamba install -c conda-forge cartopy
```

```bash
jupyter lab
```

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/giswqs/geebook/blob/master/chapters/08_cartoee.ipynb)

```{code-cell} ipython3
# pip install pygis
```

```{code-cell} ipython3
# pip install cartopy
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
# import the cartoee functionality from geemap
from geemap import cartoee
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
```

```{code-cell} ipython3
for key in cartoee.create_legend():
    print(key)
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

plt.show()
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

plt.show()
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

plt.show()
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

plt.show()
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

plt.show()
```

+++ {"tags": []}

## Using custom projections

+++

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

## Using multiple data layers

+++

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

countries = ee.FeatureCollection(geemap.examples.get_ee_path('countries'))
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

## Adding a scale bar and legend

### Scale bar

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

+++

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

plt.show()
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

plt.show()
```

#### Adding north arrow, scale bar, and legend

```{code-cell} ipython3
from matplotlib.lines import Line2D
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

plt.show()
```

## Creating animations

+++ {"tags": []}

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

plt.show()
```

### Create timelapse animations

```{code-cell} ipython3
import os
```

```{code-cell} ipython3
cartoee.get_image_collection_gif(
    ee_ic=collection,
    out_dir=os.getcwd(),
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
    file_format="jpg",
    north_arrow_dict=north_arrow_dict,
    scale_bar_dict=scale_bar_dict,
    verbose=True,
)
```

```{code-cell} ipython3
geemap.show_image('animation.gif')
```

## Plotting vector data

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

features = ee.FeatureCollection(geemap.examples.get_ee_path('countries'))

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

![](https://i.imgur.com/RTFGotE.jpg)

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

projection = ccrs.EqualEarth(central_longitude=-180)
ax = cartoee.get_map(features, region=bbox, proj=projection, style=style)
ax.set_title(label='Countries', fontsize=15)

plt.show()
```

![](https://i.imgur.com/GagRINK.jpg)

+++

### Plot a styled vector

```{code-cell} ipython3
import geemap.colormaps as cm
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
```

```{code-cell} ipython3
fc = ee.FeatureCollection("WRI/GPPD/power_plants").filter(
    ee.Filter.inList('fuel1', fuels)
)
```

```{code-cell} ipython3
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
styled_fc = geemap.ee_vector_style(fc, column="fuel1", labels=fuels, color=colors, pointSize=1)
```

```{code-cell} ipython3
Map = geemap.Map()
Map.addLayer(styled_fc, {}, 'Power Plants')
Map.add_legend(title="Power Plant Fuel Type", labels=fuels, colors=colors)
Map
```

```{code-cell} ipython3
from matplotlib.lines import Line2D
```

```{code-cell} ipython3
legend = []
```

```{code-cell} ipython3
for index, fuel in enumerate(fuels):
    item =            Line2D(
                    [],
                    [],
                    marker="o",
                    color='#' + colors[index],
                    label=fuel,
                    markerfacecolor='#' + colors[index],
                    markersize=10,
                    ls="",
                )
    legend.append(item)
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# plot the result with cartoee using a PlateCarre projection (default)
ax = cartoee.get_map(styled_fc, region=bbox, basemap='ROADMAP')
ax.set_title(label='Countries', fontsize=15)
cartoee.add_gridlines(ax, interval=30)
cartoee.add_legend(ax, legend_elements=legend)

plt.show()
```

```{code-cell} ipython3
Map = geemap.Map()

palette = cm.palettes.gist_earth
features = ee.FeatureCollection(geemap.examples.get_ee_path('countries'))
features_styled = geemap.vector_styling(features, column="NAME", palette=palette)

Map.add_styled_vector(features, column="NAME", palette=palette, layer_name='Polygon')
Map.setCenter(-14.77, 34.70, 2)
Map
```

```{code-cell} ipython3
Map = geemap.Map()

palette = cm.palettes.gist_earth
features = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017')
features_styled = geemap.vector_styling(features, column="abbreviati", palette=palette)

Map.add_styled_vector(features, column="abbreviati", palette=palette, layer_name='Polygon')
Map.setCenter(-14.77, 34.70, 2)
Map
```

```{code-cell} ipython3
car
```

```{code-cell} ipython3
features_styled.first().propertyNames().getInfo()
```

```{code-cell} ipython3
image = features_styled.style(**{"styleProperty": "style"})
```

```{code-cell} ipython3
proj = ee.Projection("EPSG:3857")
```

```{code-cell} ipython3
image = image.setDefaultProjection(proj)
```

```{code-cell} ipython3
Map.addLayer(image)
```

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

# plot the result with cartoee using a PlateCarre projection (default)
ax = cartoee.get_map(image, region=bbox, style=style)
ax.set_title(label='Countries', fontsize=15)
cartoee.add_gridlines(ax, interval=30)

plt.show()
```

```{code-cell} ipython3
image.projection().getInfo()
```

```{code-cell} ipython3
---
jupyter:
  source_hidden: true
tags: []
---
bbox = [179, -88, -179, 88]
fig = plt.figure(figsize=(15, 10))

ax = cartoee.get_map(image, region=bbox)
ax.set_title(label='Countries', fontsize=15)
cartoee.add_gridlines(ax, interval=30)

plt.show()
```

![](https://i.imgur.com/reecFZo.jpg)

```{code-cell} ipython3
fig = plt.figure(figsize=(15, 10))

projection = ccrs.EqualEarth(central_longitude=-180)
ax = cartoee.get_map(features_styled, region=bbox, proj=projection)
ax.set_title(label='Countries', fontsize=15)

plt.show()
```

![](https://i.imgur.com/uW9p8vS.jpg)

## References

- https://geemap.org/notebooks/50_cartoee_quickstart/
- https://geemap.org/notebooks/51_cartoee_projections/
- https://geemap.org/notebooks/52_cartoee_gif/
- https://geemap.org/notebooks/57_cartoee_blend/
- https://geemap.org/notebooks/61_cartoee_scalebar/
- https://geemap.org/notebooks/66_cartoee_legend/
- https://geemap.org/notebooks/69_cartoee_vector
- https://geemap.org/notebooks/112_cartoee_basemap/
