# Teton Alpine Classifier

Mostly for my benefit -- practicing using Google Earth Engine's Random Forest classification to model terrain types in Wyoming's Teton Range.
For preliminary weak truth values, I am using Dynamic World's terrain classification labels. 

Using this color key: 
```
class_palette = [
    "419bdf",  # 0 water: medium blue
    "397d49",  # 1 trees: dark green
    "88b053",  # 2 grass: light olive green
    "7a87c6",  # 3 flooded vegetation: muted blue-purple
    "e49635",  # 4 crops: orange
    "dfc35a",  # 5 shrub/scrub: yellow-tan
    "c4281b",  # 6 built: red
    "a59b8f",  # 7 bare: gray-beige
    "b39fe1",  # 8 snow/ice: light purple
]
```
Dynamic World maps Jackson Hole like this:
<p>
  <img width="1714" height="2000" alt="image" src="https://github.com/user-attachments/assets/393c3800-fe20-4222-bbd1-36d08e8f5476" />
</p>

Note that I masked out classes 3, 4, and 6 (flooded vegetation, crops, and built) because I am mainly interested in classifying alpine terrain. So those areas are collectively black above. 

Also, note that Dynamic World is wrong in a few places. In particular, it predicts the existence of several nonexistent alpine lakes on the flanks of 
the central Cathedral Group. 

## Dataset and model

To predict class, I am using Sentinel-2 and DEM data. These are the features I used, with the bottom 3 derived purely from a DEM:
```
features = [
    # Sentinel-2 reflectance
    "B2", "B3", "B4",      # blue, green, red
    "B8",                 # near infrared
    "B11", "B12",         # shortwave infrared

    # Spectral indices
    "NDVI",               # vegetation greenness
    "NDWI",               # water / wetness
    "NDSI",               # snow / ice
    "BSI",                # bare soil / rock tendency

    # Terrain
    "elevation",
    "slope",
    "aspect"
]
```
A single composite Sentinel-2 image was produced by taking the median of each reflectance band (B2, B3, B4, B8, B11, B12) across the summer months, and then calculating the spectral indices using those medians. Then, out of a random sample of 1600 pixels across the AOI (200 of each relevant class), 70 percent were sorted into a training set and 30 percent into a validation set. Finally, applying Random Trees classification produced:

```
Accuracy: 0.7506925207756233
Kappa: 0.7006771385139804
```
And the following confusion matrix:

<img width="781" height="690" alt="image" src="https://github.com/user-attachments/assets/642c2bef-4161-4555-a8c6-a490937f8398" />

 Some classes are correctly predicted at a high rate: for example, the model was perfect when predicting the 58 water pixels, and performed fairly well with grass and trees as well. But it was pretty bad at distinguishing ice and bare rock. Since they are visually difficult to differentiate, and Sentinel-2 uses optical data, this makes some sense. 

## Adding Sentinel-1 data

To perhaps remedy this, I added the VV and VH bands of Sentinel-1 images to the model features. Accuracy metrics improved slightly to 
```
Accuracy: 0.7903682719546742
Kappa: 0.748228966873247
```
and the confusion matrix became: 

<img width="781" height="690" alt="image" src="https://github.com/user-attachments/assets/cb01e977-b30d-4aa5-b9c7-1de5170f2fc1" />

Also, here is the Random Forest classification map. Interestingly, it infers a lot more terrain detail than the Dynamic World map, perhaps because Dynamic World uses solely optical data to estimate terrain class.

<img width="1028" height="1200" alt="image" src="https://github.com/user-attachments/assets/201a2e30-6f01-4ced-a629-ed7832975c30" />


