# Docker for NMR software
Docker image for NMR software. Running on Ubuntu 16.04 LTS.

The [README with links can be found by clicking here](https://github.com/tlinnet/docker_relax/blob/master/README.md)

[A screencast can be seen on YouTube by clicking here.](https://youtu.be/6isTXHYw9Uc)

The purpose is to make a container with relevant NMR software for daily processing of data. 

Consider this workflow:

* Unpack data with commands **bruker**, **varian** or **qMDD** to test.ft2
* Inspect test.ft2 in **nmrDraw** or start a **JupyterLab**, and make images with **nmrglue** and matplotlib.
* Use the **JupyterLab**, to write your work process and include images and math text.
* Prepare data files in **JupyterLab**
* Execute a script for **relax**, depending on the data files.

You now have a Jupyter notebook, with images+math+workflow, which you can share and send to collaborators. See [the example section of relaxation analysis, for such an example](#relaxationanalysis) 

**This images includes software for:**

* [relax](http://www.nmr-relax.com/) with [OpenDX](http://wiki.nmr-relax.com/OpenDX) -> [Jump to commands](#relax)
* [NMRPipe](https://www.ibbr.umd.edu/nmrpipe/install.html) -> [Jump to commands](#NMRPipe)
* [MddNMR](http://mddnmr.spektrino.com/download) -> [Jump to commands](#MddNMR)
* [nmrglue](https://www.nmrglue.com/) -> [Jump to commands](#nmrglue)
* Art Palmers software: [ModelFree4](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/modelfree), [FastModelFree](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/modelfree), [Quadric](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/quadric-diffusion), [PDBinertia](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/pdbinertia) -> [Jump to commands](#Palmer)
* [Sparky](http://www.cgl.ucsf.edu/home/sparky) -> [Jump to commands](#Sparky)
* [CcpNmr Analysis 2.4](http://www.ccpn.ac.uk/v2-software/downloads) -> [Jump to commands](#Analysis)
* [JupyterLab](http://jupyter.org)-> [Jump to commands](#Jupyter)
* [mMass Mass Spectrometry Tool](http://mmass.org/)-> [Jump to commands](#mmass)

For deleting images, go to -> [Developer section](#Developer)<br>
For examples, go to -> [Examples section](#examples)

# Get prebuild image:
```bash
docker pull tlinnet/relax
```
# Running docker with image <a name="run"></a>
[Link to run reference:](https://docs.docker.com/v1.11/engine/reference/commandline/run)

```bash
# See images on machine
docker images
```
## Running on a mac <a name="runmac"></a>
```bash
# First make sure XQuartz is running
open -a XQuartz
# In XQuartz -> Preferences > Security, make sure the tick 
# "Allow connections from network clients" is ON.

# Then set DISPLAY options.
xhost + `ifconfig|grep "inet "|grep -v 127.0.0.1|cut -d" " -f2`

# Then make alias and run.
alias dr='docker run -ti --rm -p 8888:8888 -e DISPLAY=$(ifconfig|grep "inet "|grep -v 127.0.0.1|cut -d" " -f2):0 -v /tmp/.X11-unix:/tmp/.X11-unix -v "$PWD":/home/jovyan/work --name relax tlinnet/relax'
# Run it
# With no arguments, starts Jupyter notebook
dr
# Or else start bash, to start programs
dr bash
```
## Easy run of docker by adding alias to shell profile file
To make this easier on a **mac**, consider adding this to **HOME/.bash_profile**

```bash
# Start docker, if it is not running
alias drdocker='open -a /Applications/Docker.app/Contents/MacOS/Docker'
# Start  XQuartz, if it is not running
alias drx='open -a XQuartz; xhost + `ifconfig|grep "inet "|grep -v 127.0.0.1|cut -d" " -f2`'

# Run "Docker Relax": dr
alias dr='docker run -ti --rm -p 8888:8888 -e DISPLAY=$(ifconfig|grep "inet "|grep -v 127.0.0.1|cut -d" " -f2):0 -v /tmp/.X11-unix:/tmp/.X11-unix -v "$PWD":/home/jovyan/work --name relax tlinnet/relax'

# Run "Docker Relax Execute ": For example: dre bash
# This is when then Docker Relax image is already running.
alias dre='docker exec -it relax'
```
## Running on windows <a name="runwin"></a>
First make sure Xwindow is running. I'm using [vcxsrv](https://sourceforge.net/projects/vcxsrv/) installed by [Chocolatey](https://chocolatey.org/).
You can follow the procedure [here](https://dev.to/darksmile92/run-gui-app-in-linux-docker-container-on-windows-host-4kde).

If you want to share files to docker follow this [link](https://docs.sevenbridges.com/docs/mount-a-usb-drive-in-a-docker-container); the very last entry titled "MOUNT THE USB DRIVE IN A DOCKER CONTAINER ON A WINDOWS HOST WITH DOCKER FOR WINDOWS".

```bash
# Open an admin powershell.

# Then set DISPLAY options.
set-variable -name DISPLAY -value YOUR-IP:0.0

# Then run relax.
docker run -ti --rm -e DISPLAY=$DISPLAY tlinnet/relax relax -g

# Or if you want to link to a folder on your computer
docker run -v [source path]:[destination path] -ti --rm -e DISPLAY=$DISPLAY tlinnet/relax relax -g
# For example
docker run -v e:\docker\relax:/home/jovyan/work -ti --rm -e DISPLAY=$DISPLAY tlinnet/relax relax -g
 ```
# Windows alias uses the "function" command. 
## See documentation [here](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_functions?view=powershell-6).
```
# Need to impliment variable so nmrDraw, sparky, etc. can be run the same as on mac.
function dr {
    Param([switch]$g)
        if ($g) {
            docker run -v e:\docker\relax:/home/jovyan/work -ti --rm -e DISPLAY=$DISPLAY tlinnet/relax relax -g }
        else {
            docker run -v e:\docker\relax:/home/jovyan/work -ti --rm -e DISPLAY=$DISPLAY --name relax tlinnet/relax }
    
}

# NOTE: A work in progress... directory does not connect to environment properly.
function dr {
    Param(
        [Parameter(Mandatory=$true, position=0)]
        [string]
        $val1,
        [Parameter(Mandatory=$false, position=1)]
        [string]
        $val2,
        [Parameter(Mandatory=$false, position=2)]
        [string]
        $val3,
        [Parameter(Mandatory=$false)]
        [switch]
        $g,
        [Parameter(Mandatory=$false)]
        [switch]
        $l)
                        
        if ($g) {
            docker run -v "${pwd}:/home/jovyan/work" -ti --rm -e DISPLAY=$DISPLAY tlinnet/relax $val1 $val2 -g }
        if($l) {
            docker run -v "${pwd}:/home/jovyan/work" -ti --rm -e DISPLAY=$DISPLAY tlinnet/relax $val1 -l $val2 $val3 }
        else {
            docker run -v "${pwd}:/home/jovyan/work" -ti --rm -e DISPLAY=$DISPLAY tlinnet/relax $val1 $val2 }    
}
```
# Installed programs
## relax with OpenDX <a name="relax"></a>
* [relax](http://www.nmr-relax.com/) with [OpenDX](http://wiki.nmr-relax.com/OpenDX)

```bash
# Start relax in GUI
dr relax -g

# Start OpenDX
dr dx

# Try OpenMPI for running with multiple CPU. Not tested.
# To start into bash
dr bash
# Then in terminal try
mpirun --version
mpirun -np 2 echo "hello world"
mpirun --report-bindings -np 2 echo "hello world"
```
## nmrPipe 
* [NMRPipe](https://www.ibbr.umd.edu/nmrpipe/install.html)

```bash
# Start nmrDraw. It apparently takes 1-2 min to open window?
dr nmrDraw
```
## MddNMR <a name="MddNMR"></a>
* [MddNMR](http://mddnmr.spektrino.com/download)

```bash
# First need to start terminal before running qMDD
dr bash
qMDD
```
## nmrglue <a name="nmrglue"></a>
* [nmrglue](https://www.nmrglue.com/)

Have a look here, for longer example together with JupyterLab [is explained here.](#nmrglue_ex)

```bash
dr python -c "import nmrglue; print(nmrglue.__version__)"
```


## Art Palmers software <a name="Palmer"></a>
* Art Palmers: [ModelFree4](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/modelfree), [FastModelFree](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/modelfree), [Quadric](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/quadric-diffusion), [PDBinertia](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/pdbinertia)

```bash
# modelfree
dr modelfree4

# FastModelFree
dr setupFMF

# quadric-diffusion
dr quadric_diffusion
dr r2r1_tm

# PDBinertia
dr pdbinertia
```
## Sparky <a name="Sparky"></a>
* [Sparky](http://www.cgl.ucsf.edu/home/sparky)

```bash
dr sparky
```
## CcpNmr Analysis 2.4 <a name="Analysis"></a>
* [CcpNmr Analysis 2.4](http://www.ccpn.ac.uk/v2-software/downloads)

```bash
dr analysis
```
## JupyterLab <a name="Jupyter"></a>
First make aliases as described in [aliases for mac](#runmac)

Then run

```bash
# Jupyter Notebook is running by default.
dr
```

Then visit in our browser: [http://0.0.0.0:8888](http://0.0.0.0:8888)<br>
NOTE: If you by accident use: **http://0.0.0.0:8888/tree**, the JupyterLab extension will NOT work.

## mMass <a name="mmass"></a>
* [mMass Mass Spectrometry Tool](http://mmass.org/)-> [Jump to commands](#mmass)

GUI for working with Mass Spectrometry

```bash
dr mmass
# Open mzML files right away
dr mmass Analysis.mzML
```
# Developer <a name="Developer"></a>

To open a bash terminal in the container, when it is running

```bash
docker exec -it relax bash
# Or with the alias defined from above
dre bash
```
## Build own image.
[Link to build reference:](https://docs.docker.com/v1.11/engine/reference/commandline/build)

This will build 6 docker images, chained after each other.
This is to save time in the building phase.

1. Image makes apt-get install of packages
2. Install python packages with pip and conda
3. Setup the user
4. Setup NMRPipe and qMDD
5. Setup Palmers software, Sparky and Analysis
6. Setup mMass
7. Clone and scons build relax
8. Build the main Dockerfile, with relax updated last

```bash
source build_Dockerfile.sh
```
Restart docker on a mac, if it "hangs".

```bash
killall Docker
drdocker
```

Delete container and images. This will destroy all your images and containers. <br>
**It will not be possible to restore them!**

```bash
# Delete all containers:
docker ps
docker rm $(docker ps -a -q)
docker rm --force $(docker ps -a -q)

# Delete all dangling images
docker images -f dangling=true
docker rmi $(docker images -qf dangling=true)

# Delete all images
docker images
docker rmi $(docker images -q)
```
## References
* <http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker> <br>
* <https://blog.jessfraz.com/post/docker-containers-on-the-desktop>

# Examples of use <a name="examples"></a>
## nmrglue in JupyterLab <a name="nmrglue_ex"></a>

First go to a folder, on your computer, where you can download nmrglue example files.

* [nmrglue examples](https://github.com/jjhelmus/nmrglue/tree/master/examples)
* [nmrglue archive](https://code.google.com/archive/p/nmrglue/downloads)


### separate/separate_2d_bruker
[This example](https://github.com/jjhelmus/nmrglue/tree/master/examples/separate/separate_2d_bruker) contains a Python script separate.py which separates 2D spectra from an array of 2D data in a Bruker data set.

We can use curl and unzip from the container already. 

```bash
# First make a directory where to download example files
mkdir -p $HOME/Downloads/nmrglue_ex
cd $HOME/Downloads/nmrglue_ex

dr curl -O https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/nmrglue/example_separate_2d_bruker.zip
dr unzip example_separate_2d_bruker.zip
```

Then start a Jupyter. The **dr** alias [is explained here.](#runmac)

```bash
# Start Docker Relax Labbook
dr
```
Then visit in our browser: [http://0.0.0.0:8888](http://0.0.0.0:8888).

Create a new Python 3 notebook. Paste this is into cells, and execute 
with shift+enter

```python
import nmrglue as ng

# read in the NMR data
dic, data = ng.bruker.read('separate_2d_bruker/arrayed_data.dir', shape=(7360, 640), cplex=True)
```

```python
# Write it out
array_size = 23
for i in range(array_size):
    dir_name = "separate_2d_bruker/%02d"%(i)
    print("Creating directory:", dir_name)
    ng.bruker.write(dir_name, dic, data[i::array_size], overwrite=True)

# list files
%ls separate_2d_bruker/00
```
Voila!

### jbnmr_examples/s4_2d_plotting
[This example](https://github.com/jjhelmus/nmrglue/tree/master/examples/jbnmr_examples/s4_2d_plotting) is taken from Listing S4 from the 2013 JBNMR nmrglue paper. In this example a 2D SSNMR spectrum is visualized using the script plot_2d_pipe_spectrum.py

```bash
mkdir -p $HOME/Downloads/nmrglue_ex
cd $HOME/Downloads/nmrglue_ex

curl -O https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/nmrglue/jbnmr_s4_2d_plotting.zip
unzip jbnmr_s4_2d_plotting.zip
```
Then start a JupyterLab. The **dr** alias [is explained here.](#runmac)

```bash
# Start Docker Relax Labbook
dr
```
Then visit in our browser: [http://0.0.0.0:8888](http://0.0.0.0:8888).

Create a new Python 3 notebook. Paste this is into cells, and execute 
with shift+enter


Jupyter **magic commands** starts with **%**. [See more here](http://ipython.readthedocs.io/en/stable/interactive/magics.html) and [here.](https://www.dataquest.io/blog/jupyter-notebook-tips-tricks-shortcuts/)

```bash
# This will list all magic commands
%lsmagic
# List the folder content
%ls s4_2d_plotting

import nmrglue as ng
import matplotlib.pyplot as plt
%matplotlib inline

# read in data
dic, data = ng.pipe.read("s4_2d_plotting/test.ft2")

# find PPM limits along each axis
uc_15n = ng.pipe.make_uc(dic, data, 0)
uc_13c = ng.pipe.make_uc(dic, data, 1)
x0, x1 = uc_13c.ppm_limits()
y0, y1 = uc_15n.ppm_limits()

# plot the spectrum
fig = plt.figure(figsize=(10, 10))
fig = plt.figure()
ax = fig.add_subplot(111)
cl = [8.5e4 * 1.30 ** x for x in range(20)]
ax.contour(data, cl, colors='blue', extent=(x0, x1, y0, y1), linewidths=0.5)

# add 1D slices
x = uc_13c.ppm_scale()
s1 = data[uc_15n("105.52ppm"), :]
s2 = data[uc_15n("115.85ppm"), :]
s3 = data[uc_15n("130.07ppm"), :]
ax.plot(x, -s1 / 8e4 + 105.52, 'k-')
ax.plot(x, -s2 / 8e4 + 115.85, 'k-')
ax.plot(x, -s3 / 8e4 + 130.07, 'k-')

# label the axis and save
ax.set_xlabel("13C ppm", size=20)
ax.set_xlim(183.5, 167.5)
ax.set_ylabel("15N ppm", size=20)
ax.set_ylim(139.5, 95.5)
fig.savefig("spectrum_2d.png")
```
![Result](https://raw.githubusercontent.com/jjhelmus/nmrglue/master/examples/jbnmr_examples/s4_2d_plotting/spectrum_2d.png)

### relaxation analysis <a name="relaxationanalysis"></a>

[This example](https://github.com/jjhelmus/nmrglue/tree/master/examples/jbnmr_examples/s12-s15_relaxation_analysis) is taken from Listing S12 - S15 in the 2013 JBNMR nmrglue paper. In this example a series of 3D NMRPipe files containing relaxation trajectories for a solid state NMR experiment and analyzed.

**The code has here been refactored to a complete analysis in JupyterLab.**

```bash
mkdir -p $HOME/Downloads/nmrglue_ex
cd $HOME/Downloads/nmrglue_ex

# Copy the notebook
curl -O https://raw.githubusercontent.com/tlinnet/docker_relax/master/JupyterLab/relaxation_analysis.ipynb
```

Then start a Jupyter. The **dr** alias [is explained here.](#runmac)

```bash
# Start Docker Relax Labbook
dr
```
Then visit in our browser: [http://0.0.0.0:8888](http://0.0.0.0:8888).

Open notebook. Go throug cells and execute with shift+enter.

[Please see the notebook online here for reference, and follow it.](https://github.com/tlinnet/docker_relax/blob/master/JupyterLab/relaxation_analysis.ipynb)

**Everything is handled in Jupyter**

* The data is downloaded with bash command curl
* The data is unpacked with bash command unzip
* The data is arranged into folders
* The data is analysed with [NMRPipe](https://spin.niddk.nih.gov/NMRPipe/ref/scripts/) script [peakHN.tcl](https://spin.niddk.nih.gov/NMRPipe/ref/scripts/peakhn_tcl.html)
* The spectrum is plotted in matplotlib in Jupyter
* An analysis script for **relax** is written and executed
* All data is analysed in **relax**
