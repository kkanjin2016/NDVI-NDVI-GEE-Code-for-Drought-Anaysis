# NDVI-NDVI-GEE-Code-for-Drought-Anaysis


//Define Study Area
var table = Study_Area_polygon


// Load Sentinel-2 data
var sentinelData = ee.ImageCollection("COPERNICUS/S2_SR")
                      .filterDate('2022-01-01', '2022-04-30') 
                      .filterBounds(table)
                      .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20));

// Calculate NDVI
var ndvi = sentinelData.map(function(image) {
  var ndvi = image.normalizedDifference(['B8', 'B4']).rename('NDVI');
  return image.addBands(ndvi);
});

// Calculate NDWI
var ndwi = sentinelData.map(function(image) {
  var ndwi = image.normalizedDifference(['B3', 'B8']).rename('NDWI');
  return image.addBands(ndwi);
});

// Visualize the results
var ndviParams = {min: -1, max: 1, palette: ['blue', 'white', 'green']};
var ndwiParams = {min: -1, max: 1, palette: ['white', 'yellow', 'green']};



// Clip NDVI and NDWI to the geometry for visualization
var clippedNDVI = ndvi.map(function(image) {
  return image.clip(table);
});
var clippedNDWI = ndwi.map(function(image) {
  return image.clip(table);
});

// Adjusted visualization
Map.addLayer(clippedNDVI.select('NDVI'), ndviParams, 'Clipped NDVI');
Map.addLayer(clippedNDWI.select('NDWI'), ndwiParams, 'Clipped NDWI');


Export.image.toDrive({
  image: clippedNDVI.select('NDVI').toBands().reduce(ee.Reducer.lastNonNull()),
  description: 'Clipped_NDVI010122_043022',
  scale: 10, // Adjust scale to the resolution you need
  region: table,
  fileFormat: 'GeoTIFF',
  formatOptions: {
    cloudOptimized: true
  }
});

// Export the last image of the NDWI collection to Google Drive
Export.image.toDrive({
  image: clippedNDWI.select('NDWI').toBands().reduce(ee.Reducer.lastNonNull()),
  description: 'Clipped_NDWI010122_043022',
  scale: 10, // Adjust scale to the resolution you need
  region: table,
  fileFormat: 'GeoTIFF',
  formatOptions: {
    cloudOptimized: true
  }
});
