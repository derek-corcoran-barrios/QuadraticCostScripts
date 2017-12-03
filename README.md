## Scripts for network flow optimization with quadratic cost

In this repository I have the scripts needed to go from niche model projections. To the optimization in ampl.


### Going from niche models to .dat files

The *QuadCostAmpl* package was developed with the R functions to go from the Species Distribution Models of Species in several time slices to the data needed by the ampl model. The source package is in this repocitory (*QuadCostAmpl_0.1.0.tar.gz*), or if you prefer to install it from github use the following code:

```
devtools::install_github("derek-corcoran-barrios/QuadCostAmpl")
```

The function needed to go from the projected rasters to the .dat files used by AMPL is called *MultiSppQuad*, which has the following parameters:

* *Stacklist:* a list of Stacks with a stack for each species. Each stack contains eight rasters of corresponding to eight time slices showing a binary response of where the species suitable habitat is, all stacks have to have the same extent. 
* *Dist:* the maximum dispersal distance (in meters) of the species to be modeled in the AMPL model, it can be one distance or a vector of distances.
* *name:* the name of the .dat file that will be exported.
* *nchains:* the target number of chains
* *costlayer:* raster with the costs of each cell

This function returns a *.dat* file needed to feed the *AMPL* model

The function depends on the *dplyr*, *gdistance*, *raster* and *tidyr* packages

As an example to run the function using the example data set in the Package:

```
if (!require("pacman")) install.packages("pacman")
pacman::p_load(dplyr, gdistance, raster, tidyr)
library(QuadCostAmpl)

data("BinSpp")
data("Cost")
MultiSppQuad(Stacklist = BinSpp, Dist = 1000000, name = "Two", costlayer = Cost)
```

This will create a file called *Two.dat* in your working directory with the data of the two Hypothetical species developed for the package.

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

