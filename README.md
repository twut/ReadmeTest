# Basisbestand 3D geluid v0.3
Tools voor het genereren van een nationaal 3D basisbestand geluid

## General
- All intermediary GPKG files that are input to the Geoflow process, need to contain 2D Polygons only (!). Because we use FME for the pre-processing and FME always writes PolygonZ GPKG files, these intermediary files need to be cast to Polygon with GDAL. Or LineString in case of heightlines. For example:

ogr2ogr -overwrite 37ez2_3dbag_2D.gpkg 37ez2_3dbag.gpkg -dim 2 pand_bag -nlt POLYGON

ogr2ogr -f "gpkg" 37hn1_hl.gpkg 37hn1_hl_tmp.gpkg hoogtelijnen_out -nlt LINESTRING

## BGT cleaning

Convert GML to GPKG, discretize CurvePolygon, clip to AHN tile extent (with buffer for heightlines), validate and repair geometry.

0. Download BGT tiles from PDOK

1. Convert the GML to GPKG with GDAL: prepare_bgt.sh

2. runner_bgt_clean_main.fmw

    calls: runner_bgt_clean_sub_01.fmw
    
        calls: bgt_clean.fmw

3. cast the FME GeoPackage output to Polygons with cast_to_polygon.sh

Output: /bgt/cleaned

Optionally, bgt_clean.fmw can be set up to include a buffer for each tile.

## BAG cleaning

Remove and rename attibutes, repair geometry, remove gaps and slivers, remove underground buildings, add objects from the Overigbouwwerk layer from BGT.

0. Download 3D BAG

1. gebouwen/bag_combine_overigebouwerken.fmw

## Bodemvlakken

Input: /bgt/cleaned/*.gpkg and Tile_grid file

Run `runner_bgt_generalize.fmw`.

It calls `bgt_generalize_bodemvlakken.fmw` and all the tiles in the tile grid will be processed.

## Buildings

Input: the cleaned BAG footprints

The LoD1 and LoD1.3 building models were generated with geoflow.

## Terrain

Input: AHN3 point cloud, AHN3 tile index

The TIN is generated with 3dfier, using the `simplification_tinsimp` parameter to adjust the error threshold.
