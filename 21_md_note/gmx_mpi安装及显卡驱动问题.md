```
安装Ubuntu后重启出现perform MOK management
系统版本：Ubuntu 20.04 LTS

perform mok management
安装完显卡驱动后，系统需要重启加载驱动，在重启系统时，会出现一个蓝色背景的界面 perform mok management , 选择 continue reboot， 可能导致新安装的 N 卡驱动没有加载，正确的做法如下： 
(1)当进入蓝色背景的界面perform mok management 后，选择 enroll mok , 
(2)进入enroll mok 界面，选择 continue , 
(3)进入enroll the key 界面，选择 yes , 
(4)接下来输入你在安装驱动时输入的密码， 
(5)之后会跳到蓝色背景的界面perform mok management 选择第一个 reboot

这样，重启后N卡驱动就加载了，恭喜你，Ubuntu 安装成功。可以在系统信息处看到显卡已经是独立显卡。
```





# Ubuntu18.04成功安装Nvidia驱动（解决禁用默认第三方驱动Nouveau后无法进入系统的问题）

```
nvidia-smi
```





```shell
确认安装环境
1. NVIDIA显卡已经正常安装

2. nouveau已经禁用

可以使用下面命令查看，如果没有输出代表成功：

lsmod | grep nouveau
如果正确安装了NVIDIA的驱动就会禁止掉了。

3. 验证系统是否安装了gcc

终端输入下面命令查看是否安装:

vincent@dell-Inspiron-7559 Dir:~
·····$gcc --version
gcc (Ubuntu 7.3.0-16ubuntu3) 7.3.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE
```





```shell
#安装完程序以后检验是否成功
cd /usr/local/cuda/bin

./nvcc -V
```



![img](https://img-blog.csdnimg.cn/20210314202026725.png)

在darknet的make编译的时候 出现上述问题的解决方案:
原因：这是不支持compute_30的gpu构架，这是由于GPU太新，与CUDA版本不兼容导致
根据自己CUDA的版本注释不同的行。在gromacs-2019.6/



# gmx Gromacs安装教程（+MPI+GPU加速版）及相关

## GMX安装

### 一、安装fftw-3.2.2，安装路径为/opt/fftw-3.2.2

http://www.fftw.org/fftw-3.3.8.tar.gz下载.

```shell
tar xzvf fftw-3.3.4.tar.gz   
cd fftw-3.3.4  
./configure --prefix=/opt/fftw-3.3.4 --enable-sse2 --enable-avx --enable-float --enable-shared  --enable-avx2
```

如果你的CPU相对较新，支持AVX2指令集，可再加上 `--enable-avx2` 选项以获得更好性能。

```shell
sudo make 
sudo make install
```

make时可调用-j，代表调用所有核并行编译，建议用于编译的核不要超过4个？即`make -j 4`。
编译完后可以把FFTW的解压目录和压缩包删掉。

打开~/.bashrc（gedit ~/.bashrc） 复制：

```shell
# For FFTW
export LD_LIBRARY_PATH=/opt/fftw-3.3.4/lib:$LD_LIBRARY_PATH
export PATH=/opt/fftw-3.3.4/bin:$PATH
```

保存后 `source ~/.bashrc` 并用 `vi ~/.bashrc`检查一下。

### 二、安装CMake

先检查cmake –version，如果显示没有安装则先安装cmake

```shell
tar xzvf cmake-2.8.12.2.tar.gz
cd cmake-2.8.12.2
./configure --prefix=/opt/cmake-2.8.12.2
sudo make
sudo make install
```

打开~/.bashrc 复制：

```shell
# For CMake
export PATH=/opt/cmake-2.8.12.2/bin:$PATH
```

保存后 `source ~/.bashrc` 并用 `cmake --version` 查看版本。

### 三、安装cuda

去https://developer.nvidia.com/cuda-downloads下载CUDA toolkit并安装.

```shell

```



### 四、安装OpenMPI

[http://www.open-mpi.org下载OpenMPI最新版本](http://www.open-mpi.xn--orgopenmpi-8i2p938t1tdwra964k1w5g/).

```shell
./configure --prefix=/opt/openmpi
make all install -j
```

之后在用户目录下的.bashrc末尾加入以下两行

```shell
export PATH=$PATH:/opt/openmpi/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/openmpi/lib
```

如xilock的为：

```shell
# openmpi
export PATH=/THFS/opt/openmpi/2.0.2/bin/:$PATH
export LD_LIBRARY_PATH=/THFS/opt/openmpi/2.0.2/lib:$LD_LIBRARY_PATH
```

然后重新进入终端使以上语句生效。

**注意，若装了mpi版本的还想再装一个非mpi版本的来供单节点使用，则需要重新编译一个gromacs且需要在编译gromacs时把mpi给OFF掉。**

### 五、安装Gromacs-5.1.4

```shell
tar xzvf gromacs-5.1.4.tar.gz 
cd gromacs-5.1.4
mkdir build
cd build

export CMAKE_PREFIX_PATH=/<fftw的路径/>
cmake .. -DCMAKE_INSTALL_PREFIX=/opt/gromacs-5.1.4-gpu -DGMX_MPI=ON -DGMX_GPU=ON -DCUDA_TOOLKIT_ROOT_DIR=/<cuda的路径/>

sudo make 
```

进行make时，若此前安装的是cuda9.0以上版本，则可能会报错“nvcc fatal : Unsupported gpu architecture ‘compute_20’ while cuda9.0 is installed”
这是由兼容性导致的，此时需要更改一下gromacs里面的文件“gromacs-5.1.2/cmake/gmxManageNvccConfig”，将其中几处带“-gencode;arch=compute_20,code=compute_20”的行处理掉（如将其注释掉）即可。

```shell
sudo make install
```

**注意，也可不做`sudo make`直接`sudo make install`**

打开~/.bashrc 复制：

```shell
# For Gromacs
source /opt/gromacs-5.1.4-gpu/bin/GMXRC
```

保存并 `source ~/.bashrc`

### 六、测试

```shell
gmx_mpi mdrun
```





