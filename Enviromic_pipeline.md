# **Envirotyping Pipeline with EnvRtype**

*last update: 21th October 2020*

* [Software](#P1)
* [1. Daily weather and elevation](#P2)
* [2. Soil Features](#P3)

              
<div id="P1" />


# Software

```{r, eval=FALSE}
library(devtools)
install_github('allogamous/EnvRtype')
library(EnvRtype)
```



# Raw-Data Collection

<div id="P2" />

### 1. Daily weather and elevation

> * Creating vectors or a data.frame with latitude, longitude, environment identification and collection time intervals (beginning and end) facilitates the sampling of multiple environments.

```{r, eval=FALSE}
env.i = c("1_AN","1_PI","2_AN","2_PI") # environment ID
lat = c(-22.875,-22.705,-22.875,-22.705) # latitude coordinates
lon = c(-47.997,-47.637,-47.997,-47.637) # longitude coordinates
plant.date = c("2016-01-26","2016-01-21","2017-01-12","2017-01-10") # year-month-day
harv.date = c('2016-08-01',"2016-07-14","2017-07-25","2017-07-15")
```

> * Thus, now just put this information in the get_weather function that will connect with the NasaPower and SRTM bases using the nasapower and raster R packages.

```{r, eval=FALSE}
# collecting weather data
df.clim = get_weather(env.id = env.i,lat = lat,lon = lon,start.day = plant.date,end.day = harv.date,country = 'BRA') 
```

> * Lets consider tropical maize, with cardinals Tbase1 = 8,Tbase2 = 45,Topt1 = 30,Topt2 = 37

```{r, eval=FALSE}
# Processing Raw-weather data
df.clim = processWTH(env.data = df.clim,Tbase1 = 8,Tbase2 = 45,Topt1 = 30,Topt2 = 37)
head(df.clim) # novel processed df.clim set
```

<div id="P3" />

### 2. Soil Features and other raster-based properties

> * Soil Features can be download from https://soilgrids.org/. We made available a [ZIP file containing some soil features for the locations above described](https://github.com/allogamous/EnvRtype/blob/master/SoilGrid.zip).

>* Here we used the extract_GIS function to collect environmental covariables from raster files. Below there is a code for automatize this extraction for multiple raster files disposed into a same directory.

```{r, eval=FALSE}
home.dir = getwd()
dir = paste0(home.dir,'/SoilGrid')
unzip(zipfile = 'SoilGrid.zip')

(soil_grid = list.files(path = dir,pattern = 'tif'))
(soil_name = gsub(soil_grid,pattern = '.tif',replacement = ''))

require(raster)
env.data = data.frame(env = env.i,LAT = lat, LON = lon)
soil_data = c()
for(i in 1:length(soil_grid)) soil_data = rbind(soil_data,data.frame(Feature = soil_name[i],extract_GIS(covraster = raster(paste0(dir,'/',soil_grid[i])),name.out = 'Soil_Grid',env.data = env.data)))

head(soil_data)

```


> * Finally, you can combine the weather and soil data into a single data.frame

```{r, eval=FALSE}
require(reshape2)
(soil_data = dcast(soil_data,env~Feature,value.var = 'Soil_Grid'))
df.clim = merge(df.clim,soil_data,by='env')

```


