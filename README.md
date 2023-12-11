# rgdal_rgeos_maptools_transition
how to substitute rgdal functions with new functions from other packages

rgdal was a package archived from CRAN on 2023-10-16: <br> https://cran.r-project.org/web/packages/rgdal/index.html

about transition, description from: <br> https://www.r-bloggers.com/2023/06/upcoming-changes-to-popular-r-packages-for-spatial-data-what-you-need-to-do/


# The issue

Three popular R packages for spatial data handling won’t be available on CRAN after October 2023.1 These packages are:

rgdal: a package that provides bindings to the GDAL and PROJ libraries. In other words, it gives a capability of reading and writing spatial data formats, creating coordinate reference systems and projecting geometries.
rgeos: a package that provides bindings to the GEOS library. It allows to perform various spatial operations, such as buffering, clipping, or unioning.
maptools: an older set of various tools for manipulating spatial data pre-dating rgdal and rgeos.

As you can see, two of these packages, rgdal and rgeos link and allow access to external (non-R) spatial libraries for reading and writing spatial data formats and performing spatial operations, such as reprojections and spatial joins.

If you are a user or developer of any of these packages, you need to be aware of the upcoming changes. From the user perspective, you won’t be able to install these packages from CRAN soon, and thus your workflows will be broken. On the other hand, if you are a developer and your package uses some of the above packages, it will stop working and you won’t be able to retain (or submit) it to CRAN. Very soon you will need to find some alternatives to these packages. Gladly, most of rgdal, rgeos, and maptools functionality is already available in modern R packages, such as sf and terra, as you will see below.


# The context  

rgdal, maptools, and rgeos have a few things in common. While they are independent, they are also closely related to each other – all of them depend on the sp package for spatial data representation in R.2 Another thing they have in common is that currently we have more modern alternatives to them, such as sf (first released in 2016) and terra (2020). Finally, the main author of rgdal, maptools, and rgeos is the same person – Roger Bivand, who retired from his position at the Norwegian School of Economics in 2021.

                                       
# How to test if your code is affected? 
                                    
The most basic way to test if your code is affected by the upcoming changes is to check if you use rgdal, rgeos, and maptools in your scripts or as dependencies in your packages. If so – your workflows may be broken soon!

There is also a possibility that you use the sp package but not the affected packages. In this case, you may still be touched by the changes, because the sp package had interacted with rgdal and rgeos in the background. For example, if you run the spTransform() function from the sp package, it used the rgdal package in the background. Thus, you may add the following code to your script to check if it will work in the future:

options("sp_evolution_status" = 2) # use sf instead of rgdal and rgeos in sp 
<br> library(sp)

If you get an error, it means that your code is affected by the changes.

There is also another possible situation – you are using some packages that depend on the affected packages. This one is the most difficult to check, because you need to inspect the dependencies of all the packages you use.

                                     
# Solutions for users                                                

My first advice would be: do yourself a favor, and just stop using the retired packages today. Write your new scripts using modern alternatives, such as sf or terra. In most cases they are easy to use, and you will be able to find a lot of examples and other resources online.

If you have some old scripts that use the retired packages, you have a few options and your decision should depend on your circumstances. If you do not plan to use them in the future, you can just leave them as they are. On the other hand, if you want to perform similar spatial operations, you should consider rewriting the scripts using modern alternatives – see the tables below for the equivalents of the most popular functions from rgdal, rgeos, and maptools.

You also may have old sp objects that you want to use in the future. Then, you can convert them to the modern equivalents with functions such as 

sf::st_as_sf() <br>
or <br>
terra::vect()

                                      
# Solutions for package developers    

Look at your package dependencies and check if you use any of the affected packages. If that is the case, you need to find alternatives to them. In general, there are two main alternative options:

sf package, which is a modern alternative to rgdal and rgeos, and also provides a spatial vector data representation in R.
terra package, which gives support to working with spatial vector and raster data in R.

install.packages("sf") <br>
library(sf) <br>
install.packages("terra") <br>
library(terra) 

You just need to be aware that functions in these packages are not always identical to the ones in the affected packages. They may have different arguments, different defaults, expect different inputs, or return different outputs. For example, rgdal’s 

readOGR() <br> 
function returns a Spatial*DataFrame object, while sf’s <br> 
read_sf() <br> 
returns an sf object and terra’s <br> 
vect() <br> 
returns a SpatVector object.


# Alternative functions         
                                      
| rgdal |                 | sf	                  | terra
| ----------------------- | --------------------- | ------------------
| readOGR()	              | read_sf()	            | vect()
| writeOGR()	            | write_sf()	          | writeVector()

The rgeos functions also have their equivalents in sf and terra.

Interestingly, the most often used maptools functions have been already deprecated for a long time and alternative tools for the same purposes existed, e.g.,

unionSpatialPolygons() <br>
could be replaced with <br>
rgeos::gUnaryUnion() <br>
and <br>
maptools::spRbind() <br>
with sp::rbind()<br>

# The table below shows their modern substitutes:

rgeos	                    sf	                      terra
gArea()	                st_area()	                  expanse()
gBuffer()	              st_buffer()	                buffer()
gCentroid()	            st_centroid()	              centroids()
gDistance()	            st_distance()	              distance()
gIntersection()	        st_intersection()	          crop()
gIntersects()	          st_intersects()	            relate()

maptools	                    sf	                    terra
unionSpatialPolygons()	    st_union()	              aggregate()
spRbind()	                  rbind()	                  rbind()

The sp package is not going to be removed from CRAN soon, but it is good to stop using it because it is no longer being actively developed. Here you can find some replacements for its basic functions:

sp	                    sf	                        terra
bbox()	                st_bbox()	                  ext()
coordinates()	          st_coordinates()	          crds()
identicalCRS()	        st_crs(x) == st_crs(y)	    crs(x) == crs(y)
over()	                st_intersects()	            relate()
point.in.polygon()	    st_intersects()	            relate()
proj4string()	          st_crs()	                  crs()
spsample()	            st_sample()	                spatSample()
spTransform()	          st_transform()	            project()

You also may want to visit the comparison table:
https://github.com/r-spatial/evolution/blob/main/pkgapi_230305_refs.csv
and the migration wiki:
https://github.com/r-spatial/sf/wiki/Migrating
to get more complete lists of alternative functions to the ones from rgdal, rgeos, and maptools.

Additionally, you may still want to accept sp objects as inputs and return them as outputs of your functions. In such case, you can use the 

sf::st_as_sf() 
or 
terra::vect() 

functions to convert sp objects to the modern equivalents, perform required spatial operations with sf/terra, and then convert the results back to sp objects with as(x, "Spatial"). Packages depending on sp would also need to add sf as their weak dependency.

