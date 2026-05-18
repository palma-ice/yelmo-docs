# Physics

This section describes the continuum equations and numerical methods that Yelmo
uses to solve the coupled momentum balance of grounded ice and floating ice
shelves. The goal is to document what the code actually implements, with the
same variable names used in the source so that the equations here can be traced
back to specific routines.

The ice flow in Yelmo is described as a slow, incompressible, gravity-driven
flow of a power-law (Glen) fluid. The full Stokes problem is never solved.
Instead Yelmo offers three approximations, in increasing order of fidelity to
Stokes:

| Approximation | Vertical shear | Membrane stresses | Use case |
|---|---|---|---|
| [SIA](sia.md)  | yes (closure)        | no  | slow grounded interior, used as a *complement* to DIVA in hybrid mode |
| [SSA](ssa.md)  | no (plug flow)       | yes | floating shelves, fast-streaming grounded ice |
| [DIVA](diva.md) | yes (closure) | yes | first-class momentum balance, valid from slow interior to streaming flow and shelves |

DIVA is the recommended choice in Yelmo and is solved as a single 2D problem
for the depth-averaged horizontal velocity. SSA is recovered from DIVA in the
limit of vanishing vertical shear, and SIA is recovered in the limit of
vanishing membrane stresses and large basal drag. All three formulations share
the same driving stress, the same Glen viscosity, and (where applicable) the
same family of friction laws — see [Parameters](../parameters.md) for the
parameter choices.

The fourth page in this section documents how the DIVA / SSA momentum balance
is [discretised and solved](solvers.md), including the two solvers currently
supported: the **residual** assembler inherited from Yelmo v1, and the
**energy** assembler that minimises a discrete energy functional.

## Notation used throughout

- $u, v$: horizontal velocity components $[\mathrm{m\,a^{-1}}]$.
- $\bar u, \bar v$: depth-averaged horizontal velocity (the DIVA/SSA unknowns).
- $u_b, v_b$: basal horizontal velocity.
- $H$: ice thickness $[\mathrm{m}]$; $z_s$ and $z_b$: ice surface and base elevations.
- $\zeta \in [0,1]$: normalised vertical coordinate, $\zeta=0$ at the base, $\zeta=1$ at the surface.
- $\rho_i$, $g$: ice density and gravitational acceleration.
- $A(T')$: Glen rate factor, depending on the pressure-corrected temperature $T'$
  and (where relevant) the water content $\omega$. $n=n_\mathrm{glen}$ is the Glen exponent (typically 3).
- $\dot\varepsilon_{ij}$: components of the horizontal strain-rate tensor.
- $\dot\varepsilon_e$: effective strain rate, the second invariant of $\dot\varepsilon_{ij}$.
- $\eta$: effective viscosity; $N = \int_0^H \eta\,\mathrm{d}z \equiv \bar\eta H$: depth-integrated effective viscosity.
- $\boldsymbol\tau_d$: gravitational driving stress.
- $\boldsymbol\tau_b$: basal shear stress.
- $\beta$: basal friction coefficient, defined by $\boldsymbol\tau_b = \beta\,\mathbf u_b$.

All horizontal fields live on the Arakawa C-grid (called the **ac-grid** in
Yelmo): scalars at cell centres (`aa`-nodes), $u$ on the right face of each cell
(`acx`-nodes), $v$ on the top face (`acy`-nodes), and cross-derivative
quantities at the cell corners (`ab`-nodes).
