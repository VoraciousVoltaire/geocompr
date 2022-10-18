# Bridges to GIS software {#gis}

## Prerequisites {-}

- This chapter requires QGIS\index{QGIS}, SAGA\index{SAGA} and GRASS\index{GRASS} to be installed and the following packages to be attached:


```r
library(sf)
library(terra)
```

<!--toDo:jn-->
<!-- qgisprocess to CRAN -->


```r
# remotes::install_github("paleolimbot/qgisprocess")
library(qgisprocess)
library(Rsagacmd)
library(rgrass)
library(rstac)
library(gdalcubes)
```

## Introduction

A defining feature of R is the way you interact with it:
you type commands and hit `Enter` (or `Ctrl+Enter` if writing code in the source editor in RStudio\index{RStudio}) to execute them interactively.
This way of interacting with the computer is called a command-line interface (CLI)\index{command-line interface} (see definition in the note below).
CLIs are not unique to R.^[
Other 'command-lines' include terminals for interacting with the operating system and other interpreted languages such as Python.
Many GISs originated as a CLI:
it was only after the widespread uptake of computer mice and high-resolution screens in the 1990s that GUIs\index{graphical user interface} became common.
GRASS, one of the longest-standing GIS\index{GIS} programs, for example, relied primarily on command-line interaction before it gained a sophisticated GUI [@landa_new_2008].
]
In dedicated GIS\index{GIS} packages, by contrast, the emphasis tends to be on the graphical user interface (GUI)\index{graphical user interface}.
You *can* interact with QGIS\index{QGIS}, SAGA\index{SAGA}, GRASS\index{GRASS} and gvSIG from system terminals and embedded CLIs\index{command-line interface}, but 'pointing and clicking' is the norm.
This means many GIS\index{GIS} users miss out on the advantages of the command-line according to Gary Sherman, creator of QGIS\index{QGIS} [@sherman_desktop_2008]:

> With the advent of 'modern' GIS software, most people want to point and click their way through life. That’s good, but there is a tremendous amount of flexibility and power waiting for you with the command line. Many times you can do something on the command line in a fraction of the time you can do it with a GUI.

The 'CLI vs GUI'\index{graphical user interface} debate can be adversial but it does not have to be; both options can be used interchangeably, depending on the task at hand and the user's skillset.^[GRASS GIS and PostGIS are popular in academia and industry and can be seen as products which buck this trend as they are built around the command-line.]
The advantages of a good CLI\index{command-line interface} such as that provided by R (and enhanced by IDEs\index{IDE} such as RStudio\index{RStudio}) are numerous.
A good CLI:

- Facilitates the automation of repetitive tasks
- Enables transparency and reproducibility, the backbone of good scientific practice and data science
- Encourages software development by providing tools to modify existing functions and implement new ones
- Helps develop future-proof programming skills which are in high demand in many disciplines and industries
- Is user-friendly and fast, allowing an efficient workflow

On the other hand, GUI-based GIS\index{GIS} systems (particularly QGIS\index{QGIS}) are also advantageous.
A good GIS GUI:

- Has a 'shallow' learning curve meaning geographic data can be explored and visualized without hours of learning a new language
- Provides excellent support for 'digitizing' (creating new vector datasets), including trace, snap and topological tools^[
The **mapedit** package allows the quick editing of a few spatial features but not professional, large-scale cartographic digitizing.
]
- Enables georeferencing (matching raster images to existing maps) with ground control points and orthorectification
- Supports stereoscopic mapping (e.g., LiDAR and structure from motion)
- Provides access to spatial database management systems with object-oriented relational data models, topology and fast (spatial) querying

Another advantage of dedicated GISs is that they provide access to hundreds of 'geoalgorithms' (computational recipes to solve geographic problems --- see Chapter \@ref(algorithms)).
Many of these are unavailable from the R command line, except via 'GIS bridges', the topic of (and motivation for) this chapter.^[
An early use of the term 'bridge' referred to the coupling of R with GRASS\index{GRASS} [@neteler_open_2008].
]

\BeginKnitrBlock{rmdnote}<div class="rmdnote">A command-line interface is a means of interacting with computer programs in which the user issues commands via successive lines of text (command lines).
`bash` in Linux and `PowerShell` in Windows are its common examples.
CLIs can be augmented with IDEs such as RStudio for R, which provides code auto-completion and other features to improve the user experience.</div>\EndKnitrBlock{rmdnote}

R originated as an interface language.
Its predecessor S provided access to statistical algorithms in other languages (particularly FORTRAN\index{FORTRAN}), but from an intuitive read-evaluate-print loop (REPL) [@chambers_extending_2016].
R continues this tradition with interfaces to numerous languages, notably C++\index{C++}, as described in Chapter \@ref(intro).
R was not designed as a GIS.
However, its ability to interface with dedicated GISs gives it astonishing geospatial capabilities.
R is well known as a statistical programming language, but many people are unaware of its ability to replicate GIS workflows, with the additional benefits of a (relatively) consistent CLI.
Furthermore, R outperforms GISs in some areas of geocomputation\index{geocomputation}, including interactive/animated map making (see Chapter \@ref(adv-map)) and spatial statistical modeling (see Chapter \@ref(spatial-cv)).
This chapter focuses on 'bridges' to three mature open source GIS products (see Table \@ref(tab:gis-comp)): QGIS\index{QGIS} (via the package **qgisprocess**\index{qgisprocess (package)}; Section \@ref(rqgis)), SAGA\index{SAGA} (via **Rsagacmd**\index{Rsagacmd (package)}; Section \@ref(saga)) and GRASS\index{GRASS} (via **rgrass**\index{rgrass (package)}; Section \@ref(grass)).^[
Though not covered here, it is worth being aware of so-called R-ArcGIS bridge (see https://github.com/R-ArcGIS/r-bridge) that allows R to be used from within ArcGIS\index{ArcGIS}
One can also use R scripts from within QGIS\index{QGIS} (see https://docs.qgis.org/3.16/en/docs/training_manual/processing/r_intro.html).
Finally, it is also possible to use R from the GRASS GIS\index{GRASS} command line (see https://grasswiki.osgeo.org/wiki/R_statistics/rgrass).
]


Table: (\#tab:gis-comp)Comparison between three open-source GIS. Hybrid refers to the support of vector and raster operations.

|GIS   |First release |No. functions |Support |
|:-----|:-------------|:-------------|:-------|
|QGIS  |2002          |>1000         |hybrid  |
|SAGA  |2004          |>600          |hybrid  |
|GRASS |1982          |>500          |hybrid  |

To complement the R-GIS bridges, the second part of the chapter gives a brief introduction to interfaces to spatial libraries (Section \@ref(gdal)), spatial databases\index{spatial database} (Section \@ref(postgis)), and cloud-based processing of Earth observation data (Section \@ref(cloud)).

## QGIS through **qgisprocess** {#rqgis}

<!--toDo:jn-->
<!-- how to mention/use the QGIS package (https://en.cahik.cz//2022/03/07/r-package-qgis/) here? -->

QGIS\index{QGIS} is one of the most popular open-source GIS [Table \@ref(tab:gis-comp); @graser_processing_2015]. 
Its main advantage lies in the fact that it provides a unified interface to several other open-source GIS.
This means that you have access to GDAL\index{GDAL}, GRASS\index{GRASS}, and SAGA\index{SAGA} through QGIS\index{QGIS} [@graser_processing_2015]. 
Since version 3.14, QGIS provides a command line API\index{API}, `qgis_process`, that allows to run all these geoalgorithms (frequently more than 1000, depending on your set-up) outside of the QGIS GUI.

The **qgisprocess** package \index{qgisprocess (package)} wraps this QGIS command-line utility, and thus makes it possible to call QGIS, GDAL, GRASS, and SAGA algorithms from the R session.
Before running **qgisprocess**\index{qgisprocess (package)}, make sure you have installed QGIS\index{QGIS} and all its (third-party) dependencies such as SAGA\index{SAGA} and GRASS\index{GRASS}.


```r
library(qgisprocess)
#> Using 'qgis_process' in the system PATH.
#> QGIS version: 3.26.1-Buenos Aires
#> ...
```

This package automatically tries to detect a QGIS installation and complains if it cannot find it.^[You can see details of the detection process with `qgis_configure()`.]
There are a few possible solutions when the configuration fails: you can set `options(qgisprocess.path = "path/to/your_qgis_process")`, or set up the `R_QGISPROCESS_PATH` environment variable.
<!--toDo:jn-->
<!-- link to the vignette https://github.com/paleolimbot/qgisprocess/pull/31/files when it is online -->
The above approaches can also be used when you have more than one QGIS installation and want to decide which one to use.

Next, we can find which providers (meaning different software) are available on our computer.


```r
qgis_providers()
#> # A tibble: 6 × 2
#>   provider provider_title   
#>   <chr>    <chr>            
#> 1 3d       QGIS (3D)        
#> 2 gdal     GDAL             
#> 3 grass7   GRASS            
#> 4 native   QGIS (native c++)
#> 5 qgis     QGIS             
#> 6 saga     SAGA
```

The output table affirms that we can use QGIS geoalgorithms (`native`, `qgis`, `3d`) and external ones from the third-party providers GDAL, SAGA and GRASS through the QGIS interface.

We are now ready for some QGIS geocomputation from within R!
Let's try two example case studies.
The first one shows how to unite two polygonal datasets with different borders\index{union} (Section \@ref(qgis-vector)).
The second one focuses on deriving new information from a digital elevation model represented as a raster (Section \@ref(qgis-raster)).

### Vector data {#qgis-vector}

Consider a situation when you have two polygon objects with different spatial units (e.g., regions, administrative units).
Our goal is to merge these two objects into one, containing all of the boundary lines and related attributes.
We use again the incongruent polygons we have already encountered in Section \@ref(incongruent) (Figure \@ref(fig:uniondata)).
Both polygon datasets are available in the **spData** package, and for both we would like to use a geographic CRS\index{CRS!geographic} (see also Chapter \@ref(reproj-geo-data)).


```r
data("incongruent", "aggregating_zones", package = "spData")
incongr_wgs = st_transform(incongruent, "EPSG:4326")
aggzone_wgs = st_transform(aggregating_zones, "EPSG:4326")
```

<div class="figure" style="text-align: center">
<img src="10-gis_files/figure-html/uniondata-1.png" alt="Illustration of two areal units: incongruent (black lines) and aggregating zones (red borders). " width="100%" />
<p class="caption">(\#fig:uniondata)Illustration of two areal units: incongruent (black lines) and aggregating zones (red borders). </p>
</div>

To find an algorithm to do this work, we can search the output of the `qgis_algorithms()` function.
This function returns a data frame containing all of the available providers and the algorithms they contain.^[Therefore, if you cannot see an expected provider, it is probably because you still need to install some external GIS software.] 


```r
qgis_algo = qgis_algorithms()
```

The `qgis_algo` object has a lot of columns, but usually, we are only interested in the `algorithm` column that combines information about the provider and the algorithm name.
Assuming that the short description of the function contains the word "union"\index{union}, we can run the following code to find the algorithm of interest:


```r
grep("union", qgis_algo$algorithm, value = TRUE)
#> [1] "native:multiunion" "native:union"      "saga:fuzzyunionor" "saga:polygonunion"
```

One of the algorithms on the above list, `"native:union"`, sounds promising.
The next step is to find out what this algorithm does and how we can use it.
This is the role of the `qgis_show_help()`, which returns a short summary of what the algorithm does, its arguments, and outputs.^[We can also extract some of information independently with `qgis_description()`, `qgis_arguments()`, and `qgis_outputs()`.]
This makes it output rather long.
Here, we only present the arguments`"native:union"` expects.


```r
alg = "native:union"
qgis_arguments(alg) |> 
  dplyr::mutate(acceptable_values = unlist(acceptable_values)) |>
  dplyr::select(name, description, acceptable_values)
```

Apparently, these are `INPUT`, `OVERLAY`, `OVERLAY_FIELDS_PREFIX`, and `OUTPUT`.
It seems that some of the above arguments expect a "path to a vector layer" (column: `acceptaple_values`).
However, the **qgisprocess** package also allows to provide `sf` objects as well in these cases.^[Objects from the **terra** and **stars** package can be used when a "path to a raster layer" is expected.]
Though this is really convenient, if your spatial data is already available in your R session, we recommend to provide the path to your spatial data on disk when you only read it in to submit it to a **qgisprocess** algorithm because the first thing **qgisprocess** does when executing a geoalgorithm is to export the spatial data living in your R session back to disk in a format known to QGIS such as .gpkg or .tif files.

Finally, we can let QGIS\index{QGIS} do the work.
The main function of **qgisprocess** is `qgis_run_algorithm()`.
It accepts the used algorithm name and a set of named arguments shown in the help list, and performs expected calculations.
In our case, three arguments seem important - `INPUT`, `OVERLAY`, and `OUTPUT`.
The first one, `INPUT`, is our main vector object `incongr_wgs`, while the second one, `OVERLAY`, is `aggzone_wgs`.
The last argument, `OUTPUT`, expects a path to a new vector file; however, if we do not provide the path, **qgisprocess** will automatically create a temporary file.
The `.quiet` argument only tells **qgisprocess** to be less verbose.


```r
union = qgis_run_algorithm(alg, INPUT = incongr_wgs, OVERLAY = aggzone_wgs, 
                           .quiet = TRUE)
union
```

Running the above line of code will save our two input objects into temporary .gpkg files, run the selected algorithm on them, and return a temporary .gpkg file as the output.
The **qgisprocess** package stores the `qgis_run_algorithm()` result as a list containing, in this case, a path to the output file.
We can either read this file back into R with `read_sf()` (e.g., `union_sf = read_sf(union[[1]])`) or directly with `st_as_sf()`:


```r
union_sf = st_as_sf(union)
```

Note that the QGIS\index{QGIS} union\index{vector!union} operation merges the two input layers into one layer by using the intersection\index{vector!intersection} and the symmetrical difference of the two input layers (which, by the way, is also the default when doing a union operation in GRASS\index{GRASS} and SAGA\index{SAGA}).
This is **not** the same as `st_union(incongr_wgs, aggzone_wgs)` (see Exercises)!

Our result, `union_sf`, is a multipolygon with a larger number of features than two input objects .
Notice, however, that many of these polygons are small and do not represent real areas but are rather a result of our two datasets having a different level of detail.
These artifacts of error are called sliver polygons (see red-colored polygons in the left panel of Figure \@ref(fig:sliver))
One way to identify slivers is to find polygons with comparatively very small areas, here, e.g., 25000 m^2^, and next remove them.
Let's search for an appropriate algorithm.


```r
grep("clean", qgis_algo$algorithm, value = TRUE)
```

This time the found algorithm, `v.clean`, is not included in QGIS, but GRASS GIS.
GRASS GIS's `v.clean` is a powerful tool for cleaning topology of spatial vector data. 
Importantly, we can use it through **qgisprocess**.

Similarly to the previous step, we should start by looking at this algorithm's help.


```r
qgis_show_help("grass7:v.clean")
```

We have omitted the output here, because the help text is quite long and contains a lot of arguments.^[Also note that these arguments, contrary to the QGIS's ones, are in lower case.]
This is because `v.clean` is a multi tool -- it can clean different types of geometries and solve different types of topological problems.
For this example, let's focus on just a few arguments, however, we encourage you to visit [this algorithm's documentation](https://grass.osgeo.org/grass78/manuals/v.clean.html) to learn more about `v.clean` capabilities.


```r
qgis_arguments("grass7:v.clean") |>
  dplyr::select(name, description) |>
  dplyr::slice_head(n = 4)
#> # A tibble: 4 × 2
#>   name      description                              
#>   <chr>     <chr>                                    
#> 1 input     Layer to clean                           
#> 2 type      Input feature type                       
#> 3 tool      Cleaning tool                            
#> 4 threshold Threshold (comma separated for each tool)
```

The main argument for this algorithm is `input` -- our vector object.
Next, we need to select a tool -- a cleaning method. ^[It is also possible to select several tools, which will then be executed sequentially.]
About a dozen tools exist in `v.clean` allowing to remove duplicate geometries, remove small angles between lines, or remove small areas, among others.
In this case, we are interested in the latter tool, `rmarea`.
Several of the tools, `rmarea` included, expect an additional argument `threshold`, whose behavior depends on the selected tool.
In our case, the `rmarea` tool removes all areas smaller or equal to a provided `threshold`. 

Let's run this algorithm and convert its output into a new `sf` object `clean_sf`.


```r
clean = qgis_run_algorithm("grass7:v.clean", input = union_sf,
                           tool = "rmarea", threshold = 25000, .quiet = TRUE)
clean_sf = st_as_sf(clean)
```

The result, the right panel of \@ref(fig:sliver), looks as expected -- sliver polygons are now removed.

<div class="figure" style="text-align: center">
<img src="figures/10-sliver.png" alt="Sliver polygons colored in red (left panel). Cleaned polygons (right panel)." width="100%" />
<p class="caption">(\#fig:sliver)Sliver polygons colored in red (left panel). Cleaned polygons (right panel).</p>
</div>

### Raster data {#qgis-raster}

Digital elevation models (DEMs) contain elevation information for each raster cell.
They are used for many purposes, including satellite navigation, water flow models, surface analysis, or visualization.
Here, we are interested in deriving new information from a DEM raster that could be used as predictors for statistical learning.
Various terrain parameters, for example, can be helpful for the prediction of landslides (see Chapter \@ref(spatial-cv))

For this section, we will use `dem.tif` -- a digital elevation model\index{digital elevation model} of the Mongón study area (downloaded from the Land Process Distributed Active Archive Center, see also `?dem.tif`)
It has a resolution of about 30 by 30 meters and uses a projected CRS.


```r
library(qgisprocess)
library(terra)
dem = rast(system.file("raster/dem.tif", package = "spDataLarge"))
```

The **terra** package's `terrain()` command already allows the calculation of several fundamental topographic characteristics such as slope, aspect, TPI (*Topographic Position Index*), TRI (*Topographic Ruggedness Index*), roughness, and flow directions.
However, GIS programs offer many more terrain characteristics, some of which can be more suitable in certain contexts.
For example, the topographic wetness index (TWI) was found useful in studying hydrological and biological processes [@sorensen_calculation_2006].
Let's search the algorithm list for this index using `"wetness"` as keyword.


```r
qgis_algo = qgis_algorithms()
grep("wetness", qgis_algo$algorithm, value = TRUE)
```

An output of the above code suggests that the desired algorithm exists in the SAGA GIS software.^[TWI can be also calculated using the `r.topidx` GRASS GIS function.]
Though SAGA is a hybrid GIS, its main focus has been on raster processing, and here, particularly on digital elevation models\index{digital elevation model} (soil properties, terrain attributes, climate parameters). 
Hence, SAGA is especially good at the fast processing of large (high-resolution) raster\index{raster} datasets [@conrad_system_2015].

The `"saga:sagawetnessindex"` algorithm is actually a modified TWI, that results in a more realistic soil moisture potential for the cells located in valley floors [@bohner_spatial_2006].


```r
qgis_show_help("saga:sagawetnessindex")
# output not shown here
```

Here, we stick with the default values for all arguments.
Therefore, we only have to specify one argument -- the input `DEM`.
Of course, when applying this algorithm you should make sure that the default values are in correspondence with your study aim.^[The additional arguments of `"saga:sagawetnessindex"` are well-explained at https://gis.stackexchange.com/a/323454/20955.]
`"saga:sagawetnessindex"` returns not one but four rasters -- catchment area, catchment slope, modified catchment area, and topographic wetness index.


```r
dem_wetness = qgis_run_algorithm("saga:sagawetnessindex", DEM = dem, 
                                 .quiet = TRUE)
```

The result, `dem_wetness`, is a list with file paths to the four outputs.
We can read a selected output by providing an output name in the `qgis_as_terra()` function.


```r
dem_wetness_twi = qgis_as_terra(dem_wetness$TWI)
```

You can see the output TWI map on the left panel of Figure \@ref(fig:qgis-raster-map).
The topographic wetness index is unitless.
Its low values represent areas that will not accumulate water, while higher values show areas that will accumulate water at increasing levels.

Information from digital elevation models can also be categorized, for example, to geomorphons -- the geomorphological phonotypes consisting of 10 classes that represent terrain forms, such as slopes, ridges, or valleys [@jasiewicz_geomorphons_2013].
These phonotypes are used in many studies, including landslide susceptibility, ecosystem services, human mobility, and digital soil mapping. 

The original implementation of the geomorphons' algorithm was created in GRASS GIS, and we can find it in the **qgisprocess** list as `"grass7:r.geomorphon"`:


```r
grep("geomorphon", qgis_algo$algorithm, value = TRUE)
#> [1] "grass7:r.geomorphon"
qgis_show_help("grass7:r.geomorphon")
# output not shown
```

Calculation of geomorphons requires an input DEM (`elevation`), and can be customized with a set of optional arguments.
It includes, `search` -- a length for which the line-of-sight is calculated, and ``-m`` -- a flag specifying that the search value will be provided in meters (and not the number of cells).
More information about additional arguments can be found in the original paper and the [GRASS GIS documentation](https://grass.osgeo.org/grass78/manuals/r.geomorphon.html).


```r
dem_geomorph = qgis_run_algorithm("grass7:r.geomorphon", elevation = dem, 
                                    `-m` = TRUE, search = 120, .quiet = TRUE)
```

Our output, `dem_geomorph$forms`, contains a raster file with 10 categories -- each one representing a terrain form.
We can read it into R with `qgis_as_terra()`, and then visualize it (the right panel of Figure \@ref(fig:qgis-raster-map)) or use it in our subsequent calculations.


```r
dem_geomorph_terra = qgis_as_terra(dem_geomorph$forms)
```

Interestingly, there are connections between some geomorphons and the TWI values, as shown in Figure \@ref(fig:qgis-raster-map).
The largest TWI values mostly occur in valleys and hollows, while the lowest values are seen, as expected, on ridges.

<div class="figure" style="text-align: center">
<img src="figures/10-qgis-raster-map.png" alt="Topographic wetness index (TWI, left panel) and geomorphons (right panel) derived for the Mongón study area." width="100%" />
<p class="caption">(\#fig:qgis-raster-map)Topographic wetness index (TWI, left panel) and geomorphons (right panel) derived for the Mongón study area.</p>
</div>

## SAGA GIS {#saga}

The System for Automated Geoscientific Analyses (SAGA\index{SAGA}; Table \@ref(tab:gis-comp)) provides the possibility to execute SAGA modules via the command line interface\index{command-line interface} (`saga_cmd.exe` under Windows and just `saga_cmd` under Linux) (see the [SAGA wiki on modules](https://sourceforge.net/p/saga-gis/wiki/Executing%20Modules%20with%20SAGA%20CMD/)).
In addition, there is a Python interface (SAGA Python API\index{API}).
**Rsagacmd**\index{Rsagacmd (package)} uses the former to run SAGA\index{SAGA} from within R.

We will use **Rsagacmd** in this section to delineate areas with similar values of the normalized difference vegetation index (NDVI) of the Mongón study area in Peru from the 22nd of September 2000 (the left panel of Figure \@ref(fig:sagasegments)) by using a seeded region growing algorithm from SAGA GIS.^[Read Section \@ref(local-operations) on details of how to calculate NDVI from a remote sensing image.]


```r
ndvi = rast(system.file("raster/ndvi.tif", package = "spDataLarge"))
```

To start using **Rsagacmd**, we need to run the `saga_gis()` function.
It serves two main purposes: 

- It dynamically^[This means that the available libraries will depend on the installed SAGA GIS version.] creates a new object that contains links to all valid SAGA-GIS libraries and tools
- It sets up general package options, such as `raster_backend` (R package to use for handling raster data), `vector_backend` (R package to use for handling vector data), and `cores` (a maximum number of CPU cores used for processing, default: all)


```r
library(Rsagacmd)
saga = saga_gis(raster_backend = "terra", vector_backend = "sf")
```

Our `saga` object contains connections to all of the available SAGA tools.
It is organized as a list of libraries (groups of tools), and inside of a library it has a list of tools.
We can access any tool with the `$` sign (remember to use TAB for autocompletion).

The seeded region growing algorithm works in two main steps [@adams_seeded_1994;@bohner_image_2006].
First, initial cells ("seeds") are generated by finding the cells with the smallest variance in local windows of a specified size.
Then, the region growing algorithm is used to merge neighboring pixels of the seeds to create homogeneous areas.


```r
sg = saga$imagery_segmentation$seed_generation
```

In the above example, we first opened the `imagery_segmentation` library and then its `seed_generation` tool.
We also assigned it to the `sg` object, not to retype the whole tool code in our next steps.^[You can read more about the tool at https://saga-gis.sourceforge.io/saga_tool_doc/8.3.0/imagery_segmentation_2.html.]
If we just type `sg`, we will get a quick summary of the tool and a data frame with its parameters, descriptions, and defaults.
You may also use `tidy(sg)` to extract just the parameters' table.
The `seed_generation` tool requires at least one input (`features`): a raster data.
We are able to provide a set of additional parameters, including `band_width` that specifies the size of initial polygons.


```r
ndvi_seeds = sg(ndvi, band_width = 2)
plot(ndvi_seeds$seed_grid)
```

Our output is a list of three objects: `variance` -- a raster map of local variance, `seed_grid` -- a raster map with the generated seeds, and `seed_points` -- a spatial vector object with the generated seeds.

The second SAGA GIS tool we use is `seeded_region_growing`.^[You can read more about the tool at https://saga-gis.sourceforge.io/saga_tool_doc/8.3.0/imagery_segmentation_3.html.]
The `seeded_region_growing` tool requires two inputs: our `seed_grid` calculated in the previous step and the `ndvi` raster object.
Additionally, we can specify several parameters, such as `normalize` to standardize the input features, `neighbour` (4 or 8-neighborhood), and `method`.
The last parameter can be set to either `0` or `1` (region growing is based on raster cells' values and their positions or just the values).
For a more detailed description of the method, see @bohner_image_2006.

Here, we will only change `method` to `1`, meaning that our output regions will be created only based on the similarity of their NDVI values.


```r
srg = saga$imagery_segmentation$seeded_region_growing
ndvi_srg = srg(ndvi_seeds$seed_grid, ndvi, method = 1)
plot(ndvi_srg$segments)
```

The tool returns a list of three objects: `segments`, `similarity`, `table`.
The `similarity` object is a raster showing similarity between the seeds and the other cells, and `table` is a data frame storing information about the input seeds.
Finally, `ndvi_srg$segments` is a raster with our resulting areas (the right panel of Figure \@ref(fig:sagasegments)).
We can convert it into polygons with `as.polygons()` and `st_as_sf()` (Section \@ref(spatial-vectorization)).


```r
ndvi_segments = as.polygons(ndvi_srg$segments) |> 
  st_as_sf()
```

<div class="figure" style="text-align: center">
<img src="figures/10-saga-segments.png" alt="Normalized difference vegetation index (NDVI, left panel) and NDVi-based segments derived using t he seeded region growing algorithm for the Mongón study area." width="100%" />
<p class="caption">(\#fig:sagasegments)Normalized difference vegetation index (NDVI, left panel) and NDVi-based segments derived using t he seeded region growing algorithm for the Mongón study area.</p>
</div>

The resulting polygons (segments) represent areas with similar values. 
They can also be further aggregated into larger polygons using various techniques, such as clustering (e.g., k-means), regionalization (e.g., SKATER) or supervised classification methods.
You can try to do it in Exercises.

R also has other tools to achieve the goal of creating polygons with similar values (so-called segments).
It includes the **SegOptim** package [@goncalves_segoptim_2019] that allows running several image segmentation algorithms and **supercells** [@nowosad_extended_2022] that implements superpixels algorithm SLIC to work with geospatial data.

## GRASS GIS {#grass}

The U.S. Army - Construction Engineering Research Laboratory (USA-CERL) created the core of the Geographical Resources Analysis Support System (GRASS)\index{GRASS} (Table \@ref(tab:gis-comp); @neteler_open_2008) from 1982 to 1995. 
Academia continued this work since 1997.
Similar to SAGA\index{SAGA}, GRASS focused on raster processing in the beginning while only later, since GRASS 6.0, adding advanced vector functionality [@bivand_applied_2013].

Here, we introduce **rgrass**\index{rgrass (package)} with one of the most interesting problems in GIScience - the traveling salesman problem\index{traveling salesman}.
Suppose a traveling salesman would like to visit 24 customers.
Additionally, he would like to start and finish his journey at home which makes a total of 25 locations while covering the shortest distance possible.
There is a single best solution to this problem; however, to check all of the possible solutions it is (mostly) impossible for modern computers [@longley_geographic_2015].
In our case, the number of possible solutions correspond to `(25 - 1)! / 2`, i.e., the factorial of 24 divided by 2 (since we do not differentiate between forward or backward direction).
Even if one iteration can be done in a nanosecond, this still corresponds to 9837145 years.
Luckily, there are clever, almost optimal solutions which run in a tiny fraction of this inconceivable amount of time.
GRASS GIS\index{GRASS} provides one of these solutions (for more details, see [v.net.salesman](https://grass.osgeo.org/grass80/manuals/v.net.salesman.html)).
In our use case, we would like to find the shortest path\index{shortest route} between the first 25 bicycle stations (instead of customers) on London's streets (and we simply assume that the first bike station corresponds to the home of our traveling salesman\index{traveling salesman}).


```r
data("cycle_hire", package = "spData")
points = cycle_hire[1:25, ]
```

Aside from the cycle hire points data, we need a street network for this area.
We can download it with from OpenStreetMap\index{OpenStreetMap} with the help of the **osmdata** \index{osmdata (package)} package (see also Section \@ref(retrieving-data)).
To do this, we constrain the query of the street network (in OSM language called "highway") to the bounding box\index{bounding box} of `points`, and attach the corresponding data as an `sf`-object\index{sf}.
`osmdata_sf()` returns a list with several spatial objects (points, lines, polygons, etc.), but here, we only keep the line objects with their related ids.^[As a convenience to the reader, one can attach `london_streets` to the global environment using `data("london_streets", package = "spDataLarge")`.]


```r
library(osmdata)
b_box = st_bbox(points)
london_streets = opq(b_box) |>
  add_osm_feature(key = "highway") |>
  osmdata_sf() 
london_streets = london_streets[["osm_lines"]]
london_streets = dplyr::select(london_streets, osm_id)
```

Now that we have the data, we can go on and initiate a GRASS\index{GRASS} session.
Luckily, `linkGRASS()` of the **link2GI** packages lets to set up the GRASS environment with just one line of code.^[For a more complete description of setting up the GRASS environment read the appendix at the end of this section.]
The only thing you need to provide is a spatial object which determines the projection and the extent of the spatial database\index{spatial database}.
First, `linkGRASS()` finds all GRASS\index{GRASS} installations on your computer.
Since we have set `ver_select` to `TRUE`, we can interactively choose one of the found GRASS-installations.
If there is just one installation, the `linkGRASS()` automatically chooses it.
Second, `linkGRASS()` establishes a connection to GRASS GIS.


```r
library(rgrass)
link2GI::linkGRASS(london_streets, ver_select = TRUE)
```

Before we can use GRASS geoalgorithms\index{geoalgorithm}, we need to add data to GRASS's spatial database\index{spatial database}.
Luckily, the convenience function `write_VECT()` does this for us.
(Use `write_RAST()` for raster data.)
In our case, we add the street and cycle hire point data while using only the first attribute column, and name them as `london_streets` and `points` in GRASS.


```r
write_VECT(terra::vect(london_streets), vname = "london_streets")
write_VECT(terra::vect(points[, 1]), vname = "points")
```

The **rgrass** package expects its inputs and gives its outputs as **terra** objects. 
Therefore, we need to convert our `sf` spatial vectors to **terra**'s `SpatVector`s using the `vect()` function to be able to use `write_VECT()`.^[You can learn more how to convert between spatial classes in R by reading the (Conversions between different spatial classes in R)[https://geocompr.github.io/post/2021/spatial-classes-conversion/] blog post and the 
(Coercion between object formats)[https://CRAN.R-project.org/package=rgrass/vignettes/coerce.html] vignette] 

Now, both of datasets exist in the GRASS GIS database.
To perform our network\index{network} analysis, we need a topological clean street network.
GRASS's `"v.clean"` takes care of the removal of duplicates, small angles and dangles, among others.
Here, we break lines at each intersection to ensure that the subsequent routing algorithm can actually turn right or left at an intersection, and save the output in a GRASS object named `streets_clean`.


```r
execGRASS(cmd = "v.clean", input = "london_streets", output = "streets_clean",
          tool = "break", flags = "overwrite")
```

\BeginKnitrBlock{rmdnote}<div class="rmdnote">To learn about the possible arguments and flags of the GRASS GIS modules you can you the `help` flag.
For example, try `execGRASS("g.region", flags = "help")`.</div>\EndKnitrBlock{rmdnote}

It is likely that a few of our cycling station points will not lie exactly on a street segment.
However, to find the shortest route\index{shortest route} between them, we need to connect them to the nearest streets segment.
`"v.net"`'s connect-operator does exactly this.
We save its output in `streets_points_con`.


```r
execGRASS(cmd = "v.net", input = "streets_clean", output = "streets_points_con",
          points = "points", operation = "connect", threshold = 0.001,
          flags = c("overwrite", "c"))
```

The resulting clean dataset serves as input for the `"v.net.salesman"` algorithm, which finally finds the shortest route between all cycle hire stations.
One of its arguments is `center_cats`, which requires a numeric range as input.
This range represents the points for which a shortest route should be calculated.
Since we would like to calculate the route for all cycle stations, we set it to `1-25`.
To access the GRASS help page of the traveling salesman\index{traveling salesman} algorithm\index{algorithm}, run `execGRASS("g.manual", entry = "v.net.salesman")`.


```r
execGRASS(cmd = "v.net.salesman", input = "streets_points_con",
          output = "shortest_route", center_cats = paste0("1-", nrow(points)),
          flags = "overwrite")
```

To see our result, we read the result into R, convert it into an sf-object keeping only the geometry, and visualize it with the help of the **mapview** package (Figure \@ref(fig:grass-mapview) and Section \@ref(interactive-maps)).


```r
route = read_VECT("shortest_route") |>
  st_as_sf() |>
  st_geometry()
mapview::mapview(route) + points
```

<div class="figure" style="text-align: center">
<img src="figures/10_shortest_route.png" alt="Shortest route (blue line) between 24 cycle hire stations (blue dots) on the OSM street network of London." width="80%" />
<p class="caption">(\#fig:grass-mapview)Shortest route (blue line) between 24 cycle hire stations (blue dots) on the OSM street network of London.</p>
</div>



There are a few important considerations to note in the process:

- We could have used GRASS's spatial database\index{spatial database} (based on SQLite) which allows faster processing.
That means we have only exported geographic data at the beginning.
Then we created new objects but only imported the final result back into R.
To find out which datasets are currently available, run `execGRASS("g.list", type = "vector,raster", flags = "p")`.
- We could have also accessed an already existing GRASS spatial database from within R.
Prior to importing data into R, you might want to perform some (spatial) subsetting\index{vector!subsetting}.
Use `"v.select"` and `"v.extract"` for vector data.
`"db.select"` lets you select subsets of the attribute table of a vector layer without returning the corresponding geometry.
- You can also start R from within a running GRASS\index{GRASS} session [for more information please refer to @bivand_applied_2013].
- Refer to the excellent [GRASS online help](https://grass.osgeo.org/grass82/manuals/) or `execGRASS("g.manual", flags = "i")` for more information on each available GRASS geoalgorithm\index{geoalgorithm}.

**GRASS GIS set up appendix**

It is also possible to set up a location and a mapset manually to use GRASS\index{GRASS} from within R.
First of all, we need to find out if and where GRASS is installed on the computer.


```r
library(link2GI)
link = findGRASS()
```

GRASS GIS differs from many other GIS software in its approach for handling input data -- it puts all of the input data in a GRASS spatial database.
The GRASS geodatabase \index{spatial database} system is based on SQLite.
Consequently, different users can easily work on the same project, possibly with different read/write permissions.
However, one has to set up this spatial database\index{spatial database} (also from within R), and users might find this process a bit intimidating in the beginning.
First of all, the GRASS database requires its own directory, which, in turn, contains a location (see the [GRASS GIS Database](https://grass.osgeo.org/grass82/manuals/grass_database.html) help pages at [grass.osgeo.org](https://grass.osgeo.org/grass82/manuals/index.html) for further information).
The location stores the geodata for one project or one area.
Within one location, several mapsets can exist that typically refer to different users or different tasks.
Each location also has PERMANENT -- a mandatory mapset that is created automatically.
PERMANENT stores the projection, the spatial extent and the default resolution for raster data.
In order to share geographic data with all users of a project, the database owner can add spatial data to the PERMANENT mapset.
So, to sum it all up -- the GRASS geodatabase may contain many locations (all data in one location have the same CRS), and each location can store many mapsets (groups of datasets).
Please refer to @neteler_open_2008 and the [GRASS GIS quick start](https://grass.osgeo.org/grass80/manuals/helptext.html) for more information on the GRASS spatial database\index{spatial database} system.

Next, we need to point to the path to GRASS GIS, and specify where to store the spatial database\index{spatial database} (gisDbase), name the location `london`, and use the PERMANENT mapset.


```r
library(rgrass)
grass_path = link$instDir[[1]]
initGRASS(gisBase = grass_path, gisDbase = tempdir(), 
          location = "london", mapset = "PERMANENT", override = TRUE)
```

After GRASS GIS initialization, we are able to use the `execGRASS()` function.
It expects the name of the GRASS GIS module (e.g., `"g.proj"` or `"g.region"`) and potentially a few additional arguments or flags, and then it executes the given task in GRASS GIS.
Here, let's use it to define the projection, the extent, and the resolution: `"g.proj"` sets the used CRS, while `"g.region"` is used to specify the extent and resolution.


```r
execGRASS("g.proj", flags = c("c", "quiet"), srid = "EPSG:4326")
b_box = st_bbox(london_streets)
execGRASS("g.region", flags = c("quiet"),
          n = as.character(b_box["ymax"]), s = as.character(b_box["ymin"]),
          e = as.character(b_box["xmax"]), w = as.character(b_box["xmin"]),
          res = "1")
```

In this example, use are using the "EPSG:4326" CRS and setting our extent to the bounding box of the `london_streets` dataset.
You can check if it worked correctly by running `execGRASS("g.proj", flags = "p")` and `execGRASS("g.region", flags = "p")`.

## When to use what?

To recommend a single R-GIS interface is hard since the usage depends on personal preferences, the tasks at hand and your familiarity with different GIS\index{GIS} software packages which in turn probably depends on your field of study.
As mentioned previously, SAGA\index{SAGA} is especially good at the fast processing of large (high-resolution) raster\index{raster} datasets, and frequently used by hydrologists, climatologists and soil scientists [@conrad_system_2015].
GRASS GIS\index{GRASS}, on the other hand, is the only GIS presented here supporting a topologically based spatial database which is especially useful for network analyses but also simulation studies.
QGIS is much more user-friendly compared to GRASS- and SAGA-GIS, especially for first-time GIS users, and probably the most popular open-source GIS.
Therefore, **qgisprocess**\index{qgisprocess (package)} is an appropriate choice for most use cases.
Its main advantages are:

- A unified access to several GIS, and therefore the provision of >1000 geoalgorithms (Table \@ref(tab:gis-comp)) including duplicated functionality, e.g., you can perform overlay-operations using QGIS-\index{QGIS}, SAGA-\index{SAGA} or GRASS-geoalgorithms\index{GRASS}
- Automatic data format conversions (SAGA uses `.sdat` grid files and GRASS uses its own database format but QGIS will handle the corresponding conversions)
- Its automatic passing of geographic R objects to QGIS geoalgorithms\index{geoalgorithm} and back into R
- Convenience functions to support the access of the online help, named arguments and automatic default value retrieval (**rgrass**\index{rgrass (package)} inspired the latter two features)

By all means, there are use cases when you certainly should use one of the other R-GIS bridges.
Though QGIS is the only GIS providing a unified interface to several GIS\index{GIS} software packages, it only provides access to a subset of the corresponding third-party geoalgorithms (for more information please refer to @muenchow_rqgis:_2017).
Therefore, to use the complete set of SAGA and GRASS functions, stick with **Rsagacmd**\index{Rsagacmd (package)} and **rgrass**. 
Finally, if you need topological correct data and/or spatial database management functionality such as multi-user access, we recommend the usage of GRASS. 
In addition, if you would like to run simulations with the help of a geodatabase\index{spatial database} [@krug_clearing_2010], use **rgrass** directly since **qgisprocess** always starts a new GRASS session for each call.

Please note that there are a number of further GIS software packages that have a scripting interface but for which there is no dedicated R package that accesses these: gvSig, OpenJump, Orfeo Toolbox and TauDEM.

## Other bridges

The focus of this chapter, so far, was on R interfaces to Desktop GIS\index{GIS} software.
We emphasize these bridges because dedicated GIS software is well-known and a common 'way in' to understanding geographic data.
They also provide access to many geoalgorithms\index{geoalgorithm}.

Other 'bridges' include interfaces to spatial libraries (Section \@ref(gdal) shows how to access the GDAL\index{GDAL} CLI\index{command-line interface} from R), spatial databases\index{spatial database} (see Section \@ref(postgis)) and web mapping services (see Chapter \@ref(adv-map)).
This section provides only a snippet of what is possible.
Thanks to R's\index{R} flexibility, with its ability to call other programs from the system and integration with other languages (notably via **Rcpp** and **reticulate**\index{reticulate (package)}), many other bridges are possible.
The aim is not to be comprehensive, but to demonstrate other ways of accessing the 'flexibility and power' in the quote by @sherman_desktop_2008 at the beginning of the chapter.

### Bridges to GDAL {#gdal}

As discussed in Chapter \@ref(read-write), GDAL\index{GDAL} is a low-level library that supports many geographic data formats.
GDAL is so effective that most GIS programs use GDAL\index{GDAL} in the background for importing and exporting geographic data, rather than re-inventing the wheel and using bespoke read-write code.
But GDAL\index{GDAL} offers more than data I/O.
It has [geoprocessing tools](https://gdal.org/programs/index.html) for vector and raster data, functionality to create [tiles](https://gdal.org/programs/gdal2tiles.html#gdal2tiles) for serving raster data online, and rapid [rasterization](https://gdal.org/programs/gdal_rasterize.html#gdal-rasterize) of vector data, all of which can be accessed via the system of R command line.

The code chunk below demonstrates this functionality:
`linkGDAL()` searches the computer for a working GDAL\index{GDAL} installation and adds the location of the executable files to the PATH variable, allowing GDAL to be called.


```r
link2GI::linkGDAL()
```

Now we can use the `system()` function to call any of the GDAL tools.
For example, `ogrinfo` provides metadata of a vector dataset.
Here we will call this tool with two additional flags: `-al` to list all features of all layers and `-so` to get a summary only (and not a complete geometry list):


```r
our_filepath = system.file("shapes/world.gpkg", package = "spData")
cmd = paste("ogrinfo -al -so", our_filepath)
system(cmd)
#> INFO: Open of `.../spData/shapes/world.gpkg'
#>       using driver `GPKG' successful.
#> 
#> Layer name: world
#> Geometry: Multi Polygon
#> Feature Count: 177
#> Extent: (-180.000000, -89.900000) - (179.999990, 83.645130)
#> Layer SRS WKT:
#> ...
```

Other commonly used GDAL tools include:

- `gdalinfo`: provides metadata of a raster dataset
- `gdal_translate`: converts between different raster file formats
- `ogr2ogr`: converts between different vector file formats
- `gdalwarp`: reprojects, transform, and clip raster datasets
- `gdaltransform`: transforms coordinates

Visit https://gdal.org/programs/ to see the complete list of GDAL tools and to read their help files.

The 'link' to GDAL provided by **link2GI** could be used as a foundation for doing more advanced GDAL work from the R or system CLI.
TauDEM (http://hydrology.usu.edu/taudem) and the Orfeo Toolbox (https://www.orfeo-toolbox.org/) are other spatial data processing libraries/programs offering a command line interface -- the above example shows how to access these libraries from the system command line via R.
This in turn could be the starting point for creating a proper interface to these libraries in the form of new R packages.

Before diving into a project to create a new bridge, however, it is important to be aware of the power of existing R packages and that `system()` calls may not be platform-independent (they may fail on some computers).
Furthermore, **sf** brings most of the power provided by GDAL\index{GDAL}, GEOS\index{GEOS} and PROJ\index{PROJ} to R via the R/C++\index{C++} interface provided by **Rcpp**, which avoids `system()` calls.

### Bridges to spatial databases {#postgis}

<!--toDo:jn-->
<!--consider referencing to 3rd edition of the PostGIS book-->

\index{spatial database}
Spatial database management systems (spatial DBMS) store spatial and non-spatial data in a structured way.
They can organize large collections of data into related tables (entities) via unique identifiers (primary and foreign keys) and implicitly via space (think for instance of a spatial join). 
This is useful because geographic datasets tend to become big and messy quite quickly.
Databases enable storing and querying large datasets efficiently based on spatial and non-spatial fields, and provide multi-user access and topology\index{topological relations} support.

The most important open source spatial database\index{spatial database} is PostGIS\index{PostGIS} [@obe_postgis_2015].^[
SQLite/SpatiaLite are certainly also important but implicitly we have already introduced this approach since GRASS\index{GRASS} is using SQLite in the background (see Section \@ref(grass)).
]
R bridges to spatial DBMSs such as PostGIS\index{PostGIS} are important, allowing access to huge data stores without loading several gigabytes of geographic data into RAM, and likely crashing the R session.
The remainder of this section shows how PostGIS can be called from R, based on "Hello real world" from *PostGIS in Action, Second Edition* [@obe_postgis_2015].^[
Thanks to Manning Publications, Regina Obe and Leo Hsu for permission to use this example.
]

The subsequent code requires a working internet connection, since we are accessing a PostgreSQL/PostGIS\index{PostGIS} database which is living in the QGIS Cloud (https://qgiscloud.com/).^[
QGIS\index{QGIS} Cloud lets you store geographic data and maps in the cloud. 
In the background, it uses QGIS Server and PostgreSQL/PostGIS.
This way, the reader can follow the PostGIS example without the need to have PostgreSQL/PostGIS installed on a local machine.
Thanks to the QGIS Cloud team for hosting this example.
]
Our first step here is to create a connection to a database by providing its name, host name, and user information.


```r
library(RPostgreSQL)
conn = dbConnect(drv = PostgreSQL(), 
                 dbname = "rtafdf_zljbqm", host = "db.qgiscloud.com",
                 port = "5432", user = "rtafdf_zljbqm", password = "d3290ead")
```

Our new object, `conn`, is just an established link between our R session and the database.
It does not store any data.

Often the first question is, 'which tables can be found in the database?'.
This can be answered with `dbListTables()` as follows:


```r
dbListTables(conn)
#> [1] "spatial_ref_sys" "topology"        "layer"           "restaurants"    
#> [5] "highways" 
```

The answer is five tables.
Here, we are only interested in the `restaurants` and the `highways` tables.
The former represents the locations of fast-food restaurants in the US, and the latter are principal US highways.
To find out about attributes available in a table, we can run `dbListFields`:


```r
dbListFields(conn, "highways")
#> [1] "qc_id"        "wkb_geometry" "gid"          "feature"     
#> [5] "name"         "state"   
```

Now, as we know the available datasets, we can perform some queries -- ask the database some questions.
The query needs to be provided in a language understandable by the database -- usually, it is SQL.
The first query will select `US Route 1` in the state of Maryland (`MD`) from the `highways` table.
Note that `read_sf()` allows us to read geographic data from a database if it is provided with an open connection to a database and a query.
Additionally, `read_sf()` needs to know which column represents the geometry (here: `wkb_geometry`).


```r
query = paste(
  "SELECT *",
  "FROM highways",
  "WHERE name = 'US Route 1' AND state = 'MD';")
us_route = read_sf(conn, query = query, geom = "wkb_geometry")
```

This results in an **sf**-object\index{sf} named `us_route` of type `MULTILINESTRING`.

As we mentioned before, it is also possible to not only ask non-spatial questions, but also query datasets based on their spatial properties.
To show this, the next example adds a 35-kilometer (35,000 m) buffer around the selected highway (Figure \@ref(fig:postgis)).


```r
query = paste(
  "SELECT ST_Union(ST_Buffer(wkb_geometry, 35000))::geometry",
  "FROM highways",
  "WHERE name = 'US Route 1' AND state = 'MD';")
buf = read_sf(conn, query = query)
```

Note that this was a spatial query using functions (`ST_Union()`\index{vector!union}, `ST_Buffer()`\index{vector!buffers}) you should be already familiar with.
You find them also in the **sf**-package, though here they are written in lowercase characters (`st_union()`, `st_buffer()`).
In fact, function names of the **sf** package largely follow the PostGIS\index{PostGIS} naming conventions.^[
The prefix `st` stands for space/time.
]

The last query will find all Hardee's restaurants (`HDE`) within the 35 km buffer zone (Figure \@ref(fig:postgis)).


```r
query = paste(
  "SELECT *",
  "FROM restaurants r",
  "WHERE EXISTS (",
  "SELECT gid",
  "FROM highways",
  "WHERE",
  "ST_DWithin(r.wkb_geometry, wkb_geometry, 35000) AND",
  "name = 'US Route 1' AND",
  "state = 'MD' AND",
  "r.franchise = 'HDE');"
)
hardees = read_sf(conn, query = query)
```

Please refer to @obe_postgis_2015 for a detailed explanation of the spatial SQL query.
Finally, it is good practice to close the database connection as follows:^[
It is important to close the connection here because QGIS Cloud (free version) allows only ten concurrent connections.
]


```r
RPostgreSQL::postgresqlCloseConnection(conn)
```



<div class="figure" style="text-align: center">
<img src="10-gis_files/figure-html/postgis-1.png" alt="Visualization of the output of previous PostGIS commands showing the highway (black line), a buffer (light yellow) and four restaurants (red points) within the buffer." width="100%" />
<p class="caption">(\#fig:postgis)Visualization of the output of previous PostGIS commands showing the highway (black line), a buffer (light yellow) and four restaurants (red points) within the buffer.</p>
</div>

Unlike PostGIS, **sf** only supports spatial vector data. 
To query and manipulate raster data stored in a PostGIS database, use the **rpostgis** package [@bucklin_rpostgis_2018] and/or use command-line tools such as `rastertopgsql` which comes as part of the PostGIS\index{PostGIS} installation. 

This subsection is only a brief introduction to PostgreSQL/PostGIS.
Nevertheless, we would like to encourage the practice of storing geographic and non-geographic data in a spatial DBMS\index{spatial database} while only attaching those subsets to R's global environment which are needed for further (geo-)statistical analysis.
Please refer to @obe_postgis_2015 for a more detailed description of the SQL queries presented and a more comprehensive introduction to PostgreSQL/PostGIS in general.
PostgreSQL/PostGIS is a formidable choice as an open-source spatial database.
But the same is true for the lightweight SQLite/SpatiaLite database engine and GRASS\index{GRASS} which uses SQLite in the background (see Section \@ref(grass)).

If your datasets are too big for PostgreSQL/PostGIS and you require massive spatial data management and query performance, it may be worth exploring large-scale geographic querying on distributed computing systems.
Such systems are outside the scope of this book but it worth mentioning that open source software providing this functionality exists.
Prominent projects in this space include [GeoMesa](http://www.geomesa.org/) and [Apache Sedona](https://sedona.apache.org/), formerly known as GeoSpark [@huang_geospark_2017], which has and R interface provided by the [**apache.sedona**](https://cran.r-project.org/package=apache.sedona) package.

## Bridges to cloud technologies and services {#cloud}

In recent years, cloud technologies have become more and more prominent on the internet. 
This also includes their use to store and process spatial data.
Major cloud computing providers (Amazon Web Services, Microsoft Azure / Planetary Computer, Google Cloud Platform, and others)\index{cloud computing} offer vast catalogs of open Earth observation data, such as the complete Sentinel-2 archive, on their platforms. 
We can use R and directly connect to and process data from these archives, ideally from a machine in the same cloud and region.

Three promising developments that make working with such image archives on cloud platforms _easier_ and _more efficient_ are the [SpatioTemporal Asset Catalog (STAC)](https://stacspec.org)\index{STAC}, the [cloud-optimized GeoTIFF (COG)](https://www.cogeo.org/)\index{COG} image file format, and the concept of data cubes\index{data cube}. 
Section \@ref(staccog) introduces these individual developments and briefly describes how they can be used from R.

Besides hosting large data archives, numerous cloud-based services\index{cloud computing} to process Earth observation data have been launched during the last few years.
It includes the OpenEO initiative -- a unified interface between programming languages (including R) and various cloud-based services.
You can find more information about OpenEO in Section \@ref(openeo).

### STAC, COGs, and data cubes in the cloud {#staccog}

The SpatioTemporal Asset Catalog (STAC)\index{STAC} is a general description format for spatiotemporal data that is used to describe a variety of datasets on cloud platforms including imagery, synthetic aperture radar (SAR) data, and point clouds. 
Besides simple static catalog descriptions, STAC-API presents a web service to query items (e.g. images) of catalogs by space, time, and other properties. 
In R, the **rstac** package [@simoes_rstac_2021] allows to connect to STAC-API endpoints and search for items. 
In the example below, we request all images from the [Sentinel-2 Cloud-Optimized GeoTIFF (COG) dataset on Amazon Web Services](https://registry.opendata.aws/sentinel-2-l2a-cogs)\index{COG} that intersect with a predefined area and time of interest. 
The result contains all found images and their metadata (e.g. cloud cover) and URLs pointing to actual files on AWS. 


```r
library(rstac)
# Connect to the STAC-API endpoint for Sentinel-2 data
# and search for images intersecting our AOI
s = stac("https://earth-search.aws.element84.com/v0")
items = s |>
  stac_search(collections = "sentinel-s2-l2a-cogs",
              bbox = c(7.1, 51.8, 7.2, 52.8), 
              datetime = "2020-01-01/2020-12-31") |>
  post_request() |> items_fetch()
```

Cloud storage differs from local hard disks and traditional image file formats do not perform well in cloud-based geoprocessing. 
Broadly speaking, the cloud-optimized GeoTIFF\index{COG} format is a specific type of GeoTIFF that makes it possible to efficiently read only parts of an image from cloud storage. 
As a result, reading rectangular subsets of an image or reading images at lower resolution becomes much more efficient. 
As an R user, you don't have to install anything to work with COGs because [GDAL](https://gdal.org)\index{GDAL} (and any package using it) can already work with COGs. However, keep in mind that the availability of COGs is a big plus while browsing through catalogs of data providers.

For larger areas of interest, requested images are still relatively difficult to work with: they may use different map projections, may spatially overlap, and the spatial resolution often depends on the spectral band. 
The **gdalcubes** package [@appel_gdalcubes_2019] can be used to abstract from individual images and to create and process image collections as four-dimensional data cubes\index{data cube}.

The code below shows a minimal example to create a lower resolution (250m) maximum NDVI composite from the Sentinel-2 images returned by the previous STAC-API search. 


```r
library(gdalcubes)
# Filter images from STAC response by cloud cover 
# and create an image collection object
collection = stac_image_collection(items$features, 
                  property_filter = function(x) {x[["eo:cloud_cover"]] < 10})
# Define extent, resolution (250m, daily) and CRS of the target data cube
v = cube_view(srs = "EPSG:3857", extent = collection, dx = 250, dy = 250,
              dt = "P1D") # "P1D" is an ISO 8601 duration string
# Create and process the data cube
cube = raster_cube(collection, v) |>
  select_bands(c("B04", "B08")) |>
  apply_pixel("(B08-B04)/(B08+B04)", "NDVI") |>
  reduce_time("max(NDVI)")
# gdalcubes_options(parallel = 8)
# plot(cube, zlim = c(0, 1))
```

To filter images by cloud cover, we provide a property filter function that is applied on each STAC\index{STAC} result item while creating the image collection. 
The function receives available metadata of an image as input list and returns a single logical value such that only images for which the function yields TRUE will be considered. 
In this case, we ignore images with 10% or more cloud cover. 
For more details, please refer to this [tutorial presented at OpenGeoHub summer school 2021](https://appelmar.github.io/ogh2021/tutorial.html).

The combination of STAC\index{STAC}, COGs\index{COG}, and data cubes\index{data cube} forms a cloud-native workflow to analyze (large) collections of satellite imagery in the cloud\index{cloud computing}. 
These tools already form a backbone, for example, of the **sits** R package, which allows land use and land cover classification of big Earth observation data.
The package builds EO data cubes from image collections available in cloud services and performs land classification of data cubes using various machine learning algorithms.
For more information about **sits** visit https://e-sensing.github.io/sitsbook/ or read the related article [@rs13132428].

### openEO

OpenEO [@schramm_openeo_2021]\index{openEO} is an initiative to support interoperability among cloud services by defining a common language for processing the data. 
The initial idea has been described in an [r-spatial.org blog post](https://r-spatial.org/2016/11/29/openeo.html) and aims at making it possible for users to change between cloud services easily with as little code changes as possible. 
The [standardized processes](https://processes.openeo.org) use a multidimensional data cube model\index{data cube} as an interface to the data. 
Implementations are available for eight different backends (see https://hub.openeo.org) to which users can connect with R, Python, JavaScript, QGIS, or a web editor and define (and chain) processes on collections. 
Since the functionality and data availability differs among the backends, the **openeo** R package [@lahn_openeo_2021] dynamically loads available processes and collections from the connected backend. 
Afterwards, users can load image collections, apply and chain processes, submit jobs, and explore and plot results. 

The following code will connect to the [openEO platform backend](https://openeo.cloud/), request available datasets, processes, and output formats, define a process graph to compute a maximum NDVI image from Sentinel-2 data, and finally executes the graph after logging in to the backend. 
The openEO\index{openEO} platform backend includes a free tier and registration is possible from existing institutional or social platform accounts. 


```r
library(openeo)
con = connect(host = "https://openeo.cloud")
p = processes() # load available processes
collections = list_collections() # load available collections
formats = list_file_formats() # load available output formats
# Load Sentinel-2 collection
s2 = p$load_collection(id = "SENTINEL2_L2A",
                       spatial_extent = list(west = 7.5, east = 8.5,
                                             north = 51.1, south = 50.1),
                       temporal_extent = list("2021-01-01", "2021-01-31"),
                       bands = list("B04","B08")) 
# Compute NDVI vegetation index
compute_ndvi = p$reduce_dimension(data = s2, dimension = "bands",
                                  reducer = function(data, context) {
                                    (data[2] - data[1]) / (data[2] + data[1])
                                  })
# Compute maximum over time
reduce_max = p$reduce_dimension(data = compute_ndvi, dimension = "t",
                                reducer = function(x, y) {max(x)})
# Export as GeoTIFF
result = p$save_result(reduce_max, formats$output$GTiff)
# Login, see https://docs.openeo.cloud/getting-started/r/#authentication
login(login_type = "oidc", provider = "egi", 
      config = list(client_id = "...", secret = "..."))
# Execute processes
compute_result(graph = result, output_file = tempfile(fileext = ".tif"))
```

## Exercises


<!-- qgisprocess 1-3 -->
E1. Compute global solar irradiation for an area of `system.file("raster/dem.tif", package = "spDataLarge")` for March 21 at 11:00 AM using the `r.sun` GRASS GIS through **qgisprocess**.



<!-- sagagis 1 -->
E2. Compute catchment area\index{catchment area} and catchment slope of `system.file("raster/dem.tif", package = "spDataLarge")` using **Rsagacmd**.



E3. Continue working on the `ndvi_segments` object created in the SAGA GIS section.
Extract average NDVI values from the `ndvi` raster and group them into six clusters using `kmeans()`. 
Visualize the results.



<!-- rgrass 1 -->
E4. Attach `data(random_points, package = "spDataLarge")` and read `system.file("raster/dem.tif", package = "spDataLarge")` into R.
Select a point randomly from `random_points` and find all `dem` pixels that can be seen from this point (hint: viewshed\index{viewshed} can be calculated using GRASS GIS).
Visualize your result.
For example, plot a hillshade\index{hillshade}, the digital elevation model\index{digital elevation model}, your viewshed\index{viewshed} output, and the point.
Additionally, give `mapview` a try.



<!-- gdal 1-2 -->
E5. Use `gdalinfo` via a system call for a raster\index{raster} file stored on disk of your choice.
What kind of information you can find there?



E6. Use `gdalwarp` to decrease the resolution of your raster file (for example, if the resolution is 0.5, change it into 1). Note: `-tr` and `-r` flags will be used in this exercise.



<!-- postgis 1? -->
E7. Query all Californian highways from the PostgreSQL/PostGIS\index{PostGIS} database living in the QGIS\index{QGIS} Cloud introduced in this chapter.



<!-- stac+gdalcubes 1 -->
E8. The `ndvi.tif` raster (`system.file("raster/ndvi.tif", package = "spDataLarge")`) contains NDVI calculated for the Mongón study area based on Landsat data from September 22nd, 2000.
Use **rstac**, **gdalcubes**, and **terra** to download Sentinel-2 images for the same area from 
2020-08-01 to 2020-10-31, calculate its NDVI, and then compare it with the results from `ndvi.tif`.