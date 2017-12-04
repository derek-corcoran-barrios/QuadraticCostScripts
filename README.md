## Scripts for network flow optimization with quadratic cost

In this repository I have the scripts needed to go from niche model projections to the optimization in ampl.


### Going from niche models to .dat files

The *QuadCostAmpl* package was developed with the R functions to go from the Species Distribution Models of Species in several time slices to the data needed by the ampl model. The source package is in this repository (*QuadCostAmpl_0.1.0.tar.gz*), or if you prefer to install it from github use the following code:

```
devtools::install_github("derek-corcoran-barrios/QuadCostAmpl")
```

The package depends on the *dplyr*, *gdistance*, *raster* and *tidyr* packages. The function needed to go from the projected rasters to the .dat files used by AMPL is called *MultiSppQuad*, which has the following parameters:

* *Stacklist:* a list of Stacks with a stack for each species. Each stack contains eight rasters of corresponding to eight time slices showing a binary response of where the species suitable habitat is, all stacks have to have the same extent. 
* *Dist:* the maximum dispersal distance (in meters) of the species to be modeled in the AMPL model, it can be one distance or a vector of distances.
* *name:* the name of the .dat file that will be exported.
* *nchains:* the target number of chains
* *costlayer:* raster with the costs of each cell

This function returns a *.dat* file needed to feed the *AMPL* model

As an example to run the function using the example data set in the Package:

```
if (!require("pacman")) install.packages("pacman")
pacman::p_load(dplyr, gdistance, raster, tidyr)
library(QuadCostAmpl)

data("BinSpp")
data("Cost")
MultiSppQuad(Stacklist = BinSpp, Dist = 1000000, name = "Two", costlayer = Cost, nchains = 8)
```

This will create a file called *Two.dat* in your working directory with the data of the two hypothetical species developed for the package, the result of this is stored in this repository.

### Going from .dat files to optimization results

#### Models

The *.dat* file will be passed through a model (*.mod* file). In this repository there are two models:

* *MultiCost.mod:* Which will take a *.dat* database with one or multiple species and a layer of costs associated with it, and minimize the buying cost assuming quadratic costs for the flow.

* *MultiCostLM.mod:* Which will take a *.dat* database with one or multiple species and a layer of costs associated with it, and minimize the buying cost assuming linear costs for the flow, this model was included only to compare the results of the quadratic cost.

#### Running the models

*.run* files when scripted correctly can make running models in AMPL a lot easier. What they usually do is to load a *.dat* file (your data), and pass it through a *.mod* file (your model), and it returns your results in a *.txt* file.

If you have a *.run* file, lets call it ```Example.run```, the only thing you have to do is to run the following code in AMPL:

```
include Example.run;
```

This repository has two *.run* files:

* *MultiSpp.run:* This file will run a *.dat* file with multiple species through one of the *.mod* models stated above, and it will return two *.txt* files with results. One with the index proposed by Phillips for each cell corresponding to the maximum amount of flow that will pass through a cell in any of the time slices, that is one value per cell. Also, second *.txt* file with the flow passing through each cell on each time, that is one value per cell per time slice.

* *MultiSppLoop.run:* Does the same as the *.run* file above, but it can loop through several *.dat* files, the only clause is that these files have the same name and only differ on a number indicating the ID of the case, or species to be looped through. 

### Working with the results in R

#### Getting the results back into R and transforming them into a stack


After getting the results from ampl, we can import them to analyse them back in the *R* environment, for that the *GetQuadIndex* was develop, which from the result gotten from the AMPL model, it will develop a stack with the index for each species, and a data frame with the results. The parameters of the function are:

* *Stacklist:* The original stacklist used in function MultiSppQuad
* *AmplFile:* The path to the file given by the AMPL model
* *plot:* logical, weather to plot or not the stack, defaults to TRUE

This function will return a list with a list with a stack with the index 
for each species, and a data frame with the results, this can be tested with the result (*Two.txt*) included in this repository:

```
#Load the data
data("BinSpp")
#Use the function with the 
Index <- GetQuadIndex(Stacklist = BinSpp, AmplFile = "https://raw.githubusercontent.com/derek-corcoran-barrios/QuadraticCostScripts/master/Two.txt")
```

![title](https://github.com/derek-corcoran-barrios/QuadraticCostScripts/blob/master/Results1.png)

```
head(Index[[2]])
```

|Species  | CellID|     Index|
|:--------|------:|---------:|
|SpeciesA |      3| 0.1848551|
|SpeciesA |      4| 1.0000000|
|SpeciesA |      5| 1.0000000|
|SpeciesA |      6| 1.0000000|
|SpeciesA |      7| 1.0000000|
|SpeciesA |     14| 0.8151449|

#### Turning the stack into a gif

In order to inspect the Species flow more closely the *GetQuadFlow* function was developed, this takes the second result of the *AMPL* model and turns it into a stack or a gif for the one of the species, while making a data frame with the result of every species. The parameters of the function are the following:

* *Stacklist:* The original stacklist used in function MultiSppQuad.
* *AmplFile:* The path to the file given by the AMPL model.
* *Species:*	An integer number of the species to be plotted. Even when the plot is made for this species, the data frame is made for all the species.
* *plot:*	logical, whether to plot or not the stack, defaults to TRUE
* *gif:*	logical, whether to make a gif or not the stack, defaults to FALSE

An example using the data in the repository:

```
#Load the data
data("BinSpp")
#Use the function 
Index2 <- GetQuadFlow(Stacklist = BinSpp, AmplFile = "https://raw.githubusercontent.com/derek-corcoran-barrios/QuadraticCostScripts/master/TwoY.txt", Species = 1, plot = FALSE, gif = TRUE)
```

![title2](https://github.com/derek-corcoran-barrios/QuadraticCostScripts/blob/master/animation.gif)

head(Index2[[2]])


|Species  | CellID| Time|     Index|
|:--------|------:|----:|---------:|
|SpeciesA |      3|    0| 0.1848551|
|SpeciesA |      3|    1| 0.0546383|
|SpeciesA |      3|    2| 0.0000000|
|SpeciesA |      3|    3| 0.0000000|
|SpeciesA |      3|    4| 0.0000000|
|SpeciesA |      3|    5| 0.0000000|
