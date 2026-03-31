# README: NDWI Glacier Lake Processing (2010–2020)

## Overview

This project processes stacked Landsat and Sentinel-2 NDWI datasets (2010–2020) to identify glacial lakes in the HKKH region, compute lake statistics, quantify uncertainty, and merge with lake characteristic data (e.g., area, volume, slope, connectivity). The workflow generates median NDWI, standard deviation, water masks, lake area, and integrates geomorphic and hydrologic parameters for comprehensive analysis.

## Requirements

* Python 3.8+
* Libraries:

  * rasterio
  * rioxarray
  * xarray
  * numpy
  * pandas
  * scipy

## Input Data

* **NDWI raster stack**: multi-year NDWI dataset (2010–2020) in stacked GeoTIFF format.

  * Shape: [time or band, y, x]
  * Can be Landsat (30 m) or Sentinel-2 (10 m) data.
* **Lake characteristic dataset**: CSV or DataFrame containing lake properties such as area, volume, dam slope, connectivity index, and upstream watershed.
* Optional: separate green and NIR bands if computing NDWI from raw imagery.

## Workflow

1. **Load raster stack** using `rioxarray`. Supports both single-band stacks and multi-band files.
2. **Compute NDWI** (if raw bands are used):

   ```
   NDWI = (Green - NIR) / (Green + NIR + 1e-6)
   ```
3. **Stack NDWI images** across years/seasons.
4. **Compute median NDWI** to reduce noise from clouds, snow, and seasonal variability.
5. **Compute standard deviation** as an uncertainty measure.
6. **Generate water mask** using threshold `NDWI > 0.3`.
7. **Label individual lakes** using connected components.
8. **Filter small lakes**: minimum area threshold = 0.002 km².
9. **Compute lake statistics**:

   * Lake ID
   * Area in km²
   * Mean NDWI uncertainty
10. **Merge with lake characteristic dataset** to combine NDWI-derived metrics with geomorphic and hydrologic parameters.
11. **Export results** to CSV for further analysis.

## Pixel Area Considerations

* **Landsat 5/7/8**: 30×30 m → 900 m² per pixel.
* **Sentinel-2**: 10×10 m → 100 m² per pixel.
* If resampling Sentinel-2 to 30 m to match Landsat, use 900 m² per pixel.
* Ensure consistency in pixel resolution when calculating lake areas.

## Output

* **Merged DataFrame** with lake statistics and characteristics (`lake_id`, `area_km2`, `uncertainty`, volume, slope, connectivity, etc.).
* Optional CSV export.
* Water mask raster per year/season (optional).

## Notes

* Ensure the stack contains consistent resolution across sensors if combining Landsat and Sentinel-2.
* Pre-monsoon (March–May) and post-monsoon (September–November) composites can be processed separately to capture seasonal variability.
* NDWI threshold and minimum lake area can be adjusted based on study requirements.

## Example Usage

```python
# Load stack and process NDWI
ndwi_ds = rxr.open_rasterio('NDWI_2010_2020_stack.tif', masked=True).squeeze()
# Compute median, std, water mask, and extract lake stats
# Merge with lake characteristic dataset
lake_df = pd.merge(lake_df, lake_characteristics_df, on='lake_id')
```

## References

* Standard NDWI methodology for glacial lakes.
* Thresholds follow accepted practices for turbid, high-altitude lakes.
* Compatible with multi-temporal composite analysis for GLOF susceptibility studies.
