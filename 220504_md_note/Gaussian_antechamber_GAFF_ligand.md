20220504

[几种生成有机分子GROMACS拓扑文件的工具 - 思想家公社的门口：量子化学·分子模拟·二次元 (sobereva.com)](http://sobereva.com/266)

Gaussian

```shell
# Gaussian 安装
#解压
rar x Gaussian_16_C.01_AVX_Linux_x64.part1.rar 
#解压
tar -Jxvf Gaussian 16 C.01 AVX Linux x64
#新建Gaussion文件夹
mkdir Gaussion
#设置环境变量
vim ~/.bashrc 
export g16root=/home/tmu/software/Downloads/Gaussion
export GAUSS_EXEDIR=$g16root/g16
export GAUSS_SCRDIR=$g16root/src
source ~/.bashrc
#安装
cd /home/tmu/software/Downloads/Gaussion/g16
./bsd/install 
```

Gauss view 6

```shell
安装在windows电脑中了，因为买的破解版没有gauss view 6 linux版本的。
Gauss view 6和Gaussian配套使用
```

利用Gaussian和Gauss view6 去对小分子进行优化

```shell
#使用gaussian优化的小分子最好是自己画的，不要用从PDB数据库中下载得到的，可以用chemdraw画出来，用chem3D生成gjf格式文件再导入Gview中，需要编辑好gjf文件。
%nprocshared=60 #本电脑可以用的线程多点儿，需要看电脑配置定
%mem=6GB
%chk=*.chk #这个名字要和后续处理的小分子名字一致。
opt freq b3lyp/6-31g scrf=(smd,solvent=water) geom=connectivity iop(6/33=2,6/42=6)
#这里的参数可以修改，一般是用DFT体系的。
#使用gaussian软件去优化准备好gjf格式的小分子
$g16root/g16/g16 /home/tmu/Public/MD_la_220429/md_pykfk382la_220430/fbp2.gjf
#linux版本gaussian生成文件为log后缀，利用antechamber转化为mol2格式文件
antechamber -i /home/tmu/Public/MD_la_220429/md_pykfk382la_220430/chem_fbp4.log -fi gout -o /home/tmu/Public/MD_la_220429/chem_fbp.mol2 -fo mol2 -c resp -rn fbp

#antechamber -i /home/tmu/Public/MD_la_220429/sele.mol2 -fi mol2 -o /home/tmu/Public/MD_la_220429/fbp.mol2 -fo mol2  -rn fbp
#parmchk2  mol2转换为frcmod格式
parmchk2 -i fbp2.mol2 -f mol2 -o fbp2.frcmod

#利用tleap，生成GAFF力场的文件：.prmtop rst7格式
tleap 
-I: Adding /home/tmu/software/Miniconda3/envs/AmberTools21/dat/leap/prep to search path.
-I: Adding /home/tmu/software/Miniconda3/envs/AmberTools21/dat/leap/lib to search path.
-I: Adding /home/tmu/software/Miniconda3/envs/AmberTools21/dat/leap/parm to search path.
-I: Adding /home/tmu/software/Miniconda3/envs/AmberTools21/dat/leap/cmd to search path.

Welcome to LEaP!
(no leaprc in search path)
> source leaprc.gaff
----- Source: /home/tmu/software/Miniconda3/envs/AmberTools21/dat/leap/cmd/leaprc.gaff
----- Source of /home/tmu/software/Miniconda3/envs/AmberTools21/dat/leap/cmd/leaprc.gaff done
Log file: ./leap.log
Loading parameters: /home/tmu/software/Miniconda3/envs/AmberTools21/dat/leap/parm/gaff.dat
Reading title:
AMBER General Force Field for organic molecules (Version 1.81, May 2017)
> fbp=loadmol2 fbp2.mol2
Loading Mol2 file: ./fbp2.mol2
Reading MOLECULE named fbp
> loadamberparams fbp2.frcmod
Loading parameters: ./fbp2.frcmod
Reading force field modification type file (frcmod)
Reading title:
Remark line goes here
> saveamberparm fbp fbp.prmtop fbp.rst7
Checking Unit.
Building topology.
Building atom parameters.
Building bond parameters.
Building angle parameters.
Building proper torsion parameters.
Building improper torsion parameters.
 total 0 improper torsions applied
Building H-Bond parameters.
Incorporating Non-Bonded adjustments.
Not Marking per-residue atom chain types.
Marking per-residue atom chain types.
  (Residues lacking connect0/connect1 - 
   these don't have chain types marked:

	res	total affected

	fbp	1
  )
 (no restraints)
> quit
	Quit

Exiting LEaP: Errors = 0; Warnings = 0; Notes = 0.

#acpypea生成  *.amb2gmx格式的一个文件
acpype -p fbp.prmtop -x fbp.rst7 
==========================================================================
| ACPYPE: AnteChamber PYthon Parser interfacE v. 2022.1.3 (c) 2022 AWSdS |
==========================================================================
Converting Amber input files to Gromacs ...
WARNING: residue label 'fbp' in 'fbp.prmtop' is not all UPPERCASE
WARNING: this may raise problem with some applications like CNS
==> Writing GROMACS files

==> Disambiguating lower and uppercase atomtypes in GMX top file, even if identical.

==> Writing GMX dihedrals for GMX 4.5 and higher.

Total time of execution: less than a second
#文件中包括 .log;.mdp;.gro;.top;.itp;.sh文件
```



GAFF力场是对应Amber软件处理大分子蛋白的，因此不能使用GROMACS软件中的gromos,OPLS-AA力场等等，以后要注意。这次最后虽然没有用到高斯处理的小分子，但是学习了这个软件的用法。
