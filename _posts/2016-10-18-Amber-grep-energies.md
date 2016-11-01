---
title: "How to extract energies from Amber output"
layout: single
comments: ture 
tags: Amber MD 
categories: programming 
---
### Extract energies from the original Amber output file  

The original Amber output file original\_traj.out (using **imin=0**) has the following formatted energy information:

```
 NSTEP =        1   TIME(PS) =    6284.501  TEMP(K) =     0.01  PRESS =     0.0
 Etot   =     67386.5680  EKtot   =         3.0855  EPtot      =     67383.4824
 BOND   =      9615.1469  ANGLE   =     56210.8071  DIHED      =     54263.5755
 1-4 NB =      2015.7884  1-4 EEL =    -15328.5333  VDWAALS    =    -49773.5105
 EELEC  =     10380.2083  EHBOND  =         0.0000  RESTRAINT  =         0.0000
 Ewald error estimate:   0.3266E-03
```
To retrieve total potential energy EPtot, use the following Linux command  
 
```bash
$ grep "EPtot" original_traj.out | awk '{print $9}' > output.dat
```

### Extract energies from the recalculated Amber output file 

The recalculated output file with input of _-y mdcrd_ (using **imin=5**) recalculated_traj.dat that has the following formatted energy information:

```
   NSTEP       ENERGY          RMS            GMAX         NAME    NUMBER
      1       4.4956E+04     1.2906E+01     8.5241E+01     C3       1202

 BOND    =     5845.2592  ANGLE   =    39723.5975  DIHED      =    54838.0804
 VDWAALS =   -51803.8662  EEL     =    10218.8723  HBOND      =        0.0000
 1-4 VDW =     1337.8408  1-4 EEL =   -15203.4226  RESTRAINT  =        0.0000
minimization completed, ENE= 0.44956362E+05 RMS= 0.129056E+02
minimizing coord set #    33

  Maximum number of minimization cycles reached.
```
However, total potential energy is not precise (4.4956E+04), instead one can use the following awk script called *get-energies.awk* to grep and sum up all the contributions to the total potential energy by   

```bash
$ awk -f get-energies.awk recalculated_traj.dat > output.dat
```

The awk script *get-energies.awk* is as follows:  

```bash
BEGIN{
    s = 0
}
{
    if ($0 ~ /^ BOND/) {
        s += $3 + $6 + $9
    }
    if ($0 ~ /^ VDWAALS/) {
        s += $3 + $6 + $9
    }
    if ($0 ~ /^ 1-4 VDW/) {
        s += $4 + $8 + $11
        printf "%f \n", s
        s = 0 
    }
}
```
