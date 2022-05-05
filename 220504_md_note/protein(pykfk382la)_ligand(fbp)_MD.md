20220504

```css
source /home/tmu/software/Downloads/gromacs-2019.6/bin/GMXRC
```

这次做的是一个新的修饰的分子动力学模拟, 需要用到新的大分子力场，itp，top，gro文件。

由于GROMCS重新编译了，要重新添加54a8力场，命令在启用的时候gmx_mpi

步骤参考Justin的Gromacs Tutorial:  protein-ligand

```shell
#已经有了蛋白质的itp;top;gro文件
gmx_mpi editconf -f fbp.pdb -o fbp.gro
合并gro文件pykfk382las_fbp.gro
拓扑文件添加信息
; Include ligand topology
#include "fbp.itp"
fbp                1

#从这里就需要把小分子的力场加到拓扑文件中了。
gmx_mpi editconf -f pykfk382las_fbp.gro -o newbox.gro -bt cubic -d 1.0
gmx_mpi solvate -cp newbox.gro -cs spc216.gro -p topol.top -o solv.gro
vmd solv.gro
gmx_mpi grompp -f ions.mdp -c solv.gro -p topol.top -o ions.tpr -maxwarn 1
gmx_mpi genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -np 10
>14 SOL
gmx_mpi grompp -f em.mdp -c solv_ions.gro -p topol.top -o em.tpr -maxwarn 2

#限制蛋白质
posre_Protein_chain_A.itp
posre_Protein_chain_A2.itp

#能量最小化
gmx_mpi mdrun -v -deffnm em
gmx_mpi energy -f em.edr -o em10.xvg

#限制配体
gmx_mpi make_ndx -f fbp.gro  -o index_lig.ndx
...
 > 0 & ! a H*
 > q
gmx_mpi genrestr -f fbp.gro -n index_lig.ndx -o posre_fbp.itp -fc 1000 1000 1000
 >3  #选3
 修改top文件
 
 ; Ligand position restraints
#ifdef POSRES_LIG
#include "posre_fbp.itp"
#endif
 
 
 gmx_mpi make_ndx -f em.gro -o index.ndx
  0 System              : 71563 atoms
  1 Protein             :  4208 atoms
  2 Protein-H           :  3298 atoms
  3 C-alpha             :   432 atoms
  4 Backbone            :  1296 atoms
  5 MainChain           :  1729 atoms
  6 MainChain+Cb        :  2123 atoms
  7 MainChain+H         :  2147 atoms
  8 SideChain           :  2061 atoms
  9 SideChain-H         :  1569 atoms
 10 Prot-Masses         :  4208 atoms
 11 non-Protein         : 67355 atoms
 12 Other               : 67355 atoms
 13 5YHW                :    16 atoms
 14 SOL                 : 67338 atoms
 15 NA                  :     1 atoms

> 1 | 13
> q

#nvt;npt;md
gmx_mpi grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -n index.ndx -o nvt.tpr -maxwarn 2
gmx_mpi mdrun -v -deffnm nvt

gmx_mpi grompp -f npt.mdp -c nvt.gro -t nvt.cpt -r nvt.gro -p topol.top -n index.ndx -o npt.tpr -maxwarn 2
gmx_mpi mdrun -v -deffnm npt

gmx_mpi grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top  -o md_0_10.tpr -maxwarn 2
gmx_mpi mdrun -v -deffnm md_0_10
```

这里很重要的就是在topol.top中的include的itp文件，需要按顺序。

forcefield parameters需要设置为小分子所用力场。

前期小分子力场，Amber对应GAFF；Charmm对应CGenFF； GROMOS的对应gromos (ATB网站生成的) 。GAFF需要用到高斯。

chain topologies 是之前做好的蛋白质 itp 文件，由于之前的蛋白断裂，itp分为了两个。

Position restraint file 需要自己设置

water topology；ions要根据小分子的力场来换。

```shell
;
;	File 'topol.top' was generated
;	By user: dpetrov (67742)
;	On host: drazen-desktop
;	At date: Fri Apr 22 17:03:17 2022
;
;	This is a standalone topology file
;
;	Created by:
;	                     :-) GROMACS - gmx pdb2gmx, 2020 (-:
;	
;	Executable:   /pool/dpetrov/GROMACS/gromacs-2020/build_my_comp/build/bin/gmx
;	Data prefix:  /pool/dpetrov/GROMACS/gromacs-2020/build_my_comp/build
;	Working dir:  /home/dpetrov/poso/PTM/additions/K_2hyd_but/lactyl/R_version
;	Command line:
;	  gmx pdb2gmx -f mod_fixed_fixed_pykf.pdb -ignh -ter
;	Force field was read from current directory or a relative path - path added.
;

; Include forcefield parameters
#include "./gromos54a7_atb.ff/forcefield.itp"

; Include chain topologies
#include "topol_Protein_chain_A.itp"
#include "topol_Protein_chain_A2.itp"

; Include Position restraint file
#ifdef POSRES
#include "posre_Protein_chain_A.itp"
#include "posre_Protein_chain_A2.itp"
#endif

; Include ligand topology
#include "fbp_ua.itp"

 ; Ligand position restraints
#ifdef POSRES_LIG
#include "posre_fbp.itp"
#endif

; Include water topology
#include "./gromos54a7_atb.ff/spc.itp"

#ifdef POSRES_WATER
; Position restraint for each water oxygen
[ position_restraints ]
;  i funct       fcx        fcy        fcz
   1    1       1000       1000       1000
#endif

; Include topology for ions
#include "./gromos54a7_atb.ff/ions.itp"

[ system ]
; Name
Protein was modified by http://coil.msp.univie.ac.at/ in water

[ molecules ]
; Compound        #mols
Protein_chain_A     1
Protein_chain_A2     1
fbp                 1
SOL         116477
NA               10

```



