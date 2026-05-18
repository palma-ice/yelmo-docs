# SIA — Shallow Ice Approximation

The SIA is the complementary limit to SSA: membrane stresses are
neglected and the horizontal stress balance is dominated by the basal
drag exerted by vertical shear. Yelmo's SIA solver is implemented in
[`src/physics/velocity_sia.f90`](https://github.com/palma-ice/yelmo/blob/main/src/physics/velocity_sia.f90)
and is used to provide the vertical-shear contribution to the velocity
field in the hybrid SIA + SSA mode and as a diagnostic on its own.

Crucially, Yelmo formulates SIA in **stress form**: the shear stress and
the basal stress are computed *from the driving stress* $\boldsymbol\tau_d$
exactly as in DIVA, and the velocity is then obtained by integrating
Glen's flow law vertically. This is mathematically equivalent to the
familiar closed-form SIA expression in terms of $\nabla z_s$, but it
keeps the link to the Stokes balance explicit, uses the same
$\boldsymbol\tau_d$ field as DIVA/SSA (including the same lateral and
grounding-line treatments), and makes the SIA → DIVA limit transparent.

## Continuum equations

The horizontal momentum balance, neglecting membrane stresses and
vertical accelerations, reduces to a balance between the vertical
gradient of horizontal shear stress and the horizontal pressure gradient
of the gravitational driving:

$$
\partial_z \tau_{xz} \;=\; \rho_i\, g\, \partial_x z_s,
\qquad
\partial_z \tau_{yz} \;=\; \rho_i\, g\, \partial_y z_s.
$$

Integrating from $\zeta$ upward to the stress-free surface and writing
the result in terms of the same driving stress as DIVA/SSA gives the
linear-with-depth profile

$$
\tau_{xz}(\zeta) \;=\; -(1-\zeta)\,\tau_{d,x},
\qquad
\tau_{yz}(\zeta) \;=\; -(1-\zeta)\,\tau_{d,y},
$$

with the driving stress

$$
\tau_{d,x} \;=\; \rho_i\, g\, \bar H^{\,x}\, \partial_x z_s,
\qquad
\tau_{d,y} \;=\; \rho_i\, g\, \bar H^{\,y}\, \partial_y z_s,
$$

evaluated on ac-faces by the *same* `calc_driving_stress` routine used
by DIVA and SSA (see [DIVA](diva.md)). The basal stress is recovered at
$\zeta = 0$: $(\tau_{xz},\tau_{yz})|_{\zeta=0} = -\boldsymbol\tau_d$,
i.e. all of the driving stress is balanced at the bed, as expected when
membrane stresses are dropped. The shear stress profile is built by
`calc_shear_stress_3D`.

## Velocity from Glen's flow law

With the shear-stress field in hand, the horizontal shear strain rates
follow directly from Glen's law,

$$
\dot\varepsilon_{xz}
\;=\; A(T'(\zeta))\, \tau_e^{\,n-1}\,\tau_{xz},
\qquad
\dot\varepsilon_{yz}
\;=\; A(T'(\zeta))\, \tau_e^{\,n-1}\,\tau_{yz},
$$

where the effective stress under SIA assumptions is
$\tau_e^{\,2} = \tau_{xz}^{\,2} + \tau_{yz}^{\,2}$.
Since $\dot\varepsilon_{xz} = \tfrac{1}{2}\partial_z u$ in the SIA limit,
the horizontal velocity is obtained by integrating from the bed upward:

$$
u(\zeta) \;=\; u_b \,+\, 2\int_0^\zeta A(T'(\zeta'))\,\tau_e^{\,n-1}\,\tau_{xz}\,H\,\mathrm d\zeta',
$$

and analogously for $v$. The integral is evaluated layer-by-layer with
the trapezoidal rule (routine `calc_uxy_sia_3D`), so the discrete update
between vertical levels $\zeta_{k-1}$ and $\zeta_k$ reads

$$
u(\zeta_k) \;=\; u(\zeta_{k-1})
\,+\, A\,\Delta\zeta\,H\,\bigl(\tau_{xz,k}^{\,2}+\tau_{yz,k}^{\,2}\bigr)^{(n-1)/2}
\bigl(\tau_{xz,k} + \tau_{xz,k-1}\bigr),
$$

and analogously for $v$. The depth-averaged velocity
$\bar{\mathbf u}^{(\mathrm{SIA})}$ is obtained by trapezoidal
integration over $\zeta$.

In hybrid mode, Yelmo uses $\bar{\mathbf u}^{(\mathrm{SIA})}$ to provide
the shearing contribution that is added to the basal velocity produced
by the membrane (DIVA/SSA) solve. In a pure-SIA configuration, the
basal velocity is either zero (frozen bed) or comes from a sliding law
of choice (not the default in current Yelmo versions).

## Relationship to the textbook SIA

Substituting the closed-form $\tau_{xz} = -(1-\zeta)\rho_i g H\partial_x z_s$
into the integral above and assuming $A$ constant with $\zeta$ recovers
the familiar SIA velocity

$$
u(\zeta) \;=\; u_b
\,-\, 2\,(\rho_i g)^n\,A\,H^{n+1}\,|\nabla z_s|^{\,n-1}\,\partial_x z_s
   \bigl[\,1 - (1-\zeta)^{n+1}\,\bigr]\,/\,(n+1),
$$

but Yelmo does not collapse to that form internally. Keeping the
stress-form integral is what allows the SIA solver to share
$\boldsymbol\tau_d$, $A(T'(\zeta))$ and the discretisation conventions
with the DIVA solver and the rest of the model.
