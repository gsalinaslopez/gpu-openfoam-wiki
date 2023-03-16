## Docker Engine
Install docker engine: <https://docs.docker.com/engine/>

Verify that the Docker Engine installation is successful by running the `hello-world` image:

`$ docker run hello-world`
## OpenFoam Docker

### opencfd docker images
<https://hub.docker.com/u/opencfd/>

Get all image flavours available:
```
$ docker pull opencfd/openfoam-default:2212
$ docker pull opencfd/openfoam-dev:2212
$ docker pull opencfd/openfoam-run:2212
```

### openfoam-docker script
Get the openfoam-docker script from the [packaging/containers](https://develop.openfoam.com/packaging/containers) repositories.

Instructions to execute the script: <https://develop.openfoam.com/Development/openfoam/-/wikis/precompiled/docker#running-openfoam-in-a-container>

Verify script with a dry run: 

`$ openfoam-docker -2212 -dry-run`

### Example run
`$ docker run --rm -t -i --user=1000:1000 --volume=$(pwd):/home/openfoam opencfd/openfoam-run:2212`

Within the container:

`openfoam$ blockMesh -help`

Should output something like this:
```
Usage: blockMesh [OPTIONS]                                                                
Options:                                                                                  
  -case <dir>       Case directory (instead of current directory)                         
  -dict <file>      Alternative blockMeshDict 
  -merge-points     Geometric point merging instead of topological merging
                    [default for 1912 and earlier].
  -no-clean         Do not remove polyMesh/ directory or files
  -region <name>    Specify alternative mesh region
  -sets             Write cellZones as cellSets too (for processing purposes)
  -time <time>      Specify a time to write mesh to (default: constant)
  -verbose          Force verbose output. (Can be used multiple times)
  -write-vtk        Write topology as VTU file and exit
  -doc              Display documentation in browser
  -help             Display short help and exit
  -help-compat      Display compatibility options and exit
  -help-full        Display full help and exit

Block mesh generator.

  The ordering of vertex and face labels within a block as shown below.
  For the local vertex numbering in the sequence 0 to 7:
    Faces 0, 1 (x-direction) are left, right. 
    Faces 2, 3 (y-direction) are front, back. 
    Faces 4, 5 (z-direction) are bottom, top. 

                        7 ---- 6
                 f5     |\     :\     f3
                 |      | 4 ---- 5     \
                 |      3.|....2 |      \
                 |       \|     \|      f2
                 f4       0 ---- 1
    Y  Z
     \ |                f0 ------ f1
      \|
       o--- X

Using: OpenFOAM-2212 (2212) - visit www.openfoam.com
Build: _fd8c5e00-20221221
Arch:  LSB;label=32;scalar=64
```
