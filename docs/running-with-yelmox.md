# Running with YelmoX

YelmoX is a separate repository that is designed to provide supplementary libraries and programs that allow running ice-sheet simulations with realistic boundary (e.g., climate and ocean) forcing and interactions (e.g., isostatic rebound).

![Ice sheet interactions](img/yelmo_test_color.png)

![YelmoX interface](img/yelmox_overview.png)

Here you can find the basic information and steps needed to get **YelmoX** running.

## Super-quick start

A summary of commands to get started is given below. For more detailed information see subsequent sections.

```

# Before doing anything, make sure dependencies are installed (Lis, NetCDF, runner)

##########################

# Clone repository
git clone https://github.com/palma-ice/yelmox.git
git clone git@github.com:palma-ice/yelmox.git # or via ssh

# Clone yelmo into a sub-directory too
cd yelmox
git clone https://github.com/palma-ice/yelmo.git
git clone git@github.com:palma-ice/yelmo.git # or, via ssh

# Check out your yelmox branch of interest (optional)
git checkout my-branch  # for an existing branch 
git checkout -b my-new-branch # or, to create a new one 

# Do the same for yelmo branch if needed, and return to yelmox directory
cd yelmo 
git checkout my-branch
cd .. 

# Enter yelmo directory and configure it for compiling
cd yelmo
python3 config.py config/snowball_gfortran  # or the right config file for your system

# Return to yelmox directory and configure it for compiling
cd ..
python3 config.py config/snowball_gfortran  # or the right config file for your system


# Now, compile the default program
make clean 
make yelmox 

# Link to `ice_data` repository wherever you have it saved on your system
ln -s path_to/ice_data 

# Copy the runylmox config file to the main directory
cp config/runylmox.js ./

# Run a test simulation of Antarctica for 1000 yrs
./runylmox -r -e yelmox -n par/yelmo_Antarctica.nml -o output/ant-test -p ctrl.time_end=1e3
```

## Standard YelmoX simulations 

To run YelmoX, by default we use the program `yelmox.f90`. This program currently makes use of `snapclim` for the climatic forcing and `smbpal` for the snowpack and surface mass balance calculations.

## Spin-up simulation for ISMIP6-based runs (ISMIP6, ABUMIP)

First make sure your distribution of `yelmox` and `yelmo` are up to date.

```
cd yelmo
git pull 
cd ..

# In the main yelmox directory, change to the branch 'tfm2021' or 'abumip-2021': 
git pull 
git checkout tfm2021

# From main directory of yelmox, also reconfigure to adopt all changes:
python3 config.py config/snowball_gfortran 
cp config/runylmox.js ./

# Link to ice_data path as needed:
ln -s /media/Data/ice_data ice_data
```

Now compile as normal, but with the `yelmox_ismip6` program:

```
make clean 
make yelmox_ismip6 
```

You are now ready to run some ISMIP6 simulations. If you have a spinup simulation available you can skip the rest of this section.

The next step is to get a spin-up simulation ready. To run a reasonable simulation for Antarctica that will optimize basal friction coefficient `cb_ref`, run the following commands:

```
# Specify run choices, to run locally in the background:
runopt='-r'
# or, to submit job to a cluster, eg:
runopt='-s -q priority -w 5'

# Define output folder 
fldr=output/ismip6/spinup_32km

./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr} -p ctrl.run_step="spinup_ismip6" opt_L21.cf_min=1e-3 opt_L21.fill_method="cf_min" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.003
```

## ISMIP6 simulations 

Make sure you already have a spinup simulation available, and that the parameters of the spinup will match those supplied here. The next step is to run different experiments of interest that restart from the spinup experiment.

```
# Define restart file path as a bash variable, for example, on snowball:
file_restart=/home/robinson/ismip6/spinup_32km_05/0/yelmo_restart.nc

# Define output folder as a bash variable
fldr=output/ismip6

# Specify run choices, to run locally in the background:
runopt='-r'
# or, to submit job to a cluster, eg:
runopt='-s -q priority -w 5'

# Call the Yelmo commands...

# ctrl
./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/ctrl -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.003

# exp05
./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/exp05 -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="rcp85" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.003

# exp09
./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/exp09 -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="rcp85" tf_cor.name="dT_nl_95" marine_shelf.gamma_quad_nl=21000 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.003

# exp10
./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/exp10 -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="rcp85" tf_cor.name="dT_nl_5" marine_shelf.gamma_quad_nl=9620 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.003

# exp13
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/ismip6/exp13 -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="rcp85" tf_cor.name="dT_nl_pigl" marine_shelf.gamma_quad_nl=159000 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.003

```


## ABUMIP

Make sure you already have a spinup simulation available.

Use the following commands to run the three main experiments of interest. Note that `abuk` and `abum` may run much more slowly than `abuc`. The parameter values applied in the commands below ensure that the model parameters correspond to those used in the restart simulation, although many of them like ocean temp. anomalies in different basins or calving parameters, are no longer relevant in the ABUMIP context. It is important, however, to specify `ydyn.ssa_lat_bc='marine'`, as it is relevant for this experiment to apply `marine` boundary conditions. This is generally not used currently, as it makes the model much less stable. 

Note that an equilibrium spin-up simulation has already been performed, which gives good agreement with the present-day ice sheet. These results have been saved in a restart file, from which your simulations will begin (see below). 

```
# Define restart file path as a bash variable
file_restart=/home/robinson/ismip6/spinup_32km_05/0/yelmo_restart.nc

# Define output folder as a bash variable
fldr=output/ismip6/abumip_32km

# Specify run choices, to run locally in the background:
runopt='-r'
# or, to submit job to a cluster, eg:
runopt='-s -q priority -w 5'

# Call the Yelmo commands...

# ABUC - control experiment

./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/abuc -p abumip.scenario="abuc" ctrl.run_step="abumip_proj" yelmo.restart=${file_restart} abumip_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.003 isostasy.method=0 ydyn.ssa_lat_bc='floating'

# ABUK - Ocean-kill experiment
./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/abuk -p abumip.scenario="abuk" ctrl.run_step="abumip_proj" yelmo.restart=${file_restart} abumip_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.003 isostasy.method=0 ytopo.calv_tau=1e-3 ydyn.ssa_lat_bc='marine'

# ABUM - High shelf melt (400 m/yr)
./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/abum -p abumip.scenario="abum" ctrl.run_step="abumip_proj" yelmo.restart=${file_restart} abumip_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.003 isostasy.method=0 ydyn.ssa_lat_bc='floating'

```

That's it! 