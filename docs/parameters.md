# Parameters

Here important parameter choices pertinent to running
**Yelmo** will be documented. Each section will
outline a specific parameter or set of related parameters.
The author of each section and the date last updated
will apear in the heading, to maintain traceability
in the documentation (since code usually changes over time).


## gvel_par ##
**Alexander Robinson, 2016-11-06**

```
&gvel_par
    mix_method  = 2      ! 0: None, 1: ssa+sia 2: ssa vel. fraction 3: Hwat fraction
    ssa_vref    = 100    ! [m/a] Reference SSA velocity (denominator in tanh function)
    Hmin        = 0      ! [m] Minimum water pressure to activate ssa
    Hmax        = 50     ! [m] Maximum water pressure to reach full ssa conditions (mix_method=3)
/
```

These parameters control the *hybrid* nature of **Yelmo**.
`mix_method` (formerly known as `i_resolmeca`) is a switch that
specifies how the SIA and SSA velocity solutions should be combined:

- `mix_method=0`: None, SIA and SSA domains are treated separately.
- `mix_method=1`: At any point where SSA is calculated, the SIA and SSA solutions are added.
- `mix_method=2`: SIA and SSA solutions are combined as a weighted average with the fraction of SSA determined by the SSA velocities (ie, method of Bueler and Brown, 2009).
- `mix_method=3`: SIA and SSA solutions are combined as a weighted average with the fraction of SSA determined by the basal water pressure.

*Note that the case `i_resolmeca=1` (ssa+sia, if ssa>sia) no longer exists.*

`ssa_vref` **[m/a]** is a scaling coefficient for `mix_method=3`. Default value is `ssa_vref=100`.

`Hmin` and `Hmax` are the minimum and maximum water pressure values, respectively, that define a linear transition between SIA and SSA (ie, below `Hmin` velocity is defined solely by SIA, while above `Hmax`, velocity is defined solely by SSA).

Note that `Hmax` is only used with `mix_method=3`, however `Hmin` is also
the threshold for calculating the SSA solution. Typically `Hmin` is set to a low value to avoid artificially constraining the SSA regions.

## bdrag_par ##
**Alexander Robinson, 2016-10-31**

```
&bdrag_par
    toblim            = 0.7e5  ! creo que no sirven
    tobmax            = 1000.0 ! creo que no sirven
    hmin_rock         = 0.0    ! North=400.0; South=50.0
    hmax_rock         = 500.0
    hmin_sed          = 1.0
    beta_lim          = 1500.0    ! [Pa??]
    beta_max          = 1000.0
    beta_max_island   =  500.0
    beta_min          =   10.0
    beta_sia          = 1e5
    cf_stream         = 20.0
    cf_fac_sed        = 0.1
    cf_fac_coast      = 0.01
    cf_fac_island     = 0.01
    stream_choice     = 0     ! 0: water-pressure; 1: stream-tracing
    limit_friction    = FALSE ! True: the beta has boundaries & SSA points are eliminated if beta is big enough
                              ! False: beta is not limited and SSA can occupate whatever point for whatever beta

/
```

The parameters in this section control the calculation of dragging in **Yelmo**, in particular the dragging coefficient *Βeta*.
For now, most of the parameters are in fact either no longer used, or
in the process of being phased out in favor of new approaches.
However, the four `cf_*` parameters are critical:

- `cf_stream` **[1e5 (m/a)^-1]** This defines the base dragging coefficient, which applies for inland ice streams without sediments.
- `cf_fac_sed` The scaling factor relative to `cf_stream` for sediment-covered points, ∈ (0,1).
- `cf_fac_coast` The scaling factor relative to `cf_stream` for coastal points, ∈ (0,1).
- `cf_fac_island` The scaling factor relative to `cf_stream` for "island" points, ∈ (0,1).

Normally `cf_fac_coast` and `cf_fac_island` are close to zero.
