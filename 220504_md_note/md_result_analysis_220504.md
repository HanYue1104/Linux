





```shell
#聚类
cutoff的值设置的太小了，分出的簇太多了。
cd clust/
gmx_mpi cluster -f ../md_0_10_center.xtc -s ../md_0_10.tpr -cutoff 0.25 -b 500 -wcl 10 -method gromos -sz 
```



```shell
#氢键  -num参数
可以跑一个聚类的直方图和一个以时间为横轴的图
gmx_mpi hbond -f md_0_10_center.xtc -s md_0_10.tpr -n distance.ndx  -num protein_lig_h_enok343hib.xvg 
```





```shell
#MD
氢键的数量（蛋白质和配体的）：NHBs
蛋白质结合位点质心与配体的距离 distance
配体的均方根偏差：RMSD   骨架（backbone）的配体RMSD
IRMSD


#结果分析第一步：需要进行周期性的限制
 vmd md_0_10.gro md_0_10.xtc
 gmx_mpi trjconv -s md_0_10.tpr -f md_0_10.xtc -o md_0_10_center.xtc -center -pbc mol -ur compact
 Choose "Protein" for centering and "System" for output.
 vmd md_0_10.gro md_0_10_center.xtc
 gmx_mpi trjconv -s md_0_10.tpr -f md_0_10_center.xtc -o start_eno.pdb -dump 0
 gmx_mpi trjconv -s md_0_10.tpr -f md_0_10_center.xtc -o last_eno.pdb -dump 100000
 #100000代表100ns
 Select group for output  "System"
 gmx_mpi trjconv -s md_0_10.tpr -f md_0_10_center.xtc -o md_0_10_fit.xtc -fit rot+trans
 Choose "Backbone" to perform least-squares fitting to the protein backbone, and "System" for output. 

 
 #.xtc轨迹文件就相当于trr
 
```





```shell
gmx_mpi make_ndx -f em.gro -o rmsdindex.ndx
gmx_mpi rms -s md_0_10.tpr -f md_0_10_center.xtc -o rmsd_pro_ligeno.xvg -n rmsdindex.ndx  -tu ns
#rmsd
复合物的结构稳定性变化
可以做一下配体的RMSD
gmx_mpi rms -s md_0_10.tpr -f md_0_10_center.xtc -o rmsd_eno.xvg -tu ns
xmgrace rmsd_eno.xvg
 gmx_mpi rms -s md_0_10.tpr -f md_0_10_center.xtc -o rmsd1_eno.xvg -tu ns
ligand  先选backbone再选ligand     protein  先选backbone 再选backbone
#rmsf
gmx_mpi make_ndx -f em.gro -o rmsfindex.ndx 
gmx_mpi rmsf -s md_0_10.tpr -f md_0_10_center.xtc -o rmsfeno.xvg -n rmsfindex.ndx -res -b 0 -e 100000 

gmx rmsf -s md_0_10.tpr -f md_0_10_center.xtc -o ./rmsf/rmsf_eno1.xvg  -res -b 0 -e 100000
xmgrace rmsf/rmsf_eno1.xvg 
#Rg


#dssp


#表面静电势

#氢键
gmx_mpi make_ndx -f em.gro -o lig.ndx
gmx_mpi  hbond -s md_0_10.tpr -f md_0_10_center.xtc -num hbond_eno1.xvg -n lig.ndx
xmgrace hbond_eno1.xvg 
```









```shell
#protein 342---lig distance
gmx_mpi make_ndx -f em.gro -o distance.ndx

gmx_mpi distance -s md_0_10.tpr -f md_0_10_center.xtc -select 18  -oall dist342_ligenokhib1.xvg -oh disthistenokhib1.xvg -n distance.ndx  -tu ns -len 2 -binw 0.01
#eno
5YHW_&_O4_Protein_&_r_342_&_HZ2:
  Number of samples:  5001
  Average distance:   0.89999  nm
  Standard deviation: 0.17894  nm
 
 
 > 13 & a O4

Copied index group 13 '5YHW'
Found 1 atoms with name O4
Merged two groups with AND: 16 1 -> 1

 16 5YHW_&_O4           :     1 atoms

> 1 & r 342 & a HZ2

Copied index group 1 'Protein'
Merged two groups with AND: 4208 13 -> 13
Found 40 atoms with name HZ2
Merged two groups with AND: 13 40 -> 1

 17 Protein_&_r_342_&_HZ2:     1 atoms

> 16 | 17

Copied index group 16 '5YHW_&_O4'
Copied index group 17 'Protein_&_r_342_&_HZ2'
Merged two groups with OR: 1 1 -> 2

 18 5YHW_&_O4_Protein_&_r_342_&_HZ2:     2 atoms

> q
#distance 167-343

> 1 & r 342 & a HZ2

Copied index group 1 'Protein'
Merged two groups with AND: 4208 13 -> 13
Found 40 atoms with name HZ2
Merged two groups with AND: 13 40 -> 1

 16 Protein_&_r_342_&_HZ2:     1 atoms

> 1 & r 166 & a OE2

Copied index group 1 'Protein'
Merged two groups with AND: 4208 10 -> 10
Found 29 atoms with name OE2
Merged two groups with AND: 10 29 -> 1

 17 Protein_&_r_166_&_OE2:     1 atoms

> 16 | 17

Copied index group 16 'Protein_&_r_342_&_HZ2'
Copied index group 17 'Protein_&_r_166_&_OE2'
Merged two groups with OR: 1 1 -> 2

 18 Protein_&_r_342_&_HZ2_Protein_&_r_166_&_OE2:     2 atoms

> q
#370-342
gmx_mpi distance -s md_0_10.tpr -f md_0_10_center.xtc -select 18 -oall ./distance/dist370_342.xvg -oh ./distance/disteno370_342.xvg -n ./distance/distance370_342eno.ndx  -tu ns -len 2 -binw 0.01

> 1 & r 342 & a HZ2

Copied index group 1 'Protein'
Merged two groups with AND: 4208 13 -> 13
Found 40 atoms with name HZ2
Merged two groups with AND: 13 40 -> 1

 16 Protein_&_r_342_&_HZ2:     1 atoms

> 1 & r 370 & a HE2

Copied index group 1 'Protein'
Merged two groups with AND: 4208 14 -> 14
Found 34 atoms with name HE2
Merged two groups with AND: 14 34 -> 1

 17 Protein_&_r_370_&_HE2:     1 atoms

> 16 | 17

Copied index group 16 'Protein_&_r_342_&_HZ2'
Copied index group 17 'Protein_&_r_370_&_HE2'
Merged two groups with OR: 1 1 -> 2

 18 Protein_&_r_342_&_HZ2_Protein_&_r_370_&_HE2:     2 atoms

> q
#Arg371-k342
> 1 & r 342 & a HZ2

Copied index group 1 'Protein'
Merged two groups with AND: 4208 13 -> 13
Found 40 atoms with name HZ2
Merged two groups with AND: 13 40 -> 1

 16 Protein_&_r_342_&_HZ2:     1 atoms

> 1 & r 371 & a NH2

Copied index group 1 'Protein'
Merged two groups with AND: 4208 17 -> 17
Found 0 atoms with name 2HH2

 17 Protein_&_r_371     :    17 atoms

> 16 | 17

Copied index group 16 'Protein_&_r_342_&_HZ2'
Copied index group 17 'Protein_&_r_371'
Merged two groups with OR: 1 17 -> 18

 18 Protein_&_r_342_&_HZ2_Protein_&_r_371:    18 atoms

> q








#enok343hib
5YHW_&_O4_Protein_&_r_342_&_HI1:
  Number of samples:  5001
  Average distance:   1.53552  nm
  Standard deviation: 0.11085  nm

> 13 & a O4

Copied index group 13 '5YHW'
Found 1 atoms with name O4
Merged two groups with AND: 16 1 -> 1

 16 5YHW_&_O4           :     1 atoms

> 1 & r 342 & a HI1

Copied index group 1 'Protein'
Merged two groups with AND: 4213 18 -> 18
Found 1 atoms with name HI1
Merged two groups with AND: 18 1 -> 1

 17 Protein_&_r_342_&_HI1:     1 atoms

> 16 | 17

Copied index group 16 '5YHW_&_O4'
Copied index group 17 'Protein_&_r_342_&_HI1'
Merged two groups with OR: 1 1 -> 2

 18 5YHW_&_O4_Protein_&_r_342_&_HI1:     2 atoms

> q

#distance 167-343
gmx_mpi make_ndx -f em.gro -o distance/distance166_342enokhib.ndx

gmx_mpi distance -s md_0_10.tpr -f md_0_10_center.xtc -select 18 -oall ./distance/distanceenokhib166_342.xvg -oh ./distance/distenokhib166_342.xvg -n ./distance/distance166_342enokhib.ndx  -tu ns -len 2 -binw 0.01

> 1 & r 342 & a HI1

Copied index group 1 'Protein'
Merged two groups with AND: 4213 18 -> 18
Found 1 atoms with name HI1
Merged two groups with AND: 18 1 -> 1

 16 Protein_&_r_342_&_HI1:     1 atoms

> 1 & r 166 & a OE2

Copied index group 1 'Protein'
Merged two groups with AND: 4213 10 -> 10
Found 29 atoms with name OE2
Merged two groups with AND: 10 29 -> 1

 17 Protein_&_r_166_&_OE2:     1 atoms

> 16 | 17

Copied index group 16 'Protein_&_r_342_&_HI1'
Copied index group 17 'Protein_&_r_166_&_OE2'
Merged two groups with OR: 1 1 -> 2

 18 Protein_&_r_342_&_HI1_Protein_&_r_166_&_OE2:     2 atoms

> Q


Syntax error: "Q"

> q


GROMACS reminds you: "Shake Yourself" (YES)
 #His370-khib343

> 1 & r 342 & a HI1

Copied index group 1 'Protein'
Merged two groups with AND: 4213 18 -> 18
Found 1 atoms with name HI1
Merged two groups with AND: 18 1 -> 1

 16 Protein_&_r_342_&_HI1:     1 atoms

> 1 & r 370 & a HE2

Copied index group 1 'Protein'
Merged two groups with AND: 4213 14 -> 14
Found 34 atoms with name HE2
Merged two groups with AND: 14 34 -> 1

 17 Protein_&_r_370_&_HE2:     1 atoms

> 16 | 17

Copied index group 16 'Protein_&_r_342_&_HI1'
Copied index group 17 'Protein_&_r_370_&_HE2'
Merged two groups with OR: 1 1 -> 2

 18 Protein_&_r_342_&_HI1_Protein_&_r_370_&_HE2:     2 atoms

> q

#Arg371-khib342
gmx_mpi distance -s md_0_10.tpr -f md_0_10_center.xtc -select 18 -oall ./distance/distanceenokhib371_342.xvg -oh ./distance/distenokhib371_342.xvg -n ./distance/distance371_342enokhib.ndx  -tu ns -len 2 -binw 0.01

> 1 & r 342 & a HI1

Copied index group 1 'Protein'
Merged two groups with AND: 4213 18 -> 18
Found 1 atoms with name HI1
Merged two groups with AND: 18 1 -> 1

 16 Protein_&_r_342_&_HI1:     1 atoms

> 1 & r 371 & a NH2

Copied index group 1 'Protein'
Merged two groups with AND: 4213 17 -> 17
Found 17 atoms with name NH2
Merged two groups with AND: 17 17 -> 1

 17 Protein_&_r_371_&_NH2:     1 atoms

> 16 | 17

Copied index group 16 'Protein_&_r_342_&_HI1'
Copied index group 17 'Protein_&_r_371_&_NH2'
Merged two groups with OR: 1 1 -> 2

 18 Protein_&_r_342_&_HI1_Protein_&_r_371_&_NH2:     2 atoms

> q

```





```shell
#MM/PBSA结合能
gmx_MMPBSA -O -i mmpbsa.in -cs md_0_10.tpr -ci index.ndx -cg 1 13 -ct md_0_10_center.xtc -cp topol.top -lm 2pg_2_23.mol2 

antechamber -i 2pg_2_23.pdb -fi pdb -o 2pg_2_23.mol2 -fo mol2 -c bcc -s 2

[INFO   ] Starting
usage: gmx_MMPBSA [-h] [-v] [--input-file-help] [-O] [-prefix <file prefix>]
                  [-i FILE] [-xvvfile XVVFILE] [-o FILE] [-do FILE] [-eo FILE]
                  [-deo FILE] [-nogui] [-s] [-cs <Structure File>]
                  [-ci <Index File>] [-cg index index] [-ct [TRJ ...]]
                  [-cp <Topology>] [-cr <PDB File>] [-rs <Structure File>]
                  [-ri <Index File>] [-rg index] [-rt [TRJ ...]]
                  [-rp <Topology>] [-lm <Structure File>]
                  [-ls <Structure File>] [-li <Index File>] [-lg index]
                  [-lt [TRJ ...]] [-lp <Topology>] [-make-mdins] [-use-mdins]
                  [-rewrite-output] [--clean]

gmx_MMPBSA is a new tool based on AMBER's MMPBSA.py aiming to perform end-
state free energy calculations with GROMACS files. This program is an
adaptation of Amber's MMPBSA.py and essentially works as such. As gmx_MMPBSA
adapts MMPBSA.py, since it has all the resources of this script and work with
any GROMACS version. This program will calculate binding free energies using
end-state free energy methods on an ensemble of snapshots using a variety of
implicit solvent models.This is the core of gmx_MMPBSA and it will do all the
calculations

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  --input-file-help     Print all available options in the input file.
                        (default: False)

Miscellaneous Options:
  -O, --overwrite       Allow output files to be overwritten (default: False)
  -prefix <file prefix>
                        Prefix for intermediate files. (default: _GMXMMPBSA_)

Input and Output Files:
  These options specify the input files and optional output files.

  -i FILE               MM/PBSA input file. (default: None)
  -xvvfile XVVFILE      XVV file for 3D-RISM. (default: /home/tmu/software/Min
                        iconda3/envs/AmberTools21/dat/mmpbsa/spc.xvv)
  -o FILE               Output file with MM/PBSA statistics. (default:
                        FINAL_RESULTS_MMPBSA.dat)
  -do FILE              Output file for decomposition statistics summary.
                        (default: FINAL_DECOMP_MMPBSA.dat)
  -eo FILE              CSV-format output of all energy terms for every frame
                        in every calculation. File name forced to end in
                        [.csv]. This file is only written when specified on
                        the command-line. (default: None)
  -deo FILE             CSV-format output of all energy terms for each printed
                        residue in decomposition calculations. File name
                        forced to end in [.csv]. This file is only written
                        when specified on the command-line. (default: None)
  -nogui                No open gmx_MMPBSA_ana after all calculations finished
                        (default: True)
  -s, --stability       Perform stability calculation. Only the complex
                        parameters are required. If ligand is non-Protein
                        (small molecule) type, then ligand *.mol2 file is
                        required. In any other case receptor and ligand
                        parameters will be ignored. See description bellow
                        (default: False)

Complex:
  Complex files and info that are needed to perform the calculation. If the
  receptor and/or the ligand info is not defined, we generate them from that
  of the complex.

  -cs <Structure File>  Structure file of the complex. If it is Protein-Ligand
                        (small molecule) complex and -cp is not defined, make
                        sure that you define -lm option. See -lm description
                        below. Allowed formats: *.tpr (recommended), *.pdb,
                        *.gro (default: None)
  -ci <Index File>      Index file of the bound complex. (default: None)
  -cg index index       Groups of receptor and ligand in complex index file.
                        The notation is as follows: "-cg <Receptor group>
                        <Ligand group>", ie. -cg 1 13 (default: None)
  -ct [TRJ ...]         Complex trajectories. Make sure the trajectory is
                        fitted and pbc have been removed. Allowed formats:
                        *.xtc (recommended), *.trr, *.pdb (specify as many as
                        you'd like). (default: None)
  -cp <Topology>        The complex Topology file. When it is defined -lm
                        option is not needed (default: None)
  -cr <PDB File>        Complex Reference Structure file. This option is
                        optional but recommended (Use the PDB file used to
                        generate the topology in GROMACS). If not defined, the
                        chains ID assignment (if the structure used in -cs
                        does not have chain IDs) will be done automatically
                        according to the structure (can generate wrong
                        mapping). (default: None)

Receptor:
  Receptor files and info that are needed to perform the calculation. If the
  receptor info is not defined, we generate it from that of the complex.

  -rs <Structure File>  Structure file of the unbound receptor for multiple
                        trajectory approach. Allowed formats: *.tpr
                        (recommended), *.pdb, *.gro (default: None)
  -ri <Index File>      Index file of the unbound receptor. (default: None)
  -rg index             Receptor group in receptor index file. Notation: "-lg
                        <Receptor group>", e.g. -rg 1 (default: None)
  -rt [TRJ ...]         Input trajectories of the unbound receptor for
                        multiple trajectory approach. Allowed formats: *.xtc
                        (recommended), *.trr, *.pdb, *.gro (specify as many as
                        you'd like). (default: None)
  -rp <Topology>        Topology file of the receptor. (default: None)

Ligand:
  Ligand files and info that are needed to perform the calculation. If the
  ligand are not defined, we generate it from that of the complex.

  -lm <Structure File>  A *.mol2 file of the unbound ligand used to
                        parametrize ligand for GROMACS using Antechamber. Must
                        be defined if Protein-Ligand (small molecule) complex
                        was define and -cp or -lp option are not defined. No
                        needed for Proteins, DNA, RNA, Ions, Glycans or any
                        ligand parametrized in the Amber force fields. Must be
                        the Antechamber output *.mol2. (default: None)
  -ls <Structure File>  Structure file of the unbound ligand. If ligand is a
                        small molecule and -lp is not defined, make sure that
                        you define above -lm option. Allowed formats: *.tpr
                        (recommended), *.pdb, *.gro (default: None)
  -li <Index File>      Index file of the unbound ligand. Only if tpr file was
                        define in -ls. (default: None)
  -lg index             Ligand group in ligand index file. Notation: "-lg
                        <Ligand group>", e.g. -lg 13 (default: None)
  -lt [TRJ ...]         Input trajectories of the unbound ligand for multiple
                        trajectory approach. Allowed formats: *.xtc
                        (recommended), *.trr, *.pdb, *.gro (specify as many as
                        you'd like). (default: None)
  -lp <Topology>        Topology file of the ligand. (default: None)

Miscellaneous Actions:
  -make-mdins           Create the input files for each calculation and quit.
                        This allows you to modify them and re-run using -use-
                        mdins (default: False)
  -use-mdins            Use existing input files for each calculation. If they
                        do not exist with the appropriate names, gmx_MMPBSA
                        will quit in error. (default: False)
  -rewrite-output       Do not re-run any calculations, just parse the output
                        files from the previous calculation and rewrite the
                        output files. (default: False)
  --clean               Clean temporary files and quit. (default: False)

gmx_MMPBSA is an effort to implement the GB/PB and others calculations in
GROMACS. Based on MMPBSA.py (version 16.0) and AmberTools20

```



```shell
start_enokhib2和last_enokhib2(100ns)的RMS=1.964,把align好的蛋白结构保存到pdb文件中了。

```

