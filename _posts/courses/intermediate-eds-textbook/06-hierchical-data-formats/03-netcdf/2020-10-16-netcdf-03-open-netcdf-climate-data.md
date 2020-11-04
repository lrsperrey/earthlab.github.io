---
layout: single
title: "How to Open and Process NetCDF 4 Data Format in Open Source Python"
excerpt: "Historic and projected climate data are most often stored in netcdf 4 format. Learn how to open and process MACA version 2 climate data for the Continental United States using the open source python package, xarray."
authors: ['Leah Wasser']
dateCreated: 2020-10-23
modified: 2020-11-04
category: [courses]
class-lesson: ['netcdf4']
permalink: /courses/use-data-open-source-python/hierarchical-data-formats-hdf/use-netcdf-in-python-xarray/
nav-title: 'Netcdf in Python'
week: 6
course: "intermediate-earth-data-science-textbook"
sidebar:
  nav:
author_profile: false
comments: true
order: 3
topics:
  remote-sensing: ['modis']
  reproducible-science-and-programming:
  spatial-data-and-gis: ['raster-data']
---

{% include toc title="In This Chapter" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-ship" aria-hidden="true"></i> Open NETCDF 4 Climate Data in Open Source Python Using Xarray

In this chapter, you will learn how to work with Climate Data Sets (MACA v2 for the United states) stored in netcdf 4 format using open source **Python**.


## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this chapter, you will be able to:

* Download MACA v2 climate data in `netcdf 4` format
* Open and process netcdf4 data using `xarray`
* Export climate data in tabular format to `.csv` format

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What You Need

OPTIONAL: If you want to explore the netcdf 4 files in a graphics based tool, you can download th <a href="https://www.hdfgroup.org/downloads/hdfview/" target="_blank">free HDF viewer</a> from the HDF Group website. 

</div>


{:.input}
```python
import os

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
# netcdf4 needs to be installed in your environment for this to work
import xarray as xr
import rioxarray
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import seaborn as sns
import geopandas as gpd
import earthpy as et

# Plotting options
sns.set(font_scale=1.3)
sns.set_style("white")

# Optional - set your working directory if you wish to use the data
# accessed lower down in this notebook (the USA state boundary data)
os.chdir(os.path.join(et.io.HOME,
                      'earth-analytics',
                      'data'))
```



## Get Started wtih MACA Version 2 Data Using Open Source Python

In this lesson you will work with historic projected MACA 2 data that represents 
maximum monthly temperature for the Continental United States (CONUS). 
 
`agg_macav2metdata_tasmax_BNU-ESM_r1i1p1_historical_1950_2005_CONUS_monthly`

The file name itself tells you a lot about the data. 

1. **macav2metdata**: the data are the MACA version two data which are downsampled to the extent of the continental United S tates
2. **tasmax**: Max temperature is the parameter contained within the data
3. **BNU-ESM**: This is the climate (CLM) model used to generate the data
4. **historical**: these data are the modeled historical values for the years **1950 - 2005**
5. **CONUS**: These data are for the **CON**tinental **U**nited **S**tates boundary
6. **monthly**: These data are aggregated monthly (rather than daily)

Below, you will learn how to open and work with MACA 2 data using open source Python tools. 
You will use the **xarray** package which requires the `netcdf4` package to work with netcdf data. 
The <a href="https://github.com/earthlab/earth-analytics-python-env" target="_blank">most current earth-analytics-python environment </a> contains all of the 
packages that you need to complete this tutorial.

To begin, you open up the data using `xarray.open_dataset`.

{:.input}
```python
# The (online) url for the MACAv2 data
data_path = "http://thredds.northwestknowledge.net:8080/thredds/dodsC/agg_macav2metdata_tasmax_BNU-ESM_r1i1p1_historical_1950_2005_CONUS_monthly.nc"

# Open the data using a context manager
with xr.open_dataset(data_path) as file_nc:
    # You can open the data using the code below, however it you use rio.write_crs
    # it will ensure that crs for your data persist throughout your analysis
    # max_temp_xr = file_nc
    max_temp_xr = file_nc.rio.write_crs(file_nc.rio.crs, inplace=True)

# View xarray object
max_temp_xr
```

{:.output}
{:.execute_result}



<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt, dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
Dimensions:          (lat: 585, lon: 1386, time: 672)
Coordinates:
  * lat              (lat) float64 25.06 25.1 25.15 25.19 ... 49.31 49.35 49.4
  * lon              (lon) float64 235.2 235.3 235.3 235.4 ... 292.9 292.9 292.9
  * time             (time) object 1950-01-15 00:00:00 ... 2005-12-15 00:00:00
    crs              int64 0
Data variables:
    air_temperature  (time, lat, lon) float32 ...
Attributes:
    description:                     Multivariate Adaptive Constructed Analog...
    id:                              MACAv2-METDATA
    naming_authority:                edu.uidaho.reacch
    Metadata_Conventions:            Unidata Dataset Discovery v1.0
    Metadata_Link:                   
    cdm_data_type:                   FLOAT
    title:                           Monthly aggregation of downscaled daily ...
    summary:                         This archive contains monthly downscaled...
    keywords:                        monthly, precipitation, maximum temperat...
    keywords_vocabulary:             
    standard_name_vocabulary:        CF-1.0
    history:                         No revisions.
    comment:                         
    geospatial_bounds:               POLYGON((-124.7722 25.0631,-124.7722 49....
    geospatial_lat_min:              25.0631
    geospatial_lat_max:              49.3960
    geospatial_lon_min:              -124.7722
    geospatial_lon_max:              -67.0648
    geospatial_lat_units:            decimal degrees north
    geospatial_lon_units:            decimal degrees east
    geospatial_lat_resolution:       0.0417
    geospatial_lon_resolution:       0.0417
    geospatial_vertical_min:         0.0
    geospatial_vertical_max:         0.0
    geospatial_vertical_resolution:  0.0
    geospatial_vertical_positive:    up
    time_coverage_start:             2000-01-01T00:0
    time_coverage_end:               2004-12-31T00:00
    time_coverage_duration:          P5Y
    time_coverage_resolution:        P1M
    date_created:                    2014-05-15
    date_modified:                   2014-05-15
    date_issued:                     2014-05-15
    creator_name:                    John Abatzoglou
    creator_url:                     http://maca.northwestknowledge.net
    creator_email:                   jabatzoglou@uidaho.edu
    institution:                     University of Idaho
    processing_level:                GRID
    project:                         
    contributor_name:                Katherine C. Hegewisch
    contributor_role:                Postdoctoral Fellow
    publisher_name:                  REACCH
    publisher_email:                 reacch@uidaho.edu
    publisher_url:                   http://www.reacchpna.org/
    license:                         Creative Commons CC0 1.0 Universal Dedic...
    coordinate_system:               WGS84,EPSG:4326
    grid_mapping:                    crs</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-f20bdb4b-2939-407c-9f3a-41567f44fb37' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-f20bdb4b-2939-407c-9f3a-41567f44fb37' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>lat</span>: 585</li><li><span class='xr-has-index'>lon</span>: 1386</li><li><span class='xr-has-index'>time</span>: 672</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-333e0ea8-aa3e-4e52-85be-f4f6fdd1be20' class='xr-section-summary-in' type='checkbox'  checked><label for='section-333e0ea8-aa3e-4e52-85be-f4f6fdd1be20' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>lat</span></div><div class='xr-var-dims'>(lat)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>25.06 25.1 25.15 ... 49.35 49.4</div><input id='attrs-bad04655-bd75-4d12-8050-d0d54e3dbe62' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-bad04655-bd75-4d12-8050-d0d54e3dbe62' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-03ec8a5c-241c-4fa4-bf2b-932927593e3b' class='xr-var-data-in' type='checkbox'><label for='data-03ec8a5c-241c-4fa4-bf2b-932927593e3b' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>latitude</dd><dt><span>standard_name :</span></dt><dd>latitude</dd><dt><span>units :</span></dt><dd>degrees_north</dd><dt><span>axis :</span></dt><dd>Y</dd><dt><span>description :</span></dt><dd>Latitude of the center of the grid cell</dd></dl></div><div class='xr-var-data'><pre>array([25.063078, 25.104744, 25.14641 , ..., 49.312691, 49.354359, 49.396023])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>lon</span></div><div class='xr-var-dims'>(lon)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>235.2 235.3 235.3 ... 292.9 292.9</div><input id='attrs-ea4b6fa1-597e-4839-98c2-ae033abb3ac5' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-ea4b6fa1-597e-4839-98c2-ae033abb3ac5' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c3ff6e66-eda1-4228-aac5-bd81ccb7a36a' class='xr-var-data-in' type='checkbox'><label for='data-c3ff6e66-eda1-4228-aac5-bd81ccb7a36a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>degrees_east</dd><dt><span>description :</span></dt><dd>Longitude of the center of the grid cell</dd><dt><span>long_name :</span></dt><dd>longitude</dd><dt><span>standard_name :</span></dt><dd>longitude</dd><dt><span>axis :</span></dt><dd>X</dd></dl></div><div class='xr-var-data'><pre>array([235.227844, 235.269501, 235.311157, ..., 292.851929, 292.893585,
       292.935242])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>1950-01-15 00:00:00 ... 2005-12-...</div><input id='attrs-3a255280-5435-4ac4-ae1d-6dba80b98ad6' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-3a255280-5435-4ac4-ae1d-6dba80b98ad6' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-72ee8221-8695-419f-9618-55dcf2d8b8cf' class='xr-var-data-in' type='checkbox'><label for='data-72ee8221-8695-419f-9618-55dcf2d8b8cf' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>description :</span></dt><dd>days since 1900-01-01</dd></dl></div><div class='xr-var-data'><pre>array([cftime.DatetimeNoLeap(1950, 1, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(1950, 2, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(1950, 3, 15, 0, 0, 0, 0), ...,
       cftime.DatetimeNoLeap(2005, 10, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2005, 11, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2005, 12, 15, 0, 0, 0, 0)], dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>crs</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int64</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-16afd9ae-4ab7-46da-ac36-a2eb1ffb11df' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-16afd9ae-4ab7-46da-ac36-a2eb1ffb11df' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-aa6c3e9f-ed6e-48dc-b660-1ed62529aa4f' class='xr-var-data-in' type='checkbox'><label for='data-aa6c3e9f-ed6e-48dc-b660-1ed62529aa4f' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCRS[&quot;undefined&quot;,DATUM[&quot;undefined&quot;,ELLIPSOID[&quot;undefined&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1,ID[&quot;EPSG&quot;,9001]]]],PRIMEM[&quot;undefined&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],CS[ellipsoidal,2],AXIS[&quot;longitude&quot;,east,ORDER[1],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],AXIS[&quot;latitude&quot;,north,ORDER[2],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>undefined</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>undefined</dd><dt><span>geographic_crs_name :</span></dt><dd>undefined</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCRS[&quot;undefined&quot;,DATUM[&quot;undefined&quot;,ELLIPSOID[&quot;undefined&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1,ID[&quot;EPSG&quot;,9001]]]],PRIMEM[&quot;undefined&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],CS[ellipsoidal,2],AXIS[&quot;longitude&quot;,east,ORDER[1],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],AXIS[&quot;latitude&quot;,north,ORDER[2],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-5bbd7e76-2765-414b-b1c6-7c5824c8701f' class='xr-section-summary-in' type='checkbox'  checked><label for='section-5bbd7e76-2765-414b-b1c6-7c5824c8701f' class='xr-section-summary' >Data variables: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>air_temperature</span></div><div class='xr-var-dims'>(time, lat, lon)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-3ddac6c8-c84b-41c8-a375-4a91309e7299' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-3ddac6c8-c84b-41c8-a375-4a91309e7299' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-3875fb06-c5f4-4488-ae83-e727f67a8045' class='xr-var-data-in' type='checkbox'><label for='data-3875fb06-c5f4-4488-ae83-e727f67a8045' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>Monthly Average of Daily Maximum Near-Surface Air Temperature</dd><dt><span>units :</span></dt><dd>K</dd><dt><span>grid_mapping :</span></dt><dd>crs</dd><dt><span>standard_name :</span></dt><dd>air_temperature</dd><dt><span>height :</span></dt><dd>2 m</dd><dt><span>cell_methods :</span></dt><dd>time: maximum(interval: 24 hours);mean over days</dd><dt><span>_ChunkSizes :</span></dt><dd>[ 10  44 107]</dd></dl></div><div class='xr-var-data'><pre>[544864320 values with dtype=float32]</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-72d53799-f1fe-4bab-b60e-fc3f444154fd' class='xr-section-summary-in' type='checkbox'  ><label for='section-72d53799-f1fe-4bab-b60e-fc3f444154fd' class='xr-section-summary' >Attributes: <span>(47)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>description :</span></dt><dd>Multivariate Adaptive Constructed Analogs (MACA) method, version 2.3,Dec 2013.</dd><dt><span>id :</span></dt><dd>MACAv2-METDATA</dd><dt><span>naming_authority :</span></dt><dd>edu.uidaho.reacch</dd><dt><span>Metadata_Conventions :</span></dt><dd>Unidata Dataset Discovery v1.0</dd><dt><span>Metadata_Link :</span></dt><dd></dd><dt><span>cdm_data_type :</span></dt><dd>FLOAT</dd><dt><span>title :</span></dt><dd>Monthly aggregation of downscaled daily meteorological data of Monthly Average of Daily Maximum Near-Surface Air Temperature from College of Global Change and Earth System Science, Beijing Normal University (BNU-ESM) using the run r1i1p1 of the historical scenario.</dd><dt><span>summary :</span></dt><dd>This archive contains monthly downscaled meteorological and hydrological projections for the Conterminous United States at 1/24-deg resolution. These monthly values are obtained by aggregating the daily values obtained from the downscaling using the Multivariate Adaptive Constructed Analogs (MACA, Abatzoglou, 2012) statistical downscaling method with the METDATA (Abatzoglou,2013) training dataset. The downscaled meteorological variables are maximum/minimum temperature(tasmax/tasmin), maximum/minimum relative humidity (rhsmax/rhsmin),precipitation amount(pr), downward shortwave solar radiation(rsds), eastward wind(uas), northward wind(vas), and specific humidity(huss). The downscaling is based on the 365-day model outputs from different global climate models (GCMs) from Phase 5 of the Coupled Model Inter-comparison Project (CMIP3) utlizing the historical (1950-2005) and future RCP4.5/8.5(2006-2099) scenarios. </dd><dt><span>keywords :</span></dt><dd>monthly, precipitation, maximum temperature, minimum temperature, downward shortwave solar radiation, specific humidity, wind velocity, CMIP5, Gridded Meteorological Data</dd><dt><span>keywords_vocabulary :</span></dt><dd></dd><dt><span>standard_name_vocabulary :</span></dt><dd>CF-1.0</dd><dt><span>history :</span></dt><dd>No revisions.</dd><dt><span>comment :</span></dt><dd></dd><dt><span>geospatial_bounds :</span></dt><dd>POLYGON((-124.7722 25.0631,-124.7722 49.3960, -67.0648 49.3960,-67.0648, 25.0631, -124.7722,25.0631))</dd><dt><span>geospatial_lat_min :</span></dt><dd>25.0631</dd><dt><span>geospatial_lat_max :</span></dt><dd>49.3960</dd><dt><span>geospatial_lon_min :</span></dt><dd>-124.7722</dd><dt><span>geospatial_lon_max :</span></dt><dd>-67.0648</dd><dt><span>geospatial_lat_units :</span></dt><dd>decimal degrees north</dd><dt><span>geospatial_lon_units :</span></dt><dd>decimal degrees east</dd><dt><span>geospatial_lat_resolution :</span></dt><dd>0.0417</dd><dt><span>geospatial_lon_resolution :</span></dt><dd>0.0417</dd><dt><span>geospatial_vertical_min :</span></dt><dd>0.0</dd><dt><span>geospatial_vertical_max :</span></dt><dd>0.0</dd><dt><span>geospatial_vertical_resolution :</span></dt><dd>0.0</dd><dt><span>geospatial_vertical_positive :</span></dt><dd>up</dd><dt><span>time_coverage_start :</span></dt><dd>2000-01-01T00:0</dd><dt><span>time_coverage_end :</span></dt><dd>2004-12-31T00:00</dd><dt><span>time_coverage_duration :</span></dt><dd>P5Y</dd><dt><span>time_coverage_resolution :</span></dt><dd>P1M</dd><dt><span>date_created :</span></dt><dd>2014-05-15</dd><dt><span>date_modified :</span></dt><dd>2014-05-15</dd><dt><span>date_issued :</span></dt><dd>2014-05-15</dd><dt><span>creator_name :</span></dt><dd>John Abatzoglou</dd><dt><span>creator_url :</span></dt><dd>http://maca.northwestknowledge.net</dd><dt><span>creator_email :</span></dt><dd>jabatzoglou@uidaho.edu</dd><dt><span>institution :</span></dt><dd>University of Idaho</dd><dt><span>processing_level :</span></dt><dd>GRID</dd><dt><span>project :</span></dt><dd></dd><dt><span>contributor_name :</span></dt><dd>Katherine C. Hegewisch</dd><dt><span>contributor_role :</span></dt><dd>Postdoctoral Fellow</dd><dt><span>publisher_name :</span></dt><dd>REACCH</dd><dt><span>publisher_email :</span></dt><dd>reacch@uidaho.edu</dd><dt><span>publisher_url :</span></dt><dd>http://www.reacchpna.org/</dd><dt><span>license :</span></dt><dd>Creative Commons CC0 1.0 Universal Dedication(http://creativecommons.org/publicdomain/zero/1.0/legalcode)</dd><dt><span>coordinate_system :</span></dt><dd>WGS84,EPSG:4326</dd><dt><span>grid_mapping :</span></dt><dd>crs</dd></dl></div></li></ul></div></div>





By default `xarray` does not handle spatial operations. However, if you 
load `rioxarray` it adds additional spatial functionality (supported by 
`rasterio` that supports handling:

* coordinate reference systems
* reprojection
* clipping 

and more. Below you grab the `coordinate reference system` of the 
climate data using the `.rio.crs` method that is available
because you have rioxarray loaded in this notebook.

{:.input}
```python
# For later - grab the crs of the data using rioxarray
climate_crs = max_temp_xr.rio.crs
climate_crs
```

{:.output}
{:.execute_result}



    CRS.from_wkt('GEOGCS["undefined",DATUM["undefined",SPHEROID["undefined",6378137,298.257223563]],PRIMEM["undefined",0],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AXIS["Longitude",EAST],AXIS["Latitude",NORTH]]')





### Work With the NetCDF Data Structure - A Hierarchical Data Format

The object above is hierarchical and contains metadata making it self-describing.
There are three dimensions to consider when working with this data which represent the 
x,y and z dimensions of the data:

1. latitude
2. longitude
3. time 

The latitude and longitude dimension values reprensent an arrya containing the point location
value of each pixel in decimal degrees. These are location of each pixel in your data. The 
time array similarly represents the time location for each array in the data cube. 
This particular dataset contains historical modeleled monthly max temperature values for the 
Continental United States. 

{:.input}
```python
# View first 5 latitude values
max_temp_xr["air_temperature"]["lat"].values[:5]

print("The min and max latitude values in the data is:", 
      max_temp_xr["air_temperature"]["lat"].values.min(), 
      max_temp_xr["air_temperature"]["lat"].values.max())
print("The min and max longitude values in the data is:", 
      max_temp_xr["air_temperature"]["lon"].values.min(), 
      max_temp_xr["air_temperature"]["lon"].values.max())

```

{:.output}
    The min and max latitude values in the data is: 25.063077926635742 49.39602279663086
    The min and max longitude values in the data is: 235.22784423828125 292.93524169921875



{:.input}
```python
# View first 5 and last. 5 time values - notice the span of
# dates range from 1950 to 2005
print("The earliest date in the data is:", max_temp_xr["air_temperature"]["time"].values.min())
print("The latest date in the data is:", max_temp_xr["air_temperature"]["time"].values.max())    

```

{:.output}
    The earliest date in the data is: 1950-01-15 00:00:00
    The latest date in the data is: 2005-12-15 00:00:00



{:.input}
```python
max_temp_xr["air_temperature"]["time"].values.shape
```

{:.output}
{:.execute_result}



    (672,)





Time is the z dimension. In this dataset you have 
672 months worth of data.

These data are spatial and thus have a coordinate reference system
associated with them. You can access this information using `.crs` 

HINT `max_temp_xr["crs"]` should also work!

Below, you use `.rio.crs` to access the CRS information rather than `.crs`. Rioxarray 
generally does a better job of accessing data attributes

{:.input}
```python
# The data are lat/long (not projected)
max_temp_xr.rio.crs
```

{:.output}
{:.execute_result}



    CRS.from_wkt('GEOGCS["undefined",DATUM["undefined",SPHEROID["undefined",6378137,298.257223563]],PRIMEM["undefined",0],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AXIS["Longitude",EAST],AXIS["Latitude",NORTH]]')





## Hierarchical Formats Are Self Describing

Hierarchical data formats are self-describing. This means that the 
metadata for the data are contained within the file itself. `xarray` 
stores metadata in the `.attrs` part of the file structure using a 
Python dictionary structure. You can view the metadata using `.attrs`.

{:.input}
```python
# View metadata
metadata = max_temp_xr.attrs
metadata
```

{:.output}
{:.execute_result}



    {'description': 'Multivariate Adaptive Constructed Analogs (MACA) method, version 2.3,Dec 2013.',
     'id': 'MACAv2-METDATA',
     'naming_authority': 'edu.uidaho.reacch',
     'Metadata_Conventions': 'Unidata Dataset Discovery v1.0',
     'Metadata_Link': '',
     'cdm_data_type': 'FLOAT',
     'title': 'Monthly aggregation of downscaled daily meteorological data of Monthly Average of Daily Maximum Near-Surface Air Temperature from College of Global Change and Earth System Science, Beijing Normal University (BNU-ESM) using the run r1i1p1 of the historical scenario.',
     'summary': 'This archive contains monthly downscaled meteorological and hydrological projections for the Conterminous United States at 1/24-deg resolution. These monthly values are obtained by aggregating the daily values obtained from the downscaling using the Multivariate Adaptive Constructed Analogs (MACA, Abatzoglou, 2012) statistical downscaling method with the METDATA (Abatzoglou,2013) training dataset. The downscaled meteorological variables are maximum/minimum temperature(tasmax/tasmin), maximum/minimum relative humidity (rhsmax/rhsmin),precipitation amount(pr), downward shortwave solar radiation(rsds), eastward wind(uas), northward wind(vas), and specific humidity(huss). The downscaling is based on the 365-day model outputs from different global climate models (GCMs) from Phase 5 of the Coupled Model Inter-comparison Project (CMIP3) utlizing the historical (1950-2005) and future RCP4.5/8.5(2006-2099) scenarios. ',
     'keywords': 'monthly, precipitation, maximum temperature, minimum temperature, downward shortwave solar radiation, specific humidity, wind velocity, CMIP5, Gridded Meteorological Data',
     'keywords_vocabulary': '',
     'standard_name_vocabulary': 'CF-1.0',
     'history': 'No revisions.',
     'comment': '',
     'geospatial_bounds': 'POLYGON((-124.7722 25.0631,-124.7722 49.3960, -67.0648 49.3960,-67.0648, 25.0631, -124.7722,25.0631))',
     'geospatial_lat_min': '25.0631',
     'geospatial_lat_max': '49.3960',
     'geospatial_lon_min': '-124.7722',
     'geospatial_lon_max': '-67.0648',
     'geospatial_lat_units': 'decimal degrees north',
     'geospatial_lon_units': 'decimal degrees east',
     'geospatial_lat_resolution': '0.0417',
     'geospatial_lon_resolution': '0.0417',
     'geospatial_vertical_min': 0.0,
     'geospatial_vertical_max': 0.0,
     'geospatial_vertical_resolution': 0.0,
     'geospatial_vertical_positive': 'up',
     'time_coverage_start': '2000-01-01T00:0',
     'time_coverage_end': '2004-12-31T00:00',
     'time_coverage_duration': 'P5Y',
     'time_coverage_resolution': 'P1M',
     'date_created': '2014-05-15',
     'date_modified': '2014-05-15',
     'date_issued': '2014-05-15',
     'creator_name': 'John Abatzoglou',
     'creator_url': 'http://maca.northwestknowledge.net',
     'creator_email': 'jabatzoglou@uidaho.edu',
     'institution': 'University of Idaho',
     'processing_level': 'GRID',
     'project': '',
     'contributor_name': 'Katherine C. Hegewisch',
     'contributor_role': 'Postdoctoral Fellow',
     'publisher_name': 'REACCH',
     'publisher_email': 'reacch@uidaho.edu',
     'publisher_url': 'http://www.reacchpna.org/',
     'license': 'Creative Commons CC0 1.0 Universal Dedication(http://creativecommons.org/publicdomain/zero/1.0/legalcode)',
     'coordinate_system': 'WGS84,EPSG:4326',
     'grid_mapping': 'crs'}





{:.input}
```python
# Above you grabbed the metadata (which is a dictionary)
# Here you can print any part of the dictionary that you wish to
metadata["title"]
```

{:.output}
{:.execute_result}



    'Monthly aggregation of downscaled daily meteorological data of Monthly Average of Daily Maximum Near-Surface Air Temperature from College of Global Change and Earth System Science, Beijing Normal University (BNU-ESM) using the run r1i1p1 of the historical scenario.'





## Subsetting or "Slicing" Your Data
 
You can quickly and efficiently slice and subset your data using 
`xarray`. Below, you will learn how to slice the data using the `.sel()` method.

This will return exactly two data points at the location - the temperature 
value for each day in your temporal slice.

{:.input}
```python
# Select a single x,y combination from the data
key=400
longitude = max_temp_xr["air_temperature"]["lon"].values[key]
latitude = max_temp_xr["air_temperature"]["lat"].values[key]

print("Long, Lat values:", longitude, latitude)
```

{:.output}
    Long, Lat values: 251.89422607421875 41.72947692871094



Below, you can see the x, y location that for the latitude/longitude
location that you will use to slice the data above plotted on a map. 
The code below is an example of creating a spatial plot using
the `cartopy` package that shows the locatio nthat you selected. **This step is optional.**

{:.input}
```python
# Create a spatial map of your selected location with cartopy
extent = [-120, -70, 24, 50.5]
central_lon = np.mean(extent[:2])
central_lat = np.mean(extent[2:])

f, ax = plt.subplots(figsize=(12, 6),
                     subplot_kw={'projection': ccrs.AlbersEqualArea(central_lon, central_lat)})
ax.coastlines()
# Plot the selected location 
ax.plot(longitude-360, latitude, 
        'r*', 
        transform=ccrs.PlateCarree(),
       color="purple", markersize=10)

ax.set_extent(extent)
ax.set(title="Location of the Latitude / Longitude Being Used To to Slice Your netcdf Climate Data File")

# Adds continent boundaries to the map
ax.add_feature(cfeature.LAND, edgecolor='black')

ax.gridlines()
plt.show()
```

{:.output}
    /opt/conda/lib/python3.8/site-packages/cartopy/io/__init__.py:260: DownloadWarning: Downloading: https://naciscdn.org/naturalearth/50m/physical/ne_50m_land.zip
      warnings.warn('Downloading: {}'.format(url), DownloadWarning)
    /opt/conda/lib/python3.8/site-packages/cartopy/io/__init__.py:260: DownloadWarning: Downloading: https://naciscdn.org/naturalearth/50m/physical/ne_50m_coastline.zip
      warnings.warn('Downloading: {}'.format(url), DownloadWarning)



{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_21_1.png">

</figure>




Now it is time to subset the data for the point location that 
you are interested in (the lat / lon value that you plotted above). 
You can slice the data for one single latitude, longitude location using
the `.sel()` method.

When you slice the data using one single point, your output for every 
(monthly in this case) time step in the data will be a single pixel 
value representing max temperature. 

{:.input}
```python
# Slice the data spatially using a single lat/lon point
one_point = max_temp_xr["air_temperature"].sel(lat=latitude,
                                               lon=longitude)
one_point
```

{:.output}
{:.execute_result}



<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt, dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;air_temperature&#x27; (time: 672)&gt;
array([271.11615, 274.05585, 279.538  , ..., 286.85074, 279.6746 , 271.4411 ],
      dtype=float32)
Coordinates:
    lat      float64 41.73
    lon      float64 251.9
  * time     (time) object 1950-01-15 00:00:00 ... 2005-12-15 00:00:00
    crs      int64 0
Attributes:
    long_name:      Monthly Average of Daily Maximum Near-Surface Air Tempera...
    units:          K
    grid_mapping:   crs
    standard_name:  air_temperature
    height:         2 m
    cell_methods:   time: maximum(interval: 24 hours);mean over days
    _ChunkSizes:    [ 10  44 107]</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'air_temperature'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>time</span>: 672</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-7aa09ef2-17a5-4870-81cf-bce7e4e65bf1' class='xr-array-in' type='checkbox' checked><label for='section-7aa09ef2-17a5-4870-81cf-bce7e4e65bf1' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>271.11615 274.05585 279.538 284.42365 ... 286.85074 279.6746 271.4411</span></div><div class='xr-array-data'><pre>array([271.11615, 274.05585, 279.538  , ..., 286.85074, 279.6746 , 271.4411 ],
      dtype=float32)</pre></div></div></li><li class='xr-section-item'><input id='section-383dc887-4813-4d42-880e-bd6ab46254b5' class='xr-section-summary-in' type='checkbox'  checked><label for='section-383dc887-4813-4d42-880e-bd6ab46254b5' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>lat</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>41.73</div><input id='attrs-92fa882c-d75c-40dd-a9dc-687725a083fb' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-92fa882c-d75c-40dd-a9dc-687725a083fb' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-3f0eeefe-c66d-4905-8b81-868b30fa5f0d' class='xr-var-data-in' type='checkbox'><label for='data-3f0eeefe-c66d-4905-8b81-868b30fa5f0d' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>latitude</dd><dt><span>standard_name :</span></dt><dd>latitude</dd><dt><span>units :</span></dt><dd>degrees_north</dd><dt><span>axis :</span></dt><dd>Y</dd><dt><span>description :</span></dt><dd>Latitude of the center of the grid cell</dd></dl></div><div class='xr-var-data'><pre>array(41.72947693)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>lon</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>251.9</div><input id='attrs-c6fbacf3-cf0d-4cdc-8638-53a2c54f188f' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-c6fbacf3-cf0d-4cdc-8638-53a2c54f188f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-141fb01a-a8ca-49ce-9db8-459ddebd1238' class='xr-var-data-in' type='checkbox'><label for='data-141fb01a-a8ca-49ce-9db8-459ddebd1238' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>degrees_east</dd><dt><span>description :</span></dt><dd>Longitude of the center of the grid cell</dd><dt><span>long_name :</span></dt><dd>longitude</dd><dt><span>standard_name :</span></dt><dd>longitude</dd><dt><span>axis :</span></dt><dd>X</dd></dl></div><div class='xr-var-data'><pre>array(251.89422607)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>1950-01-15 00:00:00 ... 2005-12-...</div><input id='attrs-9344a183-5258-419c-aad1-920fb1bb8273' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-9344a183-5258-419c-aad1-920fb1bb8273' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b9afdaef-ccc5-4700-8aad-cef325cacebd' class='xr-var-data-in' type='checkbox'><label for='data-b9afdaef-ccc5-4700-8aad-cef325cacebd' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>description :</span></dt><dd>days since 1900-01-01</dd></dl></div><div class='xr-var-data'><pre>array([cftime.DatetimeNoLeap(1950, 1, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(1950, 2, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(1950, 3, 15, 0, 0, 0, 0), ...,
       cftime.DatetimeNoLeap(2005, 10, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2005, 11, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2005, 12, 15, 0, 0, 0, 0)], dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>crs</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int64</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-b9c46b7a-1f08-4a33-a1fb-a9fa464ee860' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-b9c46b7a-1f08-4a33-a1fb-a9fa464ee860' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-04c0c9fa-29f8-4310-abaf-451a887a26d2' class='xr-var-data-in' type='checkbox'><label for='data-04c0c9fa-29f8-4310-abaf-451a887a26d2' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCRS[&quot;undefined&quot;,DATUM[&quot;undefined&quot;,ELLIPSOID[&quot;undefined&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1,ID[&quot;EPSG&quot;,9001]]]],PRIMEM[&quot;undefined&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],CS[ellipsoidal,2],AXIS[&quot;longitude&quot;,east,ORDER[1],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],AXIS[&quot;latitude&quot;,north,ORDER[2],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>undefined</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>undefined</dd><dt><span>geographic_crs_name :</span></dt><dd>undefined</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCRS[&quot;undefined&quot;,DATUM[&quot;undefined&quot;,ELLIPSOID[&quot;undefined&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1,ID[&quot;EPSG&quot;,9001]]]],PRIMEM[&quot;undefined&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],CS[ellipsoidal,2],AXIS[&quot;longitude&quot;,east,ORDER[1],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],AXIS[&quot;latitude&quot;,north,ORDER[2],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-a2bc3822-b5f2-43f7-9c8f-921fbde56ea0' class='xr-section-summary-in' type='checkbox'  checked><label for='section-a2bc3822-b5f2-43f7-9c8f-921fbde56ea0' class='xr-section-summary' >Attributes: <span>(7)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>Monthly Average of Daily Maximum Near-Surface Air Temperature</dd><dt><span>units :</span></dt><dd>K</dd><dt><span>grid_mapping :</span></dt><dd>crs</dd><dt><span>standard_name :</span></dt><dd>air_temperature</dd><dt><span>height :</span></dt><dd>2 m</dd><dt><span>cell_methods :</span></dt><dd>time: maximum(interval: 24 hours);mean over days</dd><dt><span>_ChunkSizes :</span></dt><dd>[ 10  44 107]</dd></dl></div></li></ul></div></div>





When you slice the data by a single point, notice that output data
only has a single array of values. In this case these values represent
air temperature (in K) over time. 

{:.input}
```python
# Notice the shape of the output array
one_point.shape
```

{:.output}
{:.execute_result}



    (672,)





The data stored in the xarray object is a numpy array. You can process
the data in the same way you would process any other numpy array.  

{:.input}
```python
# View the first 5 values for that single point
one_point.values[:5]
```

{:.output}
{:.execute_result}



    array([271.11615, 274.05585, 279.538  , 284.42365, 294.1337 ],
          dtype=float32)





### Plot A Time Series For a Single Location 

Above you used a single point location to slice your data. Because
the data are for only one location, but over time, you can quickly 
create a scatterplot of the data using `objectname.plot()`.


{:.input}
```python
# Use xarray to create a quick time series plot
one_point.plot.line()
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_29_0.png" alt = "Standard plot output from xarray .plot() with no plot elements customized. You can make this plot look much nicer by customizing the line and point markers and colors.">
<figcaption>Standard plot output from xarray .plot() with no plot elements customized. You can make this plot look much nicer by customizing the line and point markers and colors.</figcaption>

</figure>




You can make the plot a bit prettier if you'd like using the 
standard Python matplotlib plot parameters. Below you change the marker color 
to purple and the lines to grey. `figsize` is used to adjust the 
size of the plot. 

{:.input}
```python
# You can clean up your plot as you wish using standard matplotlib approaches
f, ax = plt.subplots(figsize=(12, 6))
one_point.plot.line(hue='lat',
                    marker="o",
                    ax=ax,
                    color="grey",
                    markerfacecolor="purple",
                    markeredgecolor="purple")
ax.set(title="Time Series For a Single Lat / Lon Location")

# Uncomment the line below if you wish to export the figure as a .png file
# plt.savefig("single_point_timeseries.png")
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_31_0.png" alt = "Plot showing historic max temperature climate data plotted using xarray .plot().">
<figcaption>Plot showing historic max temperature climate data plotted using xarray .plot().</figcaption>

</figure>




### Convert Climate Time Series Data to a Pandas DataFrame & Export to a .csv File
You can convert your data to a `DataFrame` and export to 
a `.csv` file using `to_dataframe()` and `to_csv()`. 

{:.input}
```python
# Convert to dataframe -- then this can easily be exported to a csv
one_point_df = one_point.to_dataframe()
# View just the first 5 rows of the data
one_point_df.head()
```

{:.output}
{:.execute_result}



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>lat</th>
      <th>lon</th>
      <th>crs</th>
      <th>air_temperature</th>
    </tr>
    <tr>
      <th>time</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1950-01-15 00:00:00</th>
      <td>41.729477</td>
      <td>251.894226</td>
      <td>0</td>
      <td>271.116150</td>
    </tr>
    <tr>
      <th>1950-02-15 00:00:00</th>
      <td>41.729477</td>
      <td>251.894226</td>
      <td>0</td>
      <td>274.055847</td>
    </tr>
    <tr>
      <th>1950-03-15 00:00:00</th>
      <td>41.729477</td>
      <td>251.894226</td>
      <td>0</td>
      <td>279.537994</td>
    </tr>
    <tr>
      <th>1950-04-15 00:00:00</th>
      <td>41.729477</td>
      <td>251.894226</td>
      <td>0</td>
      <td>284.423645</td>
    </tr>
    <tr>
      <th>1950-05-15 00:00:00</th>
      <td>41.729477</td>
      <td>251.894226</td>
      <td>0</td>
      <td>294.133698</td>
    </tr>
  </tbody>
</table>
</div>





Once you have a dataframe object, you can export it
directly to a `.csv` file format.

{:.input}
```python
# Export data to .csv file
one_point_df.to_csv("one-location.csv")
```

## Slice Climate MACAv2 Data By Time and Location

Above you sliced the data by spatial location, selecting only one point
location at a specific latitude and longitude. You can also slice the data by time. 
Below, you slice the data at the selected lat / long location and for a 5-year time period
in the year 2000.

Notice that the output shape is 60. 60 represents 12 months a year over 5 years
for the selected lat / lon location. Again because you are slicing out data 
for a single point location, you have a total of 60 data points in the output
numpy array.

{:.input}
```python
start_date = "2000-01-01"
end_date = "2005-01-01"
temp_2000_2005 = max_temp_xr["air_temperature"].sel(time=slice(start_date, end_date),
                                                    lat=45.02109146118164,
                                                    lon=243.01937866210938)
temp_2000_2005
```

{:.output}
{:.execute_result}



<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt, dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;air_temperature&#x27; (time: 60)&gt;
array([273.74884, 277.1434 , 280.0702 , 280.6042 , 289.23993, 290.86853,
       297.70242, 301.36722, 296.4382 , 291.29434, 275.8434 , 273.0037 ,
       275.154  , 278.77863, 282.29105, 283.87787, 290.53387, 296.701  ,
       297.4607 , 301.16116, 295.5421 , 287.3863 , 276.85397, 274.35953,
       278.0117 , 275.68024, 282.12674, 286.41693, 292.4094 , 297.30606,
       300.84628, 302.10593, 288.83698, 288.44   , 276.38992, 276.5348 ,
       275.86145, 276.5708 , 280.8476 , 284.35712, 292.14087, 293.66147,
       301.09534, 302.7656 , 294.99573, 287.5425 , 277.80093, 273.25894,
       274.09232, 276.94736, 280.25446, 283.47614, 289.63776, 290.5187 ,
       301.8519 , 300.40402, 290.73633, 289.60886, 277.12424, 275.7836 ],
      dtype=float32)
Coordinates:
    lat      float64 45.02
    lon      float64 243.0
  * time     (time) object 2000-01-15 00:00:00 ... 2004-12-15 00:00:00
    crs      int64 0
Attributes:
    long_name:      Monthly Average of Daily Maximum Near-Surface Air Tempera...
    units:          K
    grid_mapping:   crs
    standard_name:  air_temperature
    height:         2 m
    cell_methods:   time: maximum(interval: 24 hours);mean over days
    _ChunkSizes:    [ 10  44 107]</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'air_temperature'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>time</span>: 60</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-9fb79e67-d540-4147-9ebc-f216cdeff4a7' class='xr-array-in' type='checkbox' checked><label for='section-9fb79e67-d540-4147-9ebc-f216cdeff4a7' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>273.74884 277.1434 280.0702 280.6042 ... 289.60886 277.12424 275.7836</span></div><div class='xr-array-data'><pre>array([273.74884, 277.1434 , 280.0702 , 280.6042 , 289.23993, 290.86853,
       297.70242, 301.36722, 296.4382 , 291.29434, 275.8434 , 273.0037 ,
       275.154  , 278.77863, 282.29105, 283.87787, 290.53387, 296.701  ,
       297.4607 , 301.16116, 295.5421 , 287.3863 , 276.85397, 274.35953,
       278.0117 , 275.68024, 282.12674, 286.41693, 292.4094 , 297.30606,
       300.84628, 302.10593, 288.83698, 288.44   , 276.38992, 276.5348 ,
       275.86145, 276.5708 , 280.8476 , 284.35712, 292.14087, 293.66147,
       301.09534, 302.7656 , 294.99573, 287.5425 , 277.80093, 273.25894,
       274.09232, 276.94736, 280.25446, 283.47614, 289.63776, 290.5187 ,
       301.8519 , 300.40402, 290.73633, 289.60886, 277.12424, 275.7836 ],
      dtype=float32)</pre></div></div></li><li class='xr-section-item'><input id='section-0d21528d-c573-489f-a98f-8e1142ee8bf8' class='xr-section-summary-in' type='checkbox'  checked><label for='section-0d21528d-c573-489f-a98f-8e1142ee8bf8' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>lat</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>45.02</div><input id='attrs-de1de8a1-6307-4013-9182-4e56fb91eb27' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-de1de8a1-6307-4013-9182-4e56fb91eb27' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-64e3ec6c-39a6-4b09-9e21-546440abbc5e' class='xr-var-data-in' type='checkbox'><label for='data-64e3ec6c-39a6-4b09-9e21-546440abbc5e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>latitude</dd><dt><span>standard_name :</span></dt><dd>latitude</dd><dt><span>units :</span></dt><dd>degrees_north</dd><dt><span>axis :</span></dt><dd>Y</dd><dt><span>description :</span></dt><dd>Latitude of the center of the grid cell</dd></dl></div><div class='xr-var-data'><pre>array(45.02109146)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>lon</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>243.0</div><input id='attrs-d01a486b-a49a-421b-a0d6-d20f408dc96b' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-d01a486b-a49a-421b-a0d6-d20f408dc96b' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-2714beef-6094-4a5a-aa1e-fbd2c14e2415' class='xr-var-data-in' type='checkbox'><label for='data-2714beef-6094-4a5a-aa1e-fbd2c14e2415' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>degrees_east</dd><dt><span>description :</span></dt><dd>Longitude of the center of the grid cell</dd><dt><span>long_name :</span></dt><dd>longitude</dd><dt><span>standard_name :</span></dt><dd>longitude</dd><dt><span>axis :</span></dt><dd>X</dd></dl></div><div class='xr-var-data'><pre>array(243.01937866)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>2000-01-15 00:00:00 ... 2004-12-...</div><input id='attrs-bd38f274-60a1-4106-b54f-fd3583d553a1' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-bd38f274-60a1-4106-b54f-fd3583d553a1' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-fc23b6d2-c6fe-4fb8-bca6-799a96a80928' class='xr-var-data-in' type='checkbox'><label for='data-fc23b6d2-c6fe-4fb8-bca6-799a96a80928' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>description :</span></dt><dd>days since 1900-01-01</dd></dl></div><div class='xr-var-data'><pre>array([cftime.DatetimeNoLeap(2000, 1, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 2, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 3, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 4, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 5, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 6, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 7, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 8, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 9, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 10, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 11, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2000, 12, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 1, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 2, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 3, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 4, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 5, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 6, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 7, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 8, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 9, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 10, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 11, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2001, 12, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 1, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 2, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 3, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 4, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 5, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 6, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 7, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 8, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 9, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 10, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 11, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2002, 12, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 1, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 2, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 3, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 4, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 5, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 6, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 7, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 8, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 9, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 10, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 11, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2003, 12, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 1, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 2, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 3, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 4, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 5, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 6, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 7, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 8, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 9, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 10, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 11, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(2004, 12, 15, 0, 0, 0, 0)], dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>crs</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int64</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-62d277a4-5e2b-43e8-80b1-5e6ef1df6ba3' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-62d277a4-5e2b-43e8-80b1-5e6ef1df6ba3' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-6033d685-388d-4b25-be7f-f831b64b6088' class='xr-var-data-in' type='checkbox'><label for='data-6033d685-388d-4b25-be7f-f831b64b6088' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCRS[&quot;undefined&quot;,DATUM[&quot;undefined&quot;,ELLIPSOID[&quot;undefined&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1,ID[&quot;EPSG&quot;,9001]]]],PRIMEM[&quot;undefined&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],CS[ellipsoidal,2],AXIS[&quot;longitude&quot;,east,ORDER[1],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],AXIS[&quot;latitude&quot;,north,ORDER[2],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>undefined</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>undefined</dd><dt><span>geographic_crs_name :</span></dt><dd>undefined</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCRS[&quot;undefined&quot;,DATUM[&quot;undefined&quot;,ELLIPSOID[&quot;undefined&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1,ID[&quot;EPSG&quot;,9001]]]],PRIMEM[&quot;undefined&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],CS[ellipsoidal,2],AXIS[&quot;longitude&quot;,east,ORDER[1],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],AXIS[&quot;latitude&quot;,north,ORDER[2],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-818371fd-c0a4-4633-bbf1-bf0e50c67125' class='xr-section-summary-in' type='checkbox'  checked><label for='section-818371fd-c0a4-4633-bbf1-bf0e50c67125' class='xr-section-summary' >Attributes: <span>(7)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>Monthly Average of Daily Maximum Near-Surface Air Temperature</dd><dt><span>units :</span></dt><dd>K</dd><dt><span>grid_mapping :</span></dt><dd>crs</dd><dt><span>standard_name :</span></dt><dd>air_temperature</dd><dt><span>height :</span></dt><dd>2 m</dd><dt><span>cell_methods :</span></dt><dd>time: maximum(interval: 24 hours);mean over days</dd><dt><span>_ChunkSizes :</span></dt><dd>[ 10  44 107]</dd></dl></div></li></ul></div></div>





{:.input}
```python
temp_2000_2005.shape
```

{:.output}
{:.execute_result}



    (60,)





{:.input}
```python
# View the 60 data points (raster cell values) associated with the spatial and temporal subset
temp_2000_2005.values
```

{:.output}
{:.execute_result}



    array([273.74884, 277.1434 , 280.0702 , 280.6042 , 289.23993, 290.86853,
           297.70242, 301.36722, 296.4382 , 291.29434, 275.8434 , 273.0037 ,
           275.154  , 278.77863, 282.29105, 283.87787, 290.53387, 296.701  ,
           297.4607 , 301.16116, 295.5421 , 287.3863 , 276.85397, 274.35953,
           278.0117 , 275.68024, 282.12674, 286.41693, 292.4094 , 297.30606,
           300.84628, 302.10593, 288.83698, 288.44   , 276.38992, 276.5348 ,
           275.86145, 276.5708 , 280.8476 , 284.35712, 292.14087, 293.66147,
           301.09534, 302.7656 , 294.99573, 287.5425 , 277.80093, 273.25894,
           274.09232, 276.94736, 280.25446, 283.47614, 289.63776, 290.5187 ,
           301.8519 , 300.40402, 290.73633, 289.60886, 277.12424, 275.7836 ],
          dtype=float32)





Notice that in this instance you have much less data for that specific point
(5 years worth of data to be exact).

{:.input}
```python
# Plot the data just like you did above
f, ax = plt.subplots(figsize=(12, 6))
temp_2000_2005.plot.line(hue='lat',
                         marker="o",
                         ax=ax,
                         color="grey",
                         markerfacecolor="purple",
                         markeredgecolor="purple")
ax.set(title="A 5 Year Time Series of Temperature Data For A Single Location")
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_41_0.png" alt = "Plot showing 5 years of monthly max temperature data. Here the data are cleaned up a bit with marker colors and lines adjusted.">
<figcaption>Plot showing 5 years of monthly max temperature data. Here the data are cleaned up a bit with marker colors and lines adjusted.</figcaption>

</figure>




## Convert Subsetted to a DataFrame & Export to a .csv File

Similar to what you did above, you can also convert your data to a DataFrame. Once your data are in a 
DataFrame format, you can quickly export a `.csv` file.

{:.input}
```python
# Convert to dataframe -- then this can be exported to a csv if you want that
temp_2000_2005_df = temp_2000_2005.to_dataframe()
# View just the first 5 rows of the data
temp_2000_2005_df.head()
```

{:.output}
{:.execute_result}



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>lat</th>
      <th>lon</th>
      <th>crs</th>
      <th>air_temperature</th>
    </tr>
    <tr>
      <th>time</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2000-01-15 00:00:00</th>
      <td>45.021091</td>
      <td>243.019379</td>
      <td>0</td>
      <td>273.748840</td>
    </tr>
    <tr>
      <th>2000-02-15 00:00:00</th>
      <td>45.021091</td>
      <td>243.019379</td>
      <td>0</td>
      <td>277.143402</td>
    </tr>
    <tr>
      <th>2000-03-15 00:00:00</th>
      <td>45.021091</td>
      <td>243.019379</td>
      <td>0</td>
      <td>280.070190</td>
    </tr>
    <tr>
      <th>2000-04-15 00:00:00</th>
      <td>45.021091</td>
      <td>243.019379</td>
      <td>0</td>
      <td>280.604187</td>
    </tr>
    <tr>
      <th>2000-05-15 00:00:00</th>
      <td>45.021091</td>
      <td>243.019379</td>
      <td>0</td>
      <td>289.239929</td>
    </tr>
  </tbody>
</table>
</div>





{:.input}
```python
# Create filename from subset info
file_name = "monthly-temp-" + start_date + "-" + end_date + ".csv"
file_name
```

{:.output}
{:.execute_result}



    'monthly-temp-2000-01-01-2005-01-01.csv'





{:.input}
```python
# Export to a csv file to share with your friends!
temp_2000_2005_df.to_csv(file_name)
```

## Slice The Data Across a Spatial Extent For A Specific Time Period

In the above example you sliced out a subset of the data for a specific
point location over time. Below you select 
data for the entire CONUS (Continental United States) study area and analyze it 
as a spatial raster object.

Once again you use `.sel()` combined with `slice()` subset the data. 

Notice below that if you don't specify the lat and lon extent, 
it will by default return all of the pixels available in the data for the 
specified time period.

Notice that you are plotting the entire spatial extent of the data which is 
in this case called CONUS (**CON**tinental **U**nited **S**tates)

{:.input}
```python
start_date = "1950-01-15"
end_date = "1950-02-15"

two_months_conus = max_temp_xr["air_temperature"].sel(
    time=slice(start_date, end_date))
# Notice that time has a value of **2** below representing two time steps or months worth of data
two_months_conus
```

{:.output}
{:.execute_result}



<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt, dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;air_temperature&#x27; (time: 2, lat: 585, lon: 1386)&gt;
[1621620 values with dtype=float32]
Coordinates:
  * lat      (lat) float64 25.06 25.1 25.15 25.19 ... 49.27 49.31 49.35 49.4
  * lon      (lon) float64 235.2 235.3 235.3 235.4 ... 292.8 292.9 292.9 292.9
  * time     (time) object 1950-01-15 00:00:00 1950-02-15 00:00:00
    crs      int64 0
Attributes:
    long_name:      Monthly Average of Daily Maximum Near-Surface Air Tempera...
    units:          K
    grid_mapping:   crs
    standard_name:  air_temperature
    height:         2 m
    cell_methods:   time: maximum(interval: 24 hours);mean over days
    _ChunkSizes:    [ 10  44 107]</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'air_temperature'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>time</span>: 2</li><li><span class='xr-has-index'>lat</span>: 585</li><li><span class='xr-has-index'>lon</span>: 1386</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-748c058f-fa07-4a3f-ad94-78bd97487419' class='xr-array-in' type='checkbox' checked><label for='section-748c058f-fa07-4a3f-ad94-78bd97487419' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>...</span></div><div class='xr-array-data'><pre>[1621620 values with dtype=float32]</pre></div></div></li><li class='xr-section-item'><input id='section-bc933ea3-4236-433b-a150-ca0cd8491fe8' class='xr-section-summary-in' type='checkbox'  checked><label for='section-bc933ea3-4236-433b-a150-ca0cd8491fe8' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>lat</span></div><div class='xr-var-dims'>(lat)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>25.06 25.1 25.15 ... 49.35 49.4</div><input id='attrs-bec8fb96-5f7c-4e53-b1aa-40724b52d2ce' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-bec8fb96-5f7c-4e53-b1aa-40724b52d2ce' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c9c330b7-9ee8-4b54-b558-9344864e8389' class='xr-var-data-in' type='checkbox'><label for='data-c9c330b7-9ee8-4b54-b558-9344864e8389' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>latitude</dd><dt><span>standard_name :</span></dt><dd>latitude</dd><dt><span>units :</span></dt><dd>degrees_north</dd><dt><span>axis :</span></dt><dd>Y</dd><dt><span>description :</span></dt><dd>Latitude of the center of the grid cell</dd></dl></div><div class='xr-var-data'><pre>array([25.063078, 25.104744, 25.14641 , ..., 49.312691, 49.354359, 49.396023])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>lon</span></div><div class='xr-var-dims'>(lon)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>235.2 235.3 235.3 ... 292.9 292.9</div><input id='attrs-df06b36a-c24d-4a48-9060-949e00945e94' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-df06b36a-c24d-4a48-9060-949e00945e94' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-bde6b27c-f48b-42a5-a373-a3831dfed271' class='xr-var-data-in' type='checkbox'><label for='data-bde6b27c-f48b-42a5-a373-a3831dfed271' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>degrees_east</dd><dt><span>description :</span></dt><dd>Longitude of the center of the grid cell</dd><dt><span>long_name :</span></dt><dd>longitude</dd><dt><span>standard_name :</span></dt><dd>longitude</dd><dt><span>axis :</span></dt><dd>X</dd></dl></div><div class='xr-var-data'><pre>array([235.227844, 235.269501, 235.311157, ..., 292.851929, 292.893585,
       292.935242])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>1950-01-15 00:00:00 1950-02-15 0...</div><input id='attrs-d92eba10-6043-4ab3-bb5d-c04627793fb0' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-d92eba10-6043-4ab3-bb5d-c04627793fb0' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-7654fa95-55a2-46a5-8228-7d5a26ae6c7f' class='xr-var-data-in' type='checkbox'><label for='data-7654fa95-55a2-46a5-8228-7d5a26ae6c7f' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>description :</span></dt><dd>days since 1900-01-01</dd></dl></div><div class='xr-var-data'><pre>array([cftime.DatetimeNoLeap(1950, 1, 15, 0, 0, 0, 0),
       cftime.DatetimeNoLeap(1950, 2, 15, 0, 0, 0, 0)], dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>crs</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int64</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-16e4e3be-bac4-48e4-92bd-ac49206f5933' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-16e4e3be-bac4-48e4-92bd-ac49206f5933' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-186ee170-d29b-48ec-bc41-e70eb9c1711e' class='xr-var-data-in' type='checkbox'><label for='data-186ee170-d29b-48ec-bc41-e70eb9c1711e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCRS[&quot;undefined&quot;,DATUM[&quot;undefined&quot;,ELLIPSOID[&quot;undefined&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1,ID[&quot;EPSG&quot;,9001]]]],PRIMEM[&quot;undefined&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],CS[ellipsoidal,2],AXIS[&quot;longitude&quot;,east,ORDER[1],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],AXIS[&quot;latitude&quot;,north,ORDER[2],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>undefined</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>undefined</dd><dt><span>geographic_crs_name :</span></dt><dd>undefined</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCRS[&quot;undefined&quot;,DATUM[&quot;undefined&quot;,ELLIPSOID[&quot;undefined&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1,ID[&quot;EPSG&quot;,9001]]]],PRIMEM[&quot;undefined&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],CS[ellipsoidal,2],AXIS[&quot;longitude&quot;,east,ORDER[1],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]],AXIS[&quot;latitude&quot;,north,ORDER[2],ANGLEUNIT[&quot;degree&quot;,0.0174532925199433,ID[&quot;EPSG&quot;,9122]]]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-8179e20c-7ddf-4ffd-8082-48a5792a3332' class='xr-section-summary-in' type='checkbox'  checked><label for='section-8179e20c-7ddf-4ffd-8082-48a5792a3332' class='xr-section-summary' >Attributes: <span>(7)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>Monthly Average of Daily Maximum Near-Surface Air Temperature</dd><dt><span>units :</span></dt><dd>K</dd><dt><span>grid_mapping :</span></dt><dd>crs</dd><dt><span>standard_name :</span></dt><dd>air_temperature</dd><dt><span>height :</span></dt><dd>2 m</dd><dt><span>cell_methods :</span></dt><dd>time: maximum(interval: 24 hours);mean over days</dd><dt><span>_ChunkSizes :</span></dt><dd>[ 10  44 107]</dd></dl></div></li></ul></div></div>





{:.input}
```python
two_months_conus.shape
```

{:.output}
{:.execute_result}



    (2, 585, 1386)





The default return when you plot an xarray object with multipple years worth of 
data across a spatial extent is a histogram.

{:.input}
```python
two_months_conus.shape
```

{:.output}
{:.execute_result}



    (2, 585, 1386)





When you call `.plot()` on the data, the default plot is a 
histogram representing the range of raster pixel values in your data 
for all time periods (2 months in this case). 

{:.input}
```python
# Directly plot just a single day using time in "years"
two_months_conus.plot()
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_52_0.png" alt = "The default output of .plot() when you have more than a single point location selected will be a histogram. Here the histogram represents the spread of raster data values in the 2 months of modeled historical data.">
<figcaption>The default output of .plot() when you have more than a single point location selected will be a histogram. Here the histogram represents the spread of raster data values in the 2 months of modeled historical data.</figcaption>

</figure>




## Spatial Raster Plots of MACA v2 Climate Data
If you want to plot the data spatially as a raster, you can use 
`.plot()` but specify the lon and lat values as the x and y dimensions 
to plot. You can add the following parameters to your `.plot()` call to 
make sure each time step in your data plots spatially:

`col_wrap=2`: adjust how how many columns the each subplot is spread across
`col=`: what dimension is being plotted in each subplot.

In this case, you want a single raster for each month (time step) in the 
data so you specify `col='time'`. `col_wrap=1` forces the plots to 
stack on top of each other in your `matplotlib` figure. 

{:.input}
```python
# Quickly plot the data using xarray.plot()
two_months_conus.plot(x="lon",
                      y="lat",
                      col="time",
                      col_wrap=1)
plt.suptitle("Two Time Steps of Monthly Average Temp", y=1.03)
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_54_0.png" alt = "Plot showing two months of the historic max temperature climate data plotted using xarray .plot(). If you set col_wrap to 1, then you end up with one column and two rows of subplots.">
<figcaption>Plot showing two months of the historic max temperature climate data plotted using xarray .plot(). If you set col_wrap to 1, then you end up with one column and two rows of subplots.</figcaption>

</figure>




If you set `col_wrap` to `2` you end up with two columns and
one subplot in each column. 

{:.input}
```python
# Plot the data using 2 columns
two_months_conus.plot(x="lon",
                      y="lat",
                      col="time",
                      col_wrap=2)
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_56_0.png" alt = "Plot showing two months of the historic max temperature climate data plotted using xarray .plot(). If you set col_wrap to 2, then you end up with two columns.">
<figcaption>Plot showing two months of the historic max temperature climate data plotted using xarray .plot(). If you set col_wrap to 2, then you end up with two columns.</figcaption>

</figure>




## Plot Multiple MACA v2 Climate Data Raster Files With a Spatial Projection

Below you plot the same data using `cartopy` which support spatial 
projectsion. The `coastlines()` basemap is also added to the plot. 

{:.input}
```python
central_lat = 37.5
central_long = 96
extent = [-120, -70, 20, 55.5]  # CONUS

map_proj = ccrs.AlbersEqualArea(central_longitude=central_lon,
                                central_latitude=central_lat)

aspect = two_months_conus.shape[2] / two_months_conus.shape[1]
p = two_months_conus.plot(transform=ccrs.PlateCarree(),  # the data's projection
                          col='time', col_wrap=1,
                          aspect=aspect,
                          figsize=(10, 10),
                          subplot_kws={'projection': map_proj})  # the plot's projection

plt.suptitle("Two Time Steps of CONUS Historic Temperature Data", y=1)
# Add the coastlines to each axis object and set extent
for ax in p.axes.flat:
    ax.coastlines()
    ax.set_extent(extent)
```

{:.output}
    /opt/conda/lib/python3.8/site-packages/xarray/plot/facetgrid.py:373: UserWarning: Tight layout not applied. The left and right margins cannot be made large enough to accommodate all axes decorations. 
      self.fig.tight_layout()



{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_58_1.png" alt = "Map showing air temperature data on a map using the PlateCarree CRS for two months.">
<figcaption>Map showing air temperature data on a map using the PlateCarree CRS for two months.</figcaption>

</figure>





## Export Raster to Geotiff File 

In the above workflows you converted your data into a DataFrame and exported
it to a .csv file. This approach works well if you only need the summary values
and don't need any spatial information. However sometimes you may need to 
export spatial raster files.

You can export your data to a geotiff file format using rioxarray. 
To do this you will need to:

1. ensure that your xarray object has a crs defined and define it if it's missing.
2. call xarray to export your data

Notice below that your two month subset no long contains CRS information.


{:.input}
```python
# Double check the crs still exist
two_months_conus.rio.crs
```

{:.output}
{:.execute_result}



    CRS.from_wkt('GEOGCS["undefined",DATUM["undefined",SPHEROID["undefined",6378137,298.257223563]],PRIMEM["undefined",0],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AXIS["Longitude",EAST],AXIS["Latitude",NORTH]]')





At the very beginning of this lesson, you saved the crs information 
from the original xarray object. You can use that to export your geotiff data
below.


Once the crs is set, you can export to a geotiff file format.

{:.input}
```python
# Export to geotiff
file_path = "two_months_temp_data.tif"
two_months_conus.rio.to_raster(file_path)
```


Now test our your data. Reimport it using `xarray.open_rasterio`.

{:.input}
```python
# Open the data up as a geotiff
two_months_tiff = xr.open_rasterio(file_path)
two_months_tiff
```

{:.output}
{:.execute_result}



<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt, dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray (band: 2, y: 585, x: 1386)&gt;
[1621620 values with dtype=float32]
Coordinates:
  * band     (band) int64 1 2
  * y        (y) float64 25.06 25.1 25.15 25.19 25.23 ... 49.27 49.31 49.35 49.4
  * x        (x) float64 235.2 235.3 235.3 235.4 ... 292.8 292.9 292.9 292.9
Attributes:
    transform:      (0.04166599094652527, 0.0, 235.207011242808, 0.0, 0.04166...
    crs:            +proj=longlat +ellps=WGS84 +no_defs=True
    res:            (0.04166599094652527, -0.04166600148971766)
    is_tiled:       0
    nodatavals:     (-9999.0, -9999.0)
    scales:         (1.0, 1.0)
    offsets:        (0.0, 0.0)
    descriptions:   (&#x27;Monthly Average of Daily Maximum Near-Surface Air Tempe...
    AREA_OR_POINT:  Area
    cell_methods:   time: maximum(interval: 24 hours);mean over days
    grid_mapping:   crs
    height:         2 m
    long_name:      Monthly Average of Daily Maximum Near-Surface Air Tempera...
    standard_name:  air_temperature
    units:          K
    _ChunkSizes:    [ 10  44 107]</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'></div><ul class='xr-dim-list'><li><span class='xr-has-index'>band</span>: 2</li><li><span class='xr-has-index'>y</span>: 585</li><li><span class='xr-has-index'>x</span>: 1386</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-c66d67b5-4c95-4c46-b758-44686d523568' class='xr-array-in' type='checkbox' checked><label for='section-c66d67b5-4c95-4c46-b758-44686d523568' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>...</span></div><div class='xr-array-data'><pre>[1621620 values with dtype=float32]</pre></div></div></li><li class='xr-section-item'><input id='section-ab0e7e82-8b12-4dd2-93d9-bbee842cb88b' class='xr-section-summary-in' type='checkbox'  checked><label for='section-ab0e7e82-8b12-4dd2-93d9-bbee842cb88b' class='xr-section-summary' >Coordinates: <span>(3)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>band</span></div><div class='xr-var-dims'>(band)</div><div class='xr-var-dtype'>int64</div><div class='xr-var-preview xr-preview'>1 2</div><input id='attrs-51bcf2df-36f7-40be-84b6-4c6af70804c3' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-51bcf2df-36f7-40be-84b6-4c6af70804c3' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-46b122cd-a2f6-4b62-894c-7d84d302c6b5' class='xr-var-data-in' type='checkbox'><label for='data-46b122cd-a2f6-4b62-894c-7d84d302c6b5' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([1, 2])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>y</span></div><div class='xr-var-dims'>(y)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>25.06 25.1 25.15 ... 49.35 49.4</div><input id='attrs-2edcd4ce-4be4-4a85-832b-3dd72f4a8876' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-2edcd4ce-4be4-4a85-832b-3dd72f4a8876' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c59776e9-24a8-4fce-87d8-06bd99bc19c8' class='xr-var-data-in' type='checkbox'><label for='data-c59776e9-24a8-4fce-87d8-06bd99bc19c8' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([25.063078, 25.104744, 25.14641 , ..., 49.312691, 49.354357, 49.396023])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>x</span></div><div class='xr-var-dims'>(x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>235.2 235.3 235.3 ... 292.9 292.9</div><input id='attrs-928db257-a4b2-4863-a45a-7f970db87096' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-928db257-a4b2-4863-a45a-7f970db87096' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-07f37757-7e92-4fd3-a795-39b6fc37d643' class='xr-var-data-in' type='checkbox'><label for='data-07f37757-7e92-4fd3-a795-39b6fc37d643' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([235.227844, 235.26951 , 235.311176, ..., 292.85191 , 292.893576,
       292.935242])</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-37133066-da14-4a6e-be08-e1cac9f94a49' class='xr-section-summary-in' type='checkbox'  ><label for='section-37133066-da14-4a6e-be08-e1cac9f94a49' class='xr-section-summary' >Attributes: <span>(16)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>transform :</span></dt><dd>(0.04166599094652527, 0.0, 235.207011242808, 0.0, 0.04166600148971766, 25.042244925890884)</dd><dt><span>crs :</span></dt><dd>+proj=longlat +ellps=WGS84 +no_defs=True</dd><dt><span>res :</span></dt><dd>(0.04166599094652527, -0.04166600148971766)</dd><dt><span>is_tiled :</span></dt><dd>0</dd><dt><span>nodatavals :</span></dt><dd>(-9999.0, -9999.0)</dd><dt><span>scales :</span></dt><dd>(1.0, 1.0)</dd><dt><span>offsets :</span></dt><dd>(0.0, 0.0)</dd><dt><span>descriptions :</span></dt><dd>(&#x27;Monthly Average of Daily Maximum Near-Surface Air Temperature&#x27;, &#x27;Monthly Average of Daily Maximum Near-Surface Air Temperature&#x27;)</dd><dt><span>AREA_OR_POINT :</span></dt><dd>Area</dd><dt><span>cell_methods :</span></dt><dd>time: maximum(interval: 24 hours);mean over days</dd><dt><span>grid_mapping :</span></dt><dd>crs</dd><dt><span>height :</span></dt><dd>2 m</dd><dt><span>long_name :</span></dt><dd>Monthly Average of Daily Maximum Near-Surface Air Temperature</dd><dt><span>standard_name :</span></dt><dd>air_temperature</dd><dt><span>units :</span></dt><dd>K</dd><dt><span>_ChunkSizes :</span></dt><dd>[ 10  44 107]</dd></dl></div></li></ul></div></div>





Plot the data. Do you notice anything about the values 
in the histogram below?

{:.input}
```python
two_months_tiff.plot()
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_70_0.png" alt = "Histogram showing temperature values for the Continental United States. What do you notice about the range of values?">
<figcaption>Histogram showing temperature values for the Continental United States. What do you notice about the range of values?</figcaption>

</figure>




{:.input}
```python
# Plot the data - this doesn't look right!
two_months_tiff.plot(col="band")
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_71_0.png" alt = "Plot showing unmasked temperature data (containing nodata values) for the Continental United States.">
<figcaption>Plot showing unmasked temperature data (containing nodata values) for the Continental United States.</figcaption>

</figure>




The data above are plotting oddly because there are no data 
values in your array. You can mask those values using `xarray.where()`.

{:.input}
```python
two_months_tiff.rio.nodata
```

{:.output}
{:.execute_result}



    -9999.0





Below you use `.where()` to mask all values in your data that are 
equal to `-9999`, the nodata value for this data.

{:.input}
```python
# Remove no data values and try to plot again
two_months_tiff = xr.open_rasterio(file_path)

two_months_clean = two_months_tiff.where(
    two_months_tiff != two_months_tiff.rio.nodata)

two_months_clean.plot(col="band")
plt.show()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}/images/courses/intermediate-eds-textbook/06-hierchical-data-formats/03-netcdf/2020-10-16-netcdf-03-open-netcdf-climate-data/2020-10-16-netcdf-03-open-netcdf-climate-data_75_0.png" alt = "Plot showing masked temperature data for the Continental United States.">
<figcaption>Plot showing masked temperature data for the Continental United States.</figcaption>

</figure>




Above you learned how to begin to work with NETCDF4 format files containing 
climate focused data. You learned how to open and subset the data by time and location.
In the next lesson you will learn how to implement spatial subsets using shapefiles of your data.

