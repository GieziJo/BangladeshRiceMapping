/**** Start of imports. If edited, may not auto-convert in the playground. ****/
var irriVis = {"opacity":1,"bands":["IsRice"],"palette":["ffffff","26b31d"]},
    rfVis = {"opacity":1,"bands":["classification"],"palette":["ffffff","27dc5f"]};
/***** End of imports. If edited, may not auto-convert in the playground. *****/
var bb = ee.FeatureCollection("projects/ee-bangladesh-rice-mapping/assets/Shapefiles/Admin/BangladeshBB")
var IRRI2010 = ee.Image("projects/ee-bangladesh-rice-mapping/assets/Rasters/GroundTruth/IRRI_AmanRiceMap_2010")
var BangladeshBoundaries = ee.FeatureCollection("projects/ee-bangladesh-rice-mapping/assets/Shapefiles/Admin/BangladeshBoundary")


Map.centerObject(bb)

var ImageData = require("users/bobgiezi/BangladeshRiceMapping:Helpers/CofactorsForYearAsImage")
var TableData = require("users/bobgiezi/BangladeshRiceMapping:Helpers/ImagesToTableMergeRice")

var riceData = TableData.dataSet

// Remove one year for validation
var validationYear = 2020 
var riceDataValidation = riceData.filter(ee.Filter.eq('Year',validationYear))
riceData = riceData.filter(ee.Filter.neq('Year',validationYear))

// Choose which bands to include in the RF
var RFBands = ['EVIQ95', 'EVIQ05', 'EVIMedian', 'elevation', 'slope', 'pc1', 'pc2']

var printMetrics = function(riceDataTestClassifiction){
  var errorMatrix = riceDataTestClassifiction.errorMatrix({
    actual: 'IsRice',
    predicted: 'classification'
  });
  
  var results = {
        "Overall accuracy": errorMatrix.accuracy(),
        "Consumer's accuracy": errorMatrix.consumersAccuracy(),
        "Producer's accuracy": errorMatrix.producersAccuracy(),
        "Kappa": errorMatrix.kappa(),
        "Fscore": errorMatrix.fscore()
  }
  return results
}

var allResults = {}

var getClassifierForLocation = function(testLocation){

  var riceDataTrain = riceData.filter(ee.Filter.neq('Location',testLocation))
   
  var classifier = ee.Classifier
                .smileRandomForest(
                    {numberOfTrees :1000,
                    minLeafPopulation:5})
                .train({
                      features: riceDataTrain,
                      classProperty: 'IsRice',
                      inputProperties: RFBands})
  
  
  
  var riceDataTest = riceData.filter(ee.Filter.eq('Location',testLocation))
  var results = printMetrics(riceDataTest.classify(classifier))
  allResults[testLocation] = results
  
  return classifier
}

var majorityVotingImage = function(data, classifierBarisal, classifierKurigram, classifierRajshahi){
  var classificationBarisal = data.classify(classifierBarisal)
  var classificationKurigram = data.classify(classifierKurigram)
  var classificationRajshahi = data.classify(classifierRajshahi)
  
  var classification = ee.ImageCollection([classificationBarisal, classificationKurigram, classificationRajshahi]).sum().gte(2).multiply(1)
  
  return classification
}


var classifierBarisal = getClassifierForLocation('Barisal')
var classifierKurigram = getClassifierForLocation('Kurigram')
var classifierRajshahi = getClassifierForLocation('Rajshahi')


var renameElements = function(fc,oldName,newName){return fc.map(function(element){return element.set(newName, element.get(oldName))})}

var testClassification = renameElements(renameElements(riceDataValidation.classify(classifierBarisal), 'classification', 'classification_0').classify(classifierKurigram), 'classification', 'classification_1').classify(classifierRajshahi)
testClassification = testClassification.map(function(element){
  return element.set('classification', ee.Number(element.get('classification')).add(ee.Number(element.get('classification_0'))).add(ee.Number(element.get('classification_1'))).gte(2).multiply(1))
})


var results = printMetrics(testClassification)

allResults['Test'] = results

print(allResults)


var dataYearValidationMap = ImageData.getImagesForYear(2010).clip(bb.geometry())

var classificationValidationMap = majorityVotingImage(dataYearValidationMap, classifierBarisal, classifierKurigram, classifierRajshahi)

var mask = IRRI2010.neq(0)
IRRI2010 = IRRI2010.updateMask(mask)
IRRI2010 = IRRI2010.neq(6).multiply(1)
IRRI2010 = IRRI2010.rename('IsRice')
IRRI2010 = IRRI2010.clip(BangladeshBoundaries)
Map.addLayer(IRRI2010, irriVis, 'IRRI Rice Map 2010')
Map.addLayer(classificationValidationMap.clip(BangladeshBoundaries), rfVis, 'RF Classification 2010')