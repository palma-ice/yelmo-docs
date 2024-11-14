# Dependencies 

Yelmo is dependent on the following libraries:

- [NetCDF](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html)
- [Library of Iterative Solvers for Linear Systems](http://www.ssisc.org/lis/)
- [Optional] ['runner' Python library (cxesmc fork)](https://github.com/cxesmc/runner)

YelmoX is additionally dependent on the following library:

- FFTW (ver. 3.9+)

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

```bash
cd lis-2.0.18
./configure --prefix=$LISROOT --enable-f90
make
make install
make install check
```

Note: make sure to set the environment variables `CC` and `FC`, in order to set
a specific compiler, for example for gcc/gfortran use the following configure command:

```bash
CC=gcc FC=gfortran ./configure --prefix=$LISROOT --enable-f90
```

3. Add LIS path to the `LD_LIBRARY_PATH` in `.bash_profile`, `.bashrc` or `.bash_aliases`:

```bash
# lis library paths
LD_LIBRARY_PATH=$LISROOT/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH
```

That's it. LIS should now be available to use with Yelmo.

## Installing runner

1. Install `runner` to your system's Python installation via `pip`, along with dependency `tabulate`.

```bash
pip install https://github.com/cxesmc/runner/archive/refs/heads/master.zip
```

That's it! Now check that system command `job` is available by running `job -h`. If the command is not found, it means that the Python bin directory is not available in your PATH. To add it, typically something like this is needed in your .profile or .bashrc file:

```bash
PATH=${PATH}:${HOME}/.local/bin
export PATH
```
