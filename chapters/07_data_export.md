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

# Exporting Earth Engine Data

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

## Exporting images

```{code-cell} ipython3
Map = geemap.Map()

image = ee.Image('LANDSAT/LC08/C02/T1_TOA/LC08_044034_20140318').select(
    ['B5', 'B4', 'B3']
)

vis_params = {'min': 0, 'max': 0.5, 'gamma': [0.95, 1.1, 1]}

Map.centerObject(image, 8)
Map.addLayer(image, vis_params, 'Landsat')
Map
```

```{code-cell} ipython3
region = ee.Geometry.BBox(-122.5955, 37.5339, -122.0982, 37.8252)
fc = ee.FeatureCollection(region)
style = {'color': 'ffff00ff', 'fillColor': '00000000'}
Map.addLayer(fc.style(**style), {}, 'ROI')
Map
```

### To local drive

```{code-cell} ipython3
geemap.ee_export_image(image, filename="landsat.tif", scale=30, region=region)
```

```{code-cell} ipython3
projection = image.select(0).projection().getInfo()
projection
```

```{code-cell} ipython3
crs = projection['crs']
crs_transform = projection['transform']
```

```{code-cell} ipython3
geemap.ee_export_image(
    image,
    filename="landsat_crs.tif",
    crs=crs,
    crs_transform=crs_transform,
    region=region,
)
```

```{code-cell} ipython3
geemap.download_ee_image(image, filename='landsat_full.tif', scale=60)
```

```{code-cell} ipython3
fishnet = geemap.fishnet(image.geometry(), rows=4, cols=4, delta=0.5)
style = {'color': 'ffff00ff', 'fillColor': '00000000'}
Map.addLayer(fishnet.style(**style), {}, 'Fishnet')
Map
```

```{code-cell} ipython3
out_dir = 'Downloads'
geemap.download_ee_image_tiles(
    image, fishnet, out_dir, prefix="landsat_", crs="EPSG:3857", scale=30
)
```

### To Google Drive

```{code-cell} ipython3
geemap.ee_export_image_to_drive(
    image, description='landsat', folder='export', region=region, scale=30
)
```

### To Asset

```{code-cell} ipython3
assetId = 'landsat_sfo'
geemap.ee_export_image_to_asset(
    image, description='landsat', assetId=assetId, region=region, scale=30
)
```

### To Cloud Storage

```{code-cell} ipython3
bucket = 'your-bucket'
geemap.ee_export_image_to_cloud_storage(
    image, description='landsat', bucket=None, region=region, scale=30
)
```

### To NumPy array

```{code-cell} ipython3
region = ee.Geometry.BBox(-122.5003, 37.7233, -122.3410, 37.8026)
rgb_img = geemap.ee_to_numpy(image, region=region)
```

```{code-cell} ipython3
print(rgb_img.shape)
```

```{code-cell} ipython3
import matplotlib.pyplot as plt

rgb_img_test = (255 * ((rgb_img[:, :, 0:3]) + 0.2)).astype('uint8')
plt.imshow(rgb_img_test)
plt.show()
```

## Exporting image collections

```{code-cell} ipython3
point = ee.Geometry.Point(-99.2222, 46.7816)
collection = (
    ee.ImageCollection('USDA/NAIP/DOQQ')
    .filterBounds(point)
    .filterDate('2008-01-01', '2018-01-01')
    .filter(ee.Filter.listContains("system:band_names", "N"))
)
```

```{code-cell} ipython3
collection.aggregate_array('system:index').getInfo()
```

### To local drive

```{code-cell} ipython3
out_dir = 'Downloads'
geemap.ee_export_image_collection(collection, out_dir=out_dir, scale=10)
```

### To Google Drive

```{code-cell} ipython3
geemap.ee_export_image_collection_to_drive(collection, folder='export', scale=10)
```

### To Assets

```{code-cell} ipython3
geemap.ee_export_image_collection_to_asset(collection, scale=10)
```

## Exporting videos

```{code-cell} ipython3
collection = (
    ee.ImageCollection('LANDSAT/LT05/C01/T1_TOA')
    .filter(ee.Filter.eq('WRS_PATH', 44))
    .filter(ee.Filter.eq('WRS_ROW', 34))
    .filter(ee.Filter.lt('CLOUD_COVER', 30))
    .filterDate('1991-01-01', '2011-12-30')
    .select(['B4', 'B3', 'B2'])
    .map(lambda img: img.multiply(512).uint8())
)
region = ee.Geometry.Rectangle([-122.7286, 37.6325, -122.0241, 37.9592])
```

```{code-cell} ipython3
geemap.ee_export_video_to_drive(
    collection, folder='export', framesPerSecond=12, dimensions=720, region=region
)
```

## Exporting image thumbnails

```{code-cell} ipython3
roi = ee.Geometry.Point([-122.44, 37.75])
collection = (
    ee.ImageCollection('LANDSAT/LC08/C02/T1_TOA')
    .filterBounds(roi)
    .sort("CLOUD_COVER")
    .limit(10)
)

image = collection.first()
```

```{code-cell} ipython3
Map = geemap.Map()

vis_params = {
    'bands': ['B5', 'B4', 'B3'],
    'min': 0,
    'max': 0.3,
    'gamma': [0.95, 1.1, 1],
}

Map.addLayer(image, vis_params, "LANDSAT 8")
Map.setCenter(-122.44, 37.75, 8)
Map
```

```{code-cell} ipython3
out_img = 'landsat.jpg'
region = ee.Geometry.BBox(-122.5955, 37.5339, -122.0982, 37.8252)
geemap.get_image_thumbnail(image, out_img, vis_params, dimensions=1000, region=region)
```

```{code-cell} ipython3
geemap.show_image(out_img)
```

```{code-cell} ipython3
out_dir = 'Downloads'
geemap.get_image_collection_thumbnails(
    collection,
    out_dir,
    vis_params,
    dimensions=1000,
    region=region,
)
```

## Exporting feature collections

```{code-cell} ipython3
Map = geemap.Map()
fc = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017').filter(
    ee.Filter.eq('wld_rgn', 'Europe')
)

Map.addLayer(fc, {}, "Europe")
Map.centerObject(fc, 3)
Map
```

+++

### To local drive

```{code-cell} ipython3
geemap.ee_to_shp(fc, filename='europe.shp', selectors=None)
```

```{code-cell} ipython3
geemap.ee_export_vector(fc, filename='europe2.shp')
```

```{code-cell} ipython3
geemap.ee_to_geojson(fc, filename='europe.geojson')
```

```{code-cell} ipython3
geemap.ee_to_csv(fc, filename='europe.csv')
```

```{code-cell} ipython3
gdf = geemap.ee_to_gdf(fc)
gdf
```

```{code-cell} ipython3
df = geemap.ee_to_df(fc)
df
```

### To Google Drive

```{code-cell} ipython3
geemap.ee_export_vector_to_drive(
    fc, description="europe", fileFormat='SHP', folder="export"
)
```

### To Asset

```{code-cell} ipython3
geemap.ee_export_vector_to_asset(fc, description='Exporting Europe', assetId='europe')
```

## Exporting maps

```{code-cell} ipython3
Map = geemap.Map()
image = ee.Image('USGS/SRTMGL1_003')
vis_params = {
    'min': 0,
    'max': 4000,
    'palette': ['006633', 'E5FFCC', '662A00', 'D8D8D8', 'F5F5F5'],
}
Map.addLayer(image, vis_params, 'SRTM DEM', True)
Map
```

```{code-cell} ipython3
Map.to_html(
    filename="mymap.html", title="Earth Engine Map", width='100%', height='800px'
)
```

## Using the high-volume endpoint

```{code-cell} ipython3
import ee
import geemap
import logging
import multiprocessing
import os
import requests
import shutil
from retry import retry
```

```{code-cell} ipython3
ee.Initialize(opt_url='https://earthengine-highvolume.googleapis.com')
```

```{code-cell} ipython3
region = Map.user_roi

if region is None:
    region = ee.Geometry.Polygon(
        [
            [
                [-122.513695, 37.707998],
                [-122.513695, 37.804359],
                [-122.371902, 37.804359],
                [-122.371902, 37.707998],
                [-122.513695, 37.707998],
            ]
        ],
        None,
        False,
    )
```

```{code-cell} ipython3
image = (
    ee.ImageCollection('USDA/NAIP/DOQQ')
    .filterBounds(region)
    .filterDate('2020', '2021')
    .mosaic()
    .clip(region)
    .select('N', 'R', 'G')
)
```

```{code-cell} ipython3
Map = geemap.Map()
Map.addLayer(image, {}, "Image")
Map.addLayer(region, {}, "ROI", False)
Map.centerObject(region, 12)
Map
```

```{code-cell} ipython3
out_dir = 'Downloads'
params = {
    'count': 1000,  # How many image chips to export
    'buffer': 127,  # The buffer distance (m) around each point
    'scale': 100,  # The scale to do stratified sampling
    'seed': 1,  # A randomization seed to use for subsampling.
    'dimensions': '256x256',  # The dimension of each image chip
    'format': "png",  # The output image format, can be png, jpg, ZIPPED_GEO_TIFF, GEO_TIFF, NPY
    'prefix': 'tile_',  # The filename prefix
    'processes': 25,  # How many processes to used for parallel processing
    'out_dir': out_dir,  # The output directory. Default to the current working directly
}
```

```{code-cell} ipython3
def getRequests():
    img = ee.Image(1).rename("Class").addBands(image)
    points = img.stratifiedSample(
        numPoints=params['count'],
        region=region,
        scale=params['scale'],
        seed=params['seed'],
        geometries=True,
    )
    Map.data = points
    return points.aggregate_array('.geo').getInfo()
```

```{code-cell} ipython3
@retry(tries=10, delay=1, backoff=2)
def getResult(index, point):
    point = ee.Geometry.Point(point['coordinates'])
    region = point.buffer(params['buffer']).bounds()

    if params['format'] in ['png', 'jpg']:
        url = image.getThumbURL(
            {
                'region': region,
                'dimensions': params['dimensions'],
                'format': params['format'],
            }
        )
    else:
        url = image.getDownloadURL(
            {
                'region': region,
                'dimensions': params['dimensions'],
                'format': params['format'],
            }
        )

    if params['format'] == "GEO_TIFF":
        ext = 'tif'
    else:
        ext = params['format']

    r = requests.get(url, stream=True)
    if r.status_code != 200:
        r.raise_for_status()

    out_dir = os.path.abspath(params['out_dir'])
    basename = str(index).zfill(len(str(params['count'])))
    filename = f"{out_dir}/{params['prefix']}{basename}.{ext}"
    with open(filename, 'wb') as out_file:
        shutil.copyfileobj(r.raw, out_file)
    print("Done: ", basename)
```

```{code-cell} ipython3
%%time
logging.basicConfig()
items = getRequests()

pool = multiprocessing.Pool(params['processes'])
pool.starmap(getResult, enumerate(items))

pool.close()
```

```{code-cell} ipython3
Map.addLayer(Map.data, {}, "Sample points")
Map
```

```{code-cell} ipython3
geemap.ee_to_shp(Map.data, filename='points.shp')
```

## Summary

