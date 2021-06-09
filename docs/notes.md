# Notes

## `master` to `main`

Following updated conventions, the default branch is now called `main` and the branch `master` has been deleted. 

To update a working copy locally that already contains a `master` branch and therefore points to it as the default branch, the following steps should be applied:

1. Get the branch `main`.
2. Delete the local branch `master`.
3. Make sure your local repository sees `main` as the default branch.

```
# Get all branch information from the origin (github):
git fetch --all

# Get onto the new default branch:
git checkout main 

# Delete the branch master:
git branch -d master

# Clean up any branches that no longer exist at origin:
git fetch --prune origin

# Set the local 'head' to whatever is specified at the origin (which will be main):
git remote set-head origin -a
```

Done! Now your local copy should work like normal, with `main` instead of `master`.

## Thermodynamics equations 

### Ice column

Prognostic equation:

$$
\frac{\partial T}{\partial t} = \frac{k}{\rho c} \frac{\partial^2 T}{\partial z^2} - u \frac{\partial T}{\partial x} - v \frac{\partial T}{\partial y} - w \frac{\partial T}{\partial z} + \frac{\Phi}{\rho c}
$$

Ice surface boundary condition:

$$
T(z=z_{\rm srf}) = {\rm min}(T_{\rm 2m},T_0)
$$

Ice base (temperate) boundary condition:

$$
T(z=z_{\rm bed}) = T_{\rm pmp}
$$

Ice base (frozen) boundary condition:

$$
k \frac{\partial T}{\partial z} = k_r \frac{\partial T_r}{\partial z}
$$

Note, the following internal Yelmo variables are defined for convenience:

$$
Q_{\rm ice,b} = -k \frac{\partial T}{\partial z}; \quad 
Q_{\rm rock} = -k_r \frac{\partial T_r}{\partial z}
$$

### Bedrock column

Prognostic equation:

$$
\frac{\partial T_r}{\partial t} = \frac{k_r}{\rho_r c_r} \frac{\partial^2 T_r}{\partial z^2}
$$

Bedrock surface boundary condition:

$$
T_r(z=z_{\rm bed}) = T(z=z_{\rm bed})
$$

Bedrock base boundary condition:

$$
\frac{\partial T_r}{\partial z} = -\frac{Q_{\rm geo}}{k_r}
$$

## Equilibrium bedrock 

In this case, the bedrock temperature profile is prescribed to the equilibrium linear temperature profile. The slope follows:

$$
\frac{\partial T_r}{\partial z} = -\frac{Q_{\rm geo}}{k_r}
$$

and the bedrock surface temperature is given by the ice temperature at its base:

$$
T_r(z=z_{\rm bed}) = T(z=z_{\rm bed})
$$

## Active bedrock 

Yelmo calculates the temperature in the lithosphere along with the ice temperature. This can be achieved by assuming equilibrium conditions in the bedrock, i.e., that the temperature profile in the bedrock is always linear with `T_lith_s = T_ice_b` and the slope equal to `dT/dz = -Q_geo / k_lith`. Or, the temperature equation can be solved in the lithosphere together with the temperature in the ice column. 

The parameter block `ytherm_lith` controls how the lithosphere is calculated with `ytherm_lith.method=['equil','active']` deciding the two cases above. 

### Density of the upper lithosphere 



### Heat capacity of the upper lithosphere 

In both SICOPOLIS and GRISLI, a value of `cp = 1000.0 [J kg-1 K-1]` is used (referenced in Rogozhina et al., 2012; Greve, 2005; Greve, 1997). This value is adopted in Yelmo as well. 

```
cp = 1000.0    ! [J kg-1 K-1] 
```

### Heat conductivity of the upper lithosphere

Note, Yelmo expects input parameter values in units of `[J a-1 m-1 K-1]`, while much literature uses `[W m-1 K-1]`. Given the number of seconds in a year `sec_year = 31536000.0`, `kt [W m-1 K-1] * sec_year = kt [J a-1 m-1 K-1]`.

Rogozhina et al. (2012) use `kt = 2 [W m-1 K-1]` for Greenland:

```
kt = 6.3e7     ! [J a-1 m-1 K-1]
```

This value is supported by Lösing et al. (2020), who perform a Bayesian inversion for GHF in Antarctica. Assuming exponentially decreasing heat production with depth, lower values of `kt` are supported (see Fig. 7b). In a study on the global thermal characteristics of the lithosphere, Cammarano and Guerri (2017) adopt an upper crust thermal conductivity of `kt = 2.5 [W m-1 K-1]`.

To do: This study is also potentially relevant: 
[https://link.springer.com/article/10.1186/s40517-020-0159-y](https://link.springer.com/article/10.1186/s40517-020-0159-y). They show ranges of on the order of `kt = 2-3 [W m-1 K-1]` for the Canadian shield.

The above value of `kt = 2 [W m-1 K-1] = 6.3e7 [J a-1 m-1 K-1]` is adopted as the default thermal conductivity of the upper crust in Yelmo. For historical context, see other estimates below. 

From Greve (1997) and Greve (2005): 

```
kt = 9.46e7    ! [J a-1 m-1 K-1]
```

which is equivalent to `kt = 3 [W m-1 K-1]`. The source of this value is not known. 

From GRISLI: 

```
kt = 1.04e8    ! [J a-1 m-1 K-1]
``` 

which is equivalent to `kt = 3.3 [W m-1 K-1]`. The source of this value is not known.   



## How to read `yelmo_check_kill` output

The subroutine `yelmo_check_kill` is used to see if any instability is arising in the model. If so, then a restart file is written at that moment (the earlier in the instability, the better), and the model is stopped with diagnostic output to the log file. 

Note that `pc_eps` is the parameter that defines our target error tolerance in the time stepping of ice thickness evolution. At each time step, the diagnosed model error `pc_eta` is compared with `pc_eps`. If `pc_eta >> pc_eps`, this is interpreted as instability and the model is stopped. 

## Margin-front mass balance 

Following Pollard and DeConto (2012,2016), an ice-margin front melting scheme has been implemented that accounts for the melt rate along the vertical face of ice submerged by seawater. 

The frontal mass balance ($\dot{f}$, m yr$^{-1}$) is calculated as:

$$
\dot{f} = \dot{b}_{\rm eff} \frac{A_f}{A_{\rm tot}} \theta_f
$$

where $\dot{b}_{\rm eff}$ is the effective basal mass balance (the mean of the basal mass balance calculated for the ice-free neighbors), $A_{\rm tot}=\Delta x \Delta x$ is the horizontal grid area and $A_f$ is the area of the submerged faces (i.e., the sum of the depth of submerged ice for each face of the grid cell adjacent to an ice-free cell -- potentially four faces in total). $\theta_f=10$ is a scaling coefficient that implies the face mass balance should be ~10 times higher than the basal mass balance (Pollard and DeConto, 2016, appendix). 

## Calving schemes 

Here is a summary of calving schemes.

### Lipscomb et al. (2019)

$$
c = k_\tau \tau_{\rm ec}
$$

where $k_\tau$ (m yr$^{-1}$ Pa$^{-1}$) is an empirical constant and $\tau_{\rm ec}$ (Pa) is the effective calving stress, which is defined by:

$$
\tau_{\rm ec}^2 = \max(\tau_1,0)^2 + \omega_2 \max(\tau_2,0)^2
$$

$\tau_1$ and $\tau_2$ are the eigenvalues of the 2D horizontal deviatoric stress tensor and $\omega_2$ is an empirical weighting constant. For partially ice-covered grid cells (with $f_{\rm ice} < 1$), these stresses are taken from the upstream neighbor. 

The eigenvalues $\tau_1$ and $\tau_2$ are calculated from the depth-averaged (2D) stress tensor $\tau_{\rm ij}$ as follows. Given the stress tensor components $\tau_{\rm xx}$, $\tau_{\rm yy}$ and $\tau_{\rm xy}$, we can solve for the real roots $\lambda$ of the tensor from the quadratic equation:

$$
a \lambda^2 + b \lambda + c = 0 
$$

where 

$$
a = 1.0 \\
b = -(\tau_{\rm xx} + \tau_{\rm yy}) \\
c = \tau_{\rm xx}*\tau_{\rm yy} - \tau_{\rm xy}^2
$$

 


glissade_velo_higher.F90:

```
tau_xz(k,i,j) = tau_xz(k,i,j) + efvs_qp * du_dz            ! 2 * efvs * eps_xz
tau_yz(k,i,j) = tau_yz(k,i,j) + efvs_qp * dv_dz            ! 2 * efvs * eps_yz
tau_xx(k,i,j) = tau_xx(k,i,j) + 2.d0 * efvs_qp * du_dx     ! 2 * efvs * eps_xx
tau_yy(k,i,j) = tau_yy(k,i,j) + 2.d0 * efvs_qp * dv_dy     ! 2 * efvs * eps_yy
tau_xy(k,i,j) = tau_xy(k,i,j) + efvs_qp * (dv_dx + du_dy)  ! 2 * efvs * eps_xy
```

