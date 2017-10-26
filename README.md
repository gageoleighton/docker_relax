# Docker for NMR software
Docker image for building NMR software on Ubuntu

**Includes builded software for:**

* [relax](http://www.nmr-relax.com/) with [OpenDX](http://wiki.nmr-relax.com/OpenDX) -> [Jump to commands](#relax)
* [NMRPipe](https://www.ibbr.umd.edu/nmrpipe/install.html) -> [Jump to commands](#NMRPipe)
* [MddNMR](http://mddnmr.spektrino.com/download) -> [Jump to commands](#MddNMR)
* [nmrglue](https://www.nmrglue.com/) -> [Jump to commands](#nmrglue)
* Art Palmers software: [ModelFree4](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/modelfree), [FastModelFree](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/modelfree), [Quadric](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/quadric-diffusion), [PDBinertia](http://comdnmr.nysbc.org/comd-nmr-dissem/comd-nmr-software/software/pdbinertia) -> [Jump to commands](#Palmer)
* [Sparky](http://www.cgl.ucsf.edu/home/sparky) -> [Jump to commands](#Sparky)
* [CcpNmr Analysis 2.4](http://www.ccpn.ac.uk/v2-software/downloads)  -> [Jump to commands](#Analysis)
* [Pymol](https://pymolwiki.org/index.php/Main_Page)

For deleting images, go to -> [Developer section](#Developer)

# Get prebuild image:
```bash
docker pull tlinnet/docker_relax
docker images
```

## Running on a mac

[Link to run reference:](https://docs.docker.com/v1.11/engine/reference/commandline/run)

```bash
# First make sure XQuartz is running
open -a XQuartz
# In XQuartz -> Preferences > Security, make sure the tick 
# "Allow connections from network clients" is ON.

# Then set DISPLAY options. Verify if to use en1 or en0:
xhost + `ipconfig getifaddr en1`

# Then make alias and run. Set 'en1' to either en1 or en0, depending which returns IP.
alias dr='docker run -ti --rm -e DISPLAY=$(ipconfig getifaddr en1):0 -v /tmp/.X11-unix:/tmp/.X11-unix -v $PWD:/home/developer/work --name ubuntu_relax tlinnet/docker_relax'
```

Then try to run programs after making the **dr** alias:

```bash
# Start relax
dr relax
```

To make this easier, consider adding this to **HOME/.bash_profile**

```bash
# Start docker, if it is not running
alias drdocker='open -a /Applications/Docker.app/Contents/MacOS/Docker'
# Start  XQuartz, if it is not running
alias drx='open -a XQuartz; xhost + `ipconfig getifaddr en1`'
# Alias the docker run command
alias dr='docker run -ti --rm -e DISPLAY=$(ipconfig getifaddr en1):0 -v /tmp/.X11-unix:/tmp/.X11-unix -v $PWD:/home/developer/work --name ubuntu_relax tlinnet/docker_relax'
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
dr mpirun --version
dr mpirun -np 2 echo "hello world"
dr mpirun --report-bindings -np 2 echo "hello world"
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
dr qMDD
```
## nmrglue <a name="nmrglue"></a>
* [nmrglue](https://www.nmrglue.com/)

```bash
dr python -c "import nmrglue; print nmrglue.__version__"
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

## Pymol <a name="Pymol"></a>
* [Pymol](https://pymolwiki.org/index.php/Main_Page)

```bash
dr pymol
```


# Developer <a name="Developer"></a>

To open a bash terminal in the container, when it is running

```bash
docker exec -it ubuntu_relax bash
```

Build own image. [Link to build reference:](https://docs.docker.com/v1.11/engine/reference/commandline/build)

```bash
# -t : --tag=[]  Name and optionally a tag in the 'name:tag' format
# PATH to Dockerfile
docker build -t docker_relax .

# Run it
## On Linux
docker run -ti --rm -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v $PWD:/home/developer/work --name ubuntu_relax docker_relax

## On mac. Check to use either en0 or en1.
xhost + `ipconfig getifaddr en1`
docker run -ti --rm -e DISPLAY=$(ipconfig getifaddr en1):0 -v /tmp/.X11-unix:/tmp/.X11-unix -v $PWD:/home/developer/work --name ubuntu_relax docker_relax
```

Delete container and images. This will destroy all your images and containers. <br>
**It will not be possible to restore them!**

```bash
# Delete all containers:
docker ps
docker rm $(docker ps -a -q)

# Delete all dangling images
docker images -f dangling=true
docker rmi $(docker images -qf dangling=true)

# Delete all images
docker images
docker rmi $(docker images -q)
```


### References
* <http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker> <br>
* <https://blog.jessfraz.com/post/docker-containers-on-the-desktop>

