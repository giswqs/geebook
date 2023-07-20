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

# Creating Interactive Maps

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

## Plotting backends

### Ipyleaflet

```{code-cell} ipython3
import geemap
```

```{code-cell} ipython3
Map = geemap.Map()
```

```{code-cell} ipython3
Map
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4, height=600)
Map
```

```{code-cell} ipython3
Map = geemap.Map(data_ctrl=False, toolbar_ctrl=False, draw_ctrl=False)
Map
```

```{code-cell} ipython3
Map = geemap.Map(lite_mode=True)
Map
```

```{code-cell} ipython3
Map.save('ipyleaflet.html')
```

### Folium

```{code-cell} ipython3
import geemap.foliumap as geemap
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4, height=600)
Map
```

```{code-cell} ipython3
Map.save('folium.html')
```

### Plotly

```{code-cell} ipython3
import geemap.plotlymap as geemap
```

```{code-cell} ipython3
Map = geemap.Map()
Map
```

```{code-cell} ipython3
# geemap.fix_widget_error()
```

### Pydeck

```{code-cell} ipython3
import geemap.deck as geemap
```

```{code-cell} ipython3
Map = geemap.Map()
Map
```

### KeplerGL

```{code-cell} ipython3
import geemap.kepler as geemap
```

```{code-cell} ipython3
Map = geemap.Map()
Map
```

## Adding basemaps

### Built-in basemaps

```{code-cell} ipython3
import geemap
```

```{code-cell} ipython3
Map = geemap.Map(basemap='HYBRID')
Map
```

```{code-cell} ipython3
Map.add_basemap('OpenTopoMap')
```

```{code-cell} ipython3
for basemap in geemap.basemaps.keys():
    print(basemap)
```

```{code-cell} ipython3
len(geemap.basemaps)
```

### XYZ tiles

```{code-cell} ipython3
Map = geemap.Map()
Map.add_tile_layer(
    url="https://mt1.google.com/vt/lyrs=p&x={x}&y={y}&z={z}",
    name="Google Terrain",
    attribution="Google",
)
Map
```

### WMS tiles

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
url = 'https://www.mrlc.gov/geoserver/mrlc_display/NLCD_2019_Land_Cover_L48/wms?'
Map.add_wms_layer(
    url=url,
    layers='NLCD_2019_Land_Cover_L48',
    name='NLCD 2019',
    format='image/png',
    attribution='MRLC',
    transparent=True,
)
Map
```

### Planet basemaps

```{code-cell} ipython3
import os

os.environ["PLANET_API_KEY"] = "YOUR_API_KEY"
```

```{code-cell} ipython3
quarterly_tiles = geemap.planet_quarterly_tiles()
for tile in quarterly_tiles:
    print(tile)
```

```{code-cell} ipython3
monthly_tiles = geemap.planet_monthly_tiles()
for tile in monthly_tiles:
    print(tile)
```

```{code-cell} ipython3
Map = geemap.Map()
Map.add_planet_by_month(year=2020, month=8)
Map
```

```{code-cell} ipython3
Map = geemap.Map()
Map.add_planet_by_quarter(year=2019, quarter=2)
Map
```

### Basemap GUI

```{code-cell} ipython3
import os

os.environ["PLANET_API_KEY"] = "YOUR_API_KEY"
```

```{code-cell} ipython3
Map = geemap.Map()
Map
```

## Summary

