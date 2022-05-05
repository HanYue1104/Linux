使用ATB网站 [分享：生成小分子力场TOP（ITP）：使用ATB网站 - 分子模拟 - Gromacs/Amber/NAMD - 小木虫论坛-学术科研互动平台 (muchong.com)](http://muchong.com/t-14441082-1)

[几种生成有机分子GROMACS拓扑文件的工具 - 思想家公社的门口：量子化学·分子模拟·二次元 (sobereva.com)](http://sobereva.com/266)

```shell

分享：生成小分子力场TOP（ITP）：使用ATB网站（原文链接：https://mp.weixin.qq.com/s/YTGpwmkrBpCTbOPDXlOGbA）

在ATB网站生成itp文件之前，需要注册一个ATB账号，该网站主页为：
https://atb.uq.edu.au/，打开之后，自行注册账号。然后登陆网站。
用vmd或者pymol等工具创建好自己目标分子的结构文件，保存为pdb结构。然后通过网页上传，步骤如下：
1、点击网页左上角的Submit，出现下面的页面：
*Molecule type 根据你自己分子所属归类选择，不清楚话就用默认的。
*Net charge (e) 根据你自己分子对外所带的电荷是多少，填入即可。
然后点击Upload PDB file 把创建好的PDB文件上传上去。也可以自己使用其提供的画图工具创建一个。
2、上传完成后点击Next
3、出现下面界面：
4、可以点击左下角Submit This Molecule提交你自己的分子。但是，如果你使用的是常见的分子，ATB数据库里面可能会有，在此页面的下面，其会显示已经有的分子,如下面这种情况：
若有多个已存在的分子，选择一个RMSD最小的即可，然后点击Add to Saved Molecule。之后步骤和下面一样。
5、当提交完分子之后，ATB网站会帮你做优化。发后会收到一个邮件说开始计算了。算完之后也会发一个邮件提示。根据分子的大小，分子大的话几天都有可能还在算，所以耐心等待。
6、计算完成后，点击自己账户下拉菜单中的：Saved Molecules
7、找到自己上传的分子，然后点击其前面的编号：
自己上传的分子有可能计算出错，网站会提示的。
8、进入之后，点击Molecular Dynamics（MD）Files
9、网站会提供全原子力场和联合原子力场的itp和pdb文件，根据需要自行打开，然后保存。务必itp和pdb要对应一致。若是不想改变自己小分子的坐标（因为有时候是对接之后的小分子，改变坐标对接位点就会改变），点击original geometry即可。
10、把itp和pdb文件都保存下来之后，把itp文件include到总的top文件里面即可。注意top里面用到的分子名字要用itp里面的，即[ moleculetype ]字段下面的名字。建立盒子时候一定要用新保存的pdb文件，不要用原来自己创建的。
11、由于ATB网站使用的是它自己的原子类型名称，所以用gromacs软件自带的力场话，就会报错，一般报错为：Atomtype ** not found
所以，务必要下载ATB网站自己的gromos54a7力场，上传到模拟目录里面，然后在top文件里面不再调用gromacs软件自带力场的路径，改为当前你在ATB网站上下载的gromos54a7力场文件夹，路径一定不要弄错。
12、下载方法为：点击页面 Others，选择Forcefield Files，进入新界面。
13、选择下图红框中的力场即可

ATB网站最后选用Topology File和Structure File：GROMACS G54A7FF United-Atom (ITP file)；United-Atom PDB (optimised geometry)
```





