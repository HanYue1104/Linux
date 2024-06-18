

# 转录组测序上游分析

## 1.对自己的工作目录进行管理

##### 这里很重要，对自己之后的目录管理清晰明了，容易找到数据和文件存放位置

## 2.软件安装和管理

### conda安装和管理

#### 从官网下载最新版Miniconda3安装包，但速度较慢

`wget -c https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh `
​

#### 从清华镜像下载最新版Miniconda3安装包，速度很快，国内用户推荐,这里要搞清楚miniconda和Anaconda的区别：后者包含python2，本身也包含miniconda，文件也比较大，一般使用后者就可以了

```
wget -c  https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

#### 安装Miniconda3：安装过程只需要输入 yes 或者按 Enter

`bash Miniconda3-latest-Linux-x86_64.sh`
​

#### miniconda3安装成功，并成功配置好环境变量

`source ~/.bashrc`
`conda --help`
​

### 设置国内镜像源

##### 添加bioconda channel，bioconda中包含了生物信息学相关的软件

##### 安装一次，只配置一次，注意语句复制正确，

##### 一行一行复制在命令行上按enter键运行，不会出现任何提示即为正确 

##### 下面这三行配置官网的channel地址

```
conda config --add channels r 
conda config --add channels conda-forge 
conda config --add channels bioconda
```

##### 下面这四行配置清华大学的bioconda的channel地址，国内用户推荐

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
conda config --set show_channel_urls yes
conda config --set channel_priority flexible
```

##### 查看配置结果

`cat /home/teach_rna/.condarc`
​

### conda管理环境

##### 创建名为rna的软件环境来安装转录组学分析的生物信息学软件

##### 创建小环境成功，并成功安装python3版本，每建立一个小环境，安装一个python=3的软件作为依赖

`conda create -y -n rna python=3`
​

##### 查看当前conda环境

`conda info --e`
​

##### 每次运行前，激活创建的小环境rna，成功激活进入小环境，即可安装软件,这里作为新手特别容易忘

```
conda activate rna # 激活
conda deactivate # 退出小环境
```

##### 查看当前环境的python版本

`python --version`

##### 删除环境

`conda remove -n rcnn --all`

### conda管理软件

##### 可以一次安装一个软件，也可以一次安装多个软件

```
conda install -y sra-tools
conda install -y sra-tools fastqc trim-galore hisat2 subread multiqc samtools salmon fastp
```

##### 运行以下语句，不出现报错表示安装成功

```
fastqc --help
fastqc -h
#每个软件都执行一下这个命令，确保软件安装成功，以便之后调用
```

##### 一定要搞清楚你的软件被conda安装在哪

`which samtools`

##### 每个软件都执行一下这个命令，确保软件位置

##### 在当前小环境下，列举所有安装软件。

`conda list`



### 在这里因为是公司给的fq.gz文件，所以不用用aspera下载，就没有操作这步

### 3.数据完整性检验(非常重要！！！)

##### 得到md5值

md5值是公司给的，建一个txt文件，把结果放到里面

##### md5值检验

`md5sum -c md5.txt`

##### 这里我遇到了报错，因为自己学的还不太会解决，所以从网上查找解决的方法

```
md5sum 无法使用回车字符
md5sum: 'raw/k1_1.fq.gz'$'\r': 没有那个文件或目录
: FAILED open or read
```

文本中有时候有特殊字符

##### 解决方法

```
cat md5.txt | tr -d '\r' | md5sum -c -
会返回
k1_1.fq.gz: 成功
k1_2.fq.gz: 成功
。。。。
或者用这个命令解决sed  $'s/\r//' md5.txt | md5sum -c -

```

### 4.数据质控

目标：使用fastqc对原始数据进行质量评估

##### ￼激活conda环境

`conda activate rna`

##### 使用FastQC软件对多个fastq文件进行质量评估，结果输出到qc/文件夹下

```
qcdir=/home/tmu/public/RNA/raw/qc/
fqdir=/home/tmu/public/RNA/raw/fastq/
fastqc -t 4 -o $qcdir $fqdir/k*.fq.gz(*.fq.gz)
```

##### 使用MultiQc整合FastQC结果

```
查看一下 ls *.zip
multiqc *.zip
```

### 5.数据过滤

上课时是对比了两个软件过滤fq文件的区别，这里我直接只用了trim_galore来过滤

#### trim_galore过滤

```
#定义文件夹

rawdata=/home/tmu/public/RNA/raw/fastq/  
#在这里要注意最后的/，我在处理过滤的过程中因为这个遇到了好多报错，浪费了比较长的时间。
cleandata=/home/tmu/public/RNA/raw/trim_galore/

# 多个样本处理
cat /home/tmu/RNA/raw/sampleId.txt | while read id
do
   echo "trim_galore --phred33 -q 20 --length 36 --stringency 3 --fastqc --paired --max_n 3 -o ${cleandata} ${rawdata}${id}_1.fq.gz ${rawdata}${id}_2.fq.gz"
done >trim_galore.sh

nohup sh trim_galore.sh >trim_galore.log &

jobs
[1]+  运行中               nohup sh trim_galore.sh > trim_galore.log &
top命令查看，在这里不展示了
```

```
#多个样本处理错误操作
cat /home/tmu/RNA/raw/sampleId.txt | while read id
do
   echo "trim_galore --phred33 -q 20 --length 36 --stringency 3 --fastqc --paired --max_n 3 -o ${cleandata} ${rawdata}/${id}_1.fq.gz ${rawdata}/${id}_2.fq.gz"
done >trim_galore.sh

nohup sh trim_galore.sh >trim_galore.log &

#提交后台以后就会出现下面的错误
nohup: 忽略输入重定向错误到标准输出端
[1]+  退出 255              nohup sh trim_galore.sh > trim_galore.log
#当我把上面的/去除以后就好了，归根结底就还是路径的问题，对自己来说，路径问题还是很容易出错的，以后要多多注意
```

### 6.数据比对

上课时使用两个软件对fq数据进行比对，得到比对文件sam/bam，并探索比对结果.

在这里因为处理的是E.coli的数据，所以得自己找到E.coli的gtf.gz   fq.gz注释和参考基因组文件.

#### 参考基因组

##### 参考基因组准备:注意参考基因组版本信息

下载，Ensembl：http://asia.ensembl.org/index.html

``` 
ftp://ftp.ensemblgenomes.org/pub/bacteria/release-48/fasta/bacteria_0_collection/escherichia_coli_str_k_12_substr_mg1655/dna
```

##### 进入到参考基因组目录

`cd  ~/database/genome/Ensembl/E.coli`

##### 下载基因组序列

```
wget  ftp://ftp.ensemblgenomes.org/pub/bacteria/release-48/fasta/bacteria_0_collection/escherichia_coli_str_k_12_substr_mg1655/dna/Escherichia_coli_str_k_12_substr_mg1655.ASM584v2.dna.toplevel.fa.gz
```

##### 下载基因组注释文件

```
wget ftp://ftp.ensemblgenomes.org/pub/bacteria/release-48/gtf/bacteria_0_collection/escherichia_coli_str_k_12_substr_mg1655/Escherichia_coli_str_k_12_substr_mg1655.ASM584v2.48.gtf.gz
```

#### Hisat2比对

##### 进入参考基因组目录

`cd  ~/database/genome/Ensembl/E.coli-bacteria-release-48`

##### Hisat2构建索引(dna)

`hisat2-build Escherichia_coli_str_k_12_substr_mg1655.ASM584v2.dna.toplevel.fa colidna`

参考基因组版本

##### Hisat2比对

```
#输入输出定义文件夹

cd ~/public/RNA/raw/Mapping/hisat
index=/home/tmu/database/genome/Ensembl/E.coli-bacteria-release-48/colidna
inputdir=/home/tmu/public/RNA/raw/trim_galore
outdir=/home/tmu/public/RNA/raw/Mapping/hisat

#多个样本批量进行比对
cat /home/tmu/public/RNA/raw/trim_galore/sample.txt |while read id
do   echo "hisat2 -p 3 -x ${index} -1 ${inputdir}/${id}_1_val_1.fq.gz -2 ${inputdir}/${id}_2_val_2.fq.gz 2>${id}.log  | samtools sort -@ 1 -o ${outdir}/${id}.Hisat_aln.sorted.bam -  && samtools index ${outdir}/${id}.Hisat_aln.sorted.bam ${outdir}/${id}.Hisat_aln.sorted.bam.bai"
done >Hisat.sh

nohup sh Hisat.sh >Hisat.log &
 jobs
 [1]+  运行中               nohup sh Hisat.sh > Hisat.log &
 top
 
 #统计比对情况
 multiqc -o ./*log
```

### 7.定量

##### 使用featureCounts对bam文件进行定量。

##### 定义输入输出文件夹

```
gtf=/home/tmu/database/genome/Ensembl/E.coli-bacteria-release-48/Escherichia_coli_str_k_12_substr_mg1655.ASM584v2.48.gtf.gz
inputdir=/home/tmu/public/RNA/raw/Mapping/hisat
outputdir=/home/tmu/public/RNA/raw/Expression/featureCount

#featureCounts对bam文件进行计数
featureCounts -T 1 -p -t exon -g gene_id -a $gtf -o all.id.txt $inputdir/*.sorted.bam
```

##### 对定量结果质控

`multiqc all.id.txt.summary`

##### 得到表达矩阵

`cat all.id.txt | cut -f1,7- > counts.txt`





