/**** Start of imports. If edited, may not auto-convert in the playground. ****/
var fabdem = ee.ImageCollection("projects/sat-io/open-datasets/FABDEM"),
    EVI = ee.ImageCollection("MODIS/061/MOD13Q1");
/***** End of imports. If edited, may not auto-convert in the playground. *****/
var modisData = require("users/bobgiezi/BangladeshRiceMapping:Helpers/MODISDataPCA")

var scale = EVI.first().projection().nominalScale().getInfo()
var crs = EVI.first().projection().crs().getInfo()

var fabdemscale = fabdem.first().projection().nominalScale().getInfo()
var fabdemcrs = fabdem.first().projection().crs().getInfo()

var getImagesForYear = function(Year){
  
  // Define date range
  var date = ee.Date.parse('yyyy', ee.Number(Year).format())
              .advance(9, 'Month')
  var endDate = date.advance(8, 'Month')
  
  // Filter EVI data for date and and select evi band
  var EVIDate = EVI.filterDate(date, endDate)
  EVIDate = EVIDate.select('EVI')
  
  // Calculate EVI stats over date range
  var mean = EVIDate.mean().rename('EVIMean')
  var median = EVIDate.median().rename('EVIMedian')
  var quant = EVIDate.reduce(ee.Reducer.percentile([5,25,75,95])).rename(['EVIQ05','EVIQ25','EVIQ75','EVIQ95'])
  var maxval = EVIDate.max().rename('EVIMax')
  var minval = EVIDate.min().rename('EVIMin')
  var sumval = EVIDate.sum().rename('EVISum')
  
  // Calculate terrain indices
  var fabdemmos = fabdem.mosaic().setDefaultProjection({crs:fabdemcrs, scale:fabdemscale})
  var fabdemMosaic = fabdemmos.rename(['elevation']).reproject({crs:crs, scale:scale})
  
  var slope = ee.Terrain.slope(fabdemmos).multiply(ee.Number.expression('Math.PI')).divide(180.0).tan().rename('slope')
  
  // Get MODIS PCA for date range
  var modisImage = modisData.getDataForYear(date, endDate)
  
  // concatenate all images togther and create final multiband image
  var allvals = mean.addBands(median).addBands(quant).addBands(maxval).addBands(minval).addBands(sumval).addBands(fabdemMosaic).addBands(slope).addBands(modisImage)
  
  
  return allvals
}



exports.getImagesForYear = getImagesForYear