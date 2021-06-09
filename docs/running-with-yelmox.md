# Running with YelmoX

YelmoX is a separate repository that is designed to provide supplementary libraries and programs that allow running ice-sheet simulations with realistic boundary (e.g., climate and ocean) forcing and interactions (e.g., isostatic rebound).

![Ice sheet interactions](img/yelmo_test_color.png)

![YelmoX interface](img/yelmox_overview.png)

Here you can find the basic information and steps needed to get **YelmoX** running.

## Super-quick start

A summary of commands to get started is given below. For more detailed information see subsequent sections.

```
# Clone repository
git clone https://github.com/palma-ice/yelmox.git
git clone git@github.com:palma-ice/yelmox.git # via ssh

# Enter directory and run configuration script
cd yelmox
python config.py config/pik_ifort 

# Clone yelmo into a sub-directory and configure it too 
git clone https://github.com/palma-ice/yelmo.git
git clone git@github.com:palma-ice/yelmo.git # via ssh
cd yelmo 
python config.py config/pik_ifort 

# Return to the main directory and compile the default program
cd ../
make clean 
make yelmox 

# Link to `ice_data` repository
ln -s path_to/ice_data 

# Copy the runylmox config file to the main directory
cp config/runylmox.js ./

# Run a test simulation of Antarctica for 1000 yrs
./runylmox -r -e yelmox -o output/ant-test -n par/yelmo_Antarctica.nml
```

## Standard YelmoX simulations 

To run YelmoX, by default we use the program `yelmox.f90`. This program currently makes use of `snapclim` for the climatic forcing and `smbpal` for the snowpack and surface mass balance calculations.

## ISMIP6 simulations 

First make sure your distribution of `yelmox` and `yelmo` are up to date.

```
cd yelmo
git pull 
cd ..

# In the main yelmox directory, change to the branch 'tfm2021': 
git pull 
git checkout tfm2021

# From main directory of yelmox, also reconfigure to adopt all changes:
python config.py config/snowball_gfortran 
cp config/runylmox.js ./
```

Now compile as normal, but with the `yelmox_ismip6` program:

```
make clean 
make yelmox_ismip6 
```

That's it, you are now ready to run some ISMIP6 simulations. 

If you need to run a spinup simulation that also optimizes basal friction, run the following command:

```
# First run spinup simulation
# (steady-state present day boundary conditions)
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/ismip6/spinup_opt11
```

If you already have a spinup simulation available, you can skip that step. You can also copy one from here:

```
mkdir output/ismip6
cp -r /home/robinson/yelmox/output/ismip6/spinup_opt11 output/ismip6/
```

Next run different experiments of interest that restart from the spinup experiment.

```
# ctrl
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/ismip6/ctrl -p ctrl.run_step="transient_proj" yelmo.restart="../spinup_opt11/yelmo_restart.nc" transient_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500

# exp05
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/ismip6/exp05 -p ctrl.run_step="transient_proj" yelmo.restart="../spinup_opt11/yelmo_restart.nc" transient_proj.scenario="rcp85" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500

# exp09
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/ismip6/exp09 -p ctrl.run_step="transient_proj" yelmo.restart="../spinup_opt11/yelmo_restart.nc" transient_proj.scenario="rcp85" tf_cor.name="dT_nl_95" marine_shelf.gamma_quad_nl=21000

# exp10
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/ismip6/exp10 -p ctrl.run_step="transient_proj" yelmo.restart="../spinup_opt11/yelmo_restart.nc" transient_proj.scenario="rcp85" tf_cor.name="dT_nl_5" marine_shelf.gamma_quad_nl=9620

# exp13
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/ismip6/exp13 -p ctrl.run_step="transient_proj" yelmo.restart="../spinup_opt11/yelmo_restart.nc" transient_proj.scenario="rcp85" tf_cor.name="dT_nl_pigl" marine_shelf.gamma_quad_nl=159000

```