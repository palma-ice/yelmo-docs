# YelmoX [ISMIP6]
 
[comment]: <> (Description of the executable. Here you have to write the main purposes of the executable. It would be good to add also job references if they exist. Remember: for programming, the more complete, the better.)

This flavour reproduces the routines to produce simulations with ISMIP6 forcings in Yelmo, both for the atmospheric and oceanic components. It uses several GCM models as forcings.

## Dependencies  
[comment]: <> (OPTIONAL section, remove if necessary. Here you can include a list of the modules needed for this particular flavour, e.g. snapclim, rembo ...)

`yelmox_ismip6.f90` mainly depends on  `ismip6.f90` and `marine_shelf.f90`, but also `snapclim.f90` and `smbpal.f90`.
It also depends on `ice_data` for ISMIP6.

## Structure  
[comment]: <> (OPTIONAL section, remove if necessary. You can include here a schematic description of the executable)


This script is organized in order to read the forcings and calculate the corresponding climate with succesive updates in a loop.

Groups description:

1. Parameters reading.
2. Output definitions.
3. Initialization: ice sheet model, external models, boundary conditions, output files.
4. Choose between spinup or transient simulation.
5. Loop: updates of the objects: mainly ismip6 and marine shelf.
