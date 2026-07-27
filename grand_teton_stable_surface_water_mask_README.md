# Grand Teton stable surface-water mask

## Files

- `grand_teton_stable_surface_water_mask_90pct.tif`: binary mask (`1` = stable
  surface water, `0` = other land, `255` = outside the NPS boundary).
- `grand_teton_jrc_water_occurrence_1984_2021.tif`: source occurrence values
  clipped to the park (`0`–`100` percent, `255` = no data/outside).
- `grand_teton_nps_boundary.geojson`: official NPS legislative boundary used
  for clipping.

## Definition

Stable surface water is defined here as a JRC Global Surface Water v1.4
occurrence value greater than or equal to 90 percent during 1984–2021.

The threshold is conservative enough to exclude most seasonal and episodic
water while tolerating a small number of missed or cloud-contaminated
observations. Change the threshold to `100` if an exact permanent-water mask is
required.

## Spatial properties

- CRS: EPSG:4326 (WGS 84)
- Pixel size: 0.00025 degrees (approximately 20 × 28 m at Grand Teton)
- Raster dimensions: 2,102 columns × 2,184 rows
- Extent: -110.94825, 43.53800, -110.42275, 44.08400

## Sources

- European Commission Joint Research Centre / Google, Global Surface Water
  v1.4 occurrence, 1984–2021:
  https://global-surface-water.appspot.com/download
- National Park Service Land Resources Division, authoritative NPS boundary:
  https://services1.arcgis.com/fBc8EJBxQRMcHlei/arcgis/rest/services/NPS_Land_Resources_Division_Boundary_and_Tract_Data_Service/FeatureServer/2

