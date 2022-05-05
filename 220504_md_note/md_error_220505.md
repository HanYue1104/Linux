在用分子动力学模拟软件（Gaussian; gview; antechamber; acpype;CGenFF网站;ATB网站；gromacs等 ）遇到的报错

##### # Gaussian; gview; antechamber; acpype;CGenFF网站;ATB网站 ：都是用来生成小分子itp和top文件的。

Gaussian; gview使用的是破解版

gview5 win

Gaussian 16 linux

##### # 使用gaussion优化FBP小分子（小分子结构是从PDB数据库中截取的），最后可能会出现优化不成功 link 9999的报错。最后查出原因是因为初始结构不行，通过chemdraw和chem3D画了该结构，导出gjf，编辑gjf,重新优化即可。

##### # 使用antechamber应该在base环境下使用，不要在自己用conda建立的AmberTools21环境下使用。

##### # CGenFF：使用的是网站 https://links.jianshu.com/go?to=https%3A%2F%2Fcgenff.umaryland.edu%2F  需要注册（学术邮箱）

##### #ATB网站有两种用法，先把自己的结构提交上去看看库中有没有，查找不到的话。可以自己提交任务计算。

用以上三种方法的处理小分子生成itp和top文件有分别的笔记介绍。

需要根据大分子选用的力场来选用小分子的处理力场，不然做后续模拟的时候不兼容。

