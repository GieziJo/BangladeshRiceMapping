/**** Start of imports. If edited, may not auto-convert in the playground. ****/
var MODIS = ee.ImageCollection("MODIS/061/MOD09Q1");
/***** End of imports. If edited, may not auto-convert in the playground. *****/
var bb = ee.FeatureCollection("projects/ee-bangladesh-rice-mapping/assets/Shapefiles/Admin/BangladeshBB");

var scale = 231.65635826395828

var means_centered = function (Image, region, scale) {
  var bandNames = Image.bandNames()
  
  // Mean center the data to enable a faster covariance reducer
  // and an SD stretch of the principal components.
  var meanDict = Image.reduceRegion({
      reducer: ee.Reducer.mean(),
      geometry: region,
      scale: scale,
      maxPixels: 10e9
  });
  var means = ee.Image.constant(meanDict.values(bandNames));
  var centered = Image.subtract(means);
  
  return  centered
}
 

// This helper function returns a list of new band names.
var getNewBandNames = function(prefix,bandNames) {
  var seq = ee.List.sequence(1, bandNames.length());
  return seq.map(function(b) {
    return ee.String(prefix).cat(ee.Number(b).int());
  });
};

// This function accepts mean centered imagery, a scale and
// a region in which to perform the analysis.  It returns the
// Principal Components (PC) in the region as a new image.
var getPrincipalComponents = function(centered, scale, region, bandNames) {
  // Collapse the bands of the image into a 1D array per pixel.
  var arrays = centered.toArray();

  // Compute the covariance of the bands within the region.
  var covar = arrays.reduceRegion({
    reducer: ee.Reducer.centeredCovariance(),
    geometry: region,
    scale: scale,
    // crs:'EPSG:32737', // 
    bestEffort:true,
    maxPixels: 1e13
  });

  // Get the 'array' covariance result and cast to an array.
  // This represents the band-to-band covariance within the region.
  var covarArray = ee.Array(covar.get('array'));

  // Perform an eigen analysis and slice apart the values and vectors.
  var eigens = covarArray.eigen();
 
  // This is a P-length vector of Eigenvalues.
  var eigenValues = eigens.slice(1, 0, 1);
  
  // This is a PxP matrix with eigenvectors in rows.
  var eigenVectors = eigens.slice(1, 1);

  // Convert the array image to 2D arrays for matrix computations.
  var arrayImage = arrays.toArray(1);

  // Left multiply the image array by the matrix of eigenvectors.
  var principalComponents = ee.Image(eigenVectors).matrixMultiply(arrayImage);

  // Turn the square roots of the Eigenvalues into a P-band image.
  var sdImage = ee.Image(eigenValues.sqrt())
    .arrayProject([0]).arrayFlatten([getNewBandNames('sd',bandNames)]);

  // Turn the PCs into a P-band image, normalized by SD.
  return principalComponents
    // Throw out an an unneeded dimension, [[]] -> [].
    .arrayProject([0])
    // Make the one band array image a multi-band image, [] -> image.
    .arrayFlatten([getNewBandNames('pc', bandNames)])
    // Normalize the PCs by their SDs.
    .divide(sdImage);
};


// Calculate PCA over bands for a median time range
var getDataForYear = function(dateStart, dateEnd){
  var MODISSubCollection = MODIS.filterDate(dateStart, dateEnd)
  
  MODISSubCollection = MODISSubCollection
  .map(function(image){return image.select(['sur_refl_b01', 'sur_refl_b02'])})
  
  var MODISComp = MODISSubCollection
    .median()
  
  
  
  MODISComp = MODISComp.clip(bb.geometry())
  
  
  MODISComp = means_centered(MODISComp, bb.geometry(), scale)
      
      
  var pca = getPrincipalComponents(MODISComp, scale, bb.geometry(), MODISComp.bandNames())
  
  
  MODISComp = MODISComp
  .addBands(pca)
    
  return MODISComp
}

exports.getDataForYear = getDataForYear