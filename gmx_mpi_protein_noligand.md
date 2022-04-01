

```shell
 2015  cd MD20211117_eno/
 2016  ls
 2017  gmx_mpi rms -s md_0_10.tpr -f md_0_10_center.xtc  -o rms_enopro.xvg -tu ns
 2018  cd ..
 2019  cd MD_20220321_enok343noligand/
 2020  ls
 2021  gmx_mpi pdb2gmx -f enok343hib_finall.pdb -o enok343hib_finall.gro -water spce
 2022  gmx_mpi editconf -f enok343hib_finall.gro -o emok343hib_newbox.gro -c -d 1.0 -bt cubic
 2023  gmx_mpi solvate -cp enok343hib_finall.gro -cs spc216.gro -o enok343hib_solv.gro -p topol.top 
 2024  gmx grompp -f ions.mdp -c enok343hib_solv.gro  -p topol.top -o ions.tpr
 2025  gmx_mpi grompp -f ions.mdp -c enok343hib_solv.gro  -p topol.top -o ions.tpr
 2026  gmx_mpi grompp -f ions.mdp -c enok343hib_solv.gro  -p topol.top -o ions.tpr -maxwarn 1
 2027  gmx_mpi genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -np 2
 2028  gmx grompp -f minim.mdp -c solv_ions.gro -p topol.top -o em.tpr
 2029  gmx_mpi grompp -f minim.mdp -c solv_ions.gro -p topol.top -o em.tpr
 2030  gmx_mpi grompp -f minim.mdp -c solv_ions.gro -p topol.top -o em.tpr  -maxwarn 1
 2031  gmx_mpi  mdrun -v -deffnm em
 2032  gmx_mpi grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr
 2033  gmx_mpi grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr -maxwarn 1
 2034  gmx_mpi mdrun -deffnm nvt
 2035  gmx_mpi mdrun -v -deffnm nvt
 2036  gmx_mpi grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -o npt.tpr
 2037  gmx_mpi grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -o npt.tpr -maxwarn 1
 2038  gmx mdrun -deffnm npt
 2039  gmx_mpi  mdrun -v -deffnm npt
 2040  gmx_mpi grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -n index.ndx -o md_0_10.tpr -maxwarn 1
 2041  gmx_mpi grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -n index.ndx -o md_0_10.tpr -maxwarn 1 -maxwarn 1
 2042  gmx_mpi grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr
 2043  gmx_mpi grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr -maxwarn 1
 2044  gmx_mpi mdrun -deffnm md_0_1
 2045  gmx_mpi mdrun -v -deffnm md_0_1
 2046  ls
 2047  gmx_mpi rms -s md_0_1.tpr -f md_0_10_center.xtc -o rmsd_enok343hib.xvg -tu ns
```

