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

# Building Interactive Web Apps

```{contents}
:local:
:depth: 2
```

## Introduction

## Technical requirements

```bash
conda install -n base mamba -c conda-forge
mamba create -n gee -c conda-forge geemap pygis
```

```bash
conda activate gee
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

## Building an Earth Engine App using JavaScript

```javascript
// Get an NLCD image by year.
var getNLCD = function (year) {
  // Import the NLCD collection.
  var dataset = ee.ImageCollection("USGS/NLCD_RELEASES/2019_REL/NLCD");

  // Filter the collection by year.
  var nlcd = dataset.filter(ee.Filter.eq("system:index", year)).first();

  // Select the land cover band.
  var landcover = nlcd.select("landcover");
  return ui.Map.Layer(landcover, {}, year);
};
```

```javascript
// Create a dictionary with each year as the key
// and its corresponding NLCD image layer as the value.
var images = {
  2001: getNLCD("2001"),
  2004: getNLCD("2004"),
  2006: getNLCD("2006"),
  2008: getNLCD("2008"),
  2011: getNLCD("2011"),
  2013: getNLCD("2013"),
  2016: getNLCD("2016"),
  2019: getNLCD("2019"),
};
```

```javascript
// Create the left map, and have it display the first layer.
var leftMap = ui.Map();
leftMap.setControlVisibility(false);
var leftSelector = addLayerSelector(leftMap, 0, "top-left");

// Create the right map, and have it display the last layer.
var rightMap = ui.Map();
rightMap.setControlVisibility(true);
var rightSelector = addLayerSelector(rightMap, 7, "top-right");

// Adds a layer selection widget to the given map, to allow users to
// change which image is displayed in the associated map.
function addLayerSelector(mapToChange, defaultValue, position) {
  var label = ui.Label("Select a year:");

  // This function changes the given map to show the selected image.
  function updateMap(selection) {
    // mapToChange.layers().set(0, ui.Map.Layer(images[selection]));
    mapToChange.layers().set(0, images[selection]);
  }

  // Configure a selection dropdown to allow the user to choose
  // between images, and set the map to update when a user
  // makes a selection.
  var select = ui.Select({ items: Object.keys(images), onChange: updateMap });
  select.setValue(Object.keys(images)[defaultValue], true);

  var controlPanel = ui.Panel({
    widgets: [label, select],
    style: { position: position },
  });

  mapToChange.add(controlPanel);
}
```

```javascript
// Set the legend title.
var title = "NLCD Land Cover Classification";

// Set the legend position.
var position = "bottom-right";

// Define a dictionary that will be used to make a legend
var dict = {
  names: [
    "11 Open Water",
    "12 Perennial Ice/Snow",
    "21 Developed, Open Space",
    "22 Developed, Low Intensity",
    "23 Developed, Medium Intensity",
    "24 Developed, High Intensity",
    "31 Barren Land (Rock/Sand/Clay)",
    "41 Deciduous Forest",
    "42 Evergreen Forest",
    "43 Mixed Forest",
    "51 Dwarf Scrub",
    "52 Shrub/Scrub",
    "71 Grassland/Herbaceous",
    "72 Sedge/Herbaceous",
    "73 Lichens",
    "74 Moss",
    "81 Pasture/Hay",
    "82 Cultivated Crops",
    "90 Woody Wetlands",
    "95 Emergent Herbaceous Wetlands",
  ],

  colors: [
    "#466b9f",
    "#d1def8",
    "#dec5c5",
    "#d99282",
    "#eb0000",
    "#ab0000",
    "#b3ac9f",
    "#68ab5f",
    "#1c5f2c",
    "#b5c58f",
    "#af963c",
    "#ccb879",
    "#dfdfc2",
    "#d1d182",
    "#a3cc51",
    "#82ba9e",
    "#dcd939",
    "#ab6c28",
    "#b8d9eb",
    "#6c9fb8",
  ],
};
```

```javascript
// Create a panel to hold the legend widget.
var legend = ui.Panel({
  style: {
    position: position,
    padding: "8px 15px",
  },
});

// Function to generate the legend.
function addCategoricalLegend(panel, dict, title) {
  // Create and add the legend title.
  var legendTitle = ui.Label({
    value: title,
    style: {
      fontWeight: "bold",
      fontSize: "18px",
      margin: "0 0 4px 0",
      padding: "0",
    },
  });
  panel.add(legendTitle);

  var loading = ui.Label("Loading legend...", { margin: "2px 0 4px 0" });
  panel.add(loading);

  // Creates and styles 1 row of the legend.
  var makeRow = function (color, name) {
    // Create the label that is actually the colored box.
    var colorBox = ui.Label({
      style: {
        backgroundColor: color,
        // Use padding to give the box height and width.
        padding: "8px",
        margin: "0 0 4px 0",
      },
    });

    // Create the label filled with the description text.
    var description = ui.Label({
      value: name,
      style: { margin: "0 0 4px 6px" },
    });

    return ui.Panel({
      widgets: [colorBox, description],
      layout: ui.Panel.Layout.Flow("horizontal"),
    });
  };

  // Get the list of palette colors and class names from the image.
  var palette = dict.colors;
  var names = dict.names;
  loading.style().set("shown", false);

  for (var i = 0; i < names.length; i++) {
    panel.add(makeRow(palette[i], names[i]));
  }

  rightMap.add(panel);
}

addCategoricalLegend(legend, dict, title);
```

```javascript
// Create a SplitPanel to hold the adjacent, linked maps.
var splitPanel = ui.SplitPanel({
  firstPanel: leftMap,
  secondPanel: rightMap,
  wipe: true,
  style: { stretch: "both" },
});

// Set the SplitPanel as the only thing in the UI root.
ui.root.widgets().reset([splitPanel]);
var linker = ui.Map.Linker([leftMap, rightMap]);
leftMap.setCenter(-100, 40, 4);
```

## Publishing an Earth Engine App from the Code Editor

## Developing an Earth Engine App using geemap

```bash
conda create -n gee python
conda activate gee
conda install mamba
mamba install -c conda-forge geospatial
```

```bash
conda activate gee
jupyter notebook
```

```{code-cell} ipython3
import ee
import geemap
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
Map
```

```{code-cell} ipython3
# Import the NLCD collection
dataset = ee.ImageCollection('USGS/NLCD_RELEASES/2019_REL/NLCD')

# Filter the collection to the 2019 product
nlcd2019 = dataset.filter(ee.Filter.eq('system:index', '2019')).first()

# Select the land cover band
landcover = nlcd2019.select('landcover')

# Display land cover on the map
Map.addLayer(landcover, {}, 'NLCD 2019')
Map
```

```{code-cell} ipython3
title = 'NLCD Land Cover Classification'
Map.add_legend(title=title, builtin_legend='NLCD')
```

```{code-cell} ipython3
legend_dict = {
    '11 Open Water': '466b9f',
    '12 Perennial Ice/Snow': 'd1def8',
    '21 Developed, Open Space': 'dec5c5',
    '22 Developed, Low Intensity': 'd99282',
    '23 Developed, Medium Intensity': 'eb0000',
    '24 Developed High Intensity': 'ab0000',
    '31 Barren Land (Rock/Sand/Clay)': 'b3ac9f',
    '41 Deciduous Forest': '68ab5f',
    '42 Evergreen Forest': '1c5f2c',
    '43 Mixed Forest': 'b5c58f',
    '51 Dwarf Scrub': 'af963c',
    '52 Shrub/Scrub': 'ccb879',
    '71 Grassland/Herbaceous': 'dfdfc2',
    '72 Sedge/Herbaceous': 'd1d182',
    '73 Lichens': 'a3cc51',
    '74 Moss': '82ba9e',
    '81 Pasture/Hay': 'dcd939',
    '82 Cultivated Crops': 'ab6c28',
    '90 Woody Wetlands': 'b8d9eb',
    '95 Emergent Herbaceous Wetlands': '6c9fb8',
}
title = 'NLCD Land Cover Classification'
Map.add_legend(title=title, legend_dict=legend_dict)
```

```{code-cell} ipython3
dataset.aggregate_array("system:id").getInfo()
```

```{code-cell} ipython3
years = ['2001', '2004', '2006', '2008', '2011', '2013', '2016', '2019']
```

```{code-cell} ipython3
# Get an NLCD image by year
def getNLCD(year): # Import the NLCD collection.
dataset = ee.ImageCollection('USGS/NLCD_RELEASES/2019_REL/NLCD')

    # Filter the collection by year.
    nlcd = dataset.filter(ee.Filter.eq('system:index', year)).first()

    # Select the land cover band.
    landcover = nlcd.select('landcover')
    return landcover
```

```{code-cell} ipython3
# Create an NLCD image collection for the selected years
collection = ee.ImageCollection(ee.List(years).map(lambda year: getNLCD(year)))
```

```{code-cell} ipython3
collection.aggregate_array('system:id').getInfo()
```

```{code-cell} ipython3
labels = [f'NLCD {year}' for year in years]
labels
```

```{code-cell} ipython3
Map.ts_inspector(
    left_ts=collection, right_ts=collection, left_names=labels, right_names=labels
)
Map
```

## Publishing an Earth Engine App using a local web server

```bash
cd /path/to/ngrok/dir
conda activate gee
voila --no-browser nlcd_app.ipynb
```

```bash
cd /path/to/ngrok/dir
ngrok http 8866
```

```bash
voila --no-browser --strip_sources=False nlcd_app.ipynb
```

```bash
ngrok http -auth="username:password" 8866
```

## Publishing an Earth Engine App using cloud platforms

```bash
web: voila --port=$PORT --no-browser --strip_sources=True --enable_nbextensions=True --MappingKernelManager.cull_interval=60 --MappingKernelManager.cull_idle_timeout=120 notebooks/
```

## Building an Earth Engine App Using Streamlit

```bash
conda activate gee
mamba install -c conda-forge streamlit
```

```bash
streamlit hello
```

```bash
git config --global user.name "Firstname Lastname"
git config --global user.email user@example.com
```

```bash
git clone https://github.com/USERNAME/geemap-apps.git
```

```{code-cell} ipython3
import ee
import streamlit as st
import geemap.foliumap as geemap
```

```{code-cell} ipython3
# Get an NLCD image by year.
def getNLCD(year):
    # Import the NLCD collection.
    dataset = ee.ImageCollection("USGS/NLCD_RELEASES/2019_REL/NLCD")

    # Filter the collection by year.
    nlcd = dataset.filter(ee.Filter.eq("system:index", year)).first()

    # Select the land cover band.
    landcover = nlcd.select("landcover")
    return landcover
```

```{code-cell} ipython3
# The main app.
def app():

    st.header("National Land Cover Database (NLCD)")

    # Create a layout containing two columns
    row1_col1, row1_col2 = st.columns([3, 1])

    # Create an interactive map
    Map = geemap.Map()

    # Select the eight NLCD epochs after 2000.
    years = ["2001", "2004", "2006", "2008", "2011", "2013", "2016", "2019"]

    # Add a dropdown list and checkbox to the second column.
    with row1_col2:
        selected_year = st.multiselect("Select a year", years)
        add_legend = st.checkbox("Show legend")

    # Add selected NLCD image to the map based on the selected year.
    if selected_year:
        for year in selected_year:
            Map.addLayer(getNLCD(year), {}, "NLCD " + year)

        if add_legend:
            Map.add_legend(title="NLCD Land Cover", builtin_legend="NLCD")
        with row1_col1:
            Map.to_streamlit(height=600)

    else:
        with row1_col1:
            Map.to_streamlit(height=600)
```

```{code-cell} ipython3
from apps import home, basemaps, customize, datasets, opacity, nlcd
```

```{code-cell} ipython3
apps.add_app("NLCD", nlcd.app)
```

```bash
conda activate gee
streamlit run app.py
```

## Conclusions

