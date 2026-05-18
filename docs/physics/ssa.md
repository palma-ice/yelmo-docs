# SSA — Shallow Shelf Approximation

The SSA is the membrane-stress balance that DIVA reduces to in the limit
of vanishing vertical shear. It treats the ice column as a vertical plug:
the horizontal velocity is independent of $\zeta$, so $\bar{\mathbf u}$
*is* the basal velocity, $\bar{\mathbf u} = \mathbf u_b$. The
implementation is in
[`src/physics/velocity_ssa.f90`](https://github.com/palma-ice/yelmo/blob/main/src/physics/velocity_ssa.f90).

Yelmo uses the same depth-integrated 2D solver as DIVA — the difference
is purely in the closure. SSA is the appropriate model for floating ice
shelves (no basal drag, no vertical shear) and the right physical limit
in fast-streaming grounded ice.

## Continuum equations

The depth-integrated stress balance has the same shape as the DIVA
equation but with the **raw** friction $\beta$ instead of
$\beta_\mathrm{eff}$:

$$
\partial_x \!\left[\, 2\,N\,(2\,\bar\varepsilon_{xx} + \bar\varepsilon_{yy})\,\right]
+ \partial_y \!\left[\, 2\,N\,\bar\varepsilon_{xy}\,\right]
- \beta\,\bar u
\;=\; \tau_{d,x},
$$

$$
\partial_y \!\left[\, 2\,N\,(2\,\bar\varepsilon_{yy} + \bar\varepsilon_{xx})\,\right]
+ \partial_x \!\left[\, 2\,N\,\bar\varepsilon_{xy}\,\right]
- \beta\,\bar v
\;=\; \tau_{d,y},
$$

with $N = \bar\eta H$, driving stress $\boldsymbol\tau_d$ and friction law
$\boldsymbol\tau_b = \beta\,\mathbf u_b$ defined exactly as in [DIVA](diva.md).

## What is dropped relative to DIVA

The effective strain rate omits the $\partial_z u$ and $\partial_z v$
terms:

$$
\dot\varepsilon_e^{\,2}
\;=\; \dot\varepsilon_{xx}^{\,2}
   + \dot\varepsilon_{yy}^{\,2}
   + \dot\varepsilon_{xx}\,\dot\varepsilon_{yy}
   + \tfrac{1}{4}\,\dot\varepsilon_{xy}^{\,2}
   + \varepsilon_0^{\,2},
$$

so the Glen viscosity $\eta$ depends only on horizontal strain rates.
There is no F-integral closure: $F_2 \to 0$, so

$$
\beta_\mathrm{eff} \to \beta, \qquad u_b \to \bar u.
$$

The basal stress diagnosed after the solve is simply
$\boldsymbol\tau_b = \beta\,\bar{\mathbf u}$.

For ice shelves $\beta = 0$ identically and only the membrane terms and
$\boldsymbol\tau_d$ remain — this is the SSA in its purest form.
