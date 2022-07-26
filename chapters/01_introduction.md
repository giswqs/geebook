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

# Introducing GEE and Geemap

```{contents}
:local:
:depth: 2
```

## Introduction

## What is Geospatial Data Science

## What is Google Earth Engine

## What is geemap

## Installing geemap

### Installing with conda

```bash
conda create -n gee python
conda activate gee
conda install -c conda-forge geemap
```

```bash
conda install -c conda-forge mamba
mamba install -c conda-forge pygis
```

### Installing with pip

```bash
pip install geemap
```

### Installing from source

```bash
git clone https://github.com/giswqs/geemap
cd geemap
pip install .
```

```bash
pip install git+https://github.com/giswqs/geemap
```

### Upgrading geemap

```bash
pip install -U geemap
```

```bash
conda update -c conda-forge geemap
```

```{code-cell} ipython3
import geemap
geemap.update_package()
```

### Using Docker

## Creating a Jupyter notebook

```bash
conda activate gee
```

```bash
jupyter lab
```

## Earth Engine authentication

```{code-cell} ipython3
import ee
ee.Authenticate()
```

```{code-cell} ipython3
ee.Initialize()
```

## Using Google Colab

Click the **Open in Colab** button below to open this notebook in Google Colab:

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/giswqs/geebook/blob/master/chapters/01_introduction.ipynb)

```bash
!pip install geemap
```

```{code-cell} ipython3
import geemap
Map = geemap.Map()
Map
```

## Using geemap with a VPN

```{code-cell} ipython3
import geemap
geemap.set_proxy(port=your-port-number)
Map = geemap.Map()
Map
```

## Key features of geemap

## Useful resources

## Summary

