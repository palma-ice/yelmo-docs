# YelmoX Bipolar
 
[comment]: <> (Description of the executable. Here you have to write the main purposes of the executable. It would be good to add also job references if they exist. Remember: for programming, the more complete, the better.)

This flavour is intended to communicate two `yelmo` domains using an ocean box model in between, i.e., reproducing the bipolar seesaw with a simple and computationally efficient approach.

Some notes:
- An additional type is built, `yelmox`. It contains everything that an individual `yelmox` domain needs to work.
- The coding principle is to reproduce 1:1 `yelmox.f90` in those groups in order to be completely analogue to 1-domain `yelmox` executables.

## Dependencies  
[comment]: <> (OPTIONAL section, remove if necessary. Here you can include a list of the modules needed for this particular flavour, e.g. snapclim, rembo ...)

`yelmox_bipolar.f90` mainly needs `snapclim.f90` and `obm.f90`.

## Structure  
[comment]: <> (OPTIONAL section, remove if necessary. You can include here a schematic description of the executable)

This script is organized in "groups" in order to make the code more clear. Each group is a subroutine that contains the specified lines of the original `yelmox.f90` code. Some exceptions are the groups that involve actions related with the coupling between domains and/or the ocean box model.

Groups description:

1. Parameters reading.
2. Output definitions.
3. Initialization: ice sheet model, external models, boundary conditions, output files.
4. Loop: spinup procedure, ice sheet, external models, output.
