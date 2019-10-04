# Getting started 

This file describes the basic information and steps needed to get **Yelmo**
running.

## Dependencies

- NetCDF library (preferably version 4.0 or higher)
- LIS: [Library of Iterative Solvers for Linear Systems](http://www.ssisc.org/lis/)

See: [Dependencies](https://palma-ice.github.io/yelmo-docs/dependencies/)

OPTIONAL:
- Python 3.x, which is only needed for automatic configuration of the Makefile
and the use of the script `run_yelmo.py` for job preparation and submission.

## Directory structure
```fortran
    config/
        Configuration files for compilation on different systems.
    input/
        Location of any input data needed by the model.
    libs/
        Auxiliary libraries nesecessary for running the model.
    libyelmo/
        Folder containing all compiled files in a standard way with
        lib/, include/ and bin/ folders.
    output/
        Default location for model output.
    par/ 
        Default parameter files that manage the model configuration.
    src/
        Source code for Yelmo.
    tests/ 
        Source code and analysis scripts for specific model benchmarks and tests.
```

## Usage

**Yelmo** is hosted in a git repository. Follow the steps below to
obtain the code, compile it and run a test simulation.

### 1. Get the code.

Clone the repository from [https://github.com/palma-ice/yelmo](https://github.com/palma-ice/yelmo):
```
git clone git@github.com:palma-ice/yelmo.git $YELMOROOT
cd $YELMOROOT 
```
where `$YELMOROOT` is the installation directory.

If you plan to make changes to the code, it is wise to check out a new branch:
```
git checkout -b user-dev
```
You should now be working on the branch `user-dev`.

### 2. Create the system-specific Makefile.

To compile Yelmo, you need to generate a Makefile that is appropriate for your system. In the folder `config`, you need to specify a configuration file that defines the compiler and flags, including definition of the paths to the `NetCDF` and `LIS` libraries. You can use another file in the config folder as a template, e.g.,

```
cd config
cp pik_ifort myhost_mycompiler
```
then modify the file `myhost_mycompiler` to match your paths. Back in `$YELMOROOT`, you can then generate your Makefile with the provided python configuration script:

```
cd $YELMOROOT
python config.py config/myhost_mycompiler
```
The result should be a Makefile in `$YELMOROOT` that is ready for use. 

### 3. Compile the code.

Now you are ready to compile Yelmo as a static library:
```
make clean    # This step is very important to avoid errors!!
make yelmo-static [debug=1]
```
This will compile all of the Yelmo modules and libraries (as defined in `config/Makefile_yelmo.mk`),
and link them in a static library. All compiled files can be found in the folder `libyelmo/`. 

Once the static library has been compiled, it can be used inside of external Fortran programs and modules
via the statement `use yelmo`. 
To include/link yelmo-static during compilation of another program, its location must be defined:
```
INC_YELMO = -I${YELMOROOT}/include
LIB_YELMO = -L${YELMOROOT}/include -lyelmo
```

Alternatively, several test programs exist in the folder `tests/` to run Yelmo 
as a stand-alone ice sheet.
For example, it's possible to run different EISMINT benchmarks, MISMIP benchmarks and the
ISIMIP6 INITMIP simulation for Greenland, respectively:
```
make benchmarks    # compiles the program `libyelmo/bin/yelmo_benchmarks.x`
make mismip        # compiles the program `libyelmo/bin/yelmo_mismip.x`
make initmip       # compiles the program `libyelmo/bin/yelmo_initmip.x`
```

The Makefile additionally allows you to specify debugging compiler flags with the option `debug=1`, in case you need to debug the code (e.g., `make benchmarks debug=1`). Using this option, the code will run much slower, so this option is not recommended unless necessary.

### 4. Run the model.

Once an executable has been created, you can run the model. This can be
achieved via the included Python job submission script `run_yelmo.py`. The following steps
are carried out via the script:

1. The output directory is created.
2. The executable is copied to the output directory
3. The relevant parameter files are copied to the output directory.
4. Links to the input data paths (`input` and `ice_data`) are created in the output directory. (Note `ice_data` is typically linked to an external data repository, and is not necessary.)
4. The executable is run from the output directory, either as a background process or it is submitted to the queue (the script currently supports `qsubmit` and `sbatch` commands).

To run a benchmark simulation, for example, use the following command:

```
python run_yelmo.py -r -e benchmarks output/test par/yelmo_EISMINT.nml 
```
where the option `-r` implies that the model should be run as a background process. If this is omitted, then the output directory will be populated, but no executable will be run, while `-s` instead will submit the simulation to cluster queue system instead of running in the background. The option `-e` lets you specify the executable. For some standard cases, shortcuts have been created:
```
benchmarks = libyelmo/bin/yelmo_benchmarks.x 
mismip     = libyelmo/bin/yemo_mismip.x 
initmip    = libyelmo/bin/yelmo_initmip.x 
```
The last two mandatory arguments are always the output/run directory and the parameter file to be used for this simulation. In the case of the above simulation, the output directory is defined as `output/test`, where all model parameters (loaded from the file `par/yelmo_EISMINT.nml`) and model output can be found. 

<!--
Additional powerful job submission functionality is available by using the Python `runner` tool. The `job run` command of the `runner` package allows you to change parameter values at the command line and perform large ensembles. For example, you can run an ensemble of two simulations as above but with different grid resolutions using:

```
job run -f -o output/test -p eismint.dx=25.0,50.0 -- python run_yelmo.py -s -x -e benchmarks {} par/yelmo_EISMINT.nml
```
where `-f` means execute the command without confirmation, `-o` is the parent ensemble directory, `-p` is the argument to allow you to list parameters to modify and `eismint.dx=25.0,50.0` specifies that the parameter `dx` in the namelist group `eismint` should be given a value of `25.0` in one simulation and `50.0` in the second simulation. This command will modify the specified parameters found in the parameter file `par_yelmo_EISMINT.nml` and then run the simulation from the run directory `output/test/{}` where `{}` is the name of the directory corresponding to that parameter combination. Using the option `-o` means that the specific run directory name is simply the number of the simulation, so the run directory for the above command would be `output/test/0`. Using the option `-a` means that the run directory will have a name corresponding to the parameter combination specified. Also note that for `run_yelmo.py` to work properly when called by `job run`, the additional option `-x` must be specified.

The list of parameter choices corresponding to each directory can be found in the file:
```
output/test/params.txt
```

More information about running jobs can be found in the help of `run_yelmo.py` and `runner`:

```
./run_yelmo.py -h
job -h 
job run -h 
```
-->
