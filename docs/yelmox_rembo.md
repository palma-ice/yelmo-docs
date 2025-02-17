# YelmoX [YelmoX-REMBO]
 
[comment]: <> (Description of the executable. Here you have to write the main purposes of the executable. It would be good to add also job references if they exist. Remember: for programming, the more complete, the better.)

This flavour couples `yelmo` with the regional energy-moisture balance model for Greenland `rembo1` (Robinson et al., 2010). 
Therefore, `yelmox_rembo` works only for the Greenland ice sheet and in a similar way to `yelmox`, but the surface mass balance is calculated with `rembo1`.

## Dependencies  
[comment]: <> (OPTIONAL section, remove if necessary. Here you can include a list of the modules needed for this particular flavour, e.g. snapclim, rembo ...)

`yelmox_rembo.f90` mainly needs all `.f90` files of `rembo1` and also `snapclim.f90` and `hyster.f90`. It works with two namelists: `rembo_Greenland.nml` and `yelmo_Greenland_rembo.nml`.


## Structure  
[comment]: <> (OPTIONAL section, remove if necessary. You can include here a schematic description of the executable)

This script is organized in order to read the forcings and calculate the corresponding climate with succesive updates in a loop:

1. Parameters reading.
2. Output definitions.
3. Initialization: ice sheet, external modeles (sea level, isostasy, climate and marine melt models), boundary conditions and output files. 
4. Loop: choice between snapclim or hyster boundary forcing and update of the objects (isostasy, sealevel, ice sheet, climate and mass balance.

