# Project Results Documentation

## 📌 Introduction

This repository contains the results of data extraction from the [HydroSHEDS](https://www.hydrosheds.org/) database.
The HydroSHEDS database provides raster and shapefile datasets for global hydrology, typically used in GIS applications.

The purpose of this extraction and processing was to convert these spatial datasets into **matrix formats** (double-entry grids organized by latitude and longitude).
In other words, the project transforms geospatially projected values into tabular matrix data, enabling easier integration and analysis outside GIS software.

---

## 🌍 Processed Datasets

### 🔹 Rasters

* **Elevation**

  * Original resolution: 15 arc-seconds (\~450 m at the equator).
  * Aggregated to 0.01° resolution by computing cell means.

* **Flow direction, flow accumulation, and mask**

  * Original resolution: 30 arc-seconds (\~900 m at the equator).
  * Extracted directly at this resolution (no upscaling possible).

* **Surface classification**

  * Original resolution: 15 arc-seconds (\~450 m at the equator).
  * Extracted directly at this resolution (no upscaling possible).
  * For this extraction, the product used out of the database is the GLWD\_combined\_classes.

---

### 🔹 Shapefiles

* **Watersheds**

  * Derived from the Pfafstetter coding system (see HydroBASINS documentation).
  * Multiple hierarchical levels were considered.
  * Global grids were created at resolutions of **0.01°**, **0.1°**, **0.25°**, and **0.5°**.
  * For each grid cell, the most frequent watershed ID was recorded in the final matrix.
  * Each resolution corresponds to a different Pfafstetter level:

    * 0.01° → level 9
    * 0.1° → level 7
    * 0.25° → level 5
    * 0.5° → level 3
  * Attribute tables of HydroBASINS and BasinATLAS were joined and exported as CSV files for each level of extraction.

* **Lakes**

  * Lake IDs were extracted for each 0.01° grid cell.
  * If multiple lakes occurred within a single cell, IDs were concatenated in the format:
    `XXXXXXXX | YYYYYYYY`
  * The attribute tables of HydroLAKES, LakeATLAS and LakeTEMP(Aggregated) were joined and exported to CSV.

* **Rivers**

  * Source data is polyline-based, where each river segment is treated as an independent object.
  * Extracting lists of IDs per 0.01° grid cell was impractical.
  * Instead, the original attribute table of HydroRIVERS was joined to RiverATLAS and exported, with additional attributes for **start and end coordinates** of each river segment.

---

## ⚠️ Important Notes

### Coordinate Grids (Latitude/Longitude)

Datasets extracted at **0.01° resolution** were all generated from the same underlying grid.

* Their latitude and longitude coordinates are therefore perfectly aligned, allowing seamless comparison between matrices (e.g., elevation, rivers, lakes, watersheds).

In contrast, datasets extracted at their **original resolutions** (15 or 30 arc-seconds) may not share identical coordinates:

* For example, flow direction, flow accumulation and surface class rasters preserve their native grid.
* This means that **direct cell-to-cell alignment across products is not guaranteed** at those resolutions.
  Users should exercise caution when cross-referencing these rasters.

### Zone 35 (Far East Asia)

Zone **35** is a special case because it covers the **easternmost part of Asia**, which also appears at the western edge of global maps.
As a result, extractions for this zone can appear in two possible formats:

* A **single matrix** spanning longitudes from -180° to 180°, with a **gap** for longitudes outside zone 35.
* Or **two separate matrices**, one for the western fragment and one for the eastern fragment.

⚠️ Users should be aware of this behavior when handling results from zone 35.

---

## 🔗 Cross-referencing Between Datasets

* In the attribute tables for **rivers** and **lakes**, you will find the field `HYBAS_L12`.
* This attribute corresponds to the watershed identifier at **Pfafstetter level 12** in which the feature is located.
* As a result, rivers, lakes, and watershed polygons can be cross-referenced through this identifier.

---

## 📂 Folder Organization

The extracted datasets are organized both **by data type** and **by geographical zone**.

* Global 0.01° grids are too large to cover the entire globe, and would not be meaningful for the analysis.

* Therefore, the **Pfafstetter watershed division** was used as a basis for spatial partitioning.

  * At the **second level**, each continent is subdivided into up to 9 zones based on watershed boundaries.

* As a result, extractions were conducted for **61 distinct zones** worldwide.

* **results/**
  Main output folder.

  * Contains **9 subfolders** numbered `1` to `9`, each representing a major study division.
  * A PDF map of the 61 zones.
  * Each of these folders includes:

    * Subfolders for each sub-zone (e.g., `11`, `12`, …).
    * A PDF map of all corresponding sub-zones.
  * Within each sub-zone folder, you will find:

    * The result folders for that specific sub-zone.
    * A PDF map of the sub-zone itself.

* **Data subfolders (per sub-zone)**
  Each sub-zone contains results organized by data type.
  The following folders are created systematically:

  * `XX_elevation_avg/` → elevation rasters averaged to 0.01°
  * `XX_flow_accumulation/` → flow accumulation matrices
  * `XX_flow_direction/` → flow direction matrices
  * `XX_lakes_ID_list/` → list of lake identifiers per grid cell
  * `XX_land_mask/` → land mask grids
  * `XX_rivers/` → river network attribute tables
  * `XX_surface_class/` → surface classification matrices
  * `XX_watershed_ID/` → watershed identifiers

  Inside each of these folders you will find:

  * `Value_zone_XX.csv` → the extracted matrix
  * `Value_attributes_zone_XX.csv` → the corresponding attribute table (if applicable)

  **Special case – Watersheds:**
  Since watershed data were extracted at multiple resolutions, the folder `XX_watershed_ID/` contains four subfolders:

  * `Resolution_001/`
  * `Resolution_01/`
  * `Resolution_025/`
  * `Resolution_05/`

  ⚠️  The watershed attribute tables (`Value_attributes_zone_XX.csv`) have been exported for **all existing basins** in the HydroSHEDS source data.
  However, not all basins are represented in the extracted matrices:

  * Small basins whose surface area was too limited to cover an entire grid cell were excluded during the matrix extraction process.
  * As a result, **some basins listed in the attribute tables may not appear in the corresponding matrix**.
    This is expected and does not indicate a processing error.

📂 For a complete and detailed overview of the folder hierarchy, please refer to the file `folder_structure.txt`, located in the same directory as this README.

---

## 📑 Documentation

At the same level as the `results` folder, you will find a dedicated documentation directory containing:

* The original HydroSHEDS documentation.
* An Excel file listing all available attributes with their units for each source dataset.
* A Word document explaining the construction of `HYBAS_ID` and `PFAF_ID` (watershed identifiers).
* A schematic diagram illustrating the relationships between original datasets, derived products, and resolutions.

In addition, you will also find a folder:

* **Scripts\_extraction/**

  * This folder contains the Python scripts used for data extraction on **Google Colab**.
  * They are designed to be executed within Colab, connected to a Google Drive.
  * Each script includes detailed comments describing its purpose and usage.

---

## 📧 Contact

For any question about this dataset, please contact:
**Raphaëlle Camphuis** – \[[raphaelle.camphuis@gmail.com](mailto:raphaelle.camphuis@gmail.com)]