# DIVA — Depth-Integrated Viscosity Approximation

DIVA is the first-class momentum balance solved in Yelmo. It is a 2D
(depth-integrated) problem for the horizontal velocity, augmented with a
1D vertical closure that recovers the full 3D velocity field. The formulation
follows [Goldberg (2011)](https://doi.org/10.1017/S0022143000004585),
[Arthern et al. (2015)](https://doi.org/10.1002/2014JF003239) and
[Lipscomb et al. (2019)](https://doi.org/10.5194/gmd-12-3199-2019).

DIVA is valid uniformly from slow grounded ice (where vertical shear dominates,
the SIA limit) to fast streaming ice and floating shelves (where membrane
stresses dominate, the SSA limit). It does so by retaining the depth-integrated
membrane stress balance of SSA while reintroducing vertical shear through a
closure on $\partial_z u$ that is consistent with the Glen flow law.

The implementation is in [`src/physics/velocity_diva.f90`][diva_src].

## Continuum equations

### Depth-integrated stress balance

The unknown is the depth-averaged horizontal velocity
$\bar{\mathbf u} = (\bar u, \bar v)$. The momentum balance is the
depth integral of the horizontal Stokes equations, with the assumption
that the longitudinal stresses do not vary in the vertical
($\partial_z u$ contributes only to the strain-rate invariant, not as an
independent unknown):

$$
\partial_x \!\left[\, 2\,N\,(2\,\bar\varepsilon_{xx} + \bar\varepsilon_{yy})\,\right]
+ \partial_y \!\left[\, 2\,N\,\bar\varepsilon_{xy}\,\right]
- \beta_\mathrm{eff}\,\bar u
\;=\; \tau_{d,x},
$$

$$
\partial_y \!\left[\, 2\,N\,(2\,\bar\varepsilon_{yy} + \bar\varepsilon_{xx})\,\right]
+ \partial_x \!\left[\, 2\,N\,\bar\varepsilon_{xy}\,\right]
- \beta_\mathrm{eff}\,\bar v
\;=\; \tau_{d,y},
$$

with the depth-integrated effective viscosity

$$
N \;=\; \int_0^H \eta\,\mathrm dz \;\equiv\; \bar\eta\, H
$$

(stored in the code as `visc_eff_int`). The horizontal strain rates are
$\bar\varepsilon_{xx} = \partial_x \bar u$,
$\bar\varepsilon_{yy} = \partial_y \bar v$,
$\bar\varepsilon_{xy} = \tfrac{1}{2}(\partial_y \bar u + \partial_x \bar v)$.

The **driving stress** on the right-hand side is the depth-integrated
gravitational force, computed in the staggered form

$$
\tau_{d,x} \;=\; \rho_i\, g\, \bar H^{\,x}\, \partial_x z_s,
\qquad
\tau_{d,y} \;=\; \rho_i\, g\, \bar H^{\,y}\, \partial_y z_s,
$$

evaluated on ac-faces by [`calc_driving_stress`][diva_taud] in
`velocity_general.f90`. A grounding-line variant
(`calc_driving_stress_gl`) replaces $H$ with a flotation-corrected
thickness so that no spurious driving stress is generated at the
flotation contact. At calving fronts the lateral boundary
contributes a depth-integrated water-pressure imbalance
$\tau_{l,\mathrm{int}}$ that enters as a Neumann condition.

### Effective viscosity from Glen's flow law

The effective viscosity follows Glen's law,

$$
\eta \;=\; \tfrac{1}{2}\, A^{-1/n}\, \dot\varepsilon_e^{\,(1-n)/n},
$$

(routine [`calc_viscosity_glen`][diva_visc] in `deformation.f90`).
The effective strain rate $\dot\varepsilon_e$ in DIVA uses the
horizontal strain rates plus a contribution from the (closure-defined)
vertical shear of the depth-averaged velocity profile, regularised by
$\varepsilon_0$ to avoid singular viscosity in stagnant ice:

$$
\dot\varepsilon_e^{\,2}
\;=\; \dot\varepsilon_{xx}^{\,2}
   + \dot\varepsilon_{yy}^{\,2}
   + \dot\varepsilon_{xx}\,\dot\varepsilon_{yy}
   + \tfrac{1}{4}\,\dot\varepsilon_{xy}^{\,2}
   + \tfrac{1}{4}\,(\partial_z u)^2
   + \tfrac{1}{4}\,(\partial_z v)^2
   + \varepsilon_0^{\,2}.
$$

This is Eq. (21) of Lipscomb et al. (2019). The two
$(\partial_z u)^2,(\partial_z v)^2$ terms are what make DIVA different
from SSA: they reintroduce the vertical-shear contribution to the
deformation rate, and hence soften the ice in places where vertical
shear is large (slow, grounded interior).

### Vertical-shear closure

Because the depth-integrated equation does not solve for the full vertical
velocity profile, $\partial_z u$ and $\partial_z v$ are obtained from a
closure that is consistent with the Stokes balance under DIVA's
assumptions. With the basal shear stress $\boldsymbol\tau_b$ acting at $\zeta=0$
and vanishing shear stress at the surface,

$$
\partial_z u(\zeta) \;=\; \frac{\tau_{b,x}}{\eta(\zeta)}\,(1-\zeta),
\qquad
\partial_z v(\zeta) \;=\; \frac{\tau_{b,y}}{\eta(\zeta)}\,(1-\zeta).
$$

(Lipscomb et al. 2019, Eq. 36; routine `calc_vertical_shear_3D`.)
Define the **F-integrals**

$$
F_n \;=\; \int_0^1 \frac{H}{\eta(\zeta)}\,(1-\zeta)^n\,\mathrm d\zeta,
$$

(routine `calc_F_integral`, Arthern et al. 2015 Eq. 7, Lipscomb et al.
2019 Eq. 30). These are precisely the integrals needed to relate the
depth-averaged velocity to the basal velocity:

$$
\bar u \;=\; u_b \,+\, \tau_{b,x}\,F_2,
\qquad
\bar v \;=\; v_b \,+\, \tau_{b,y}\,F_2.
$$

### Basal stress closure and effective drag

With a basal friction law of the form $\boldsymbol\tau_b = \beta\,\mathbf u_b$
(see [Parameters](../parameters.md) for the supported choices of $\beta$),
the relation $\bar u = u_b + \tau_{b,x}F_2 = u_b(1 + \beta F_2)$ inverts
to express $\boldsymbol\tau_b$ directly in terms of the depth-averaged
velocity:

$$
\boldsymbol\tau_b \;=\; \beta_\mathrm{eff}\,\bar{\mathbf u},
\qquad
\beta_\mathrm{eff} \;=\; \frac{\beta}{1 + \beta\,F_2}.
$$

This is Eq. (33) of Lipscomb et al. (2019), implemented by
`calc_beta_eff`. In the no-slip limit ($\beta \to \infty$) this reduces
to $\beta_\mathrm{eff} = 1/F_2$ (Lipscomb et al. 2019 Eq. 35), which is
the form used when the user requests `no_slip = .TRUE.`. The effective
drag $\beta_\mathrm{eff}$ — not the raw $\beta$ — is what appears in the
depth-integrated momentum equation above.

### Picard iteration

The viscosity $\eta(\bar{\mathbf u})$ and the effective drag
$\beta_\mathrm{eff}(\bar{\mathbf u})$ both depend on the solution, so the
momentum equation is nonlinear. Yelmo solves it by Picard iteration:

1. Compute $\eta$ from the current $\bar{\mathbf u}$ (and the current
   3D viscosity $\eta(\zeta)$).
2. Relax $\eta$ in log-space against the previous iterate
   (Sandip et al. 2023): `picard_relax_visc`.
3. Compute $N$, the F-integrals $F_n$, and $\beta_\mathrm{eff}$.
4. Assemble and solve the linear system for $\bar{\mathbf u}$ (see
   [Numerical solution](solvers.md)).
5. Relax $\bar{\mathbf u}$ against the previous iterate
   (`picard_relax_vel`) and test an L2-relative convergence criterion.

When the iteration converges, the basal stress
$\boldsymbol\tau_b = \beta_\mathrm{eff}\bar{\mathbf u}$ and basal velocity
$u_b = \bar u - \tau_{b,x} F_2$ are diagnosed, and the 3D horizontal
velocity field is reconstructed by integrating the vertical-shear closure
from the bed upward:

$$
u(\zeta) \;=\; u_b \,+\, \tau_{b,x}\, \tilde F_1(\zeta),
\qquad
\tilde F_1(\zeta) \;=\; \int_0^\zeta \frac{H}{\eta(\zeta')}\,(1-\zeta')\,\mathrm d\zeta'.
$$

(Lipscomb et al. 2019 Eq. 29; routine `calc_vel_horizontal_3D`.)

## Limits

- **SSA limit**: as $\partial_z u, \partial_z v \to 0$, the F-integrals
  enter only through $\beta_\mathrm{eff}\to\beta$ (no vertical-shear
  enhancement of $\dot\varepsilon_e$, no basal/depth-averaged offset). The
  depth-integrated stress balance reduces to the SSA equation — see
  [SSA](ssa.md).
- **SIA limit**: as membrane gradients become negligible relative to
  vertical-shear gradients, the balance collapses to
  $\boldsymbol\tau_b \approx \boldsymbol\tau_d$, and the closure on
  $\partial_z u$ reproduces the SIA velocity profile — see [SIA](sia.md).

[diva_src]: https://github.com/palma-ice/yelmo/blob/main/src/physics/velocity_diva.f90
[diva_taud]: https://github.com/palma-ice/yelmo/blob/main/src/physics/velocity_general.f90
[diva_visc]: https://github.com/palma-ice/yelmo/blob/main/src/physics/deformation.f90
