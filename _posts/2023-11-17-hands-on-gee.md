---
layout: post
title:  "Hands On: Getting Started with Google Earth Engine"
categories: [hands-on, GEE]
featured: true
image: https://developers.google.com/static/earth-engine/images/og_social_media.png
comments: false
hidden: false
---

Learn the basics of Google Earth Engine and download a Landsat based vegetation index time series from 1985 to today.




## Basics

* use a chromium based browser (Google Chrome, Brave)
* GEE Web Editor Functionalities
	* Scripts, Docs, Assets
	* Code editor
	* Inspector, Console, Tasks
	* Map


## Assets

* Add a project
* Work in Progress! More and more collaboration features
* New -> Shapefile -> Amtsvenn.zip
* Klick on it to get path

## 01_featurecollections

Basic input and output of featurecollections

```js
var aoi = ee.FeatureCollection("projects/rsforum-tutorial/assets/amtsvenn");

print("Amtsvenn", aoi)
Map.addLayer(aoi, {}, "Amtsvenn")

// define a dictionary
var vis = {color: "red"}
Map.addLayer(aoi, vis, "Amtsvenn in red")
```

Map.addLayer:
* object to visualize
* visualization parameters (as a dictionary)
* name


## 02_imagecollections

* [[ImageCollections]]
* Consist of images
* Images consist of multiple bands
* Images have properties

https://developers.google.com/earth-engine/datasets/catalog/landsat-8
We need Landsat 8 - Collection 2 - Tier 1

* How to read the documentation
	* .this

```js

var aoi = ee.FeatureCollection("projects/rsforum-tutorial/assets/amtsvenn");

// load full image collection: doesnt work, to big
var landsat8 = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
print(landsat8, "Full Landsat")


// built in filter functions
var l8 = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .filterBounds(aoi)
  .filterDate("2021-01-01", "2021-12-01")


print(l8, "Amtsvenn Landsat")
Map.addLayer(l8, {}, "Amtvenn")


// filter functions based on Properties
var l8_cloudfree = l8.filter(ee.Filter.lt('CLOUD_COVER', 30))
print(l8_cloudfree, "Amtsvenn Cloudfree")

// select bands
l8_cloudfree = l8_cloudfree.select("SR_B4", "SR_B3")
print(l8_cloudfree, "Amtsvenn Bands")



// All in one

var l8 = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .filterBounds(aoi)
  .filterDate("2021-01-01", "2021-12-01")
  .filter(ee.Filter.lt('CLOUD_COVER', 30))
  .select("SR_B4", "SR_B3")
```


## 03_bandmath

## .map
write a function with an image as an input
return an image
map the function over the image collection

```js

var aoi = ee.FeatureCollection("projects/rsforum-tutorial/assets/amtsvenn");


// define function for NDWI for one image
var NDWI = function(img){
  var ndwi = img.normalizedDifference(["SR_B5", "SR_B6"]).rename("NDWI")
  return img.addBands(ndwi)
}



// apply the function to all the images with map
var l8 = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .filterBounds(aoi)
  .filterDate("2021-01-01", "2021-12-01")
  .filter(ee.Filter.lt('CLOUD_COVER', 30))
  .select("SR_B5", "SR_B6")
  .map(NDWI)

print(l8)


// visualize first image NDWI
var l8_ndwi = l8.select("NDWI").first()
Map.addLayer(l8_ndwi, {palette: ["red", "white", "green"]}, "NDWI")
```


## 04_visualize_export


* require() and export

```js

var aoi = ee.FeatureCollection("projects/rsforum-tutorial/assets/amtsvenn");


// define function for NDWI for one image
var NDWI = function(img){
  var ndwi = img.normalizedDifference(["SR_B5", "SR_B6"]).rename("NDWI")
  return img.addBands(ndwi)
}



// apply the function to all the images with map
var l8 = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .filterBounds(aoi)
  .filterDate("2021-01-01", "2021-12-01")
  .filter(ee.Filter.lt('CLOUD_COVER', 30))
  .select("SR_B5", "SR_B6")
  .map(NDWI)
  .select("NDWI")


print(l8, "ImageCollection")



// image collection to multiband image
var ndwi_ts = l8.sort("system:time_start").toBands().clip(aoi)
print(ndwi_ts, "Multiband Image")

// composites
var ndwi_median = l8.reduce(ee.Reducer.median()).clip(aoi)
print(ndwi_median, "Median Image")


Map.addLayer(ndwi_median, {palette: ["red", "white", "green"]}, "NDWI Median")


// Export
Export.image.toDrive(
  {image: ndwi_ts,
  scale: 30,
  region: aoi
  }
)

```



## 05_sampling

```js
var aoi = ee.FeatureCollection("projects/rsforum-tutorial/assets/amtsvenn");


// define function for NDWI for one image
var NDWI = function(img){
  var ndwi = img.normalizedDifference(["SR_B5", "SR_B6"]).rename("NDWI")
  return img.addBands(ndwi)
}



// apply the function to all the images with map
var l8 = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .filterBounds(aoi)
  .filterDate("2021-01-01", "2021-12-01")
  .filter(ee.Filter.lt('CLOUD_COVER', 30))
  .select("SR_B5", "SR_B6")
  .map(NDWI)
  .select("NDWI")


print(l8, "ImageCollection")



// image collection to multiband image
var ndwi_ts = l8.sort("system:time_start").toBands().clip(aoi)

// Image to table (extract)
var ndwi_sample = ndwi_ts.sampleRegions(aoi)
print(ndwi_sample, "Sample Regions")

Export.table.toDrive({
  collection: ndwi_sample
})



var l8Chart = ui.Chart.image.series(l8.select('NDWI'), aoi)
  .setChartType('ScatterChart')
  .setOptions({
   title: 'Landsat 8 NDWI at Amtsvenn',
   trendlines: {
     0: {color: 'CC0000'}
   },
   lineWidth: 1,
   pointSize: 3,
  });
print(l8Chart);




```



## Misc thought

Git Backup
	On the cogwheel next to the repo in the code editor is the link for cloning
	To initialize git with google services got to
	https://cloud.google.com/source-repositories/docs/authentication?hl=de





