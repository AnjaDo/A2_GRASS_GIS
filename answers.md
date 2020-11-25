# Assignment 2: GIS analyses using GRASS GIS


## 1. Create new location in GRASS GIS


Creating new location with Info from GHS_POP_E2015_GLOBE_R2019A_54009_250_V1_0_18_3.tif

```
gdalinfo GHS_POP_E2015_GLOBE_R2019A_54009_250_V1_0_18_3.tif
gives us World Mollweide = EPSG:54009
```

## 2. Import data with `v.import`, `v.in.ogr`, `r.import` or `r.in.ogr`.


### 2.1 Import Motorways vectorlayer

```
v.import input=/Users/anjadoppelmayr/Desktop/Fossgis/Assignment2/A2_GRASS_GIS/data motorways.shp layer=motorways output=motorways
```

### 2.2 Import Administrative Districts of Baden-Württemberg

Importing only districts of Baden-Württemberg (ID_1=1) to a temporal location
```
v.in.ogr input=/Users/anjadoppelmayr/Desktop/Fossgis/Assignment2/A2_GRASS_GIS/data/gadm28_adm2_germany.shp output=GADM where=ID_1=1 location=temp
```

Projecting and loading the data into the Permanent mapset
```
v.proj location=temp mapset=PERMANENT input=GADM output=GADM
```

### 2.3 Importing Global Human Settlement Layer

```
r.import input=/Users/anjadoppelmayr/Desktop/Fossgis/Assignment2/A2_GRASS_GIS/data/GHS_POP_E2015_GLOBE_R2019A_54009_250_V1_0_18_3/GHS_POP_E2015_GLOBE_R2019A_54009_250_V1_0_18_3.tif output=HSLA
```

## 3. Calculate the total population of the districts


### 3.1 Set the region

Setting the Region to GADM and checking if the resolution of the region is 250 by 250 meters.
```
g.region vector=GADM@PERMANENT
g.region -p
```

### 3.2 Rasterizing the districts

For the cells to contain OBJECTID _attr_ is selected and the referring column.

```
v.to.rast input=GADM@PERMANENT output=GADM_Rast use=attr attribute_column=OBJECTID

```

### 3.3 Calculate the population of each district

With the new raster file as base and the Global Human Settlement Layer as cover, the population is summed up.
```
r.stats.zonal base=GADM_Rast@PERMANENT cover=HSLA@PERMANENT method=sum output=Pop
```
### 3.4. Evaluate the population estimate

Using the "Query raster/vector maps" tool we can derive the following numbers:

| Districts     | Calculated     | Official |
| :-----------: |:--------------:| :-------:|
| Heidelberg    | 150.007        | 160.355  |
| Baden-Baden   | 520.47,47      | 55.123   |
| Ludwigsburg   | 522.012,2      | 545.423  |

The (recent) official numbers are slightly higher than the calculated population numbers. This can be due to the Global Human Settlement Layer, which provides only estimates up until 2015. [GHSL Data] (https://ghsl.jrc.ec.europa.eu/download.php?ds=pop)

## 4. Calculate total population living within 1km of motorways


Checking _map units_ to know what to insert in _distance_ in buffer calculation
```
g.proj -p 
```

Creating a buffer around Motorway-lines with a distance of 1km
```
v.buffer input=motorways@PERMANENT type=line output=motorways_buffer distance=1000
```

Checking current region and setting region to buffered motorways layer
```
g.region -p
g.region vector=motorways_buffer@PERMANENT
```

Creating raster from vector layer by using _val_ to assign cells a value
```
v.to.rast input=motorways_buffer@PERMANENT output=motorways_buff_raster use=val 
```

With the motorways buffer as base and Human Settlement Layer as cover summing up the population within 1km of the motorways
```
r.stats.zonal base=motorways_buff_raster@PERMANENT cover=HSLA@PERMANENT method=sum output=Pop_within_1km_motorway
```

Printing population value
```
r.stats -c input=Pop_within_1km_motorway@PERMANENT
```

**Result** = 304012.125