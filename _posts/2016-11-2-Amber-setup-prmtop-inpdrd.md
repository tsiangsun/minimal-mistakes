---
title: "Amber: LEAP to prepare force field and initial coordinate input files"
layout: single
comments: ture
tags: Amber
categories: programming
---

![Amber Flowchart](/images/amber.png)


In this note, we discuss how to use LEAP to generate force field (prmtop) and initial coordinate (inpcrd) input files for Amber MD program sander, follow the steps in below.

### (1) Single molecular structure

Draw molecule in ChemDraw or iQmol and save as .mol2 or .pdb format (Spartan can open .mol created by ChemDraw and save as .mol2 or .pdb files)


### (2) Charge methods

| charge method |    abbre. | index | charge method |    abbre. | index |
| ---------- | ------ | ------- |------ | ------- | -------|
| RESP	 |resp	 |1	 	| AM1-BCC|	 bcc	| 2 |
| CM1  	 |cm1	   |3	 	| CM2	   |cm2	 |4|
| ESP (Kollman)	 |esp 	 |5	 	 |Mulliken	 |mul	 |6|
|Gasteiger 	 |gas	| 7	 |	 Read in charge	| rc	 |8|
|Write out charge	| wc	 |9	 	| Delete Charge|	 dc	| 10|


#### Option 1: Gaussian charge method like RESP

```
$ antechamber -i xx.ini.mol2 -fi mol2 -o xx.mol2 -fo mol2 -rn xx  (refresh mol2 format for antechamber)
$ antechamber -i xx.mol2 -fi mol2 -o xx.gcrt -fo gcrt    (generate g09 input file)
$ g09  < xx.gcrt   > xx.gout   (Gaussian calculation of charges)
$ antechamber -i xx.gout -fi gout -o xx.mol2 -fo mol2 -c resp -rn xx  (assign charge resp)
```

(note that .mol2 file can be changed manually to match literature model values)

#### Option 2: semi-empirical method like bcc (AM1-BCC method)

```
$ antechamber -i xx.ini.mol2 -fi mol2 -o xx.mol2 -fo mol2 -c bcc -rn xx
```

### (3) Use _parmchk_ to convert .mol2 (only) into .frcmod format (customized force field)

```
$ parmchk -i xx.mol2 -f mol2 -o xx.frcmod -a Y
$ vmd xx.mol2  (view graphics of molecule)
```

### (4) Use LEAP to construct system - prepare .inpcrd and .prmtop files

#### A. Load molecules: xx.frcmod (the force field file) and xx.mol2 (the molecular geometry file)
copy both files to your leap folder, then
$ xleap

```
> source leaprc.gaff  (the general Amber force field, no topology)
```

other force fields options:
> source leaprc.ff02pol
/export/apps/Amber/12/dat/leap/parm/frcmod.ff02pol.r1  (polarizable force field) ff03.r1  
> source leaprc.ff03.r1
/export/apps/Amber/12/dat/leap/parm/frcmod.ff03  (the non-polarizable force field)


Then create molecule called xx, yy,... and load force field parameters for xx,yy,...

```
> xx = loadmol2 xx.mol2
> loadamberparams xx.frcmod
> yy = loadmol2 yy.mol2
> loadamberparams yy.frcmod
> list  
```  

#### B. Solvate your molecule with solvents and form a simulation box

##### Option 1:

```
> loadoff solvents.lib   (the default solvent models)
> list
CHCL3BOX   DC4       MEOHBOX   NMABOX    PL3       POL3BOX   QSPCFWBOX SPC       
SPCBOX     SPCFWBOX  SPF       SPG       T4E       TIP3PBOX  TIP3PFBOX TIP4PBOX  
TIP4PEWBOX TIP5PBOX  TP3       TP4       TP5       TPF
```

##### Option 2:
Use  solvateBox  <solute> <solvent> <half_length_computer_box in angstrom>

```
> solvateBox xx yy 15
> saveAmberParms xx xx.prmtop xx.inpcrd   
(here, xx is the solute and xx.prmtop and xx.inpcrd can be renamed afterwards)
> quit
```


Two input files for sander .prmtop and .inpcrd
.prmtop is the force field file including both parameters and topology information for the whole solute/solvent system

.inpcrd is the initial configuration file

##### Option 3: use a script that loads all the input files except for the solvation step
e.g.
$ xleap

```
> source script
```
the script file is as follows:

```
source leaprc.ff03.r1  OR source leaprc.ff02pol.r1
loadoff solvents.lib
hod = loadmol2   hod.mol2
leadAmberParams hod.frcmod
WAT = SPC
loadAmberParams frcmod.spce
list
```

then you type to finish the solvation step in xleap:

```
> solvateBox hod SPC  12.5
OR to use the default solvent box:
> solvateBox SPC MEOHBOX  12.0
> addIons CYN  K+ 9
> saveAmberParams SPC (the solute) spc.prmtop spc.inpcrd
> quit
```

$ vmd
New Molecule ...  --> load xx.prmtop using format AMBER7 Parm , then load xx.inpcrd using formate *AMBER7 Restart* to view the 3D structure of the system.






## Example of PCBM_DMA in Chrorobenzene solution preparation

Take the example of PCBM_DMA (residue name: PCB) as solute and Chlorobenzene (residue name: CBZ) as solvent.


[comment]: <> (to align center or right, use ``|:-------------:|`` or ``|------:|``)


|  Information  | PDB   |  mol2  | lib/prep | frcmod/dat | prmtop | inpcrd |
| --------------|:------:| :-----:| :------:| :-----:| :------:| :-----:|
| Residue/Unit Name     |  Y  | Y | Y  |   | Y |  |
| Atom Name             |  Y  | Y | Y  |   | Y |  |
| Atom Type             |     | Y | Y  | Y | Y |  |
| Charges               |     | Y | Y  |   | Y |  |
| Connectivity          |  D  | Y | Y  |   | Y |  |
| Coordinates           |  Y  | Y | D  |   |   |Y |
| Atom Mass             |     |   |    | Y | Y |  |
| Bonded param          |     |   |    | Y | Y |  |
| Nonbonded param       |     |   |    | Y | Y |  |

Here, Y means yes, and D means will be discarded when converted to force field files.


For standard residue in protein and nucleic acids PDB files, LEAP
