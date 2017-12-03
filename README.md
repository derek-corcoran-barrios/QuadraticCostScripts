## Scripts for network flow optimization with quadratic cost

In this repository I have the scripts needed to go from niche model projections. To the optimization in ampl.


### Going from niche models to .dat files

A package was developed with the R functions to use the ampl model, the source package is in this repocitory, or to install it from github use the following code:

```
devtools::install_github("derek-corcoran-barrios/QuadCostAmpl")
```

The function needed to go from the projected rasters to the .dat files needed by the AMPL its called *MultiSppQuad*, which has the following parameters:

*Stacklist:* a list of Stacks with a stack for each species. Each stack contains eight rasters of corresponding to eight time slices showing a binary response of where the species suitable habitat is, all stacks have to have the same extent. 
*Dist:* the maximum dispersal distance of the species modeled in the AMPL model, it can be one distance or a vector of distances.
*name:* the name of the .dat file that will be exported.
*nchains:* the target number of chains
*costlayer:* raster with the costs of each cell

This function returns a .dat file needed to feed the *AMPL* model

The function depends on the *dplyr*, *gdistance*, *raster* and *tidyr* packages

as an example to run the data using the data in the repository:

```
if (!require("pacman")) install.packages("pacman")
pacman::p_load(dplyr, gdistance, raster, tidyr)
library(QuadCostAmpl)

data("BinSpp")
data("Cost")
MultiSppQuad(Stacklist = BinSpp, Dist = 1000000, name = "Two", costlayer = Cost)

```

### Going from .dat files to optimization results

#### Models

The *.dat* file will be passed through a model (*.mod* file), in this repository there are two models:

* *MultiCost.mod:* Which will take a *.dat* database with one or multiple species and a layer of costs asociated with it, and minimize the buying cost asuming quadratic costs for the flow.

* *MultiCostLM.mod:* Which will take a *.dat* database with one or multiple species and a layer of costs asociated with it, and minimize the buying cost asuming linear costs for the flow, this model was included only to compare the results of the quadratic cost.

#### Running the models

*.run* Files when scripted correctly can make running models in AMPL a lot easier, what they usually do is to load a *.dat* file (your data), and pass it through a *.mod* file (your model), and it returns your results in a *.txt* file.

If you have a *.run* file, lets call it ```Example.run```, the only thing you have to do is to run the following code in AMPL:

```
include Example.run;
```

This repository has two *.run* files:

* *MultiSpp.run:* This file will run a *.dat* file with multiple species through one of the *.mod* models stated above, and it will return two *.txt* files with results. One with the index proposed by phillips for each cell corresponding to the maximum amont of flow that will pass through a cell through any of the time silces, that is one value per cell. Also, second *.txt* file with the flow passing through each cell on each time, that is one value per cell per time slice.

* *MultiSppLoop.run:* Does the same as the *.run* file above, but it can loop through several *.dat* files, the only clause is that these files have the same name and only differ on a number indicating the ID of the case, or species to be looped throug 

### Working with the results in R

#### Getting the results back into R and transforming them into a stack


#### Turning the stack into a gif

