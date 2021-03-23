# Dependencies 

Yelmo is dependent on the following libraries:

- [NetCDF](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html)
- [Library of Iterative Solvers for Linear Systems](http://www.ssisc.org/lis/)
- [Optional] ['runner' Python library (alex-robinson fork)](https://github.com/alex-robinson/runner)

Installation tips for each dependency can be found below.

## Installing NetCDF (preferably version 4.0 or higher)

The NetCDF library is typically available with different distributions (Linux, Mac, etc).
Along with installing `libnetcdf`, it will be necessary to install the package `libnetcdf-dev`.
Installing the NetCDF viewing program `ncview` is also recommended.

If you want to install NetCDF from source, then you must install both the
`netcdf-c` and subsequently `netcdf-fortran` libraries. The source code and
installation instructions are available from the Unidata website:

[https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html)

## Installing LIS

1. Download the LIS source:
[https://www.ssisc.org/lis/](https://www.ssisc.org/lis/)

2. Configure the package (where is the desired installation location),
and install it in the location of your choice (below defined as `$LISROOT`). Also, make sure to enable the Fortran90 interface:

```
cd lis-2.0.18
./configure --prefix=$LISROOT --enable-f90
make
make install
make install check
```

Note: make sure to set the environment variables `CC` and `FC`, in order to set
a specific compiler, for example for gcc/gfortran use the following configure command:

```
CC=gcc FC=gfortran ./configure --prefix=$LISROOT --enable-f90
```

3. Add LIS path to the `LD_LIBRARY_PATH` in `.bash_profile`, `.bashrc` or `.bash_aliases`:

```
# lis library paths
LD_LIBRARY_PATH=$LISROOT/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH
```

That's it. LIS should now be available to use with Yelmo.

## Installing runner

1. Install `runner` to your system's Python installation via `pip`, along with dependency `tabulate`.

```
pip install https://github.com/alex-robinson/runner/archive/refs/heads/master.zip
pip install tabulate
```

That's it! Now check that system command `job` is available by running `job -h`. 

Note that install method `python setup.py install` should be avoided if possible to maintain Python system integrity.
