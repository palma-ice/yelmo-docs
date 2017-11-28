# Yelmo

Welcome to Yelmo, an easy to use continental ice sheet model.
**Yelmo** is a 3D thermomechanical ice sheet model that
simulates continental scale ice sheets. It is a so-called hybrid
model that solves both the Shallow Ice Approximation (SIA) for
land-based ice driven by deformation, and the Shallow Shelf
Approximation (SSA) for faster moving ice streams and ice shelves.

It has been designed to operate as a stand-alone model or to be easily plugged in as a module in another program. The key to its flexibility is that no variables are defined globally and parameters are defined according to the domain being modeled. In this way, all variables and calculations are store in an object that entirely represents the model domain.

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

## Directory structure

    docs/
        Model documentation.
    input/
        Location of any input data needed by the model.
        The REMBOv2_data respository should be available here as:
        `input/REMBOv2_data` which contains some useful boundary datasets.
    libs/
        Auxiliary libraries nesecessary for running the model.
    maps/
        Generated maps and grids.
    output/
        Default location for model output.
    src/
        Source code for Yelmo.

## Classes and usage

### yelmo_class

The Yelmo class defines all data related to a model domain, such as Greenland or Antarctica. As seen below in the yelmo_class defintion, the 'class' is simply a user-defined Fortran type that contains additional types representing variousparameters, variables or sets of module variables.

    type yelmo_class

        type(param_class)     :: par      ! Parameters
        type(grid_class)      :: grid     ! Grid definition
        type(ytopo_class)     :: topo     ! topography variables
        type(yvel_class)      :: vel      ! velocity variables
        type(yen_class)       :: en       ! energy (temp, enthalpy) variables

    end type

Likewise the module variables are defined in a similar way, e.g. ytopo_class that defines variables and parameters associated with the topography:


    type ytopo_class

        type(ytopo_param_class)   :: par        ! physical parameters
        type(boundary_opt_class)  :: bnd, bnd0  ! boundary switches (bnd0 for equilibration)

        ! All variables
        type(ytopo_state_class) :: now

    end type

Submodules such as ytopo_class include parameter definitions relevant to topography calculations, as well as so-called 'boundary switches' that define whether a variable should be calculated using the module are considered as a boundary (i.e., forcing) variable. Finally the class contains an additional type that defines the state of the variables in the module.

### Example model domain intialization

The yelmo_class is defined along with a set of subroutines that can be used to initialize, access and change the object (in object-oriented language, class methods).

Assuming that a grid object has been defined previously (using the coordinates library), a yelmo_class object can be initialized using the following call:

    call yelmo_init(yelmo_grl,"Greenland",grid=grid_grl)

This will initialize the object `yelmo_grl` with the domain name "Greenland" and the model domain will match the grid defined by the grid object `grid_grl` (e.g., arrays will allocated to the size of the grid, etc.). Parameters will be initialized by first loading all of the default parameter sets from the Yelmo namelist file `src/yelmo.nml`. Next the program will look for a domain-specific namelist file that matches the name of the domain, in this case `Greenland.nml`. Any parameters that have been defined in this file will overwrite the default parameter values of the program.

There are only two additional public methods available for managing the yelmo_class object: `yelmo_update` and `yelmo_end`, which handle time-stepping of the model calculations and finalization (deallocation) of the object. `yelmo_end` is called at the end of the program or when a model domain variable should be deleted and the memory freed. `yelmo_update` is called in a time-stepping loop of a main program, e.g.:

    do year = 1, 100
        call yelmo_update(yelmo_grl,year=year)        
    end do

That's it! All options that manage the calculations to be performed have been defined in `src/yelmo.nml` and `Greenland.nml`, therefore no additional information needs to be passed except the time step.

Exchange of information between the yelmo_class object and other models, modules or even external datasets stored in files can be performed by direct access, or more rigorously by defining a subroutine in the Yelmo exchange module (`src/yelmo_exchange.f90`).


## Getting the code

Yelmo is managed using the version control software [git](http://www.git.org). To get started with a local working copy of the code, you need to (1) clone the repository to a local path, (2) create your own branch so that your changes do not conflict with the master branch and (3) synchronize your branch with the central repository so that your changes are also under version control.

    git clone robinson@cluster.pik-potsdam.de:/iplex/01/tumble/robinson/repos/yelmo LOCAL_PATH
    git checkout -b your_branch
    git push -u origin your_branch

Note use the command `git branch` at any time to see which branches are available and which branch is currently checked out (marked by the asterisk).

That's it! You now have all of the Yelmo code in the local path `LOCAL_PATH`.

### Bare (central) repository locations

PIK repository location: `/iplex/01/tumble/robinson/repos/yelmo`  
UCM repository location: `/home/fispalma25/repos/yelmo_mirror`

## Compiling Yelmo programs

Yelmo uses a Makefile to manage program compilation. Currently one test program exists for Yelmo development: `test_yelmo.x`. Compilation can be acheived via make:

    make test_yelmo    : compiles the program test_yelmo.x (ice sheet development program)
    make clean         : removes all compiled files

By default, the Makefile will choose gfortran as the compiler and use the `-O3` compiler options. To compile on the PIK cluster using ifort add `ifort=1` to the end of the command, eg.:

    make test_yelmo ifort=1

Likewise to include debugging options, use `debug=1`:

    make test_yelmo debug=1

If compiled on the PIK cluster with ifort, the library paths given in the Makefile (mainly to find the netcdf libraries) should be correct and the programs should compile without any problems. For local compilation, typical Mac locations are currently specified that may need to be modified for other computers. To do so, find these lines in the Makefile:

    netcdf_inc = /opt/local/include
    netcdf_lib = /opt/local/lib

## Running the programs

Yelmo is designed so that the source code is located in one place and each simulation produces output in a new directory. In this way, there is only one copy of the source code and compiled program, and one output directory for each program call.

By default, if no output directory is specified, the program sends output directly to the path `output` and can be run from the command line:

    ./test_yelmo.x

Otherwise the output directory can be specified as a command-line argument. In this case, the output directory should already contain a copy of the namelist files for the domains to be simulated.

    mkdir output/simulation1
    cp Greenland.nml output/simulation1/
    ./test_yelmo.x output/simulation1

_Note: In the near future, the automatic generation of output directories, as well as batch simulations and parameter changes will be handled using the python __job__ script._

## Parameter modification

Yelmo depends on a namelist file with default parameter values located in the `src` directory (`src/yelmo.nml`). This file should remain unchanged during production runs, it is only used to initially load all parameter values.

Next, in the main directory, there should be one namelist file per modeled domain (e.g., `Greenland.nml`), with each group of parameters corresponding to each model component. The name of this namelist file corresponds to the name of the domain given in the code when the domain was initialized. As the program runs, it first loads parameters from the default namelist file, then it checks if any parameters should be overwritten from the domain-specific namelist file.

This two step procedure is convenient in that usually the default values do not change and can be stored for reference in the model component namelist file. Meanwhile in the domain-specific namelist file, only the parameters of interest that need to be modified for a given run need to be specified.

## Libraries

Yelmo relies on some external libraries (some of which have been built specifically for use with it), however the Yelmo repository is mostly self-contained. If the libraries are portable, like fortran modules, then they are stored in the `libs/` subdirectory. The only external dependency is on the NetCDF libraries, which must be preinstalled on the system.

Below is a list of the libraries used in Yelmo:

__nml__ is a custom library for reading namelist parameters into a Fortran program. It has the same functionality as native nml reading, however it does not require the definition of a common block.

__coordinates__ handles the definition of grids and points on user-specified coordinate systems. It has the ability to map between different coordinate systems, interpolation and oblique stereographic projection.

__subset__ depends on the coordinates module and allows subsets of points to be generated from pre-defined grids. The subset of points can be of higher resolution than the original grid, and if so, mapping to and from the original grid is handled internally.

__insol__ provides functions to calculate the daily insolation at any latitude for any time up to 5 Ma BP or 1 Ma AP (after present).

__solvers__ provides some basic differential equation solvers.
