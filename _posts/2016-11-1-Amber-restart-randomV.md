---
title: "Amber: restart job with random initial velocities"
layout: single
comments: ture 
tags: Amber 
categories: programming 
---


Take the example of triad MD simulation on the GR state (similarly for CT1 and CT2 states). The strategy is suitable for perturbing the system by randomizing velocities or switch to another force field. At the same time we use more or less equlibrated configuration so that the system is not too far from equilibrium, thus takes shorter time to relax.

### We submit jobs by executing `qsub run.amber.pbs` scripts from directory `$HOME/triad/bent/GR/`:

Here, we start with jobindex 1000, and from NVE999 we copy a specific restart file: `$HOME/triad/bent/GR/NVE999/triad_thf_pi_065.rst_5000` 

```bash
#!/bin/bash
#PBS -S /bin/bash
#PBS -N TRIAD
#PBS -o TRIAD-GR-1000.out
#PBS -e TRIAD-GR-1000.err
#PBS -q fluxoe
#PBS -A eitan_fluxoe
#PBS -l qos=flux
#PBS -l nodes=1:ppn=12
#PBS -l walltime=10:00:00
##PBS -W depend=afterok:PREV_PBS_JOB_ID
#PBS -M xiangs@umich.edu
#PBS -m ae
#PBS -r n
#PBS -j oe
#PBS -V

TRAJ_STATE=GR
PREV_JOB_ID=999
CURR_JOB_ID=1000

$jobname="triad_$TRAJ_STATE"
echo Start on `date`
echo PBS job ID : ${PBS_JOBID}

LOCAL_DIR=$HOME/triad/bent/$TRAJ_STATE/NVE$CURR_JOB_ID

module load gcc/4.8.5 openmpi/1.10.2/gcc/4.8.5  cuda/7.5  amber/14

mkdir $LOCAL_DIR

cd $LOCAL_DIR

#cp $HOME/triad/bent/$TRAJ_STATE/NVE$PREV_JOB_ID/*.prmtop .
cp $HOME/triad/bent/$TRAJ_STATE/NVE$PREV_JOB_ID/triad_thf_pi_065.rst_5000  .
#cp $HOME/triad/bent/$TRAJ_STATE/NVE$PREV_JOB_ID/prd.in .

if [ -s "$PBS_NODEFILE" ] ; then
    echo "Running on $PBS_NODEFILE"
fi

if [ -d "$PBS_O_WORKDIR" ] ; then
    cd $PBS_O_WORKDIR
    echo "Submitted from $PBS_O_WORKDIR"
fi

### create temp dir, and copy all files to it
mkdir /scratch/eitan_fluxoe/xiangs/${PBS_JOBID}
cd /scratch/eitan_fluxoe/xiangs/${PBS_JOBID}
pwd >> $LOCAL_DIR/$jobname.PBSDIR
cp $LOCAL_DIR/* . 
cp $HOME/triad/bent/*.prmtop .
cp $HOME/triad/bent/$TRAJ_STATE/prd-randomV.in .

### execute program
mpirun -np 12 sander.MPI -O -i prd-randomV.in \
-o triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.out \
-p triad_thf_${TRAJ_STATE}.prmtop \
-c triad_thf_pi_065.rst_5000 \
-r triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.rst \
-inf triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.mdinfo 

### copy all files back to original dir
rm *.prmtop
rm *.in
cp -rf /scratch/eitan_fluxoe/xiangs/${PBS_JOBID}/* $LOCAL_DIR/

echo Job $jobname finished on `date`

### clean up all files in temp dir
cd ~ 
rm -rf /scratch/eitan_fluxoe/xiangs/${PBS_JOBID}

echo "DONE"
```
 
   
### Amber job control file `prd-randomV.in` located at `$HOME/triad/bent/GR/` is as follows, where _ntx = 1, irest = 0_ labels restart a new trajecotry by reading in R but randomizing V.


```
continue MD, read R but random V
 &cntrl
   imin = 0, 
   ntx = 1, irest = 0,
   ntb = 1, cut = 12,
   ntpr = 1, ntwx = 0, 
   ntwr = 20,
   nstlim = 5000, 
   dt = 0.001,
   ntc = 2, ntf = 2, iwrap = 1,
   ntt = 0, nscm = 1000,
   ig = -1, vlimit = 10.0,
   nmropt = 0,
   igb = 0,
   ipol = 0,
 /
 &ewald
 skinnb = 2.0
 /
&end
```




All force filed files are located at `$HOME/triad/bent/`


```bash
triad_thf_CT1.prmtop  
triad_thf_CT2.prmtop  
triad_thf_GR.prmtop  
triad_thf_PI.prmtop
```
