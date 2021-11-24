# Running with YelmoX: Greenland paleo simulations

This document describes how to get a transient paleo simulation running. It will assume that you already have cloned `yelmo` and `yelmox` and have checked out the right version and/or branch, and that you have access to the boundary data needed (e.g., in an `ice_data` folder). 

This setup will use the standard `yelmox.f90` program. 

To run a present-day spinup simulation that will also optimize the basal friction coefficient `cb_ref` using the default parameter values, use the following command:

```
# First run spinup simulation
# (steady-state present day boundary conditions)
./runylmox -r -e yelmox -n par/yelmo_Greenland.nml -o output/spinup-grl-1
```

Once a spinup is available, you can run a transient simulation.

```
# Define output folder and restart file path
fldr=output/paleo-grl-1
restart=output/spinup-grl-1/yelmo_restart.nc 

# Call run command
./runylmox -r -e yelmox -n par/yelmo_Greenland.nml -o ${fldr} -p ctrl.time_init=-158e3 ctrl.time_end=2000 ctrl.transient_clim=True ctrl.equil_method="none" yelmo.restart=${restart} 
```

Note that transient simulations have the time defined in [years CE]. In other words, `time=2000` corresponds to the time 2000 CE. There is another variable available in the code `time_bp` which is used to represent time before present, where the present day is year 0, assumed to occur at the year `time=1950`. 

To run without a restart file, then it is possible to run the above command but leave out the option `yelmo.restart=${restart}`. 

That's it! 