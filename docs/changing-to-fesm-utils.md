# Changing to fesm-utils

Until now, many source files of general helper modules were contained within each repository itself, often with duplication. So `yelmo/libs` contained ncio.f90 and nml.f90, while so did `yelmox/libs`, `FastIsostasy/src`, etc.

Now all (hopefully) of these general libraries have been moved to an independent repository called `fesmc/fesm-utils`. This way, one installation can serve for many interrelated model components that are coupled in YelmoX.

To modify an existing installation to make use of the new framework, please do the following:

```bash
# 1. Update your `fesm-utils` installation and configure/compile new modules (lis and fftw installations remain unchanged)

cd fesm-utils
git pull
cd utils
python config.py config/dkrz_levante_ifx  # replace with config file for your system
make clean
make fesmutils-static

# 2. Update all Yelmox repositories, reconfigure and move location of links to fesm-utils

# yelmox
cd yelmox
python config.py config/dkrz_levante_ifx  # replace with config file for your system
mv libs/fesm-utils ./
make clean

# yelmo
cd yelmo
python config.py config/dkrz_levante_ifx  # replace with config file for your system
mv libs/fesm-utils ./
make clean

# FastIsostasy (note: link to fesm-utils already in main directory)
cd FastIsostasy
python config.py config/dkrz_levante_ifx  # replace with config file for your system
make clean

# rembo1 (if needed)
cd rembo1
python config.py config/dkrz_levante_ifx  # replace with config file for your system
mv libs/fesm-utils ./
make clean

# 3. Check if it worked
cd yelmox
make yelmox

```

That's it! In principle all programs should run exactly the same as before.
