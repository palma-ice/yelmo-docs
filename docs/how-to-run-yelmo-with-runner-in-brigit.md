# How to install runner and run Yelmo in brigit

## How to install runner

1. Install anaconda first. Create a conda environment `runner` with python=3.7 (this is not needed, just in case you want to keep things separated):

   ```bash
   conda create --name runner python=3.7
   conda activate runner
   ```

2. Install `runner` from `pip`:

```bash
pip install https://github.com/alex-robinson/runner/archive/refs/heads/master.zip
pip install tabulate
pip install numpy
pip install scipy
pip install six
pip install pandas
```

## How to run Yelmo

### To run a simple simulation for yelmox (run in the background)

```bash
./runylmox -r -e yelmox -o output/test1 -n par/yelmo_Greenland.nml 
```

### To run a simulation with one parameter changed from the class ytopo (run in the background)

```bash
./runylmox -r -e yelmox -o output/test2 -n par/yelmo_Greenland.nml -p ytopo.kt=0.10e-2
```

### To send runs to the queue (with runner)

Copy `runylmox.js` from `config/` and make sure it is modified as it should be for brigit. The form of `runylmox.js` for brigit is something like:

```json
{
      "defaults" :
      {
         "jobname"       : "Yelmo",
         "email"         : "USER@ucm.es",
         "group"         : "",
         "omp"           : 0,
         "wall"          : 24,
         "qos"           : "normal",
         "partition"     : "long",
         "job_template"  : "config/brigit_submit_slurm"
      },
      "exe_aliases" :
      {
         "yelmox" : "libyelmox/bin/yelmox.x",
         "iso"    : "libyelmox/bin/yelmox_iso.x",
         "hyst"   : "libyelmox/bin/yelmox_hyst.x",
         "rembo"  : "libyelmox/bin/yelmox_rembo.x",
         "ismip6" : "libyelmox/bin/yelmox_ismip6.x"
      },
      "grp_aliases" : {},
      "par_paths" : {},
      "files" : [],
      "dir-special" : {},
      "links" :
         ["input","ice_data"],
      "const_paths" :
      {   "EISMINT"  : "par/yelmo_const_EISMINT.nml",
         "MISMIP3D" : "par/yelmo_const_MISMIP3D.nml"
      },
      "const_path_default" : "par/yelmo_const_Earth.nml",
      "job_queues" :
      {     "normal" :
         {   "wall" : 10000  }
      }
}
```

Then submit the job to the queue with:

```bash
jobrun ./runylmox -s -e yelmox -n par/yelmo_Greenland.nml -- -o output/test3 -p opt_L21.cf_min=1e-3,2e-3
```

To run en ensemble:

```bash
jobrun ./runylmox -s -e yelmox -n par/yelmo_Greenland.nml -- -o output/test4 -p opt_L21.cf_min=1e-3,2e-3 ytopo.H_min_flt=0.0,10.0,50.0
```

To run an ensemble from a txt file containing the permutation of parameters:

```bash
job product opt_L21.cf_min=1e-3,2e-3,3e-3,4e-3 ytopo.H_min_flt=0.0,10.0,50.0 > params.txt
jobrun ./runylmox -s -e yelmox -n par/yelmo_Greenland.nml -- -o output/test5 -i params.txt
```

To run an ensemble from a selection of values in the param file:

```bash
jobrun ./runylmox -s -e yelmox -n par/yelmo_Greenland.nml -- -o output/test6 -i params.txt -j 1,4,5-7
```
