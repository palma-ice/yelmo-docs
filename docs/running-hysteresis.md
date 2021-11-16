# Running hysteresis experiments for Antarctica

For now, we will:

- Use optimized basal friction.
- Spinup with a constant present-day climate based on the ISMIP6 protocol.

To run `yelmox` with this setup, we need the `hyst-2021` branches:

```
cd yelmox
git checkout hyst-2021

cd yelmo
git checkout hyst-2021

# Reconfigure
python3 config.py config/snowball_gfortran
cd ..
python3 config.py config/snowball_gfortran

# If not done already, link to ice_data
ln -s /media/Data/ice_data ice_data
```

We will run with the ISMIP6 standard YelmoX program, so compile:

```
make clean
make yelmox_ismip6
```

The hysteresis runs can be done in two steps:
1. First, generate a spun-up simulation with optimized basal friction.
2. Restart from the optimized, spun-up state and continue with transient forcing from the 
hysteresis module.

## Step 1: spinup

The spinup simulation runs for 30.000 years, by default. For the first `opt_L21.rel_time1=5e3` years, the shelves and grounding line are relaxed (tightly) to the present-day reference state, while the optimization of the basal friction field `cf_ref` is active. Next between `opt_L21.rel_time1=5e3` kyr and `opt_L21.rel_time2=10e3` years, the relaxation timescale is slowly increased from `opt_L21.rel_tau1=10` yrs to `opt_L21.rel_tau2=1000` yrs, to slowly allow the ice sheet more freedom to adjust its state. Basal optimization is further optimized during this time period. After `opt_L21.rel_time2=10e3` years, the relation is disabled and the ice sheet if fully prognostic. The simulation is further run until the end with continual optimization adjustments to `cf_ref`, although these are usually minor after the initial spinup period. 

To run a spinup simulation as above, use the following command:

```
# First run spinup simulation
# (steady-state present day boundary conditions)
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/hyst/spinup01 -p ctrl.run_step="spinup_ismip6" opt_L21.cf_min=1e-3 ytopo.kt=0.10e-2 tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5
```

Or an ensemble to test different parameters too:

```
fldr=output/hyst/spinup02
jobrun ./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" opt_L21.cf_min=1e-3 ytopo.kt=0.10e-2,0.20e-2,0.30e-2,0.40e-2 tf_corr_ant.ronne=0.0,0.25 tf_corr_ant.ross=0.0,0.2 tf_corr_ant.pine=-0.5,0.0
```

To make the spinup run for a longer time, like 50 kyr, set `ctrl.time_end=50e3`. 

If you already have a spinup simulation available, you can skip that step.
Alternatively, you can specify one that is ready on `snowball`:

```
yelmo.restart=/home/robinson/abumip-2021/yelmox/output/ismip6/spinup11/1/yelmo_restart.nc
```

## Step 2: transient simulations 

To run transient simulations the `run_step` should be specified as `ctrl.run_step="hysteresis_proj"`. Typically model parameters should be defined to be equivalent to those used by the restart simulation. The time control parameters of the simulation are defined in the parameter section `&hysteresis_proj`. Parameters associated with the hysteresis module can be changed in the `&hyster` section. 

Example transient simulation of `hysteresis_proj.time_end=500` years, with ramp forcing via the `hyster` module with 100 years of constant forcing, followed by a ramp over 250 years from an anomaly of 0 degC to 5 degC:

```
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/hyst/test1 -p \
ctrl.run_step="hysteresis_proj" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 \
yelmo.restart=/home/robinson/abumip-2021/yelmox/output/ismip6/spinup11/1/yelmo_restart.nc \
tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.001 \
hyster.method="ramp" hyster.dt_init=100 hyster.dt_ramp=250 hyster.f_min=0 hyster.f_max=5
```

Example transient simulation using Adaptive Quasi-Equilibrium Forcing (AQEF) with no lead-in time:

```
./runylmox -r -e ismip6 -n par/yelmo_ismip6_Antarctica.nml -o output/hyst/test2 -p \
ctrl.run_step="hysteresis_proj" tf_cor.name="dT_nl" marine_shelf.gamma_quad_nl=14500 \
yelmo.restart=/home/robinson/abumip-2021/yelmox/output/ismip6/spinup11/1/yelmo_restart.nc \
tf_corr_ant.ronne=0.25 tf_corr_ant.ross=0.2 tf_corr_ant.pine=-0.5 ytopo.kt=0.001 \
hyster.method="PI42" hyster.dt_init=0 hyster.f_min=0 hyster.f_max=5
```

Or simply further constant forcing of a control run by setting `hyster.method="const"`. 

You can add white noise (Normal distribution) to your forcing by setting the standard deviation greater than zero: `hyster.sigma=1.0`. 
