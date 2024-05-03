A repository for setting up an installation of MaRGE on MacOS.
https://github.com/josalggui/MaRGE
Based on https://github.com/josalggui/MaRGE/tree/8496e873905cb16adf16a8cc1700d6a9f4ea666b

See there for more instructions.

See also https://github.com/marcos-mri/

This was setup during the ezyMRI NerdFest Portable MRI Hackathon 2024 in Singapore https://ezymri-nerdfest.sciencesconf.org/.

# installation
We will use conda for the management of python interpreter and packages 

MaRGE also makes use of the `timeout` command which is not by default available on mac.
We will install it via the homebrew package manager.

## prerequisites
You should be somewhat familiar with the MacOS terminal application.
We will use it to run commands from the command line.

You must already have a red pitaya with a marcos-compatible image installed.
These instructions do not cover this process.
Check the marcos wiki here for more information: https://github.com/vnegnev/marcos_extras/wiki.
During the hackathon, there were however challenges communicating with these images, at times.

**With the current version of the code in this repository, the red pitaya must have IP address 192.168.1.101.**
What we do need to know from the red pitaya, is its IP address, the exact model and, if present, the gradient DAC board used.

### conda
install conda: see https://anaconda.org. Choose either the full anaconda or miniconda.
test that it works: you can type `conda` in your terminal and there is no error.

### homebrew timeout command
to install 'timeout' we will install the homebrew package manager:

https://brew.sh and follow the instructions. especially the last step where it asks you to copy a command to the terminal

Then, restart your terminal.

If everything worked, you can now type `brew` and it will not give you an error about an undefined command.

Install the `timeout` command with

`brew install coreutils`

test that it worked by typing `timeout`. This should print the instructions how to use `timeout`.


### git
Make sure git is installed.
Type `git` in the terminal and make sure that there are no error messages.
The installation of homebrew (previous step) should have also installed git.
If it's not installed (git command not found) you have to install git.
The web should have instructions on how to do that.

### 'git clone' this repository (download contents of this repository)

`git clone --recurse-submodules https://github.com/gabuzi/marcos-marge-macos.git`
This will clone all necessary code into a subdirectory `marcos-marge-macos` in the current directory.
Then `cd marcos-marge-macos` to change into this new directory.
Run all following commands from within this folder.

### Additional configurations
#### Network
You have to make sure that you have an ethernet adapter connected to your mac that is 
plugged in to the red pitaya's ethernet port.
If you are unsure which adapter you have to configure in the preferences, you can remove it and plug it back in (it might be necessary to have the device on the other end connected via a lan cable and powered up).
Then just observe which adapter changes its status to identify the right one.
Alternatively, you can click through the adapters and check their 'Details -> Hardware'.
Only those plugged into the computer will have a MAC address listed.

This network adapter must be configured with a static ip address (the option in the MacOS preferences is called 'manual').
The ip address must be in the same subnet as the red pitaya's IP address.
If in doubt use just a neighboring address (e.g. if your red pitaya has 192.168.1.101, use 192.168.1.100 but general networking concepts are out of scope for these instructions).


## Installation of MaRGE
### setup conda environment OPTION A: if you have a mac with apple silicon (M1, M2, M3 based mac)
create an x86-64-based conda environment (bm4d, a mandatory requirement for MaRGE requires this as it is not compiled for the new ARM based M1, M2, M3, M... MAC CPUs)

`CONDA_SUBDIR=osx-64 conda create -n marcos-marge python=3.9`
activate it and then persist the subdir config for any future conda commands:
`conda activate marcos-marge`
`conda config --env --set subdir osx-64`

### setup conda environment OPTION B: if you don't have that (older mac with intel CPU)
any conda environment you create will already be x86-64, so we can just use:

`conda create -n marcos-marge python=3.9`

to create a conda environment with name 'marcos-marge'
activate this environment

`conda activate marcos-marge`

## install python packages
we install the python requirements with pip.
for mac based systems, we remove one dependency which uses cuda gpus.
cuda cupy is not available on macs, so we remove this and use a modified requirements.txt file.
this may make some processing features of marge fail, but the basics should still work.

run `pip install -r requirements_macos.txt`


## setup configs
Confirm that we are in the correct conda environment by confirming that your terminal prompt line now says
`(marcos-marge)` in the beginning.

edit the file 
`marge/configs/hw_config.py.copy`
and set all your required hw specific options.
most importantly, is probably the `rp_ip_address` to point to the red pitaya (depends on the image
you have used and which way it was configured.

ADDITIONALLY: set the `bash_path` to `/bin/bash` such that the line reads
`bash_path = /bin/bash`

save the file as `hw_config.py` (i.e. remove the .copy extension)

edit the file
`marge/configs/hw_config.py.copy`
and check what you want to change.
then save as `marge/configs/hw_config.py` (i.e. remove the .copy extension)

edit the file
`marge/configs/units.py.copy`
and check what you want to change.
then save as `marge/configs/units.py` (i.e. remove the .copy extension)

edit the file
`marcos_client/local_config.py.example`
remove the `#` in front of the line `#ip_address = "192.168.1.189"` and set the ip address to the one corresponding to your red pitaya (same as in step above)

then also remove the `#` in front of the right option for the `fpga_clk_freq_MHz`

then also remove the `#` in front of the right option for the `grad_board`

save the file as `marcos_client/local_config.py` (i.e. remove the .example extension)

## make sure you can use ssh to talk to the red pitaya
type `ssh root@<xxx.xxx.xxx.xxx>` where `<xxx.xxx.xxx.xxx>` is the ip address of your red pitaya.
if you have never connected to the red pitaya before, this should show a confirmation dialog to confirm the key. Confirm with `yes`.

If you have problems with this step, make sure that
* ethernet is connected to red pitaya and it is powered on
* red pitaya has sd card inserted with correctly configured image
* you are not also in a second network (e.g. wifi) that has addresses in the same range as the red pitaya (e.g. most home wireless routers by default would do that by giving out addresses in the 192.168.1.x range. If so, disconnect wifi to check. In the long term, you should reconfigure red pitaya to use a different ip range, otherwise you won't be able to use internet on the laptop while being connected to red pitaya.

## launching MaRGE (the GUI)
make sure that your red pitaya is connected (step before)
then make sure that you are in the folder `MaRGE` of this repository.
if not, use the `cd` command to navigate to this directory.
you can use the `pwd` command to see where you are currently, and the `ls` command to list the files and directories in the current directory.

then make sure that you are in the right conda environment `(marcos-marge)` in front of your terminal if you followed these instructions.
if not, use `conda activate marcos-marge` to activate the environment (this makes all the required python packages that we have previously installed available).

type `python -m main` to launch the GUI.

You should see the `session window` appear.
Confirm your inputs and click on the "house" icon on the top left.

Then, the full GUI should open.
You should follow MARGE instructions to figure out what to do next.
You can always switch back to the terminal window that you have used to launch the GUI to check for error messages.

then click through the buttons to initialize marcos (should show a blue LED on the red pitaya),
then start the marcos server and then finally launch the currently selected sequence with the clipboard icon button.
Amber LEDs should flash if it's working.

Refer to the MaRGE and MaRCoS instructions for more info.

