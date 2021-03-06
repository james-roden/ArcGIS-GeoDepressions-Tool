# ArcGIS-GeoDepressions-Tool

Version==ArcGIS 10.3

**Currently investigating issues with ESRI error 99999 in 10.5+**

*Created by James M Roden*

An ArcGIS toolbox to semi-automatically identify and analyse geo-depressions in a bathymetric grid. These tools can be used to aid the discovery of pockmarks on the sea bed, and ultimately the location of working hydrocarbon systems.

[DOWNLOAD](https://github.com/GISJMR/ArcGIS-GeoDepressions-Tool/raw/master/ArcGIS-GeoDepressions-JMR.zip)

## Pockmarks

Pockmarks, or seafloor depressions are often associated with fluid discharge and are regarded as indicators of focused fluid seepage. First detailed by King & Maclean (1970); they suggest that gas from underlying bedrock was released sufficiently enough to put fine grain material into suspension. As this sediment drifts away from the venting point it results in a depression in the sea floor. The shape and size of pockmarks can vary greatly, however there are some relatively common traits. Hovland et al (2002) devised several morphological categories. The most common are:
* A unit pockmark (1-10m in diameter, up to 0.5m deep)
* 'Normal' pockmarks (10-700m in diameter, up to 45m deep)
* Elongate pockmarks (Similar to normal pockmarks except one axis is considerably longer)
This toolbox should facilitate in helping detect potential pockmarks at varying scales depending on the quality and resolution of the input bathymetry. 

## Identify GeoDepression Tool

Identifies depressions in a bathymetric raster using the specified z-value (difference between sink and pour point). The z-value parameter is a threshold for what sinks should be filled in the raster. A sink is a cell in the raster that has no outward flow direction (i.e. a depression). If the sink depth to pour point height is less than the specified z-value, the sink will be identified; otherwise it will be ignored. Ideally this tool will be run several times with differing z-values to capture all possible depressions. For e.g. depressions that are ~5m deep would be ignored if they were located inside a depression that is ~20m deep if the z-value was >20m. A suggested work-flow is to loop over several z-values (e.g. [3, 5, 7, 9, 12, 15]) and using 'Select by Location' in ArcMap remove the larger polygons if they contain polygons from a smaller z-value. At this stage spurious polygons from edge data and/or noise can also be removed. Once happy with the depressions polygon it can be used in the Analyse Tool to get the various statistics that will support any interpretation.

![z-value example](https://github.com/GISJMR/ArcGIS-GeoDepressions-Tool/blob/master/imgs/z-value.png)
*Illustration showing how the z-value affects the geo-depression detection. Depressions where the height between pour point and sink are larger than the specified z-value are ignored.*

### Algorithm Outline:

1. Use ArcGIS Spatial Analysis Fill tool to fill all sinks within z-value range
1. Subtract newly created fill raster from original bathymetry raster layer. Resulting layer is a depressions raster
1. Reclassify depressions raster and convert to polygons
1. Calculate area and remove all depression polygons that fall outside of the specified range. The minimum area is pre-defined as (cellsize*3)² to allow enough raster resolution to delineate a shape

## Analyse GeoDepressions Tool

Analyses z-value polygons from Identify Geo-Depressions tool. Produces polygon set with the following analytic statistics: area, perimeter, major axis, minor axis, eccentricity, azimuth, thinness ratio, diameter-depth ratio, depression depth, and low-level morpholgical characteristics. The tool outputs 3 feature classes: depression polygons, depression polygon centroids, and depression polygon deepest point.

![z-value to output](https://github.com/GISJMR/ArcGIS-GeoDepressions-Tool/blob/master/imgs/example.png)
*Results from Identify GeoDepressions tool (left); Depression polygons, deepest point, and centroid with azimuth symbology output from Analyse GeoDepressions tool (right)*

### Algorithm Outline:

1. Calculate the location of deepest point using ArcGIS Spatial Analysis Zonal Statistics and Raster Calculator
1. Smooth Polygons for better aesthetics using cellsize*3 PAEK smoothing algorithm
1. Calculate Area, Perimeter, Major Axis, Minor Axis, Eccentricity, Azimuth, Thinness Ratio, and Diameter-Depth Ratio for each polygon

![belfast bay](https://github.com/GISJMR/ArcGIS-GeoDepressions-Tool/blob/master/imgs/belfast_bay.png)
*GeoDepression centroids created from 40m resolution multibeam bathymetry data (Bathymetry from GMRT)*

### Understanding the Output

AREA_M (Area in Metres), PERIMETER (Perimeter):
Area and perimeter of the depression polygon.

MAJ_AXIS (Major Axis), MIN_AXIS (Minor Axis), ECC (Eccentricity):
The major and minor axis of the minimum bounding polygon of the depression polygon. The eccentricity is calculated from these two values. These three attributes are used to determine the general shape of the depression.

THIN_RAT (Thinness Ratio):
Thinness ratio is used to define the regularity of the depression polygon. A circle will have a value of 1, whereas the more irregular the shape the lower the value.

DIDP_RAT (Diameter/Depth Ratio):
The ratio between the diameter and depth of the depression polygon.

POCK_DEP (Depression Depth):
Depth of the depression polygon at its deepest point.

AZIMUTH (Azimuth):
Calculated from the direction of the minimum bounding area polygon; is the direction of the depression polygon. Can be used with tidal data.

MORP_CHAR (Morphological Characteristics):
These are based on *my* findings and should not necessarily be used by every user. The parameters for defining a *Potential Geo-feature caused by fluid escape* are depression polygons with a thinness ratio greater than 0.75 and a diamater-depth ratio less than 100. A polygon with a thinness ratio greater than 0.75 shows the depression has regular shape (circular/elongate), consisntent with literature.  The diameter-depth ratio means a steeper incline from sea floor surface to depression depth indicating it's potentially an anomoly in the sea floor opposed to the general ebb and flow of the sea floor.

James M Roden
gisjmr@gmail.com
