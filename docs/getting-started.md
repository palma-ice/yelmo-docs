This file describes the basic steps needed to get **Yelmo**
running.

## Dependencies

- NetCDF library (preferably version 4.0 or higher)
- LIS: [Library of Iterative Solvers for Linear Systems](http://www.ssisc.org/lis/)
- [Coordinates library](https://github.com/alex-robinson/coordinates "alex-robinson/coordinates")
- Python 2.7.x (only needed for automation of configuration of Makefile
and the use of the `run_yelmo.py` script for job preparation and submission)

OPTIONAL:

- Access to the `ice_data` repository of pre-processed
input data for the model on different domains (North, Antarctica,
Greenland)

## Directory structure

```
config/
    Location of the Makefile template and configuration files.
extra_data/
    An empty folder where user-specific input data can be stored.
input/
    Location of any input time series and other data needed by the model.
libyelmo/
    All compiled object files and binaries will be located here
    after compilation (in bin/ and include/ directories).
libs/
    Auxiliary included libraries necessary/useful for running the model.
output/
    Default location for model output.
par/
    Folder containing the default parameter files for running Yelmo.
src/
    Source code for Yelmo.
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

Now you are ready to compile the **Yelmo** program you desire (eg, `yelmo_eismint`) :

```
make clean    *This step is very important to avoid errors!!
make yelmo_eismint [debug=1]
```
(or other program `yelmo_test`, `yelmo_mismip`, `yelmo_stommel` or `yelmo_optbeta`). The `debug=1` option allows you to compile with debugging compiler options enabled, in case you need to debug the code. Using this option, the code will run much slower, so this option is not recommended for real simulations.

### 4. Run the model.

Once the executable has been created, you can run the model. This can be
achieved via a Python job submission script `run_yelmo.py`. The following steps
are carried out via the script:

1. The output directory is created.
2. The parameter files are modified according to command-line options,
then saved in the output directory.
3. The executable is copied to the output directory, along with
links to the input data paths (`input` and `ice_data`).
4. The executable is run from the output directory, either as a background process or it is submitted to the queue (currently supports `qsubmit` and `sbatch` commands).

**TO DO** __The model output of each simulation will appear in the specified output directory. The above steps can also be run for a batch ensemble of simulations, which can also be automatically generated via `runner`.__

To run a test simulation `yelmo_eismint.x`, submitted to the queue (option `-s`), use the following command:

```
./run_yelmo.py  -e yelmo_eismint.x -s -f -o output/test
```

**TO DO** Or, you can change parameters at the command line:

```
./run_yelmo.py -p yelmo_eismint.x -s -f -o output/test \&group="par1=0.0"
```
where `\&group` is the Namelist group that contains the parameter par1.
**TO DO** Or, you can run an ensemble of simulations:

```
./run_yelmo.py -p yelmo_eismint.x -s -f -a output/ens1 \&group="par1=0.0,0.5 par2=0,1,2"
```
\*Note the change of `-o` to `-a` which means that sub-directories will be automatically generated for each simulation inside the chosen directory.

More information about running jobs can be found in the help:

```
./run_yelmo.py -h
```
