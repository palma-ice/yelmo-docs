# Standard YelmoX simulations

To run YelmoX, by default we use the program `yelmox.f90`. This program currently makes use of `snapclim` for the climatic forcing and `smbpal` for the snowpack and surface mass balance calculations.

## Spin-up simulation for ISMIP6-based runs (ISMIP6, ABUMIP)

First make sure your distribution of `yelmox` and `yelmo` are up to date.

```bash
cd yelmo
git pull 
cd ..

# In the main yelmox directory, change to the branch 'tfm2021' or 'abumip-2021': 
git pull 
git checkout tfm2021

# From main directory of yelmox, also reconfigure to adopt all changes:
python3 config.py config/snowball_gfortran 

# Link to ice_data path as needed:
ln -s /media/Data/ice_data ice_data
```

Now compile as normal, but with the `yelmox_ismip6` program:

```bash
make clean 
make yelmox_ismip6 
```

You are now ready to run some ISMIP6 simulations. If you have a spinup simulation available you can skip the rest of this section.

The next step is to get a spin-up simulation ready. To do so, we will run a small ensemble of simulations that apply different calving coefficients (`ytopo.kt`) and shear-regime enhancement factors (`ymat.enh_shear`). Each simulation will run with these parameter values set, while optimizing the basal friction coefficient field `cb_ref` and the temperature anomalies imposed in different basins `tf_corr`. To run this ensemble, use the following commands:

```bash
# Specify run choices, to run locally in the background:
runopt='-r'
# or, to submit job to a cluster, eg:
runopt='-rs -q priority -w 10'

# Define the output folder
fldr=tmp/ismip6/spinup_32km_68

# Run the Yelmo ensemble

jobrun ./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 ytopo.kt=1.0e-3,1.5e-3,2.0e-3,2.5e-3 ymat.enh_shear=1,3
```

An alternative spinup procedure

```bash
# First run for 30kyr with topo relax on to spinup thermodynamics...

runopt='-rs -q priority -w 10'
fldr=tmp/ismip6/spinup_32km_69
jobrun ./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 spinup_ismip6.equil_method="relax" spinup_ismip6.time_end=30e3 spinup_ismip6.time_equil=30e3 ytopo.kt=1.0e-3,1.5e-3,2.0e-3,2.5e-3 ymat.enh_shear=1


# Testing opt spinup but with 'robin' initial temp profile
runopt='-rs -q priority -w 10'
fldr=tmp/ismip6/spinup_32km_70
jobrun ./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 ytopo.kt=1.0e-3 ymat.enh_shear=1 opt_L21.cf_max=1.0

# robin-cold but only by -2deg instead of -10deg
runopt='-rs -q priority -w 10'
fldr=tmp/ismip6/spinup_32km_71
jobrun ./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 ytopo.kt=1.0e-3 ymat.enh_shear=1 opt_L21.cf_max=0.2 opt_L21.cf_init=0.2


runopt='-rs -q short -w 10'
fldr=tmp/ismip6/spinup_32km_72
jobrun ./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 ytopo.kt=1.0e-3 opt_L21.cf_max=10,20,40,45 ydyn.beta_u0=100,300
```

## ISMIP6 simulations

Make sure you already have a spinup simulation available, and that the parameters of the spinup will match those supplied here. The next step is to run different experiments of interest that restart from the spinup experiment.

Some commands for running diagnostic short runs

```bash
### Diagnostic short runs ###

# Run a 16km spinup run with relaxation

runopt='-rs -q priority -w 1'
fldr=tmp/ismip6/spinup_16km_72_diag

jobrun ./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 ytopo.kt=1.0e-3 spinup_ismip6.equil_method="relax" yelmo.grid_name="ANT-16KM" spinup_ismip6.time_end=20 spinup_ismip6.dt2D_out=1


# Run with 16km restarting from 32km file
# (currently crashes probably because tf_corr cannot be interpolated)

runopt='-rs -q priority -w 1'
fldr=tmp/ismip6/ismip_32km_68_diag
file_restart=/p/tmp/robinson/ismip6/spinup_32km_68/0/yelmo_restart.nc 

./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/ctrl -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 ${paropt} yelmo.grid_name="ANT-16KM" transient_proj.time_end=1920 transient_proj.dt2D_out=1 ytill.is_angle=False

###
```

## Actual ISMIP6 commands

```bash
# Define output folder as a bash variable
fldr=tmp/ismip6/ismip_32km_71

# Specify run choices, to run locally in the background:
runopt='-r'
# or, to submit job to a cluster, eg:
runopt='-rs -q priority -w 10'

# Set parameter choices to match those of spinup simulation
paropt="ytopo.kt=1.0e-3"

## First run a steady-state simulation to stabilize everything

# Define restart file path as a bash variable, for example, on snowball:
file_restart=/p/tmp/robinson/ismip6/spinup_32km_71/0/yelmo_restart.nc 

# ctrl-0
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/ctrl-0 -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 transient_proj.time_end=11900 transient_proj.dt1D_out=10 transient_proj.dt2D_out=200 ${paropt}

## Next, call the Yelmo commands for the individual cases...

# Define restart file path as a bash variable, for example, on snowball:
#file_restart=/p/tmp/robinson/ismip6/spinup_32km_68/0/yelmo_restart.nc 
file_restart=/p/tmp/robinson/ismip6/ismip_32km_68/ctrl-0/yelmo_restart.nc 

# ctrl
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/ctrl -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 ${paropt}

# exp05
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/exp05 -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="rcp85" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 ${paropt}

# exp09
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/exp09 -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="rcp85" tf_cor.name="dT_nl_95" marine_shelf.gamma_quad_nl=21000 ${paropt}

# exp10
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/exp10 -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="rcp85" tf_cor.name="dT_nl_5" marine_shelf.gamma_quad_nl=9620 ${paropt}

# exp13
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/exp13 -p ctrl.run_step="transient_proj" yelmo.restart=${file_restart} transient_proj.scenario="rcp85" tf_cor.name="dT_nl_pigl" marine_shelf.gamma_quad_nl=159000 ${paropt}

```

## ABUMIP

Make sure you already have a spinup simulation available.

Use the following commands to run the three main experiments of interest. Note that `abuk` and `abum` may run much more slowly than `abuc`. The parameter values applied in the commands below ensure that the model parameters correspond to those used in the restart simulation, although many of them like ocean temp. anomalies in different basins or calving parameters, are no longer relevant in the ABUMIP context. It is important, however, to specify `ydyn.ssa_lat_bc='marine'`, as it is relevant for this experiment to apply `marine` boundary conditions. This is generally not used currently, as it makes the model much less stable.

Note that an equilibrium spin-up simulation has already been performed, which gives good agreement with the present-day ice sheet. These results have been saved in a restart file, from which your simulations will begin (see below).

```bash
# Define restart file path as a bash variable
file_restart=/p/tmp/robinson/ismip6/spinup_32km_68/0/yelmo_restart.nc 

# Define output folder as a bash variable
fldr=tmp/ismip6/abumip_32km_68

# Specify run choices, to run locally in the background:
runopt='-r'
# or, to submit job to a cluster, eg:
runopt='-rs -q priority -w 5'

debugopt="abumip_proj.time_end=20 abumip_proj.dt2D_out=1"

# Set parameter choices to match those of spinup simulation
paropt="ytopo.kt=1.0e-3"

# Call the Yelmo commands...

# ABUC - control experiment
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/abuc -p abumip.scenario="abuc" ctrl.run_step="abumip_proj" yelmo.restart=${file_restart} abumip_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 isostasy.method=0 ${paropt} 

# ABUK - Ocean-kill experiment
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/abuk -p abumip.scenario="abuk" ctrl.run_step="abumip_proj" yelmo.restart=${file_restart} abumip_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 isostasy.method=0 ${paropt}

# ABUM - High shelf melt (400 m/yr)
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/abum -p abumip.scenario="abum" ctrl.run_step="abumip_proj" yelmo.restart=${file_restart} abumip_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 isostasy.method=0 ${paropt} 

# ABUK - Ocean-kill experiment (MARINE BOUNDARY CONDITIONS)
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/abuk-marine -p abumip.scenario="abuk" ctrl.run_step="abumip_proj" yelmo.restart=${file_restart} abumip_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 isostasy.method=0 ${paropt} ydyn.ssa_lat_bc="marine"

# ABUM - High shelf melt (400 m/yr) (MARINE BOUNDARY CONDITIONS)
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/abum-marine -p abumip.scenario="abum" ctrl.run_step="abumip_proj" yelmo.restart=${file_restart} abumip_proj.scenario="ctrl" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 isostasy.method=0 ${paropt} ydyn.ssa_lat_bc="marine"

```

## Simulations with `hyster`

```bash
# Define restart file path as a bash variable, for example, on snowball:
file_restart=/p/tmp/robinson/ismip6/spinup_32km_68/0/yelmo_restart.nc 

# Define output folder as a bash variable
fldr=tmp/ismip6/ramp_32km_68

# Specify run choices, to run locally in the background:
runopt='-r'
# or, to submit job to a cluster, eg:
runopt='-rs -q priority -w 5'

# Set parameter choices to match those of spinup simulation

paropt="ytopo.kt=1.0e-3"

# Now run simulation
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/r1 -p ctrl.run_step="hysteresis_proj" yelmo.restart=${file_restart} tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 hysteresis_proj.time_end=30e3 hyster.method='ramp-time' hyster.df_sign=-1 hyster.dt_init=0 hyster.dt_ramp=10e3 hyster.f_min=-10 hyster.f_max=5 ${paropt}

# Try a periodic simulation
./runme ${runopt} -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o ${fldr}/r2 -p ctrl.run_step="hysteresis_proj" yelmo.restart=${file_restart} tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 hysteresis_proj.time_end=100e3 hyster.method='sin' hyster.df_sign=1 hyster.dt_init=0 hyster.dt_ramp=20e3 hyster.f_min=-10 hyster.f_max=5 ${paropt}


```

That's it!
