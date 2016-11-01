---
title: "How to submit consecutive Amber jobs"
layout: single
comments: ture 
tags: Amber MD script
categories: programming 
---


Take the example of triad MD simulation on the PI state. First, all force filed files and job control file are located at `$HOME/triad/bent/`

```bash
triad_thf_CT1.prmtop  
triad_thf_CT2.prmtop  
triad_thf_GR.prmtop  
triad_thf_PI.prmtop
prd.in
```

###The follwing generic `sample-run.amber.pbs` scripts are located at `$HOME/sample/`:

```bash
#!/bin/bash
#PBS -S /bin/bash
#PBS -N TRIAD
#PBS -o TRIAD-MYTRAJSTATE-MYJOBINDEX.out
#PBS -e TRIAD-MYTRAJSTATE-MYJOBINDEX.err
#PBS -q fluxoe
#PBS -A eitan_fluxoe
#PBS -l qos=flux
#PBS -l nodes=1:ppn=12
#PBS -l walltime=10:00:00
#PBS -W depend=afterok:PREV_PBS_JOB_ID
#PBS -M xiangs@umich.edu
#PBS -m ae
#PBS -r n
#PBS -j oe
#PBS -V

TRAJ_STATE=MYTRAJSTATE
PREV_JOB_ID=MYPREVJOBID
CURR_JOB_ID=MYCURRJOBID

$jobname="triad_$TRAJ_STATE"
echo Start on `date`
echo PBS job ID : ${PBS_JOBID}

LOCAL_DIR=$HOME/triad/bent/$TRAJ_STATE/NVE$CURR_JOB_ID

module load gcc/4.8.5 openmpi/1.10.2/gcc/4.8.5  cuda/7.5  amber/14

mkdir $LOCAL_DIR

cd $LOCAL_DIR

# cp $HOME/triad/bent/$TRAJ_STATE/NVE$PREV_JOB_ID/*.prmtop .
cp $HOME/triad/bent/${TRAJ_STATE}/NVE${PREV_JOB_ID}/triad_thf_${TRAJ_STATE}_${PREV_JOB_ID}.rst .
# cp $HOME/triad/bent/${TRAJ_STATE}/prd.in .

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
cp $HOME/triad/bent/$TRAJ_STATE/prd.in .

### execute program
# [1] outputing mdcrd trajectory
# mpirun -np 12 sander.MPI -O -i prd.in -o triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.out -p triad_thf_${TRAJ_STATE}.prmtop -c triad_thf_${TRAJ_STATE}_${PREV_JOB_ID}.rst -r triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.rst -x triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.mdcrd -inf triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.mdinfo 

# [2] not outputing mdcrd trajectory
mpirun -np 12 sander.MPI -O -i prd.in -o triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.out -p triad_thf_${TRAJ_STATE}.prmtop -c triad_thf_${TRAJ_STATE}_${PREV_JOB_ID}.rst -r triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.rst  -inf triad_thf_${TRAJ_STATE}_${CURR_JOB_ID}.mdinfo

### copy all files back to original dir
rm *.prmtop
rm prd.in
rm triad_thf_${TRAJ_STATE}_${PREV_JOB_ID}.rst
cp -rf /scratch/eitan_fluxoe/xiangs/${PBS_JOBID}/* $LOCAL_DIR/

echo Job $jobname finished on `date`

### clean up all files in temp dir
cd ~
rm -rf /scratch/eitan_fluxoe/xiangs/${PBS_JOBID}

echo "DONE"
```



### We submit jobs by executing `./auto-amber-serial.sh` scripts from directory `$HOME/triad/bent/PI/`:

```bash
#!/bin/bash
# This script performs consecutive Amber MD runs

jobname="TRIAD"
max=1015
min=1004
state="PI"

outfile="SUMMARY-$jobname-$state.info"

echo "Start on `date` " >> $outfile

i=$min
sed "s/MYTRAJSTATE/$state/g" $HOME/sample/sample-run.amber.pbs > $jobname-$state-$i.pbs
sed "/-W depend=afterok/d" -i $jobname-$state-$i.pbs  #delete job dependence of first job
sed "s/MYTRAJSTATE/$state/g" -i $jobname-$state-$i.pbs
sed "s/MYJOBINDEX/$i/g" -i $jobname-$state-$i.pbs
previd=$min-1
sed "s/MYPREVJOBID/$previd/g" -i $jobname-$state-$i.pbs
sed "s/MYCURRJOBID/$i/g" -i $jobname-$state-$i.pbs
chmod +x $jobname-$state-$i.pbs
JOBDEP=$(qsub ./$jobname-$state-$i.pbs) 


# submit rest of jobs with job dependence.
for (( i = $min+1 ; i <= $max ; i++ ))
do
    sed "s/MYTRAJSTATE/$state/g" $HOME/sample/sample-run.amber.pbs > $jobname-$state-$i.pbs
    sed "s/PREV_PBS_JOB_ID/$JOBDEP/g" -i $jobname-$state-$i.pbs  #job dependence
    sed "s/MYTRAJSTATE/$state/g" -i $jobname-$state-$i.pbs
    sed "s/MYJOBINDEX/$i/g" -i $jobname-$state-$i.pbs
    previd=$i-1
    sed "s/MYPREVJOBID/$previd/g" -i $jobname-$state-$i.pbs
    sed "s/MYCURRJOBID/$i/g" -i $jobname-$state-$i.pbs
    chmod +x $jobname-$state-$i.pbs
    JOBDEP=$(qsub ./$jobname-$state-$i.pbs)
done

echo "All $jobname jobs submitted on `date`." >> $outfile 2>&1
echo "All $jobname jobs submitted on `date`." 

exit

```


###Amber job control file `prd.in` located at `$HOME/triad/bent/PI/` is as follows.

```
continue MD, read R and V
 &cntrl
   imin = 0, 
   ntx = 5, irest = 1,
   ntb = 1, cut = 12,
   ntpr = 1, ntwx = 1, 
   ntwr = 1,
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