# Heat-Klang-Valley – SDS320

An exploratory remote sensing project investigating urban expansion along the Elmina expansion corridor in Klang Valley, Malaysia. The aim is to identify changes in vegetation and built-up areas and subsequently investigate their relationship with land surface temperature.

## Current status (6 October 2026)

The work so far covers the selection of a Landsat comparison pair, a local cloud check within the study area, and an openEO/STAC workflow for loading, exporting, and displaying RGB imagery. Quantitative analyses of urban expansion and temperature have not yet been completed.

| Acquisition date | Satellite | RGB bands (red, green, blue) |
| --- | --- | --- |
| 31 May 2004 | Landsat 5 | `TM_B3`, `TM_B2`, `TM_B1` |
| 17 May 2025 | Landsat 9 | `OLI_B4`, `OLI_B3`, `OLI_B2` |

The acquisitions are approximately 21 years apart and both fall in May. This helps limit seasonal differences; weather conditions and sensor differences still need to be considered in subsequent analyses.

## Study area

Elmina subset in geographic coordinates (WGS84, EPSG:4326):

```python
bbox_list = [101.50, 3.16, 101.54, 3.20]
bbox_stac = {
    "west": 101.50,
    "south": 3.16,
    "east": 101.54,
    "north": 3.20,
}
```

`bbox_list` is used for the STAC search and local cloud check; `bbox_stac` is the dictionary passed to `connection.load_stac(spatial_extent=...)`.

## Data and cloud check

Data source: Landsat Collection 2 Level-2 through the [Microsoft Planetary Computer STAC catalogue](https://planetarycomputer.microsoft.com/api/stac/v1/collections/landsat-c2-l2).

The selection is based on a cloud check using `QA_PIXEL` within the Elmina subset. The project workflow reported **0.0% cloud cover within the ROI** for both selected acquisitions. This differs from cloud cover across the entire Landsat scene. These reported values were not recalculated for this README; their precise interpretation depends on the QA bits used, the handling of invalid pixels, and the denominator used in the calculation.

## Jupyter workflow

Requirements include a Python/Jupyter environment, access to an openEO backend supporting `load_stac`, and an authenticated openEO connection named `connection`. The specific backend and authentication procedure should be documented in the project notebook.

The RGB step requires `openeo`, `rasterio`, `numpy`, and `matplotlib`; additional packages may be needed for the STAC search and ROI cloud check.

1. Connect to the openEO backend and authenticate.
2. Search for Landsat scenes through STAC and check `QA_PIXEL` within the ROI.
3. Load the two selected acquisitions using the same spatial extent.
4. Export the RGB data as GeoTIFFs.
5. Display both images side by side in Jupyter.

The latest agreed loading and export block is:

```python
landsat_url = (
    "https://planetarycomputer.microsoft.com/api/stac/v1/"
    "collections/landsat-c2-l2"
)

old_final = connection.load_stac(
    landsat_url,
    spatial_extent=bbox_stac,
    temporal_extent=["2004-05-31", "2004-06-01"],
    bands=["TM_B3", "TM_B2", "TM_B1"],
    properties={"platform": lambda x: x == "landsat-5"},
)

new_final = connection.load_stac(
    landsat_url,
    spatial_extent=bbox_stac,
    temporal_extent=["2025-05-17", "2025-05-18"],
    bands=["OLI_B4", "OLI_B3", "OLI_B2"],
    properties={"platform": lambda x: x == "landsat-9"},
)

old_final.execute_batch("elmina_2004_05_31_rgb.tif", out_format="GTiff")
new_final.execute_batch("elmina_2025_05_17_rgb.tif", out_format="GTiff")
```

The GeoTIFF filenames are the intended export names; they do not by themselves confirm completed downloads. Creating a data cube also does not confirm successful execution on the backend.

For visualisation, the three RGB bands are read with Rasterio and displayed side by side with Matplotlib. The current approach applies a contrast stretch between the 2nd and 98th percentiles separately for each image. This is for display purposes; separate image stretches do not support direct quantitative comparisons of colour or brightness.

## Next steps

- Visually inspect the RGB images and document the ROI cloud check with the specific STAC item IDs.
- Investigate vegetation changes using NDVI and/or built-up areas using an appropriate classification; load additional spectral bands for these analyses.
- Account for scaling, NoData masks, spatial alignment, and sensor differences in quantitative comparisons.
- Subsequently analyse land surface temperature using suitable thermal products. RGB bands alone do not provide temperature.

## Reproducibility

This README documents the latest status agreed during the project discussion. Full reproduction requires the current notebook, backend configuration without credentials, package versions, STAC item IDs, and an executable definition of the ROI cloud check in the repository. Credentials and tokens must not be published.
