This file describes the basic steps needed to get **Yelmo**
running.

## Dependencies

- NetCDF library (preferably version 4.0 or higher)
- LIS: [Library of Iterative Solvers for Linear Systems](http://www.ssisc.org/lis/)
- [Coordinates library](https://github.com/alex-robinson/coordinates "alex-robinson/coordinates")

OPTIONAL:
- Python 2.7.x or 3.x, which is only needed for automation of configuration of Makefile
and the use of the `run_yelmo.py` script for job preparation and submission.
- [Python library `runner`](https://gitlab.pik-potsdam.de/greenrise/runner "runner library"), which extends the `run_yelmo.py` functionality to facilitate running ensembles of simulations.
- Access to the `ice_data` repository of pre-processed
input data for the model on different domains (North, Eurasia, Greenland, Antarctica)

## Directory structure
```fortran
    config/
        Configuration files for compilation on different systems.
    docs/
        Model documentation.
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

Clone the repository, check out a new branch and make sure it is linked 'upstream' to the same branch in the central repository:

```
git clone user@airara.fis.ucm.es:/palma/repos/yelmo.git yelmo
git checkout -b user-dev
git push -u origin user-dev
```
You should now be working on the branch `user-dev`.

### 2. Housekeeping.

- You need to generate a symbolic link to the input (2D/3D/+) data folder that the model uses (typically the location of the `ice_data` repository), e.g.,

```
ln -s PATH/TO/ice_data ice_data
```
- You need to generate the Makefile that is appropriate for your system. In the folder config, you need to specify a configuration file with the name of your system that contains the paths to the `NetCDF`, `LIS` and `coordinates` libraries. See others in the config folder for a template, e.g.,

```
cd config
cp eolo your_system
```
then modify the file `your_system` to match your paths. This file should have the name that is output from your system when you run the bash command `hostname`. Back in the main `yelmo` directory, you can then generate your Makefile with the provided python configuration script:

```
cd ..
python config.py
```

### 3. Compile the code.

Now you are ready to compile the **Yelmo** program you desire (eg, `yelmo_benchmarks`) :

```
make clean    *This step is very important to avoid errors!!
make yelmo_benchmarks [debug=1]
```
(or other program `yelmo_test`, `yelmo_mismip`, `yelmo_stommel` or `yelmo_optbeta`). The `debug=1` option allows you to compile with debugging compiler options enabled, in case you need to debug the code. Using this option, the code will run much slower, so this option is not recommended for real simulations.

### 4. Run the model.

Once the executable has been created, you can run the model. This can be
achieved via a Python job submission script `run_yelmo.py`. The following steps
are carried out via the script:

1. The output directory is created.
2. The executable is copied to the output directory
3. The parameter files are copied to the output directory.
4. Links to the input data paths (`input` and `ice_data`) are created in the output directory.
4. The executable is run from the output directory, either as a background process or it is submitted to the queue (currently supports `qsubmit` and `sbatch` commands).

To run a test simulation on a cluster using the executable `yelmo_benchmarks.x`, use the following command:

```
./run_yelmo.py -s -e benchmarks output/test par/yelmo_EISMINT.nml 
```
where `-s` implies submission to the cluster queue, and `-e` lets you specify the executable. For some standard cases, shortcuts have been created:
```
benchmarks = libyelmo/bin/yelmo_benchmarks.x 
test       = libyelmo/bin/yelmo_test.x 
mismip     = libyelmo/bin/yemo_mismip.x 
```
The last two mandatory arguments are always the output/run directory and the parameter file to be used for this simulation. 

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

### Troubleshooting

`Runtime error: liblis.so.0 not found`
It may be necessary to export the LIS lib path to the linker, in order to avoid a runtime error. This can be achieved by including the following lines in the `.bash_profile` file:
```
# lis library paths 
LD_LIBRARY_PATH=/home/fispalma25/apps/lis/lis/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH
```

