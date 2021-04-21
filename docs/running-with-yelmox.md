# Running with YelmoX

YelmoX is a separate repository that is designed to provide supplementary libraries and programs that allow running ice-sheet simulations with realistic boundary (e.g., climate and ocean) forcing and interactions (e.g., isostatic rebound).

![Ice sheet interactions](img/yelmo_test_color.png)

![YelmoX interface](img/yelmox_overview.png)

Here you can find the basic information and steps needed to get **YelmoX** running.

## Super-quick start

A summary of commands to get started is given below. For more detailed information see subsequent sections.

```
# Clone repository
git clone https://github.com/palma-ice/yelmox.git
git clone git@github.com:palma-ice/yelmox.git # via ssh

# Enter directory and run configuration script
cd yelmox
python config.py config/pik_ifort 

# Clone yelmo into a sub-directory and configure it too 
git clone https://github.com/palma-ice/yelmo.git
git clone git@github.com:palma-ice/yelmo.git # via ssh
cd yelmo 
python config.py config/pik_ifort 

# Return to the main directory and compile the default program
cd ../
make clean 
make yelmox 

# Link to `ice_data` repository
ln -s path_to/ice_data 

# Run a test simulation of Greenland for 100 yrs
./runylmox -r -e yelmox -o output/grl-test -n par/yelmo_Greenland.nml
```

