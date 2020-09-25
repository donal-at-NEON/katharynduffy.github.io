# NEON AOP

## Calculating Forest Structural Diversity Metrics from NEON LiDAR Data

> Contributors: Jeff Atkins, Keith Krause, Atticus Stovall
> Authors: Elizabeth LaRue, Donal O'Leary


## Learning Objectives
After completing this tutorial, you will be able to:

* Read a NEON LiDAR file (laz) into R
* Visualize a spatial subset of the LiDAR tile
* Correct a spatial subset of the LiDAR tile for topographic varation
* Calculate 13 structural diversity metrics described in <a href="https://doi.org/10.3390/rs12091407" target="_blank">LaRue, Wagner, et al. (2020) </a>


### R Libraries to Install:

* **lidR**: `install.packages('lidR')`
* **gstat**: `install.packages('gstat')`

**Important Note:**
If you have R version 3.6 or above you'll need to update `data.table`:

```
data.table::update.dev.pkg()
```


### Data to Download

For this tutorial, we will be using two .laz files containing NEON 
AOP point clouds for 1km tiles from the <a href="https://www.neonscience.org/field-sites/field-sites-map/HARV" target="_blank">Harvard Forest (HARV)</a> and 
<a href="https://www.neonscience.org/field-sites/field-sites-map/TEAK" target="_blank">Lower Teakettle (TEAK)</a> sites.

<a href="https://drive.google.com/open?id=1Aemh0IVKvC-LoMj2AXr9k8rDQk77l8k7" target="_blank"> **Link to download .laz files on Google Drive Here.**</a>


### Recommended Skills

For this tutorial, you should have an understanding of Light Detection 
And Ranging (LiDAR) technology, specifically how discrete return lidar 
data are collected and represented in las/laz files. For more information 
on how lidar works, please see this 
<a href="https://www.youtube.com/watch?v=EYbhNSUnIdU" target="_blank"> primer video </a>
and
<a href="https://www.neonscience.org/intro-lidar-r-series" target="_blank"> Introduction to Lidar Tutorial Series.</a>

</div> 

## Introduction to Structural Diversity Metrics
Forest structure influences many important ecological processes, including 
biogeochemical cycling, wildfires, species richness and diversity, and many 
others. Quantifying forest structure, hereafter referred to as "structural 
diversity," presents a challenge for many reasons, including difficulty in 
measuring individual trees, limbs, and leaves across large areas. In order 
to overcome this challenge, today's researchers use Light Detection And 
Ranging (LiDAR) technology to measure large areas of forest. It is also 
challenging to calculate meaningful summary metrics of a forest's structure 
that 1) are ecologically relevant and 2) can be used to compare different 
forested locations. In this tutorial, you will be introduced to a few tools 
that will help you to explore and quantify forest structure using LiDAR data 
collected at two field sites of the 
<a href="https://www.neonscience.org/" target="_blank"> 
National Ecological Observatory Network</a>. 

## NEON AOP Discrete Return LIDAR
<a href="https://www.neonscience.org/data-collection/airborne-remote-sensing" target="_blank"> The NEON Airborne Observation Platform (AOP) </a>. 
has several sensors including discrete-return LiDAR, which is useful for measuring forest structural diversity that can be summarized into four categories of metrics: (1) canopy height, (2) canopy cover and openness, and (3) canopy heterogeneity (internal and external), and (4) vegetation area.

We will be comparing the structural diversity of two NEON sites that vary in their structural characteristics. 

First, we will look at <a href="https://www.neonscience.org/field-sites/field-sites-map/HARV" target="_blank">Harvard Forest (HARV)</a>, which is located in Massachusetts. It is a lower elevation, mixed deciduous and evergreen forest dominated by Quercus rubra, Acer rubrum, and Aralia nudicaulis.

Second, we will look at <a href="https://www.neonscience.org/field-sites/field-sites-map/TEAK" target="_blank">Lower Teakettle (TEAK)</a>, which is a high elevation forested NEON site in California. TEAK is an evergreen forest dominated by Abies magnifica, Abies concolor, Pinus jeffreyi, and Pinus contorta. 

As you can imagine, these two forest types will have both similarities and differences in their structural attributes. We can quantify these attributes by calculating several different structural diversity metrics, and comparing 
the results.

## Loading the LIDAR Products
To begin, we first need to load our required R packages, and set our working 
directory to the location where we saved the input LiDAR .laz files that can be 
downloaded from the NEON Data Portal.


```r
library(lidR)
```

```
## Loading required package: raster
```

```
## Loading required package: sp
```

```r
library(gstat)
library(data.table)
```

```
## 
## Attaching package: 'data.table'
```

```
## The following object is masked from 'package:raster':
## 
##     shift
```

```r
library(kableExtra)
```

Next, we will read in the LiDAR data using the `lidR::readLAS()` function. 
Note that this function can read in both `.las` and `.laz` file formats.


```r
# Read in LiDAR data 
#2017 1 km2 tile .laz file type for HARV and TEAK

#Watch out for outlier Z points - this function also allows for the
#ability to filter outlier points well above or below the landscape
#(-drop_z_blow and -drop_z_above). See how we have done this here 
#for you.

HARV <- lidR::readLAS('/Users/kdw223/Research/katharynduffy.github.io/data/NEON_D01_HARV_DP1_727000_4702000_classified_point_cloud_colorized.laz',filter = "-drop_z_below 150 -drop_z_above 325")

TEAK <- lidR::readLAS('/Users/kdw223/Research/katharynduffy.github.io/data/NEON_D17_TEAK_DP1_316000_4091000_classified_point_cloud_colorized.laz',filter = "-drop_z_below 1694 -drop_z_above 2500")
```

Let's check out: 

  1. the extent, 
  2. coordinate system, 
  3. and a 3D plot of each .laz file. 
  
*Note that on Mac computers you may need to install 
XQuartz for 3D plots - see xquartz.org*


```r
summary(HARV)
lidR::plot(HARV)
```
```
class        : LAS (v1.3 format 3)
memory       : 521.6 Mb 
extent       : 727000, 728000, 4702000, 4703000 (xmin, xmax, ymin, ymax)
coord. ref.  : +proj=utm +zone=18 +datum=WGS84 +units=m +no_defs +ellps=WGS84 +towgs84=0,0,0 
area         : 1 km²
points       : 5.94 million points
density      : 5.94 points/m²
File signature:           LASF� 
File source ID:           211 
Global encoding:
 - GPS Time Type: GPS Week Time 
 - Synthetic Return Numbers: no 
 - Well Know Text: CRS is GeoTIFF 
 - Aggregate Model: false 
Project ID - GUID:        00000000-0000-0000-0000-000000000000 
Version:                  1.3
System identifier:        LAStools (c) by rapidlasso GmbH 
Generating software:      lascolor (190812) commercial 
File creation d/y:        278/2019
header size:              235 
Offset to point data:     329 
Num. var. length record:  1 
Point data format:        3 
Point data record length: 34 
Num. of point records:    5944282 
Num. of points by return: 4581566 1255117 104991 2608 0 
Scale factor X Y Z:       0.01 0.01 0.01 
Offset X Y Z:             7e+05 4700000 0 
min X Y Z:                727000 4702000 223.2 
max X Y Z:                728000 4703000 324.99 
Variable length records: 
   Variable length record 1 of 1 
       Description:  
       Tags:
          Key 1024 value 1 
          Key 1025 value 2 
          Key 3072 value 32618 
          Key 4099 value 9001 
```

![](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/dev-aten/graphics/lidar-point-clouds/HARV1km2.JPG)

>1 km-squared point cloud from Harvard Forest showing a gentle slope covered in a continuous canopy of mixed forest.


```r
summary(TEAK)
lidR::plot(TEAK)
```

```
class        : LAS (v1.3 format 3)
memory       : 439.2 Mb 
extent       : 316000, 317000, 4091231, 4092000 (xmin, xmax, ymin, ymax)
coord. ref.  : +proj=utm +zone=11 +datum=WGS84 +units=m +no_defs +ellps=WGS84 +towgs84=0,0,0 
area         : 0.75 km²
points       : 5.01 million points
density      : 6.7 points/m²
File signature:           LASF� 
File source ID:           211 
Global encoding:
 - GPS Time Type: GPS Week Time 
 - Synthetic Return Numbers: no 
 - Well Know Text: CRS is GeoTIFF 
 - Aggregate Model: false 
Project ID - GUID:        00000000-0000-0000-0000-000000000000 
Version:                  1.3
System identifier:        LAStools (c) by rapidlasso GmbH 
Generating software:      lascolor (190812) commercial 
File creation d/y:        206/2019
header size:              235 
Offset to point data:     329 
Num. var. length record:  1 
Point data format:        3 
Point data record length: 34 
Num. of point records:    5006002 
Num. of points by return: 3315380 1182992 416154 91476 0 
Scale factor X Y Z:       0.01 0.01 0.01 
Offset X Y Z:             3e+05 4e+06 0 
min X Y Z:                316000 4091231 2117.03 
max X Y Z:                317000 4092000 2376.98 
Variable length records: 
   Variable length record 1 of 1 
       Description:  
       Tags:
          Key 1024 value 1 
          Key 1025 value 2 
          Key 3072 value 32611 
          Key 4099 value 9001
```

![](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/dev-aten/graphics/lidar-point-clouds/TEAK1km2.JPG)

>1 km-squared point cloud from Lower Teakettle showing mountainous terrain covered in a patchy conifer forest, with tall, skinny conifers clearly visible emerging from the discontinuous canopy.


## Normalizing Tree Height to Ground
To begin, we will take a look at the structural diversity of the dense mixed deciduous/evergreen 
forest of HARV. We're going to choose a 40 x 40 m spatial extent for our analysis, but first we 
need to normalize the height values of this LiDAR point cloud from an absolute elevation 
above mean sea level to height above the ground using the `lidR::lasnormalize()` function. 

This function relies on spatial interpolation, and therefore we want to perform this step on an area 
that is quite a bit larger than our area of interest to avoid edge effects. 

To be safe, we will clip out an area of 200 x 200 m, normalize it, and then clip out our smaller area of interest.


```r
# Correct for elevation 
#We're going to choose a 40 x 40 m spatial extent, which is the
#extent for NEON base plots. 
#First set the center of where you want the plot to be (note easting 
#and northing works in units of m because these data are in a UTM 
#proejction as shown in the summary above).
x <- 727500 #easting 
y <- 4702500 #northing

#Cut out a 200 x 200 m buffer by adding 100 m to easting and 
#northing coordinates (x,y).
data.200m <- 
   lasclipRectangle(HARV,
                    xleft = (x - 100), ybottom = (y - 100),
                    xright = (x + 100), ytop = (y + 100))

#Correct for ground height using a kriging function to interpolate 
#elevation from ground points in the .laz file.
#If the function will not run, then you may need to checkfor outliers
#by adjusting the 'drop_z_' arguments when reading in the .laz files.
dtm <- grid_terrain(data.200m, 1, kriging(k = 10L))
data.200m <- lasnormalize(data.200m, dtm)

#Will often give a warning if not all points could be corrected, 
#but visually check to see if it corrected for ground height. 
lidR::plot(data.200m)
#There's only a few uncorrected points and we'll fix these in 
#the next step. 

#Clip 20 m out from each side of the easting and northing 
#coordinates (x,y).
data.40m <- 
   lasclipRectangle(data.200m, 
                    xleft = (x - 20), ybottom = (y - 20),
                    xright = (x + 20), ytop = (y + 20))

data.40m@data$Z[data.40m@data$Z <= .5] <- NA  
#This line filters out all z_vals below .5 m as we are less 
#interested in shrubs/trees. 
#You could change it to zero or another height depending on interests. 

#visualize the clipped plot point cloud
lidR::plot(data.40m) 
```

![](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/dev-aten/graphics/lidar-point-clouds/HARV40mx40m.JPG)

>40 meter by 40 meter point cloud from Harvard Forest showing a cross-section of the forest structure with a complex canopy- and sub-canopy structure with many rounded crowns, characteristic of a deciduous-dominated section of forest.

## Calculating Structural Diversity Metrics
Now that we have our area of interest normalized and clipped, we can proceed with calculating 
our structural diversity metrics. 

### GENERATE CANOPY HEIGHT MODEL (CHM) 
(i.e. a 1 m2 raster grid of vegetations heights)


```r
# Structural diversity metrics


#res argument specifies pixel size in meters and dsmtin is 
#for raster interpolation
chm <- grid_canopy(data.40m, res = 1, dsmtin())  

#visualize CHM
lidR::plot(chm) 
```

### MEAN OUTER CANOPY HEIGHT (MOCH)


```r
#calculate MOCH, the mean CHM height value
mean.max.canopy.ht <- mean(chm@data@values, na.rm = TRUE) 
```
### MAX CANOPY HEIGHT


```r
#calculate HMAX, the maximum CHM height value
max.canopy.ht <- max(chm@data@values, na.rm=TRUE) 
```

### RUMPLE

```r
#calculate rumple, a ratio of outer canopy surface area to 
#ground surface area (1600 m^2)
rumple <- rumple_index(chm) 
```

### TOP RUGOSITY


```r
#top rugosity, the standard deviation of pixel values in chm and 
#is a measure of outer canopy roughness
top.rugosity <- sd(chm@data@values, na.rm = TRUE) 
```

### DEEP GAPS & DEEP GAP FRACTION

```r
#number of cells in raster (also area in m2)
cells <- length(chm@data@values) 
chm.0 <- chm
chm.0[is.na(chm.0)] <- 0 #replace NAs with zeros in CHM
#create variable for the number of deep gaps, 1 m^2 canopy gaps
zeros <- which(chm.0@data@values == 0) 
deepgaps <- length(zeros) #number of deep gaps
#deep gap fraction, the number of deep gaps in the chm relative 
#to total number of chm pixels
deepgap.fraction <- deepgaps/cells 
```
### COVER FRACTION

```r
#cover fraction, the inverse of deep gap fraction
cover.fraction <- 1 - deepgap.fraction 

#HEIGHT SD
#height SD, the standard deviation of height values for all points
#in the plot point cloud
vert.sd <- cloud_metrics(data.40m, sd(Z, na.rm = TRUE)) 


#SD of VERTICAL SD of HEIGHT
#rasterize plot point cloud and calculate the standard deviation 
#of height values at a resolution of 1 m^2
sd.1m2 <- grid_metrics(data.40m, sd(Z), 1)
#standard deviation of the calculated standard deviations 
#from previous line 
#This is a measure of internal and external canopy complexity
sd.sd <- sd(sd.1m2[,3], na.rm = TRUE) 


#some of the next few functions won't handle NAs, so we need 
#to filter these out of a vector of Z points
Zs <- data.40m@data$Z
Zs <- Zs[!is.na(Zs)]

#ENTROPY 
#entropy, quantifies diversity & evenness of point cloud heights 
#by = 1 partitions point cloud in 1 m tall horizontal slices 
#ranges from 0-1, with 1 being more evenly distributed points 
#across the 1 m tall slices 
entro <- entropy(Zs, by = 1) 

#GAP FRACTION PROFILE 
#gap fraction profile, assesses the distribution of gaps in the 
#canopy volume 
#dz = 1 partitions point cloud in 1 m horizontal slices 
#z0 is set to a reasonable height based on the age and height of 
#the study sites 
gap_frac <- gap_fraction_profile(Zs, dz = 1, z0=3) 
#defines gap fraction profile as the average gap fraction in each 
#1 m horizontal slice assessed in the previous line
GFP.AOP <- mean(gap_frac$gf) 

#VAI
#leaf area density, assesses leaf area in the canopy volume 
#k = 0.5 is a standard extinction coefficient for foliage 
#dz = 1 partitions point cloud in 1 m horizontal slices 
#z0 is set to the same height as gap fraction profile above
LADen<-LAD(Zs, dz = 1, k=0.5, z0=3) 
#vegetation area index, sum of leaf area density values for 
#all horizontal slices assessed in previous line
VAI.AOP <- sum(LADen$lad, na.rm=TRUE) 
```

### VCI
A vertical complexity index, fixed normalization of entropy metric calculated above

```r
#set zmax comofortably above maximum canopy height
#by = 1 assesses the metric based on 1 m horizontal slices in 
#the canopy
VCI.AOP <- VCI(Zs, by = 1, zmax=100) 
```

We now have 13 different structural diversity metrics.  Let's organize them into a new dataframe:


```r
#OUTPUT CALCULATED METRICS INTO A TABLE
#creates a dataframe row, out.plot, containing plot descriptors 
#and calculated metrics
HARV_structural_diversity <- 
   data.frame(matrix(c(x, y, mean.max.canopy.ht, max.canopy.ht, 
                       rumple, deepgaps,deepgap.fraction,
                       cover.fraction,top.rugosity, vert.sd, 
                       sd.sd, entro,GFP.AOP, VAI.AOP, VCI.AOP),
                     ncol = 15)) 

#provide descriptive names for the calculated metrics
colnames(HARV_structural_diversity) <- 
   c("easting", "northing", "mean.max.canopy.ht.aop",
     "max.canopy.ht.aop", "rumple.aop", "deepgaps.aop",
     "deepgap.fraction.aop","cover.fraction.aop", 
     "top.rugosity.aop", "vert.sd.aop", "sd.sd.aop", 
     "entropy.aop", "GFP.AOP.aop", "VAI.AOP.aop", "VCI.AOP.aop") 

#View the results
HARV_structural_diversity 
```


## Combining Everything Into One Function
Now that we have run through how to measure each structural diversity metric, let's create a 
convenient function to run these a little faster on the TEAK site for a comparison of structural 
diversity with HARV. 


```r
#Let's correct for elevation and measure structural diversity for TEAK
x <- 316400 
y <- 4091700

data.200m <- lasclipRectangle(TEAK, 
                              xleft = (x - 100), ybottom = (y - 100),
                              xright = (x + 100), ytop = (y + 100))

dtm <- grid_terrain(data.200m, 1, kriging(k = 10L))
data.200m <- lasnormalize(data.200m, dtm)

data.40m <- lasclipRectangle(data.200m, 
                             xleft = (x - 20), ybottom = (y - 20),
                             xright = (x + 20), ytop = (y + 20))
data.40m@data$Z[data.40m@data$Z <= .5] <- 0  
plot(data.40m)
```


![](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/dev-aten/graphics/lidar-point-clouds/TEAK40mx40m.JPG)

>40 meter by 40 meter point cloud from Lower Teakettle showing a cross-section of the forest structure with several tall, pointed conifers separated by deep gaps in the canopy.


```r
#Zip up all the code we previously used and write function to 
#run all 13 metrics in a single function. 
structural_diversity_metrics <- function(data.40m) {
   chm <- grid_canopy(data.40m, res = 1, dsmtin()) 
   mean.max.canopy.ht <- mean(chm@data@values, na.rm = TRUE) 
   max.canopy.ht <- max(chm@data@values, na.rm=TRUE) 
   rumple <- rumple_index(chm) 
   top.rugosity <- sd(chm@data@values, na.rm = TRUE) 
   cells <- length(chm@data@values) 
   chm.0 <- chm
   chm.0[is.na(chm.0)] <- 0 
   zeros <- which(chm.0@data@values == 0) 
   deepgaps <- length(zeros) 
   deepgap.fraction <- deepgaps/cells 
   cover.fraction <- 1 - deepgap.fraction 
   vert.sd <- cloud_metrics(data.40m, sd(Z, na.rm = TRUE)) 
   sd.1m2 <- grid_metrics(data.40m, sd(Z), 1) 
   sd.sd <- sd(sd.1m2[,3], na.rm = TRUE) 
   Zs <- data.40m@data$Z
   Zs <- Zs[!is.na(Zs)]
   entro <- entropy(Zs, by = 1) 
   gap_frac <- gap_fraction_profile(Zs, dz = 1, z0=3)
   GFP.AOP <- mean(gap_frac$gf) 
   LADen<-LAD(Zs, dz = 1, k=0.5, z0=3) 
   VAI.AOP <- sum(LADen$lad, na.rm=TRUE) 
   VCI.AOP <- VCI(Zs, by = 1, zmax=100) 
   out.plot <- data.frame(
      matrix(c(x, y, mean.max.canopy.ht,max.canopy.ht, 
               rumple,deepgaps, deepgap.fraction, 
               cover.fraction, top.rugosity, vert.sd, 
               sd.sd, entro, GFP.AOP, VAI.AOP,VCI.AOP),
             ncol = 15)) 
   colnames(out.plot) <- 
      c("easting", "northing", "mean.max.canopy.ht.aop",
        "max.canopy.ht.aop", "rumple.aop", "deepgaps.aop",
        "deepgap.fraction.aop", "cover.fraction.aop",
        "top.rugosity.aop","vert.sd.aop","sd.sd.aop", 
        "entropy.aop", "GFP.AOP.aop",
        "VAI.AOP.aop", "VCI.AOP.aop") 
   print(out.plot)
}

TEAK_structural_diversity <- structural_diversity_metrics(data.40m)
```

## Comparing Metrics Between Forests
How does the structural diversity of the evergreen TEAK forest compare to the mixed deciduous/evergreen forest from HARV? Let's combine the result data.frames for a direct comparison:


```r
combined_results=rbind(HARV_structural_diversity, 
                       TEAK_structural_diversity)

# Add row names for clarity
row.names(combined_results)=c("HARV","TEAK")

# Take a look to compare
combined_results
```



## Matching GEDI waveforms with NEON AOP LiDAR pointclouds

>author: Donal O'Leary

*GEDI has amazing coverage around the globe, but is limited in its spatial resolution. Here, we extract NEON pointcloud data corresponding to individual GEDI waveforms to better understand how the GEDI return waveform gets its shape.*

<div id="ds-objectives" markdown="1">

## Learning Objectives 
After completing this section you will be able to: 

* Search for GEDI data based on a NEON site bounding box
* Extract NEON LiDAR pointcloud data corresponding to a specific GEDI footprint
* Visualize NEON and GEDI LiDAR data together in 3D

## Things You’ll Need To Complete This Tutorial

### R Packages to Install
Prior to starting the tutorial ensure that the following packages are installed. 

* **raster:** `install.packages("raster")`
* **rGEDI:** `install.packages("rGEDI")`
* **sp:** `install.packages("sp")`
* **sf:** `install.packages("sf")`
* **lidR:** `install.packages("lidR")`
* **neonUtilities:** `install.packages("neonUtilities")`
* **viridis:** `install.packages("viridis")`
* **maptools:** `install.packages("maptools")`



### Example Data Set

############# Possibly add GEDI subset here?? ##############

#### Datum difference between WGS84 and NAD83
This dataset describes the differences between two common standards for vertical data in North America.
<a href="https://neondata.sharefile.com/d-sf4e35e969fc43aca" class="link--button link--arrow">
Download Dataset</a>

### Working Directory
[Donal to add text]

</div>

## Getting Started
In this tutorial, we will compare NEON and GEDI LiDAR data by comparing the information that they both capture in the same location. NEON data are actually one of the datasets used by the GEDI mission to calibrate and validate GEDI waveforms, so this makes for a valuable comparison! 

In order to compare these data, we will need to download GEDI data that overlap a NEON site. Fortunately, Carlos Silva et al. have made a convenient R package clled `rGEDI` and <a href="https://cran.r-project.org/web/packages/rGEDI/vignettes/tutorial.html">this excellent tutorial hosted on CRAN</a> desribing how to work with GEDI data in R. 

However, GEDI data are currently only available to download per complete orbit, which means that the vast majority of the orbit's data does not fall within a NEON site. The GEDI orbit datasets come in HDF5 data format, and contain about 7Gb of data, so you may want to run the first few sections of this tutorial to get the GEDI download started, and refresh your memory of HDF5 files with NEON's <a href="https://www.neonscience.org/intro-hdf5-r-series">Intro to HDF5 series</a>.

First, we will load the required libraries and set our working directory:


```r
library(rGEDI)
library(raster)
library(sp)
library(sf)
```

```
## Linking to GEOS 3.7.2, GDAL 2.4.2, PROJ 5.2.0
```

```r
library(rgl)
library(lidR)
library(neonUtilities)
library(viridis)
```

```
## Loading required package: viridisLite
```

```r
library(maptools)
```

```
## Checking rgeos availability: TRUE
```

```r
#wd <- "./data" # This will depend on your local environment
#setwd(wd)
```

Next, we will download a Canopy Height Model (CHM) tile from the Wind River Experiemental Forest (WREF) using the `byTileAOP()` function to use as a preliminary map on which to overlay the GEDI data. There is one particularly interesting mosaic tile which we will select using the `easting` and `northing` arguments. We then load the raster into R and make a simple plot. For more information about CHMs, please see our tutorial <a href="https://www.neonscience.org/chm-dsm-dtm-gridded-lidar-data">What is a CHM?</a>


```r
# Define the SITECODE as a variable because
# we will use it several times in this tutorial
SITECODE <- "WREF"

byTileAOP(dpID = "DP3.30015.001", site = SITECODE, year = 2017, 
          easting = 580000, northing = 5075000, check.size = F,
          savepath = './data')
```

```
## Downloading files totaling approximately 4.0 MB 
## Downloading 6 files
## 
## Successfully downloaded  6  files.
## NEON_D16_WREF_DP1_580000_5075000_classified_point_cloud.kml downloaded to ./data/DP3.30015.001/2017/FullSite/D16/2017_WREF_1/Metadata/DiscreteLidar/TileBoundary/kmls
## NEON_D16_WREF_DP1_580000_5075000_classified_point_cloud.shp downloaded to ./data/DP3.30015.001/2017/FullSite/D16/2017_WREF_1/Metadata/DiscreteLidar/TileBoundary/shps
## NEON_D16_WREF_DP1_580000_5075000_classified_point_cloud.dbf downloaded to ./data/DP3.30015.001/2017/FullSite/D16/2017_WREF_1/Metadata/DiscreteLidar/TileBoundary/shps
## NEON_D16_WREF_DP1_580000_5075000_classified_point_cloud.shx downloaded to ./data/DP3.30015.001/2017/FullSite/D16/2017_WREF_1/Metadata/DiscreteLidar/TileBoundary/shps
## NEON_D16_WREF_DP3_580000_5075000_CHM.tif downloaded to ./data/DP3.30015.001/2017/FullSite/D16/2017_WREF_1/L3/DiscreteLidar/CanopyHeightModelGtif
## NEON_D16_WREF_DP1_580000_5075000_classified_point_cloud.prj downloaded to ./data/DP3.30015.001/2017/FullSite/D16/2017_WREF_1/Metadata/DiscreteLidar/TileBoundary/shps
```

```r
chm <- raster('./data/DP3.30015.001/2017/FullSite/D16/2017_WREF_1/L3/DiscreteLidar/CanopyHeightModelGtif/NEON_D16_WREF_DP3_580000_5075000_CHM.tif')

plot(chm)
```

<img src="07-NEON_AOP_files/figure-html/download-chm-1.png" width="672" />

As you can see, this particular CHM is showing a conspicuous, triangle-shaped clearcut in this section of the experimental forest, where the tree canopy is much shorter than the towering 60m+ trees in the undisturbed areas. This variation will give us a variety of forest structures to investigate.

## Downloading GEDI data
This next section on downloading and working with GEDI data is loosely based on the excellent `rGEDI` package tutorial <a href="https://cran.r-project.org/web/packages/rGEDI/vignettes/tutorial.html">posted on CRAN here</a>.

The Global Ecosystem Dynamics Investigation (GEDI) is a NASA mission with the primary data collection being performed by a novel waveform lidar instrument mounted on the International Space Station (ISS). Please see <a href="https://www.sciencedirect.com/science/article/pii/S2666017220300018">this open-access paper</a> published in Science of Remote Sensing that describes this mission in detail. The ISS orbits around the world every 90 minutes, and can be tracked <a href="https://spotthestation.nasa.gov/tracking_map.cfm">on this cool NASA website</a>. 

As described <a href="https://lpdaac.usgs.gov/data/get-started-data/collection-overview/missions/gedi-overview/">here</a> on the Land Processes Distributed Active Archive Center (LP DAAC), "the sole GEDI observable is the waveform from which all other data products are derived. Each waveform is captured with a nominal ~30 m diameter."GEDI has six   As of the date of publication, GEDI data are only offered in HDF5 format, with each file containing the data for a full orbit. The LP DAAC has developed a tool that allows researchers to input a bounding box, which will return a list of every orbit that has a "shot" (waveform return) that falls within that box. Unfortunately, at this time, the tool will not subset out the specific shots that fall within that bounding box; you must download the entire orbit (~7Gb). This functionality may be improved in the future.

Our next few steps involve defining our bounding box, requesting the list of GEDI orbits that contain data relevant to that bounding box, and downloading those data. Let's focus on the extent of our CHM that we downloaded above - but we will first need to re-project the CHM from its UTM projection into WGS84. To do so, we will refer to the EPSG code for WGS84. To look up any of these codes, please see the amazing resource <a href="https://spatialreference.org/ref/epsg/4326/">spatialrerference.org</a>.


```r
# Project CHM to WGS84
chm_WGS = projectRaster(chm, crs=CRS("+init=epsg:4326"))

# Study area boundary box coordinates
ul_lat<- extent(chm_WGS)[4]
lr_lat<- extent(chm_WGS)[3]
ul_lon<- extent(chm_WGS)[1]
lr_lon<- extent(chm_WGS)[2]
```

Next, we use that bounding box information as an input to the `gedifinder()` funciton.


```r
# Specifying the optional date range, if desired
daterange=c("2019-03-25", #first date of GEDI availability
           "2020-07-15")

# Get path to GEDI data
# These lines use an API request to determine which orbits are available 
# that intersect the bounding box of interest.
# Note that you still must download the entire orbit and then subset to 
# your area of interest!
gLevel1B <- gedifinder(product="GEDI01_B",
                      ul_lat, ul_lon, lr_lat,lr_lon,
                     version="001",daterange=NULL)
# 
# # View list of available data
 gLevel1B
```

```
## [1] "https://e4ftl01.cr.usgs.gov/GEDI/GEDI01_B.001/2019.10.07/GEDI01_B_2019280204828_O04642_T03216_02_003_01.h5"
## [2] "https://e4ftl01.cr.usgs.gov/GEDI/GEDI01_B.001/2019.07.25/GEDI01_B_2019206022612_O03482_T00370_02_003_01.h5"
## [3] "https://e4ftl01.cr.usgs.gov/GEDI/GEDI01_B.001/2019.06.08/GEDI01_B_2019159013955_O02752_T01597_02_003_01.h5"
```

Great! There are several GEDI orbits available that have at least 1 'shot' within our bounding box of interest. For more information about GEDI filename conventions, and other valuable information about GEDI data, <a href="https://lpdaac.usgs.gov/data/get-started-data/collection-overview/missions/gedi-overview/#gedi-temporal-and-spatial-resolution">see this page on the LP DAAC</a>. However, as mentioned before, each of these files are quite large (~7Gb), so let's focus on just the first one for now.


```r
# Downloading GEDI data, if you haven't already
# if(!file.exists(paste0(wd,basename(gLevel1B[2])))){
#   gediDownload(filepath=gLevel1B[2],outdir=wd)
# }
```

Next, we use the rGEDI package to read in the GEDI data. First, we need to make a 'gedi.level1b' object using the `readLevel1B()` function. Next, we extract the geographic position of the center of each shot from the 'gedi.level1b' object using the `getLevel1BGeo()` function, and display the head of the resulting table.


```r
gedilevel1b<-readLevel1B(level1Bpath = file.path('./data/NEON_GEDI_subset.h5'))
#gedilevel1b<-readLevel1B(level1Bpath = file.path(wd, "GEDI01_B_2019206022612_O03482_T00370_02_003_01.h5"))
level1bGeo<-getLevel1BGeo(level1b=gedilevel1b,select=c("elevation_bin0"))
```

```
## 
```

```r
head(level1bGeo)
```

```
##    shot_number latitude_bin0 latitude_lastbin longitude_bin0 longitude_lastbin
## 1:           1      45.82541         45.82542      -121.9700         -121.9699
## 2:           2      45.82566         45.82567      -121.9693         -121.9693
## 3:           3      45.82590         45.82591      -121.9687         -121.9686
## 4:           4      45.82615         45.82616      -121.9680         -121.9680
## 5:           5      45.82639         45.82641      -121.9674         -121.9674
## 6:           6      45.82664         45.82665      -121.9667         -121.9667
##    elevation_bin0
## 1:       505.6233
## 2:       483.9717
## 3:       505.3455
## 4:       496.2213
## 5:       500.5322
## 6:       494.9825
```

## Plot GEDI footprints on CHM
Let's visualize where the GEDI footprints are located on the CHM tile. To do so, we will need to first convert the GEDI data into a spatial object. For this example, we will use a 'spatial features' object type from the 'sp' package:


```r
# drop any shots with missing latitude/longitude values
level1bGeo = level1bGeo[!is.na(level1bGeo$latitude_bin0)&
                          !is.na(level1bGeo$longitude_bin0),]

# Convert the GEDI data.frame into an 'sf' object
level1bGeo_spdf<-st_as_sf(level1bGeo, 
                          coords = c("longitude_bin0", "latitude_bin0"),
                          crs=CRS("+init=epsg:4326"))


# crop to the CHM that is in WGS84
level1bgeo_WREF=st_crop(level1bGeo_spdf, chm_WGS)
```

```
## although coordinates are longitude/latitude, st_intersection assumes that they are planar
```

```
## Warning: attribute variables are assumed to be spatially constant throughout all
## geometries
```

Next, project the GEDI geospatial data into the UTM zone that the CHM is within (Zone 10 North). These data come as the point location at the center of the GEDI footprint, so we next convert the GEDI footprint center (lat/long) into a circle using the `buffer()` function. Finally, we can plot the CHM and overlay the GEDI footprint circles, and label with the last three digits of the 'shot' number.


```r
# project to UTM
level1bgeo_WREF_UTM=st_transform(
  level1bgeo_WREF, crs=chm$NEON_D16_WREF_DP3_580000_5075000_CHM@crs)

# buffer the GEDI shot center by a radius of 12.5m 
# to represent the full 25m diameter GEDI footprint
level1bgeo_WREF_UTM_buffer=st_buffer(level1bgeo_WREF_UTM, dist=12.5)

# plot CHM and overlay GEDI data
plot(chm)
plot(level1bgeo_WREF_UTM_buffer, add=T, col="transparent")

# add labes with the last three digits of the GEDI shot_number
pointLabel(st_coordinates(level1bgeo_WREF_UTM),
           labels=level1bgeo_WREF_UTM$shot_number, cex=1)
```

<img src="07-NEON_AOP_files/figure-html/overly-GEDI-on-chm-1.png" width="480" />

## Extract Waveform for a single Shot
Let's take a look at a waveform for a single GEDI shot. We will use the last three numbers of the shot, shown a labels in the plot above, to select a waveform of interest. In this case, let's plot the the footprint labeled '002' in the northwest of the CHM.


```r
# Extracting GEDI full-waveform for a given shot_number
###ERROR HERE###
wf <- getLevel1BWF(gedilevel1b,shot_number = 20)
# wf <- getLevel1BWF(gedilevel1b,shot_number = level1bgeo_WREF_UTM_buffer$shot_number[which(level1bgeo_WREF_UTM_buffer$shot_number==1786)])

# Save current plotting parameters to revert to
oldpar <- par()

# Set up plotting paramters
par(mfrow = c(1,2), mar=c(4,4,1,1), cex.axis = 1.5)

# Plot filled-in waveform 
plot(wf, relative=FALSE, polygon=TRUE, type="l", lwd=2, col="forestgreen",
     xlab="Waveform Amplitude", ylab="Elevation (m)")
grid() #add a grid to the plot

# Plot a simple line with no fill
plot(wf, relative=TRUE, polygon=FALSE, type="l", lwd=2, col="forestgreen",
     xlab="Waveform Amplitude (%)", ylab="Elevation (m)")
grid()#add a grid to the plot
```

<img src="07-NEON_AOP_files/figure-html/extract-wf-1.png" width="672" />

```r
# Revert plotting parameters to previous values.
par(oldpar)
```

This waveform shows some noise above and below the main ecosystem return, with a fairly dense canopy around 370m elevation, an a characteristic ground return spike at about 340m. While the GEDI data are extremely valuable and offer a near-global coverage, it is hard to get a good sense of what the ecosystem really looks like from this GEDI waveform. Let's download some NEON AOP pointcloud data to pair up with this waveform to get a better sense of what GEDI is reporting.

## Download and Plot NEON AOP LiDAR pointcloud data
Here, we will use the `byTileAOP()` function from the 'neonUtilities' package to download the classified pointcloud mosaic data (DP1.30003.001). Since this is a mosaic tile like the CHM, we can just pass this function the lower left corner of the CHM tile to get the corresponding lidar pointcloud mosaic tile for our analysis.


```r
# Download the pointcloud data if you don't have it already
if (!file.exists('./data/DP1.30003.001/2017/FullSite/D16/2017_WREF_1/L1/DiscreteLidar/ClassifiedPointCloud/NEON_D16_WREF_DP1_580000_5075000_classified_point_cloud.laz')){
  byTileAOP(dpID = "DP1.30003.001", site = SITECODE, year = 2017, 
          easting = extent(chm)[1], northing=extent(chm)[3], 
          check.size = F)
}
```

After downloading the point cloud data, let's read them into our R session using the `readLAS()` function from the `lidaR` package, and plot them in 3D. Note, you may need to update your 'XQuartz' if you are using a Mac.


```r
WREF_LAS=readLAS("./data/DP1.30003.001/2017/FullSite/D16/2017_WREF_1/L1/DiscreteLidar/ClassifiedPointCloud/NEON_D16_WREF_DP1_580000_5075000_classified_point_cloud.laz")

lidR::plot(WREF_LAS)
```

Oh, yikes! There are a lot of outliers above the actual forest, and a few below. Let's use some simple statistics to throw out those outliers. We will first calculate the mean and standard deviation for the verical axis, and then use the 'filter' options of the `readLAS()` function to eliminate the vertical outliers.


```r
# ### remove outlier lidar point outliers using mean and sd statistics
Z_sd=sd(WREF_LAS@data$Z)
Z_mean=mean(WREF_LAS@data$Z)
 
# make filter string in form filter = "-drop_z_below 50 -drop_z_above 1000"
# You can increase or decrease (from 4) the number of sd's to filter outliers
f = paste("-drop_z_below",(Z_mean-4*Z_sd),"-drop_z_above",(Z_mean+4*Z_sd))

# Read in LAS file, trimming off vertical outlier points
WREF_LAS=readLAS("./data/DP1.30003.001/2017/FullSite/D16/2017_WREF_1/L1/DiscreteLidar/ClassifiedPointCloud/NEON_D16_WREF_DP1_580000_5075000_classified_point_cloud.laz",
                 filter = f)

#Plot the full LiDAR point cloud mosaic tile (1km^2)
plot(WREF_LAS)
```
Ahhh, that's better.

## Clip AOP LiDAR Pointcloud to GEDI footprints
We can now use the GEDI footprint circles (that we made using the `buffer()` function) to clip out the NEON AOP LiDAR points that correspond with the GEDI footprints:


```r
# Clip the pointcloud by the GEDI footprints created by buffering above. 
# This makes a 'list' of LAS objects
WREF_GEDI_footprints=lasclip(WREF_LAS, geometry = level1bgeo_WREF_UTM_buffer)

# we can now plot individual footprint clips
plot(WREF_GEDI_footprints[[8]])
```

## Plot GEDI Waveform with AOP Pointcloud in 3D space
Now that we can extract individual waveforms, and the AOP LiDAR pointcloud that corresponds with each GEDI waveform, let's see if we can plot them both in 3D space. We already know how to plot the AOP LiDAR points, so let's write a function to draw the GEDI waveform, too, using the `points3d()` function:


```r
for(shot_n in 1:21){

  # First, plot the NEON AOP LiDAR clipped to the GEDI footprint
  # We save the plot as an object 'p' which gives the (x,y) offset for the lower
  # left corner of the plot. The 'rgl' package offsets all (x,y) locations 
  # to this point, so we will need to subtract these values from any other 
  # (x,y) points that we want to add to the plot
  p=plot(WREF_GEDI_footprints[[shot_n]])
  
  # Extract the specific waveform from the GEDI data
  wf <- getLevel1BWF(gedilevel1b,shot_number = level1bgeo_WREF$shot_number[shot_n])
  
  # Make a new data.frame 'd' to convert the waveform data coordinates into 3D space
  d=wf@dt
  
  # normalize rxwaveform to 0-1
  d$rxwaveform=d$rxwaveform-min(d$rxwaveform)
  d$rxwaveform=d$rxwaveform/max(d$rxwaveform)
  
  # Add xy data in UTMs, and offset lower left corner of 'p'
  d$x=st_coordinates(level1bgeo_WREF_UTM[shot_n,])[1]-p[1]
  d$y=st_coordinates(level1bgeo_WREF_UTM[shot_n,])[2]-p[2]
  
  # Make a new column 'x_wf' where we place the GEDI waveform in space with the 
  # NEON AOP LiDAR data, we scale the waveform to 30m in the x-dimension, and 
  # offset by 12.5m (the radius of the GEDI footprint) in the x-dimension.
  d$x_wf=d$x+d$rxwaveform*30+12.5
  
  # Add GEDI points to 3D space in 'green' color
  points3d(x=d$x_wf, y=d$y, z=d$elevation, col="green", add=T)

}
```

Whoa, it looks like there is a bad vertical mismatch between those two data sources. This is because the two data sources have a different vertical datum. The NEON AOP data are delivered in the GEOID12A, while the GEDI data are delivered in the WGS84 native datum. We will need to convert one to the other to get them to line up correctly.

## Datum, Geoid, and how to best measure the Earth
We are seeing a vertical mismatch between the NEON and GEDI LiDAR data sources because they are using different standards for measuring the Earth. Here, we will briefly describe the main differences between these two datum models, and then show how to correct for this discrepancy.

### WGS84
As described in <a href="https://en.wikipedia.org/wiki/World_Geodetic_System">this great Wikipedia article</a> the World Geodedic System (WGS) is a global standard for cartography, geodesy, and navigation including GPS. Most people refer to "WGS84," which is the latest revision to this system in 1984. To describe elevation, WGS84 uses an idealized mathematical model to describe the 'oblate spheroid' of the Earth (basically, it looks kind of like a sphere that is wider around the equator and shorter from pole to pole). This model, called a 'datum,' defines the relative elevational surface of the Earth. However, the Earth isn't exactly this idealized shape - it has a lot of undulations caused by topogrophy and local differences in its gravitational field. In the locations where the natural variations of the Earth do not coencide with the mathematical model, it can be harder to accurately describe the elevation of certain landforms and structures. Still, for a global datum, WGS84 does a great job overall.

### GEOID12A
While WGS84 is a convenient global standard, precisely describing location information is a common problem in geography, and is often remedied by using a local datum or projection. For latitude and longitude spatial information in North America, this is often done by using the <a href="https://www.ngs.noaa.gov/datums/horizontal/north-american-datum-1983.shtml">North American Datum (NAD83)</a> or various Universal Transverse Mercator (UTM) or State Plane projections based on the NAD83 datum. The NAD83 Datum was developed in 1983 as the horizontal and geometric control datum for the United States, Canada, Mexico, and Central America. This is the official datum for the government of the United States of America, and therefore many public datasets that are specific to this geographic region use this datum. In order to further refine the vertical precision over a wide geographic area, we can model the gravitational field of the Earth - this surface is called a "Geoid". <a href="https://geodesy.noaa.gov/GEOID/geoid_def.html">According to NOAA</a>, a geoid can be defined as "the equipotential surface of the Earth’s gravity field which best fits, in a least squares sense, global mean sea level." For a nice description of the North American geoid, please <a href="https://geodesy.noaa.gov/GEOID/geophysics.shtml">see this page on the NOAA website</a> written by Dr. Allen Joel Anderson. As described by Dr. Anderson, this gravitational field is changing all the time, as can global mean sea level (due to melting glaciers, etc.). Therefore local and regional geoids must be updated regularly to reflect these changes, as shown <a href="https://geodesy.noaa.gov/GEOID/models.shtml">in this list of NOAA geoids</a>. One recent standard from 2012 is <<a href="https://geodesy.noaa.gov/GEOID/GEOID12A/">GEOID12A</a> - this is the geoid selected by NEON as the vertical reference for the Z-dimension of our LiDAR data. Note that GEOID12B superscedes GEOID12A, however they are identical everywhere except for Puerto Rico and the US Virgin Islands <a href="https://geodesy.noaa.gov/GEOID/GEOID12B/">according to NOAA</a>. If you are working with any NEON data from Puerto Rico, please be aware that these data are delivered in GEOID12A format and may need to be converted to GEOID12B for some purposes.

For more information about these standards, please see <a href="https://desktop.arcgis.com/en/arcmap/10.3/guide-books/map-projections/about-the-geoid-ellipsoid-spheroid-and-datum-and-h.htm">this ESRI help document</a>.

## Aligning the Vertical Datum
AOP data have a relatively small area of coverage, and are therefore delivered with the coordinate reference system (CRS) in the UTM zone in which they were collected, set to the GEOID12A vertical datum (roughly equaling mean sea level). Meanwhile, GEDI data are global, so they are delivered with the common WGS84 ellipsoidal refrence datum. We need to align these vertical datums to ensure that the two data products line up correctly. However, this is not a trivial step, nor as simple as a reprojection. In this example, we will keep the NEON LiDAR data in its current datum, and convert the vertical position of the GEDI data from WGS84 into GEOID12A vertical position.

NOAA has a useful <a href="https://www.ngs.noaa.gov/INFO/facts/vdatum.shtml">tool called Vdatum</a> that will convert from one datum to another. For this example, we will use Vdatum to convert from WGS84 to NAD83. Seeing as GEOID12A is based on the NAD83 datum, we can first convert the GEDI data's vertical position from WGS84 to NAD83 datum, then apply the offset between NAD83 and GEOID12A. Rather than use the Vdatum tool for each point, we will use a raster created by NEON scientists that reports the vertical difference between WGS84 and NAD83 for all points in the conterminous USA. Please <a href="https://neondata.sharefile.com/d-sf4e35e969fc43aca">visit this Sharepoint page</a> (same as the link at the top of this page) to download the raster. We will then read it into our R session:


```r
# Edit this filepath as needed for your machine
datum_diff=raster('./data/WGS84_NAD83_seperation.tif')
```

###GEOID12A Height Model
GEOID12A is a surface model that is described in terms of its relative height compared to NAD83. You can use <a href="https://www.ngs.noaa.gov/cgi-bin/GEOID_STUFF/geoid12A_prompt1.prl">this interactive webpage</a> to find the geoid height for any location within North America. However, that would be combersome to have to use this webpage for every location. Instead, we will download a <a href="https://www.ngs.noaa.gov/GEOID/GEOID12A/GEOID12A_CONUS.shtml>binary file from the NOAA website</a> that describes this geoid's height, and convert that into a raster similar to the one that we just downloaded above. We have included comments here from the NOAA website about the structure of the binary file. We use this information to extract the dimensions of this dataset in order to construct a raster in R from these binary data.


```r
# Download binary file of offset from GEOID12A to NAD83
if(!file.exists('./data/g2012au0.bin')){
  download.file("https://www.ngs.noaa.gov/PC_PROD/GEOID12A/Format_unix/g2012au0.bin", 
                destfile = './data/g2012au0.bin')
}

# Read header information. See https://www.ngs.noaa.gov/GEOID/GEOID12B/g2012Brme.txt

# File Structure
# ---------------
# The files (ASCII and binary) follow the same structure of a one-line header 
# followed by the data in row-major format. The one-line header contains four 
# double (real*8) words followed by three long (int*4) words. 
# These parameters define the geographic extent of the area:
# 
# SLAT:  Southernmost North latitude in whole degrees. 
#        Use a minus sign (-) to indicate South latitudes. 
# WLON:  Westernmost East longitude in whole degrees. 
# DLAT:  Distance interval in latitude in whole degrees 
#        (point spacing in E-W direction)
# DLON:  Distance interval in longitude in whole degrees 
#        (point spacing in N-S direction)
# NLAT:  Number of rows 
#        (starts with SLAT and moves northward DLAT to next row)
# NLON:  Number of columns 
#        (starts with WLON and moves eastward DLON to next column)
# IKIND: Always equal to one (indicates data are real*4 and endian condition)

to.read = file('./data/g2012au0.bin', "rb")
header1=readBin(to.read, double(), endian = "big", n=4)
header1 #SLAT, WLON, DLAT, DLON
```

```
## [1]  24.00000000 230.00000000   0.01666667   0.01666667
```

```r
header2=readBin(to.read, integer(), endian = "big", n=3)
header2 #NLAT, NLON, IKIND
```

```
## [1] 2041 4201    1
```

```r
GEOID12A_diff_vals=readBin(to.read, n=header2[1]*header2[2], numeric(), endian = "big", size=4)
```



```r
# Create a new raster using the dimensions extracted from the headers
GEOID12A_diff_rast <- raster(ncol=header2[2], nrow=header2[1], 
                             xmn=header1[2]-360, xmx=header1[2]+(header1[4]*header2[2])-360, 
                             ymn=header1[1], ymx=header1[1]+(header1[3]*header2[1]),
                             crs = crs(datum_diff))
GEOID12A_diff_rast <- setValues(GEOID12A_diff_rast, values = GEOID12A_diff_vals)

# we need to use the 'flip' function to put the map 'upright' because R expects to see raster values from the top left corner and fills by rows, but this dataset is delivered in sucha a way that it describes the bottom left corner and fills by rows up to the top of the image (this is actually the convention for most traditional remote sensing software - and leads to a similar problem that is explained in the Hyperspectral tutorial series.)
GEOID12A_diff_rast <- flip(GEOID12A_diff_rast, 'y')

## let's crop out only CONUS for plotting purposes - we will still refer to the fill image when extracting values.

diff_resp=resample(datum_diff, GEOID12A_diff_rast) # resample to match pixel size/registration for cropping
diff_resp=crop(diff_resp, GEOID12A_diff_rast)
GEOID12A_diff_rast=crop(GEOID12A_diff_rast, extent(diff_resp))
GEOID12A_diff_rast_mask=mask(GEOID12A_diff_rast, diff_resp)
```

Now that we have the two offset rasters, let's plot them together to compare their spatial patterns.


```r
par(mfrow=c(2,1), mar=c(2.5,2.5, 3.5,1))
plot(GEOID12A_diff_rast_mask, col=viridis(100),main="GEOID12A minus NAD83 (m)")
plot(diff_resp, main="WGS84 minus NAD83 (m)")
```

<img src="07-NEON_AOP_files/figure-html/plot-both-diff-maps-1.png" width="576" />

As you can see, the differences between the GEOID12A geoid, and the NAD83 sphereoid vary quite a lot across space, especially in mountainous areas. The magnitude of these differences is also large, upwards of 35m in some areas. Meanwhile, the differences between the NAD83 and WGS84 sphereoids shows a smooth gradient that is relatively small, with total difference less than 2m across the Conterminous USA. 

## Extract vertical offset for GEDI shots
Now that we know the relative vertical offsets between WGS84, NAD83, and GEOID12A for all of the conterminous USA, we can use the `extract()` function to retrieve those relative offsets for any location. By combining those offsets together, we can finally rectify the vertical position of the NEON and GEDI LiDAR data.

***** DO I need to convert GEDI x/y locations from WGS84 to NAD83?? *****



```r
# Make a new DF to store the GEDI footprint (x,y) locations, and the relative datum/geoid offsets
footprint_df=as.data.frame(
  cbind(level1bgeo_WREF$longitude_lastbin, level1bgeo_WREF$latitude_lastbin))

WGS_NAD_diff <- extract(datum_diff, footprint_df)

GEOID12A_NAD_diff <- extract(GEOID12A_diff_rast_mask,footprint_df)

# Add together the offsets to calculate a vector of net differences in elevation
net_diff=WGS_NAD_diff+GEOID12A_NAD_diff
```


## Plot vertically corrected GEDI waveform in 3D

Now that we have a vertical offset for each of the GEDI footprints, let's try again to plot the NEON AOP pointcloud data with the GEDI waveform.


```r
# You can enter whichever shots that you want to plot in the for loop here
#for(shot_n in 1:length(WREF_GEDI_footprints)){
for(shot_n in c(20)){

  # First, plot the NEON AOP LiDAR clipped to the GEDI footprint
  # We save the plot as an object 'p' which gives the (x,y) offset for the lower
  # left corner of the plot. The 'rgl' package offsets all (x,y) locations 
  # to this point, so we will need to subtract these values from any other 
  # (x,y) points that we want to add to the plot
  p=plot(WREF_GEDI_footprints[[shot_n]])
  
  # Extract the specific waveform from the GEDI data
  wf <- getLevel1BWF(gedilevel1b,shot_number = level1bgeo_WREF$shot_number[shot_n])
  
  # Make a new data.frame 'd' to convert the waveform data coordinates into 3D space
  d=wf@dt
  
  # normalize rxwaveform to 0-1
  d$rxwaveform=d$rxwaveform-min(d$rxwaveform)
  d$rxwaveform=d$rxwaveform/max(d$rxwaveform)
  
  # Add xy data in UTMs, and offset lower left corner of 'p'
  d$x=st_coordinates(level1bgeo_WREF_UTM[shot_n,])[1]-p[1]
  d$y=st_coordinates(level1bgeo_WREF_UTM[shot_n,])[2]-p[2]
  
  # Make a new column 'x_wf' where we place the GEDI waveform in space with the 
  # NEON AOP LiDAR data, we scale the waveform to 30m in the x-dimension, and 
  # offset by 12.5m (the radius of the GEDI footprint) in the x-dimension.
  d$x_wf=d$x+d$rxwaveform*30+12.5
  
  # Add GEDI points to 3D space in 'green' color
  # This time, subtracting the elevation difference for that shot
  points3d(x=d$x_wf, y=d$y, z=d$elevation-net_diff[shot_n], col="green", add=T)
  
}
```

##Optional - NEON base plots
You may also be interested to see if any of the GEDI footprints intersect a NEON base plot, which would allow for a direct comparison of the GEDI waveform with many of the datasets which are collected within the base plots, such as the vegetation structure data product containing height, DBH, and species identification of all trees >10cm DBH. While it is statistically pretty unlikely that a GEDI footprint will intersect with your base plot of interest, it is possible that _some_ GEDI footprint will intersetc with _some_ base plot in your study area, so we may as well take a look:


```r
setwd("./data/")

# Download the NEON TOS plots polygons directly from the NEON website
download.file(url="https://data.neonscience.org/api/v0/documents/All_NEON_TOS_Plots_V8", 
              destfile="All_NEON_TOS_Plots_V8.zip")
unzip("All_NEON_TOS_Plots_V8.zip")
NEON_all_plots <- st_read('All_NEON_TOS_Plots_V8/All_NEON_TOS_Plot_Polygons_V8.shp')
```

```
## Reading layer `All_NEON_TOS_Plot_Polygons_V8' from data source `/Users/kdw223/Research/katharynduffy.github.io/data/All_NEON_TOS_Plots_V8/All_NEON_TOS_Plot_Polygons_V8.shp' using driver `ESRI Shapefile'
## Simple feature collection with 3841 features and 36 fields
## geometry type:  POLYGON
## dimension:      XY
## bbox:           xmin: -156.6516 ymin: 17.9514 xmax: -66.82358 ymax: 71.3169
## CRS:            4326
```

```r
# Select just the WREF site
SITECODE = 'WREF'
base_plots_SPDF <- NEON_all_plots[
  (NEON_all_plots$siteID == SITECODE)&
    (NEON_all_plots$subtype == 'basePlot'),]
rm(NEON_all_plots)

base_crop=st_crop(base_plots_SPDF, extent(chm_WGS))
```

```
## although coordinates are longitude/latitude, st_intersection assumes that they are planar
```

```
## Warning: attribute variables are assumed to be spatially constant throughout all
## geometries
```

```r
plot(chm_WGS)
plot(base_crop$geometry, border = 'blue', add=T)
```

<img src="07-NEON_AOP_files/figure-html/import and plot base plots-1.png" width="672" />