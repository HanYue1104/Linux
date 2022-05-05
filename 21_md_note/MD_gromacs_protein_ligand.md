## Gromacs分子动力学模拟-protein_ligand

本示例将指导新用户完成设置包含与配体复合的蛋白质（T4 溶菌酶 L99A/M102Q）的模拟系统的过程。本教程特别关注与处理配体相关的问题，假设用户熟悉基本的 GROMACS 操作和拓扑的内容。

本教程需要 2018.x 系列的 GROMACS 版本。

ubuntu18.04  用的是2019.6的版本 

#### 安装gromacs软件

```shell
#依次执行下列命令（Linux 下需要先切换为超级用户）：
tar xfz gromacs-2019.6.tar.gz
cd gromacs-2019.6
mkdir build
cd build
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON
make
make check
sudo make install
source /usr/local/gromacs/bin/GMXRC
调用gmx
```

下载将使用的蛋白质结构文件。在本教程中，我们将使用 T4 溶菌酶 L99A/M102Q（PDB ID 3HTB）,PDB中本身带有JZ4的配体（2-丙基苯酚），我们现在面临的问题是 JZ4 配体在 GROMACS 提供的任何力场中都不是可识别的实体，因此如果您尝试通过它传递此文件，pdb2gmx 将给出致命错误。

1. Prepare the protein topology with pdb2gmx
2. Prepare the ligand topology using external tools

#### 生成拓扑文件

 **蛋白质拓扑文件**

```shell
#处理蛋白质的水分子和其他多余的离子,生成下面文件
3HTB_clean.pdb
#将小分子配体提取出来
grep JZ4 3HTB_clean.pdb > jz4.pdb
#力场文件下载
tar -zxvf charmm36-mar2019.ff.tgz
#建立蛋白质拓扑文件
gmx pdb2gmx -f 3HTB_clean.pdb -o 3HTB_processed.gro -water spc
```

选择力场

```shell
Select the Force Field:
From current directory:
 1: CHARMM36 all-atom force field (July 2017)
From '/usr/local/gromacs/share/gromacs/top':
 2: AMBER03 protein, nucleic AMBER94 (Duan et al., J. Comp. Chem. 24, 1999-2012, 2003)
 3: AMBER94 force field (Cornell et al., JACS 117, 5179-5197, 1995)
 4: AMBER96 protein, nucleic AMBER94 (Kollman et al., Acc. Chem. Res. 29, 461-469, 1996)
 5: AMBER99 protein, nucleic AMBER94 (Wang et al., J. Comp. Chem. 21, 1049-1074, 2000)
 6: AMBER99SB protein, nucleic AMBER94 (Hornak et al., Proteins 65, 712-725, 2006)
 7: AMBER99SB-ILDN protein, nucleic AMBER94 (Lindorff-Larsen et al., Proteins 78, 1950-58, 2010)
 8: AMBERGS force field (Garcia & Sanbonmatsu, PNAS 99, 2782-2787, 2002)
 9: CHARMM27 all-atom force field (CHARM22 plus CMAP for proteins)
10: GROMOS96 43a1 force field
11: GROMOS96 43a2 force field (improved alkane dihedrals)
12: GROMOS96 45a3 force field (Schuler JCC 2001 22 1205)
13: GROMOS96 53a5 force field (JCC 2004 vol 25 pag 1656)
14: GROMOS96 53a6 force field (JCC 2004 vol 25 pag 1656)
15: GROMOS96 54a7 force field (Eur. Biophys. J. (2011), 40,, 843-856, DOI: 10.1007/s00249-011-0700-9)
16: OPLS-AA/L all-atom force field (2001 aminoacid dihedrals)
```

For this tutorial, choose the CHARMM36 force field (**option 1**), listed first under "From current directory" in the list.

###### 小tip

在这里安装了**Avogadro**

```shell
sudo apt install Avogadro
#在 Avogadro 中打开 jz4.pdb，然后从“Build”菜单中选择“Add Hydrogens”。 Avogadro 将所有 H 原子构建到 JZ4 配体上。保存名为“jz4.mol2.mol2”的 .mol2 文件（文件 -> 另存为...并从下拉菜单中选择 Sybyl Mol2）。
# 打开jz4.mol2
@<TRIPOS>MOLECULE
*****
 22 22 0 0 0
SMALL
GASTEIGER

@<TRIPOS>ATOM
      1 C4         24.2940  -24.1240   -0.0710 C.3   167  JZ4167     -0.0650
      2 C7         21.5530  -27.2140   -4.1120 C.ar  167  JZ4167     -0.0613
      3 C8         22.0680  -26.7470   -5.3310 C.ar  167  JZ4167     -0.0583
      4 C9         22.6710  -25.5120   -5.4480 C.ar  167  JZ4167     -0.0199
      5 C10        22.7690  -24.7300   -4.2950 C.ar  167  JZ4167      0.1200
      6 C11        21.6930  -26.4590   -2.9540 C.ar  167  JZ4167     -0.0551
      7 C12        22.2940  -25.1870   -3.0750 C.ar  167  JZ4167     -0.0060
      8 C13        22.4630  -24.4140   -1.8080 C.3   167  JZ4167     -0.0245
      9 C14        23.9250  -24.7040   -1.3940 C.3   167  JZ4167     -0.0518
     10 OAB        23.4120  -23.5360   -4.3420 O.3   167  JZ4167     -0.5065
     11 H          25.3133  -24.3619    0.1509 H       1  UNL1        0.0230
     12 H          23.6591  -24.5327    0.6872 H       1  UNL1        0.0230
     13 H          24.1744  -23.0611   -0.1016 H       1  UNL1        0.0230
     14 H          21.0673  -28.1238   -4.0754 H       1  UNL1        0.0618
     15 H          21.9931  -27.3472   -6.1672 H       1  UNL1        0.0619
     16 H          23.0361  -25.1783   -6.3537 H       1  UNL1        0.0654
     17 H          21.3701  -26.8143   -2.0405 H       1  UNL1        0.0621
     18 H          21.7794  -24.7551   -1.0588 H       1  UNL1        0.0314
     19 H          22.2659  -23.3694   -1.9301 H       1  UNL1        0.0314
     20 H          24.5755  -24.2929   -2.1375 H       1  UNL1        0.0266
     21 H          24.0241  -25.7662   -1.3110 H       1  UNL1        0.0266
     22 H          23.7394  -23.2120   -5.1580 H       1  UNL1        0.2921
@<TRIPOS>BOND
     1     4     3   ar
     2     4     5   ar
     3     3     2   ar
     4    10     5    1
     5     5     7   ar
     6     2     6   ar
     7     7     6   ar
     8     7     8    1
     9     8     9    1
    10     9     1    1
    11     1    11    1
    12     1    12    1
    13     1    13    1
    14     2    14    1
    15     3    15    1
    16     4    16    1
    17     6    17    1
    18     8    18    1
    19     8    19    1
    20     9    20    1
    21     9    21    1
    22    10    22    1
#需要进行的第一个更改是在 MOLECULE 标题中。将“*****”替换为“JZ4”
@<TRIPOS>MOLECULE
JZ4
#修复.mol2，如下。
@<TRIPOS>ATOM
      1 C4         24.2940  -24.1240   -0.0710 C.3     1  JZ4        -0.0650
      2 C7         21.5530  -27.2140   -4.1120 C.ar    1  JZ4        -0.0613
      3 C8         22.0680  -26.7470   -5.3310 C.ar    1  JZ4        -0.0583
      4 C9         22.6710  -25.5120   -5.4480 C.ar    1  JZ4        -0.0199
      5 C10        22.7690  -24.7300   -4.2950 C.ar    1  JZ4         0.1200
      6 C11        21.6930  -26.4590   -2.9540 C.ar    1  JZ4        -0.0551
      7 C12        22.2940  -25.1870   -3.0750 C.ar    1  JZ4        -0.0060
      8 C13        22.4630  -24.4140   -1.8080 C.3     1  JZ4        -0.0245
      9 C14        23.9250  -24.7040   -1.3940 C.3     1  JZ4        -0.0518
     10 OAB        23.4120  -23.5360   -4.3420 O.3     1  JZ4        -0.5065
     11 H          25.3133  -24.3619    0.1509 H       1  JZ4         0.0230
     12 H          23.6591  -24.5327    0.6872 H       1  JZ4         0.0230
     13 H          24.1744  -23.0611   -0.1016 H       1  JZ4         0.0230
     14 H          21.0673  -28.1238   -4.0754 H       1  JZ4         0.0618
     15 H          21.9931  -27.3472   -6.1672 H       1  JZ4         0.0619
     16 H          23.0361  -25.1783   -6.3537 H       1  JZ4         0.0654
     17 H          21.3701  -26.8143   -2.0405 H       1  JZ4         0.0621
     18 H          21.7794  -24.7551   -1.0588 H       1  JZ4         0.0314
     19 H          22.2659  -23.3694   -1.9301 H       1  JZ4         0.0314
     20 H          24.5755  -24.2929   -2.1375 H       1  JZ4         0.0266
     21 H          24.0241  -25.7662   -1.3110 H       1  JZ4         0.0266
     22 H          23.7394  -23.2120   -5.1580 H       1  JZ4         0.2921

#用下面代码修复
perl sort_mol2_bonds.pl jz4.mol2 jz4_fix.mol2
#这里生成的是jz4_fix.mol2
```

最后，注意@<TRIPOS>BOND 部分中奇怪的键顺序。所有程序似乎都有自己的生成此列表的方法，但并非所有程序都生而平等。如果键未按升序列出，则在构建具有匹配坐标的正确拓扑时会出现问题。要解决此问题，请下载我编写的 **sort_mol2_bonds.pl** 脚本并执行它，修复.mol2脚本如下。

```
#!/usr/bin/perl

use strict;

# sort_mol2_bonds.pl - a script to reorder the listing in a .mol2 @<TRIPOS>BOND
# section so that the following conventions are preserved:
#   1. Atoms on each line are in increasing order (e.g. 1 2 not 2 1)
#   2. The bonds appear in order of ascending atom number
#   3. For bonds involving the same atom in the first position, the bonds appear
#       in order of ascending second atom
#
# Written by: Justin Lemkul (jalemkul@vt.edu)
#
# Distributed under the GPL-3.0 license

unless (scalar(@ARGV)==2)
{
    die "Usage: perl sort_mol2_bonds.pl input.mol2 output.mol2\n";
}

my $input = $ARGV[0];
my $output = $ARGV[1];

open(IN, "<$input") || die "Cannot open $input: $!\n";
my @in = <IN>;
close(IN);

# test for header lines that some scripts produce
unless($in[0] =~ /TRIPOS/)
{
    die "Nonstandard header found: $in[0]. Please delete header lines until the TRIPOS molecule definition.\n"; 
}

open(OUT, ">$output") || die "Cannot open $output: $!\n";

# get number of atoms and number of bonds from mol2 file
my @tmp = split(" ", $in[2]);
my $natom = $tmp[0];
my $nbond = $tmp[1];

# check
print "Found $natom atoms in the molecule, with $nbond bonds.\n";

# print out everything up until the bond section
my $i=0;
while (!($in[$i] =~ /BOND/))
{
    print OUT $in[$i];
    $i++;
}

# print the bond section header line to output
print OUT $in[$i];
$i++;

# read in the bonds and sort them
my $bondfmt = "%6d%6d%6d%5s\n";
my @tmparray; 

# sort the bonds - e.g. the one that has the
# lowest atom number in the first position and then the
# lowest atom number in the second position (swap if necessary)
for (my $j=0; $j<$nbond; $j++)
{
    my @tmp = split(" ", $in[$i+$j]);
    # parse atom numbers
    my $ai = $tmp[1];
    my $aj = $tmp[2];
    # reorder if second atom number < first
    if ($aj < $ai)
    {
        $ai = $tmp[2];
        $aj = $tmp[1];
    }
    # store new lines in a temporary array
    $tmparray[$j] = sprintf($bondfmt, $tmp[0], $ai, $aj, $tmp[3]); 
}

# loop over tmparray to find each atom number
my $nbond = 0;
for (my $x=1; $x<=$natom; $x++)
{
    my @bondarray;
    my $ntmp = scalar(@tmparray);
    for (my $b=0; $b<$ntmp; $b++)
    {
        my @tmp = split(" ", $tmparray[$b]);
        if ($tmp[1] == $x)
        {
            push(@bondarray, $tmparray[$b]);
            splice(@tmparray, $b, 1);
            $ntmp--;
            $b--;
        }
    }

    if (scalar(@bondarray) > 0) # some atoms will only appear in $aj, not $ai
    {
        my $nbondarray = scalar(@bondarray);
        if ($nbondarray > 1)
        {
            # loop over all bonds, find the one with lowest $aj
            # and then print it
            for (my $y=0; $y<$nbondarray; $y++)
            {
                my @tmp2 = split(" ", $bondarray[$y]);
                my $tmpatom = $tmp[2];
                my $lowindex = 0;
                if ($tmp2[2] < $tmpatom)
                {
                    $lowindex = $y; 
                }
                my $keep = splice(@bondarray, $lowindex, 1);
                $y--;
                $nbondarray--;
                my @sorted = split(" ", $keep);
                $nbond++;
                printf OUT $bondfmt, $nbond, $sorted[1], $sorted[2], $sorted[3]; 
            }
        }
        else
        {
            $nbond++;
            my @tmp2 = split(" ", $bondarray[0]);
            printf OUT $bondfmt, $nbond, $tmp2[1], $tmp2[2], $tmp2[3];
        }
    }
}

close(OUT);

exit;
```



**小分子配体拓扑文件**

```shell
#在本教程中，我们将使用 CGenFF 生成 JZ4 拓扑。在该教程中已经给出了拓扑生成以后的文件。在平时需要自己生成。要合适的小分子配体生成网站。
#使用外部工具准备配体拓扑
https://cgenff.umaryland.edu/
#请以 mol2 格式上传您的分子。重要的是所有氢都以正确的质子化和互变异构状态存在，并且键序正确。 xyz 坐标并不重要。最大原子数为 384，每个用户每周限制为 100 个分子。
#重要笔记：
#** CGenFF 不应用于生物大分子！仅对有机小分子使用 CGenFF；高度优化的蛋白质、核酸、碳水化合物和脂质的 CHARMM 力场可从 CHARMM 力场页面免费下载，并可轻松与 CGenFF 结合使用。
#** 除了 mol2 格式，还可以上传 pdb 格式的文件。这些文件使用 Open Babel 2.3.0 在内部转换为 mol2。但是，我们不能保证这种转换的正确性；因此，强烈建议下载中间 mol2 文件并仔细检查其连接性和键序是否有错误。
#jz4_fix.mol2 文件用于生成拓扑，访问 CGenFF 服务器，登录您的帐户，然后单击页面顶部的“上传分子”。上传 jz4_fix.mol2，CGenFF 服务器将快速返回 （扩展名 .str）形式的拓扑。将其内容从 Web 浏览器保存到名为“jz4.str”的文件中。您也可以在此处下载此文件的副本。
#使用我们之前从 MacKerell 网站下载的 cgenff_charmm2gmx.py 脚本
#cgenff_charmm2gmx_py3_nx1.py 依赖Python 3.x，NetworkX 1.11
python cgenff_charmm2gmx_py3_nx1.py JZ4 jz4_fix.mol2 jz4.str charmm36-mar2019.ff
#分子拓扑已写入jz4.itp 分子需要的附加参数写入jz4.prm，需要包含在系统.top
```

**创建complex.gro**

```shell
# 从 pdb2gmx 中，我们有一个名为“3HTB_processed.gro”的文件，其中包含我们蛋白质的经过处理的、符合力场的结构。我们还有来自 cgenff_charmm2gmx.py 的“jz4_ini.pdb”，它包含所有必需的 H 原子并匹配 JZ4 拓扑中的原子名称。使用 editconf 将此 .pdb 文件转换为 .gro 格式
gmx editconf -f jz4_ini.pdb -o jz4.gro
#将 3HTB_processed.gro 复制到要操作的新文件中，例如，将其命名为“complex.gro”，因为将 JZ4 添加到蛋白质中将生成我们的蛋白质配体复合物。接下来，简单地复制 jz4.gro 的坐标部分并将其粘贴到 complex.gro 中，在蛋白质原子的最后一行下方，框向量之前
#具体见网站教程


#由于我们在 .gro 文件中又添加了 22 个原子，因此增加 complex.gro 的第二行以反映此更改。现在坐标文件中应该有 2636 个原子。

这里很重要，后续的盒子增大会用到。
```

**创建拓扑**   Build the Topology

见网页教程，topol.top文本文件的开头和结尾部分都需要添加相应的参数

```shell
head topol.top
tail topol.top
```

#### 增大盒子和溶剂化蛋白

```shell
#定义盒子，并用水填充
gmx editconf -f complex.gro -o newbox.gro -bt dodecahedron -d 1.0

gmx solvate -cp newbox.gro -cs spc216.gro -p topol.top -o solv.gro

#这里出现报错，top和gro原子数不匹配，在complex.gro部分出现错误，已解决。
```

#### 添加离子

```shell
#ions.mdp是有现成的代码，使用 grompp 组合一个 .tpr 文件，使用任何 .mdp 文件。我使用 .mdp 文件来运行能量最小化，因为它们需要最少的参数，因此最容易维护。
#用VMD查看solv.gro

gmx grompp -f ions.mdp -c solv.gro -p topol.top -o ions.tpr

gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -nn 6  #-nn 6主要是根据离子的个数 -neutral中性
```

Your [ molecules ] directive should now look like:

```
[ molecules ]
; Compound        #mols
Protein_chain_A     1
JZ4                 1
SOL             10228
CL                  6
```

#### 添加离子

```shell
#em.mdp也是有现成代码
gmx grompp -f em.mdp -c solv_ions.gro -p topol.top -o em.tpr
```



#### 能量最小化

```shell
gmx mdrun -v -deffnm em
#在这里还可以继续查看生成的文件
```



#### 平衡我们的蛋白质配体复合物就像平衡任何其他含有蛋白质的系统。在这种情况下，有一些特殊的考虑：

1.对配体施加限制（Restraining the Ligand）

```
#Restraining the Ligand

gmx make_ndx -f jz4.gro -o index_jz4.ndx
...
 > 0 & ! a H*
 > q
 
 gmx genrestr -f jz4.gro -n index_jz4.ndx -o posre_jz4.itp -fc 1000 1000 1000
 
 如果我们只是想在蛋白质也受到限制时限制配体，请将以下几行添加到拓扑中指示的位置;如果您想要在平衡过程中进行更多控制，即独立限制蛋白质和配体，您可以控制在不同的 #ifdef 块中包含配体位置限制文件,详细的内容见教程。
```

2.温度偶联基团的处理（Thermostats）

```
#为了不使
gmx make_ndx -f em.gro -o index.ndx
...
  0 System              : 33506 atoms
  1 Protein             :  2614 atoms
  2 Protein-H           :  1301 atoms
  3 C-alpha             :   163 atoms
  4 Backbone            :   489 atoms
  5 MainChain           :   651 atoms
  6 MainChain+Cb        :   803 atoms
  7 MainChain+H         :   813 atoms
  8 SideChain           :  1801 atoms
  9 SideChain-H         :   650 atoms
 10 Prot-Masses         :  2614 atoms
 11 non-Protein         : 30892 atoms
 12 Other               :    22 atoms
 13 JZ4                 :    22 atoms
 14 CL                  :     6 atoms
 15 Water               : 30864 atoms
 16 SOL                 : 30864 atoms
 17 non-Water           :  2642 atoms
 18 Ion                 :     6 atoms
 19 JZ4                 :    22 atoms
 20 CL                  :     6 atoms
 21 Water_and_ions      : 30870 atoms
 
# Merge the "Protein" and "JZ4" groups with the following, where ">" indicates the make_ndx prompt:

> 1 | 13
> q
```



#### 平衡

```shell
#NVT   this.mdp是nvt.mdp相关代码 10:20-11:10
gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -n index.ndx -o nvt.tpr

gmx mdrun -deffnm nvt
#NPT   this.mdp是npt.mdp相关代码 11:15-12:25
gmx grompp -f npt.mdp -c nvt.gro -t nvt.cpt -r nvt.gro -p topol.top -n index.ndx -o npt.tpr

gmx mdrun -deffnm npt
```

#### MD

```shell
#MD  here是md.mdp模拟所用代码，时间设置在其中设置，原代码是设置50000steps,10000ps；该电脑完成时间为12:30到次日6:06；即1ns用时17.5h
gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -n index.ndx -o md_0_10.tpr

gmx mdrun -deffnm md_0_10
```

#### 分析

```shell
xmgrace
VMD
Av

数据分析
分析前需要检查模拟结果的质量，只有对好的模拟结果进行分析才有意义。

目录中有以下几个模拟文件，其内容如下：

topol.tpr:运行输入文件，包含模拟开始时系统的完整描述；

confout.gro:结构文件，包含最后一步的坐标和速度；

traj.trr:包含位置，速度和力随时间变化的全精度轨迹;

traj.xtc：精简轨迹，仅包含低精度（0.001 nm）的坐标；

ener.edr：与能量相关的参数随时间的变化；

md.log：包含有关模拟信息的文件。
```



#### 结果检查

其他的开始之前，必须保证模拟正常结束。有许多因素能使模拟崩溃，如力场、系统平衡不充分等问题。用gmxcheck命令检查模拟是否正常结束：

gmxcheck -f traj.xtc

查看输出，确保模拟运行了1ns。

#### 结果可视化

现在用gromacs的轨迹查看器ngmx查看轨迹。

用pymol设置轨迹，进行视频播放。

```
xmgrace   Public/HY/MD/rmsd_jz4.xvg  
可以画出RMSD的轨迹图片

Avogadro是可以为配体加氢生成mol2格式文件
```



