---
layout: default
---
**Yelmo** is a 3D thermomechanical ice sheet model that
simulates continental scale ice sheets. It is a so-called hybrid
model that solves both the Shallow Ice Approximation (SIA) for
land-based ice driven by deformation, and the Shallow Shelf
Approximation (SSA) for faster moving ice streams and ice shelves.

> ### New in v0.3 compared to previous versions ###
**Alexander Robinson, 2017-01-10**
>
- Modified f_ssa to be defined on the staggered grid (now f_ssa_mx, f_ssa_my). For a short simulation, now f_ssa looks much more consistent with other mask definitions.
- Reimplemented SIA horizontal velocity limiting - now it is applied to both basal velocity and uxbar/uybar. Perhaps a limit on diffmx/diffmy should be considered, analagous to the limit in SICOPOLIS HD_MIN/HD_MAX.
- Updated default parameter files to be consistent with new velocity limit parameter. Also set cf_fac_ values equal to 1.0.
- Updated default dtmin value to a very small number (1e-5) to make the model more stable.
- Added parameters to switch sliding laws, limit basal velocity allowed, etc.

> ### New in v0.2 compared to previous versions ###
**Alexander Robinson, 2016-11-06**
>
- *Yelmo* is now compiled as a static library with a few public
interface functions (`grisli_init`, `grisli_update`, `grisli_end`). The
stand-alone program runs the same as before, but this setup will facilitate coupling with other models.
- Default parameter files are stored in the folders `pars_default` and are
tracked by the repository. Users should copy the relevant parameter files to the main directory and modify them instead when running the model. (Previously all user changes to parameter files were tracked causing constant repository conflicts.)
- The output filenames have been simplified. Now the default 1D and 2D output files are named "grisli1D.nc" and "grisli2D.nc". Output files for specific regions have the region name appended as a suffix, eg, "grisli1D_region1.nc"
- The ice volume units in the 1D output file have been changed: [m^3] => [1e6 km^3].
- Hybrid mixing code and parameters have all been unified into the module "grisli_velocity.f90". See parameter documentation for more information (group `gvel_par`).
- Now parameter files can be loaded that have an additional suffix after the domain name, using the `-s` option from the gjob script. For example, to load the parameter file `grisli_Antarctica_test1.nml`, use the option `-s test1`.

## History ##

**Yelmo** is based on the GRISLI code developed by Catherine Ritz
(see Ritz et al., 1997), and subsequent modifications by coworkers in Grenoble and Paris until 2011.
The thermodynamic core (i.e., solvers) of **Yelmo** has not been modified
from GRISLI. Significant changes include all calculations related to
boundary conditions (surface mass balance, topography, basal melt,
sediments), as well as several helper modules to calculate the
thermodynamics (basal dragging, deformation parameters).

```
! **********************************************************************
! GRISLI      Grenoble Ice Shelves-Land Ice
! **********************************************************************
! Ont participe a l'ecriture de ce modele :
!
!               Catherine Ritz                           (tout du long)
!                   <catritz@lgge.obs.ujf-grenoble.fr>
!               Adeline Fabre                      (la partie Gremlins)
!               Vincent Rommelaere         (ice shelves et ice streams)
!               Christophe Dumas (debut f90,              (Antarctique)
!               Vincent Peyaud      (portage HN,calving, front, hydrol)
!               Cyril Mazauric                                  (AGRIF)
!
! From 2011 onwards as Yelmo :
!               Alexander Robinson
!               Jorge Alvarez-Solas
!               Marisa Montoya
!               Ilaria Tabone
!               Javier Blasco
! **********************************************************************
```
