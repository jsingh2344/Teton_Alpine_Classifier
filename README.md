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

<p>
    <img width="781" height="690" alt="image" src="https://github.com/user-attachments/assets/642c2bef-4161-4555-a8c6-a490937f8398" />
</p>

 Some classes are correctly predicted at a high rate: for example, the model was perfect when predicting the 58 water pixels, and performed fairly well with grass and trees as well. But it was pretty bad at distinguishing ice and bare rock. Since they are visually difficult to differentiate, and Sentinel-2 uses optical data, this makes some sense. 

## Adding Sentinel-1 data

To perhaps remedy this, I added the VV and VH bands of Sentinel-1 images to the model features. Accuracy metrics improved slightly to 
```
Accuracy: 0.7903682719546742
Kappa: 0.748228966873247
```
and the confusion matrix became: 

<p>
    <img width="781" height="690" alt="image" src="https://github.com/user-attachments/assets/cb01e977-b30d-4aa5-b9c7-1de5170f2fc1" />

</p>
Also, here is the Random Forest classification map. Interestingly, it infers a lot more terrain detail than the Dynamic World map, perhaps because Dynamic World uses solely optical data to estimate terrain class.

<p>
    <img width="1028" height="1200" alt="image" src="https://github.com/user-attachments/assets/201a2e30-6f01-4ced-a629-ed7832975c30" />
</p>

A few other notes:
- Interestingly, Random Forest cleaned up most of the erronous alpine lakes that Dynamic World features
- The snow/bare classification is a mess. The model in general seems to prefer snow and ice above a certain elevation, leading much of the alpine tundra/rock to turn into a sea of disconnected snow pixels


## Confidence Masking 

My next optimization was using only Dynamic World pixels with probability above a certain threshold. First, I checked how many pixels were excluded by masking out pixels whose label confidence was below each threshold level:

| Threshold | Kept | Excluded |
|---:|---:|---:|
| 0.5 | 0.5023440635648382 | 0.49765593643516176 |
| 0.6 | 0.40947052639879583 | 0.5905294736012042 |
| 0.7 | 0.27106181748878405 | 0.7289381825112159 |
| 0.75 | 0.0022276530503183025 | 0.9977723469496816 |
| 0.8 | 0.0 | 1.0 |
| 0.85 | 0.0 | 1.0 |
| 0.9 | 0.0 | 1.0 |

```0.7``` looks like the best option, so I recalibrated the model using only dynamic world pixels with label confidences above ```0.7``` (note that I am aggregating probability bands by taking the median across the summer months).

... except that these were the labels I got when I tried to take a sample: ```{'0': 200, '1': 200, '2': 31, '8': 200}```
No bare or shrub pixels made the cut. `0.65` gave a similar result, but `0.60` ran with 200 of each label class. And the result were great!
```
[[58, 0, 0, 0, 0, 0, 0, 0, 0], [0, 49, 0, 0, 0, 0, 0, 0, 0], [0, 1, 57, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 55, 0, 0, 0], [0, 0, 0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0, 0, 55, 3], [0, 0, 0, 0, 0, 0, 0, 9, 47]]
Accuracy: 0.9610778443113772
Kappa: 0.9532610684722117
{'0': 200, '1': 200, '2': 200, '5': 200, '7': 200, '8': 200}
```
, with the following confusion matrix:

<p>
    <img width="781" height="690" alt="image" src="https://github.com/user-attachments/assets/4b6e92a7-d7d0-4f01-add8-4c25ecdb0dbb" />
</p>

Here is the new RF map, in which 60 percent of pixels don't have Dynamic World labels:

<p>
    <img width="857" height="1000" alt="image" src="https://github.com/user-attachments/assets/19edd7cc-0af3-4845-be53-97a880eecbd6" />
</p>

Overlaying this classification map with USGS's National Map Topo (link to that is <a href="https://drive.google.com/file/d/120TCLUTZgQh9pSzX4kxjo5djrROSuL0q/view?usp=sharing"> here </a>) reveals a few things. First, there is a consistent pattern of
snow-classified pixels overestimating snow cover in some spots, and underestimating it in others. Avalanche and Garnet Canyons provide good examples:
<p>
    <img width="2060" height="1260" alt="image" src="https://github.com/user-attachments/assets/124e5c9d-b72d-4ecf-bc5f-71acb5888a08" />
</p>

Higher-elevation regions are receiving higher percentages of snow-classification than the topo would suggest. For starters, Icefloe Lake is completely frozen, and the basins north and south of Avalanche Divide have large estimated snowfields that the topo disagrees with. This pattern is particularly dominant in north-facing slopes, and disappears below around 9000 feet -- note that in particular, Snowdrift Lake is classified as half ice-covered, while Taminah lower in the canyon is classified as ice-free. But given how I've defined my truth values from Dynamic World, and how the USGS Topo likely works, this is a pretty obvious and sensible issue: I'm just estimating the snow cover during the median day of the summer, and topos tend to only plot non-seasonal, permanent icefields that persist through the whole year.

<p>
    <img width="1458" height="1042" alt="image" src="https://github.com/user-attachments/assets/a5553386-38f3-4737-b623-d4e892f64e94" />
</p>

Zooming in on the glacial core of the range, the opposite dynamic emerges. I'm predicting less snow specifically at the tails of glacial snowfields. 
In varying degrees, the ends of Middle Teton Glacier, Teepe Glacier, Teton Glacier, and the large snowfield just northwest of Disappointment Peak's summit (which I'm lumping in here, since it actually shows the second biggest moraine of this group) are classified as bare rock. This is an interesting result of glacial retreat: snow/ice exists in the topo in this discrepancy zone purely because it moved downhill from the accumulation zone, and not because there is a local terrain condition that allows snow to persist there throughout the year. The middle-of-summer classification doesn't overpredict in these zones because had they not existed at the bottom of glaicers, they would never have allowed for year round snow independently. 

Which, incidentally, reaffirms that the Disappointment snowfield is probably about as proper of a 'glacier' as several of the official ones in the Teton Range.

