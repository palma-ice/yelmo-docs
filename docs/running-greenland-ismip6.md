
# Testing without the ice sheet:
jobrun ./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Greenland.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" spinup_ismip6.with_ice_sheet=False spinup_ismip6.time_end=10 spinup_ismip6.dt2D_out=10

jobrun ./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Greenland.nml -- -o ${fldr} -p ctrl.run_step="transient_proj" transient_proj.with_ice_sheet=False

# Specify run choices, to run locally in the background:
runopt='-r'
# or, to submit job to a cluster, eg:
runopt='-s -q priority -w 5'

# Define the output folder
fldr=tmp/ismip6/spinup-grl_16km_2

# Run the Yelmo ensemble

fldr=tmp/ismip6/spinup-grl_16km_1
jobrun ./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Greenland.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" ymat.enh_shear=1,3


fldr=tmp/ismip6/spinup-grl_16km_2
jobrun ./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Greenland.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" opt_L21.opt_tf=True opt_L21.tf_min=-1 opt_L21.tf_max=1

fldr=tmp/ismip6/spinup-grl_16km_3
jobrun ./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Greenland.nml -- -o ${fldr} -p ctrl.run_step="spinup_ismip6" ytopo.calv_flt_method="kill" ytopo.calv_grnd_method="zero"


# Spinup is not great, but functions. Try a transient simulation now

file_restart=/p/tmp/robinson/ismip6/spinup-grl_16km_3/0/yelmo_restart.nc 

fldr=tmp/ismip6/ismip-grl_16km_3-1
jobrun ./runylmox ${runopt} -e ismip6 -n par/yelmo_ismip6_Greenland.nml -- -o ${fldr} -p yelmo.restart=${file_restart} ctrl.run_step="transient_proj" transient_proj.scenario="ctrl","rcp26","rcp85" ismip6.gcm="miroc5" ytopo.calv_flt_method="kill" ytopo.calv_grnd_method="zero" ytill.method=-1
