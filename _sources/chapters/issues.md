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

# Known Issues

Here are some known issues with Google Colab:

## Chapter 1: Introducing GEE and Geemap

## Chapter 2: Creating Interactive Maps

- [2.3.3. The plotly backend](https://book.geemap.org/chapters/02_maps.html#plotly)
- [2.3.4. The pydeck backend](https://book.geemap.org/chapters/02_maps.html#pydeck)
- [2.3.6. The heremap backend needs an API key](https://book.geemap.org/chapters/02_maps.html#heremap)
- [2.4.4. Planet basemaps need an API key](https://book.geemap.org/chapters/02_maps.html#planet-basemaps)

## Chapter 3: Using Earth Engine Data

- [3.9. Calling JavaScript functions from Python](https://book.geemap.org/chapters/03_gee_data.html#calling-javascript-functions-from-python): Calling functions from the default OEEL library works, but calling functions from custom libraries does not.

## Chapter 4: Using Local Geospatial Data

- [4.3. Local raster datasets](https://book.geemap.org/chapters/04_local_data.html#local-raster-datasets): localtileserver does not support Colab.
- [4.10. Converting NetCDF to ee.Image](https://book.geemap.org/chapters/04_local_data.html#converting-netcdf-to-ee-image): A temporary workaround is to install an older version of pyymal: `pip install pyyaml==5.4.1`
- [4.12. Reading PostGIS data](https://book.geemap.org/chapters/04_local_data.html#reading-postgis-data): PostGIS database is not supported in Colab.

## Chapter 5: Visualizing Geospatial Data

- [5.14.2. Visualizing planet imagery](https://book.geemap.org/chapters/05_data_viz.html#visualizing-planet-imagery): Needs a Planet API key.
- [5.18. Visualing NetCDF data](https://book.geemap.org/chapters/05_data_viz.html#visualing-netcdf-data): A temporary workaround is to install an older version of pyymal: `pip install pyyaml==5.4.1`
- [5.19. Visualizing LiDAR data](https://book.geemap.org/chapters/05_data_viz.html#visualizing-lidar-data): The `view_lidar()` function is not supported in Colab because pyvista does not support Colab.
- [5.20. Visualizing raster data in 3D](https://book.geemap.org/chapters/05_data_viz.html#visualizing-raster-data-in-3d): The `plot_raster_3d()` function is not supported in Colab because pyvista-xarray does not support Colab.

## Chapter 6: Analyzing Geospatial Data

## Chapter 7: Exporting Earth Engine Data

## Chapter 8: Making Maps with Cartoee

## Chapter 9: Creating Timelapses

## Chapter 10: Building Web Apps

## Chapter 11: Earth Engine Applications
