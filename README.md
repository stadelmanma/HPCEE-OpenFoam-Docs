# HPCEE-OpenFoam-Docs

Overview
========

OpenFOAM is a free open source modeling tool able to handle a variety of different types of simulations. Any number of users can use it at the same time because there are no licensing restrictions. There are currently four versions of OpenFoam available on the HPCEE system:

* Version 2.2.0
* Version 3.0.0
* Version 3.0.1
* Version 4.0

All OpenFOAM simulations must be run on one of the two scratch severs. Currently available slots on the scratch server can be viewed using the `qsc` command. For full documentation on the scratch server click [here](http://127.0.0.1:10080/index.php/documentation/31-using-scratch-servers)


Running OpenFOAM
================

Being that OpenFOAM is designed to run from the commandline it is very simple to run it through the VNC viewier terminal window. The various parameters in curly braces are replaced by your job's needs. If using a different scaratch server make sure to update the line after "Change to Scratch" otherwise the job will be rejected. The `|& tee (file)` portion saves the stdout and stderr of the preceding command to a specific file as well as printing it to the screen.

OpenFoam simulations can take a very long time for large and complex problems. If you feel that your given simulation might violate [queue time constraints](http://127.0.0.1:10080/index.php/queue-policies) submitting additional "continuation" jobs and telling them to hold until the former finishes is the best way to ensure smooth progress. Generally OpenFOAM files and commands can be configured to start from the latest time available. Information on how to hold a job can be found [here](http://127.0.0.1:10080/index.php/documentation/19-using-grid-engine) at the bottom of the page.

Below is a sample run script utilizing OpenFoam version 3.0.0

```
#!/bin/csh
## Change into the current working directory
#$ -cwd
##
## The name for the job. It will be displayed this way on qstat
#$ -N {job-name}
##
## Number of cores to request
#$ -pe mpi {np}
##
#$ -r n
##
## Queue Name
#$ -q general
##
## Scratch
#$ -l scr1=1 #using server 1, scr2=1 to use server 2

##Load Modules
module load openfoam/3.0.0

##Change to Scratch
cd /nfs/scr/1/{username}

## Copying job directory in home to scratch
cp -r ~/{job-directory} .
cd {job-directory}

##Decompose mesh and run the parallel job
decomposePar |& tee decompose-par.log
mpirun  -np {np} simpleFoam -parallel |& tee solver.log

## reconstructing mesh
reconstructPar |& tee reconstruct-par.log
```

Tips/ Pitfalls
--------------

* Keep in mind queue time constraints when running simulations
* Using an interactive parallel session can be a good way to test run scripts
* Utilization of the `-case` option for OpenFoam commands may simplify job scripts

Running ParaFoam
================

ParaFoam is OpenFOAM's natively supprted viewer that utilizates it's own, or a system installation of ParaView. On the HPCEE system OpenFOAM does not have it's own version of ParaView. The full documentation on running ParaView should be read first, [here](http://127.0.0.1:10080/index.php/documentation/25-running-paraview-on-vis-nodes). You will need to set the DISPLAY variable prior to submitting this job script. It has the form of `login<X>:<Y>`, and the easiest way to determine its value is to look at the top of the VNC viewer window. 

Finally, the following job script can be used to begin an interaction session of paraFoam on a gpu node.

```
#!/bin/csh
## Change into the current working directory
#$ -cwd
#$ -r n
##
## The name for the job. It will be displayed this way on qstat
#$ -N paraFoam
##
## Pass Display
#$ -v DISPLAY
##
## Number of cores to request
#$ -pe gpu 1

##Load Modules
module load vgl paraview/5.0.1 openfoam/4.0

##Run the parallel job
export DISPLAY="$DISPLAY"
vglrun paraFoam -builtin
```

