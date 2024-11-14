# Snapshot climate (snapclim)

The snapclim module is designed to determine climatic forcing, i.e., monthly temperature and precipitation, for a given point in time. This can be acheived by applying a temperature anomaly, or by interpolating snapshots of climate states available for different times.

## The "hybrid" method

This is my preferred method and is set up to be rather flexible, and I think is a good place to start for these simulations. It is comprised of an annual mean temperature anomaly time series from 300 kyr ago to today obtained from several spliced paleo reconstructions plus a monthly seasonal cycle over the 300 kyr obtained from a climber2 paleo run. So with the monthly values and the annual mean, you can get monthly temp anomalies over 300 kyr. There are more details in the attached manuscript that was never yet submitted...

To activate this method, in the parameter file, set the following parameters in the group "snapclim":

```bash
atm_type = "hybrid"
ocn_type = "hybrid" 
```

Then in the group "snapclim_hybrid", you can specify:

```bash
    f_eem              = 0.4     # Controls the maximum temp anomaly during the Eemian
    f_glac             = 1.0     # Controls the minimum temp anomaly during the glacial period 
    f_hol              = 0.5     # Controls the maximum temp anomaly during the Holocene
    f_seas             = 1.0     # Controls the magnitude of the seasonal cycle
    f_to               = 0.2     # Defines the oceanic temperature anomaly relative 
                                 # to the annual mean atmospheric temp anomaly
```
