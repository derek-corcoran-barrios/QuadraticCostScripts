## Scripts for network flow optimization with quadratic cost

In this repocitory I have the scripts needed to go from niche model projections. To the optimization in ampl.


### Going from niche models to .dat files

The function needed to go from the projected rasters to the .dat files needed by the AMPL software is in the QuadScript.R file and its called *MultiSppQuad*, wich has the following parameters:

*Stacklist:* a list of Stacks with a stack for each species. Each stack contains eight rasters of corresponding to eight time slices showing a binary responce of where the species suitable habitat is, all stacks have to have the same extent. 
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

load("BinSpp.rda")
load("Cost.rda")
MultiSppQuad(Stacklist = BinSpp, Dist = 1000000, name = "Two", costlayer = Cost)
```

### Going from .dat files to optimization results

### Getting the results back into R and transforming them into a stack

### Turning the stack into a gif

