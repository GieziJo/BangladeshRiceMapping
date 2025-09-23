var bb = ee.FeatureCollection("projects/ee-bangladesh-rice-mapping/assets/Shapefiles/Admin/BangladeshBB");


var ImageData = require("users/bobgiezi/BangladeshRiceMapping:Helpers/CofactorsForYearAsImage")
var TableData = require("users/bobgiezi/BangladeshRiceMapping:Helpers/ImagesToTableMergeRice")

var riceData = TableData.dataSet

// Remove one year for validation
var validationYear = 2020 
var riceDataValidation = riceData.filter(ee.Filter.eq('Year',validationYear))
riceData = riceData.filter(ee.Filter.neq('Year',validationYear))

// Choose which bands to include in the RF algorithm
var RFBands = ['EVIQ95', 'EVIQ05', 'EVIMedian', 'elevation', 'slope', 'pc1', 'pc2']


// create random forest classifier and exclude test location (leave-one-out cross-validation)
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
  
  return classifier
}

// classify data based on each leave one out classifier, then apply majority voting, i.e. if 2 ouf the 3 classifiers classify as rice, accept result
var majorityVotingImage = function(data, classifierBarisal, classifierKurigram, classifierRajshahi){
  var classificationBarisal = data.classify(classifierBarisal)
  var classificationKurigram = data.classify(classifierKurigram)
  var classificationRajshahi = data.classify(classifierRajshahi)
  
  var classification = ee.ImageCollection([classificationBarisal, classificationKurigram, classificationRajshahi]).sum().gte(2).multiply(1)
  
  return classification
}

// Create classifier for each leave one out district
var classifierBarisal = getClassifierForLocation('Barisal')
var classifierKurigram = getClassifierForLocation('Kurigram')
var classifierRajshahi = getClassifierForLocation('Rajshahi')

// Loop through years to be inferred
for(var year = 2002; year < 2023; year++){
  
  // get image with data for year
  var dataYear = ImageData.getImagesForYear(year)
  
  // classify image with majority voting/RF scheme
  var classification = majorityVotingImage(dataYear, classifierBarisal, classifierKurigram, classifierRajshahi)
  
  // Export image
  Export.image.toAsset({
    image: classification,
    scale: 250,
    region: bb.geometry(),
    assetId: 'projects/BangladeshRiceMapping/assets/InferredData/'+year,
    description: year
  })
}



