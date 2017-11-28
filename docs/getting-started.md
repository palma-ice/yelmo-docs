This file describes the basic steps needed to get **Yelmo**
running.

## Dependencies

- Intel FORTRAN compiler with Math Kernel Library (MKL).
Note: **Yelmo** can also be compiled using `gfortran`, however `ifort`
is also necessary due to the dependence on MKL.
- NetCDF library (preferably version 4.0 or higher)
- [Coordinates library](https://github.com/alex-robinson/coordinates "alex-robinson/coordinates")
- Python 2.7.x (only needed for automation of configuration of Makefile
and job submission steps)

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
input_1D/
    Location of any input time series data needed by the model.
libyelmo/
    All compiled object files and binaries will be located here
    after compilation (in bin/ and include/ directories).
libs/
    Auxiliary included libraries necessary for running the model.
output/
    Default location for model output.
pars_default/
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
- You will also need to copy the default parameter values into the main folder. Only the parameter files in the main folder will be found.

- You need to generate the Makefile that is appropriate for your system. In the folder config, you need to specify a configuration file with the name of your system that contains the paths to the NetCDF and MKL libraries. See others in the config folder for a template, e.g.,

```
cd config
cp eolo your_system
```
then modify the file `your_system` to match your paths. This file should have the name that is output from your system when you run the bash command `hostname`. Back in the main `yelmo` directory, you can then generate your Makefile with the provided python configuration script:

```
cd ..
python config.py
```

- You need to copy the parameter files for your domain from the `pars_default` folder to the main folder, eg for `North-40`:

```
cp pars_default/North.nml ./
cp pars_default/yelmo_control.nml ./
```
The parameter files copied into the main directory will be used by
the model, and will not be tracked by the repository.

### 3. Compile the code.

Now you are ready to compile **Yelmo** for your domain and grid of interest (eg, `North-40`) :

```
make clean    *This step is very important to avoid errors!!
make North-40 [debug=1]
```
(or other domains `Ant-40`, `Grl-20`, or `GRL-10`). The `debug=1` option allows you to compile with debugging compiler options enabled, in case you need to debug the code. Using this option, the code will run much slower, so this option is not recommended for real simulations.

### 4. Run the model.

Once the executable has been created, you can run the model. This can be
achieved via a Python job submission script `gjob`. The following steps
are carried out via the script:

1. The output directory is created.
2. The parameter files are modified according to command-line options,
then saved in the output directory.
3. The executable is copied to the output directory, along with
links to the input data paths (`input_1D`, `ice_data` and `extra_data`).
4. The executable is run from the output directory, either as a background process or it is submitted to the queue (currently supports `qsubmit` command).

The model output of each simulation will appear in the specified output directory. The above steps can also be run for a batch ensemble of simulations, which can also be automatically generated via `gjob`.

To run a test simulation of the domain `North-40`, submitted to the queue (option `-l`), use the following command:

```
python gjob -p North-40 -l -f -o output/test
```

Or, you can change parameters at the command line:

```
python gjob -p North-40 -l -f -o output/test \&group="par1=0.0"
```
where `\&group` is the Namelist group that contains the parameter par1.
Or, you can run an ensemble of simulations:

```
python gjob -p North-40 -l -f -a output/ens1 \&group="par1=0.0,0.5 par2=0,1,2"
```
\*Note the change of `-o` to `-a` which means that sub-directories will be automatically generated for each simulation inside the chosen directory.

More information about running jobs can be found in the help:

```
python gjob -h
```
