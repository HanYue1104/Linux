## gmx_mpi安装

### 一、安装fftw-3.2.2，安装路径为/home/tmu/software/Downloads/fftw/fftw-3.3.8

http://www.fftw.org/fftw-3.3.8.tar.gz下载.

```
tar xzvf fftw-3.3.8.tar.gz   
cd fftw/fftw-3.3.8 
./configure --prefix=/home/tmu/software/Downloads/fftw/fftw-3.3.8 --enable-sse2 --enable-avx --enable-float --enable-shared  
```

如果你的CPU相对较新，支持AVX2指令集，可再加上 `--enable-avx2` 选项以获得更好性能。

注：一般计算只需要按照前述**编译单精度版本**就够了，但如果模拟刚开始就崩溃，有时候用双精度版本可解决，但计算比单精度版慢将近一倍、trr/edr等文件体积大一倍。另外，做正则振动分析时在能量极小化和对角化Hessian矩阵的时候一般也需要用双精度版以确保数值精度。注意，**编译双精度版本时不支持GPU加速**。

**要编译双精度版本的话**，先按照前文方式编译一遍单精度版本，毕竟这之后在研究中肯定也得用。然后再重复一遍编译过程，但是在编译FFTW时去掉--enable-float，并且在使用cmake3命令时额外加上-DGMX_DOUBLE=ON选项。双精度版本的GROMACS可执行文件是gmx_d，而单精度是gmx，因此单精度和双精度的可执行文件可以同时存在于同一目录，互不冲突。

```
make 
make install
```

make时可调用-j，代表调用所有核并行编译，建议用于编译的核不要超过4个？即`make -j 4`。
编译完后可以把FFTW的解压目录和压缩包删掉。

打开~/.bashrc（vim ~/.bashrc） 复制：

```
# For FFTW
export PATH=$PATH:/home/tmu/software/Downloads/fftw/fftw-3.3.8/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/tmu/software/Downloads/fftw/fftw-3.3.8/lib
```

保存后 `source ~/.bashrc` 并用 `vim ~/.bashrc`检查一下。

### 二、安装CMake

先检查cmake –version，如果显示没有安装则先安装cmake

```
tar -xzvf cmake-3.22.1.tar.gz
cd cmake/cmake-3.22.1/
./configure --prefix=/home/tmu/software/Downloads/cmake/cmake-3.22.1
make
make install
```

打开~/.bashrc 复制：

```
# For CMake
export PATH=/home/tmu/software/Downloads/cmake/cmake-3.22.1/bin:$PATH
```

保存后 `source ~/.bashrc` 并用 `cmake --version` 查看版本。

### 三、安装cuda

第一次安装cuda的时候出现没有安装Nvidia驱动，需要解决禁用默认第三方驱动Nouveau后无法进入系统的问题。

20211221安装的时候看到了youtube的一个视频【搜索gromacs   cuda GPU】，有一个针对桌面版的ubuntu直接选择驱动的方法，这次没有遇到驱动禁用的问题，应该是在上次解决以后就可以用了。下次遇到ubuntu驱动问题就可以尝试一下这个方法。

去https://developer.nvidia.com/cuda-downloads下载CUDA toolkit并安装.

```shell
https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=18.04&target_type=deb_local
#CUDA安装
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.5.1/local_installers/cuda-repo-ubuntu1804-11-5-local_11.5.1-495.29.05-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-11-5-local_11.5.1-495.29.05-1_amd64.deb
sudo apt-key add /var/cuda-repo-ubuntu1804-11-5-local/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
```

打开~/.bashrc 复制：

```shell
export PATH=/usr/local/cuda-11.5/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.5/lib64:$LD_LIBRARY_PATH
```

### 四、安装OpenMPI

http://www.open-mpi.org下载OpenMPI最新版本.

```
cd software/Downloads/openmpi/openmpi-4.1.2/
./configure --prefix=/home/tmu/software/Downloads/openmpi/openmpi-4.1.2
make
make install
```

之后在用户目录下的.bashrc末尾加入以下两行

```
export PATH=$PATH:/home/tmu/software/Downloads/openmpi/openmpi-4.1.2/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/tmu/software/Downloads/openmpi/openmpi-4.1.2/lib
```

**注意，若装了mpi版本的还想再装一个非mpi版本的来供单节点使用，则需要重新编译一个gromacs且需要在编译gromacs时把mpi给OFF掉。**

### 五、安装Gromacs-5.1.4

```
tar xzvf gromacs-5.1.4.tar.gz 
cd gromacs-5.1.4
mkdir build
cd build

export CMAKE_PREFIX_PATH=/home/tmu/software/Downloads/fftw/fftw-3.3.8
cmake .. -DCMAKE_INSTALL_PREFIX=/home/tmu/software/Downloads/gromacs_mpi/gromacs-2019.6 -DGMX_MPI=ON -DGMX_GPU=ON -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.5 

make 
make install
```

进行make时，安装的是cuda11.5版本，报错“nvcc fatal : Unsupported gpu architecture ‘compute_30’ while cuda11.5 is installed”
这是由兼容性导致的，此时需要更改一下gromacs里面的文件“**gromacs-2019.6/cmake/gmxManageNvccConfig**”，将其中几处带“-gencode;arch=compute_30,code=compute_30”的行处理掉（如将其注释掉）即可。

```
make install
```

**注意，也可不做`sudo make`直接`sudo make install`**

打开~/.bashrc 复制：

```
# For Gromacs
source /home/tmu/software/Downloads/gromacs_mpi/gromacs-2019.6/bin/GMXRC
```

保存并 `source ~/.bashrc`

### 六、测试

```
gmx_mpi --h
```



本文参考

https://chufang.cf/2019/02/02/gmx_install/

http://sobereva.com/457









