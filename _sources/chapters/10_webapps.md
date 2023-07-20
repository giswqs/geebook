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
conda create -n gee python
conda activate gee
conda install -c conda-forge mamba
mamba install -c conda-forge pygis
```

```bash
pip install "geemap[apps]"
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

## Building JavaScript web apps

```javascript
var getNLCD = function (year) {
  var dataset = ee.ImageCollection("USGS/NLCD_RELEASES/2019_REL/NLCD");
  var nlcd = dataset.filter(ee.Filter.eq("system:index", year)).first();
  var landcover = nlcd.select("landcover");
  return ui.Map.Layer(landcover, {}, year);
};
```

```javascript
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
var leftMap = ui.Map();
leftMap.setControlVisibility(false);
var leftSelector = addLayerSelector(leftMap, 0, "top-left");

var rightMap = ui.Map();
rightMap.setControlVisibility(true);
var rightSelector = addLayerSelector(rightMap, 7, "top-right");

function addLayerSelector(mapToChange, defaultValue, position) {
  var label = ui.Label("Select a year:");

  function updateMap(selection) {
    mapToChange.layers().set(0, images[selection]);
  }

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
var title = "NLCD Land Cover Classification";
var position = "bottom-right";
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
var legend = ui.Panel({
  style: {
    position: position,
    padding: "8px 15px",
  },
});

function addCategoricalLegend(panel, dict, title) {
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

  var makeRow = function (color, name) {
    var colorBox = ui.Label({
      style: {
        backgroundColor: color,
        padding: "8px",
        margin: "0 0 4px 0",
      },
    });
    var description = ui.Label({
      value: name,
      style: { margin: "0 0 4px 6px" },
    });

    return ui.Panel({
      widgets: [colorBox, description],
      layout: ui.Panel.Layout.Flow("horizontal"),
    });
  };

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
var splitPanel = ui.SplitPanel({
  firstPanel: leftMap,
  secondPanel: rightMap,
  wipe: true,
  style: { stretch: "both" },
});

ui.root.widgets().reset([splitPanel]);
var linker = ui.Map.Linker([leftMap, rightMap]);
leftMap.setCenter(-100, 40, 4);
```

## Publishing JavaScript web apps

## Building Python Web Apps

```{code-cell} ipython3
import ee
import geemap
```

```{code-cell} ipython3
Map = geemap.Map(center=[40, -100], zoom=4)
```

```{code-cell} ipython3
dataset = ee.ImageCollection('USGS/NLCD_RELEASES/2019_REL/NLCD')
nlcd2019 = dataset.filter(ee.Filter.eq('system:index', '2019')).first()
landcover = nlcd2019.select('landcover')
Map.addLayer(landcover, {}, 'NLCD 2019')
Map
```

```{code-cell} ipython3
title = 'NLCD Land Cover Classification'
Map.add_legend(title=title, builtin_legend='NLCD')
```

```{code-cell} ipython3
dataset.aggregate_array("system:id")
```

```{code-cell} ipython3
years = ['2001', '2004', '2006', '2008', '2011', '2013', '2016', '2019']
```

```{code-cell} ipython3
def getNLCD(year):
    dataset = ee.ImageCollection('USGS/NLCD_RELEASES/2019_REL/NLCD')
    nlcd = dataset.filter(ee.Filter.eq('system:index', year)).first()
    landcover = nlcd.select('landcover')
    return landcover
```

```{code-cell} ipython3
collection = ee.ImageCollection(ee.List(years).map(lambda year: getNLCD(year)))
```

```{code-cell} ipython3
labels = [f'NLCD {year}' for year in years]
labels
```

```{code-cell} ipython3
Map.ts_inspector(
    left_ts=collection,
    right_ts=collection,
    left_names=labels,
    right_names=labels
)
Map
```

## Using Voila to deploy web apps

```bash
cd /path/to/ngrok/dir
ngrok config add-authtoken <token>
```

```bash
./ngrok config add-authtoken <token>
```

```bash
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

## Building Streamlit web apps

```bash
streamlit hello
```

```bash
pip install streamlit
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
def getNLCD(year):
    dataset = ee.ImageCollection("USGS/NLCD_RELEASES/2019_REL/NLCD")
    nlcd = dataset.filter(ee.Filter.eq("system:index", year)).first()
    landcover = nlcd.select("landcover")
    return landcover
```

```{code-cell} ipython3
st.header("National Land Cover Database (NLCD)")
row1_col1, row1_col2 = st.columns([3, 1])
Map = geemap.Map()
years = ["2001", "2004", "2006", "2008", "2011", "2013", "2016", "2019"]
with row1_col2:
    selected_year = st.multiselect("Select a year", years)
    add_legend = st.checkbox("Show legend")
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

```bash
conda activate gee
streamlit run app.py
```

## Building Solara web apps

```bash
pip install solara
```

```{code-cell} ipython3
import ee
import geemap
import solara

class Map(geemap.Map):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.add_ee_data()

    def add_ee_data(self):
        years = ['2001', '2004', '2006', '2008', '2011', '2013', '2016', '2019']
        def getNLCD(year):
            dataset = ee.ImageCollection('USGS/NLCD_RELEASES/2019_REL/NLCD')
            nlcd = dataset.filter(ee.Filter.eq('system:index', year)).first()
            landcover = nlcd.select('landcover')
            return landcover

        collection = ee.ImageCollection(ee.List(years).map(lambda year: getNLCD(year)))
        labels = [f'NLCD {year}' for year in years]
        self.ts_inspector(
            left_ts=collection,
            right_ts=collection,
            left_names=labels,
            right_names=labels,
        )
        self.add_legend(
            title='NLCD Land Cover Type',
            builtin_legend='NLCD',
            height="460px",
            add_header=False
        )

@solara.component
def Page():
    with solara.Column(style={"min-width": "500px"}):
        Map.element(
            center=[40, -100],
            zoom=4,
            height="800px",
        )
```

```bash
conda activate gee
solara run ./pages
```

## Deploying web apps on Hugging Face

```{code-cell} ipython3
import geemap
geemap.get_ee_token()
```

## Summary

