20220504

除了使用高斯软件处理小分子，还用CGenFF力场处理了参数。

参考简书笔记  https://www.jianshu.com/p/6526f7aa8bc6

CGenFF网站现已提供python3版本的charmm2gmx脚本，下载链接如下：

cgenff_charmm2gmx_py3.py

CGenFF程序可以直接生成Charmm力场的分子拓扑文件，但使用较为复杂，特此记录一下。

[CGenFF 网站](https://links.jianshu.com/go?to=https%3A%2F%2Fcgenff.umaryland.edu%2F)
 [CGenFF FAQ](https://links.jianshu.com/go?to=https%3A%2F%2Fcgenff.umaryland.edu%2FcommonFiles%2Ffaq.php%23morene)

## 使用流程

### 制作CGenFF专属`.str`文件

制作`.mol2`格式的坐标文件，氢原子也包含在内。不推荐使用MS软件，其产生的`.mol2`文件在程序中会出现警告。GView产生的`.mol2`文件需删除两个空行，必要的话可以进行`sed -i 's/Ar/ar' *.mol2`修改大小写。其他软件详见FAQ。

在[CGenFF](https://links.jianshu.com/go?to=https%3A%2F%2Fcgenff.umaryland.edu%2Finitguess%2F)网站上传`.mol2`文件，此处有三个复选框：

- Guess bond orders from connectivity
- Include parameters that are already in CGenFF
- Use CGenFF legacy v1.0

一般情况下都不用选。选项意义如字面所言。选项一重新判断键价，有一定概率出错，如果时用于其他软件的拓扑，不用选择。选项二会将原力场中已包含的信息一并列出，在后续格式转换的过程中可能会出现重复。

`.str`文件中会有一个`penalty`值，该值小于10表明结果较好，在10到50之间则说明需要进行一定的验证，大于50则需要更广泛的验证。

### 转换为Gromacs格式

格式转换需要用到[cgenff_charmm2gmx.py](https://links.jianshu.com/go?to=http%3A%2F%2Fmackerell.umaryland.edu%2Fdownload.php%3Ffilename%3DCHARMM_ff_params_files%2Fcgenff_charmm2gmx.py)，同时预先下载好Gromacs的[Charmm力场文件](https://links.jianshu.com/go?to=http%3A%2F%2Fmackerell.umaryland.edu%2Fcharmm_ff.shtml%23gromacs)。

运行改程序的命令为：

```shell
python cegnff_charmm2gmx.py RESNAME drug.mol2 drug.str charmm36.ff
#python cgenff_charmm2gmx_py3_nx1.py fbp 220504_fbp.mol2 220504_fbp.str charmm36-jul2021.ff
```

python应该使用2.*版本

`RESNAME`为残基名称例如我的残基名字是fbp，这里应该写fbp。 在`.str`文件中见"XXX     0.000!"字段中的XXX

`drug.mol2`为`.mol2`文件

`drug.str`为`.str`文件

`charmm36`为力场文件夹的**名称**，此处力场版本最好与CGenFF程序中的保持一致，常用为4.0+版本，一般力场文件会实时更新。

## 其他事项

### `.mol2`文件基本格式

```python
@<TRIPOS>MOLECULE
残基名称
原子个数    键数  残基数
type: SMALL BIOPOLMER PROTEIN NULEIC_ACID SACCHARIDE
charges: NO_CHARGES MMFF94_CHARGES USER_CHARGES GSDTELIGER等
@<TRIPOS>ATOM
原子信息
@<TRIPOS>BOND
成键信息
...
```

### 常见错误

Q：转换格式时出现错误："Error in atomgroup.py: read_mol2_coor_only: no. of atoms in mol2 (%d) and top (%d) are unequal"

A：通常由于`.mol2`文件与`.str`文件中的残基名称不一致导致。



[几种生成有机分子GROMACS拓扑文件的工具 - 思想家公社的门口：量子化学·分子模拟·二次元 (sobereva.com)](http://sobereva.com/266)

CGenFF

```shell
• CGenFF（https://cgenff.paramchem.org）：先注册，上传有机小分子的mol2文件，即可生成基于CGenFF力场的CHARMM的拓扑文件。如果mol2是gview建的，一定要事先把里面的Ar替换成ar。上传文件的时候不要选Guess bond orders from connectivity和Include parameters that are already in CGenFF。如果文件处理正常，会立刻产生str文件，点击其链接之后把里面内容拷到比如Actos.str里面。进入网页里的More Info & Tools - Utilities，点击GROMACS conversion program，可下载把str文件转换为GROMACS格式的Python脚本cgenff_charmm2gmx.py。
#gromacs里面下载用于GROMACS的CHARMM36力场文件，一般力场文件用最近更新的，现在是2022，用的是2021的。解压得到比如charmm36-nov2016.ff文件夹，将它和Actos.str、cgenff_charmm2gmx.py都放到当前目录，另外也把此文件夹拷到gromacs的top目录下。假设Actos.str里RESI后面的词是Molecu，最初上传的是Actos.mol2，则运行./cgenff_charmm2gmx.py Molecu Actos.mol2 Actos.str charmm36-nov2016.ff。在当前目录会产生molecu.top、molecu.prm、molecu.itp、molecu.pdb（从mol2转换过来的），适当调整itp和top里的分子名，调整itp里的残基名和结构文件相对应后，即可用于模拟。str文件里的力场参数有penalty指标，数值越大说明此参数可靠性越低，详见网页里的说明。
```

