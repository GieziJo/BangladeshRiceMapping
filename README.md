# Bangladesh Rice Mapping - Google Earth Engine Code

## Code

Please find the code here:
https://code.earthengine.google.com/?accept_repo=users/bobgiezi/BangladeshRiceMapping

This repository serves as a backup.

## Data

The Data folder contains the data to train the rice mapping algorithm.

Available here on gee:
```javascript
var BangladeshRiceDataPoints = ee.FeatureCollection("projects/ee-bangladesh-rice-mapping/assets/Shapefiles/GroundTruth/BangladeshRiceDataPoints")
var IRRI_AmanRiceMap_2010 = ee.Image("projects/ee-bangladesh-rice-mapping/assets/Rasters/GroundTruth/IRRI_AmanRiceMap_2010")
```