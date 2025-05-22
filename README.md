# Google Earth Engine Lectures
## Lecture by Rocio Cortse Lobos
+ ®️ Register at: https://earthengine.google.com/
+ Choose Get started on top right
+ Example: https://console.cloud.google.com/earth-engine/welcome?inv=1&invt=AbyD6Q&project=ee-ducciorocchini&supportedpurview=project

## Repo
https://github.com/rociobeatrizc/GEE?tab=readme-ov-file

## Data catalogue
+ https://developers.google.com/earth-engine/datasets?hl=it
+ Categories: https://developers.google.com/earth-engine/datasets/categories?hl=it
+ Al datasets: https://developers.google.com/earth-engine/datasets/catalog?hl=it

## Sentinel data
+ Sentinel: https://developers.google.com/earth-engine/datasets/catalog/sentinel?hl=it
+ Multispectral: https://developers.google.com/earth-engine/datasets/catalog/sentinel-2?hl=it
+ Surface reflectance: https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED
+ Dataset Availability 2017-03-28T00:00:00Z–2025-05-22T06:06:06.360000Z
+ Resolution: https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED#bands
+ At the end: Open in Code Editor

<img width="1728" alt="Screenshot 2025-05-22 at 14 49 44" src="https://github.com/user-attachments/assets/a33df403-ff81-4bf3-81a0-79c1ae90a322" />

+ Click: "I am authorised"

Make an AOI to the left and change in the code from geometry to AOI.

Then you can make a median with the following code also selecting the period:

``` javascript

// ==============================================
// Sentinel-2 Surface Reflectance - Cloud Masking and Visualization
// https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED
// ==============================================

// ==============================================
// Function to mask clouds using the QA60 band
// Bits 10 and 11 correspond to opaque clouds and cirrus
// ==============================================
function maskS2clouds(image) {
  var qa = image.select('QA60');
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;

  // Keep only pixels where both cloud and cirrus bits are 0
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
               .and(qa.bitwiseAnd(cirrusBitMask).eq(0));

  // Apply the cloud mask and scale reflectance values (0–10000 ➝ 0–1)
  return image.updateMask(mask).divide(10000);
}

// ==============================================
// Load and Prepare the Image Collection
// ==============================================

// Load Sentinel-2 SR Harmonized collection (atmospherical correction already done)
var collection = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
                   .filterBounds(aoi)
                   .filterDate('2025-05-01', '2025-06-30')              // Filter by date   
                   .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20)) // Only images with <20% cloud cover
                   .map(maskS2clouds);                                  // Apply cloud masking

// Print number of images available after filtering
print('Number of images in collection:', collection.size());

// ==============================================
// Create a median composite from the collection
// Useful when the AOI overlaps multiple scenes or frequent cloud cover
// ==============================================
var composite = collection.median().clip(aoi);

// ==============================================
// Visualization on the Map
// ==============================================

Map.centerObject(aoi, 10); // Zoom to the AOI

// Display the first image of the collection (GEE does this by default)
Map.addLayer(collection, {
  bands: ['B4', 'B8', 'B2'],  // True color: Red, Green, Blue
  min: 0,
  max: 0.3
}, 'First image of collection');

// Display the median composite image
Map.addLayer(composite, {
  bands: ['B4', 'B8', 'B2'],
  min: 0,
  max: 0.3
}, 'Median composite');
```

<img width="1728" alt="Screenshot 2025-05-22 at 15 41 55" src="https://github.com/user-attachments/assets/592114c4-7302-4e85-94f0-8320c617a973" />

Then you can export adding the following code:

``` javascript

// ==============================================
// Export to Google Drive
// ==============================================

// Export the median composite
Export.image.toDrive({
  image: composite.select(['B4', 'B3', 'B2']),  // Select RGB bands
  description: 'Sentinel2_Median_Composite',
  folder: 'GEE_exports',                        // Folder in Google Drive
  fileNamePrefix: 'sentinel2_median_2020',
  region: aoi,
  scale: 10,                                    // Sentinel-2 resolution
  crs: 'EPSG:4326',
  maxPixels: 1e13
});
```

Click on tasks in the orange button to the right console. Click on RUN.
While running the task is grey in the console with a spinning wheel.
Check it in DRIVE!








