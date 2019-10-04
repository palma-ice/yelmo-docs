# Basal friction optimization

A simple optimization program has developed that attempts
to optimize the basal friction field applied in Yelmo so
that the errors between simulated and observed ice thickness
are minimized.

Program: `tests/yelmo_opt.f90`
To compile: `make opt`
To run: `python run_yelmo.py -s -e opt output/test par/yelmo_Antarctica_opt.nml`

The program consists of the following steps:

### 1. Spin-up a steady-state ice sheet with constant forcing and fixed topography.

For this step, the restart parameter should be set to `&yelmo:restart='none'`, to
ensure that the spin-up is performed with the current parameters. Currently,
the program is hard-coded to spin-up the ice sheet for 20 kyr using SIA only,
followed by another 10 kyr using SIA+SSA, as seen in the following lines of code:
```
call yelmo_update_equil_external(yelmo1,hyd1,cf_ref,time_init,time_tot=20e3,topo_fixed=.TRUE.,dt=5.0,ssa_vel_max=0.0)
call yelmo_update_equil_external(yelmo1,hyd1,cf_ref,time_init,time_tot=10e3, topo_fixed=.TRUE.,dt=1.0,ssa_vel_max=5000.0)
```
Note that this spin-up is obtained with a fixed topography set to the present-day
observed fields (`H_ice`,`z_bed`). After the spin-up finishes, a restart file is
written in the output directory with the name `yelmo_restart.nc`. The simulation
will terminate at this point.

### 2. Optimization

The restart file from Step 1 should be saved somewhere convenient for the model
(like in the `input` folder). Then the restart parameter should be set to that
location `&yelmo:restart='PATH_TO_RESTART.nc'`. This will ensure that the spin-up
step is skipped, and instead the program will start directly with the optimization
iterations.

The optimization method follows Pollard and DeConto (2012), in that the basal
friction coefficient is scaled as a function of the error in elevation. Here we
do not modify `beta` directly, however, we assume that `beta = cf_ref * lambda_bed * N_eff * f(u)`.
`lambda_bed`, `N_eff` and `f(u)` are all controlled by parameter choices in the .nml file
like normal. Thus we are left with a unitless field `cf_ref`, which for any given
friction law varies within the range of about [0:1]. When `cf_ref=1.0`, sliding will
diminish to near zero, and `cf_ref~0.0` (near, but not zero) will give fast sliding.
This gives a convenient range for optimization.  

The program runs for `time_iter=500` years with a given initial field of `cf_ref`
(with `C_bed` and `beta` updating every time step to follow changes in `u/v` and `N_eff`). At the end
of `time_iter`, the error in ice thickness is determined and used to update `cf_ref`
via the function `update_cf_ref_thickness_simple`. The model is again run for `time_iter` years
and the process is repeated. There are a few parameters related to the optimization
that can change to improve the results. These step changes are controlled via the following options
hard-coded at the start of the program:
- `iter_steps`: array that holds the number of iterations for which the optimization parameters should be set.  
- `topo_rels`: array that holds the values of the parameter `topo_rel` to be updated.
- `topo_rel_taus`: array that holds the values of the parameter `topo_rel_tau` to be updated.
- `H_scales`: array that holds the values of the parameter `H_scale` to be updated.
Simply put, if the total iterations surpasses the current value of `iter_steps`, then the program
advances to the next parameter values in the list. When `topo_rel=1`, the ice thickness in Yelmo
is relaxed to the reference thickess over the ice shelves, where `topo_rel_tau` is the time scale of
that relaxation. A lower value of `topo_rel_tau` means the ice shelves are held closer to the observed
values. `H_scale` controls the scaling of the ice thickness error, which determines how to modify
`cf_ref` at each iteration. A higher value of `H_scale` means that changes to `cf_ref` will be
applied more slowly.
