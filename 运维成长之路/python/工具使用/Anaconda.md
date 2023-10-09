
改源 .condarc
conda config --set show_channel_urls yes 会自动生成.condarc 在主目录下


查看配置
conda info

win10 的路径引用有最长限制，修改注册表
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem
```
LongPathsEnabled 改为 1

创建虚拟环境
conda create -n wav2lip python=3.6
conda env list
conda activate xxx


pytorch 使用
CUDA 分两种，驱动API 和运行API，驱动API 指的是显卡驱动支持的最高 cuda 版本
nvidia-smi # 右上角显示的（驱动API）

CUDA nvidia 下载
https://developer.download.nvidia.com/compute/cuda/11.7.0/local_installers/cuda_11.7.0_516.01_windows.exe

nvcc -V 是anaconda 带的 CUDA 版本


conda 安装与 pip 安装包的区别
来源不同：conda 从 anaconda 的 repo 里安装，pip 从 pypi 安装
支持种类不同： conda 只支持二进制文件，pip 支持源码安装，即 python setup.py install
```
\AppData\Roaming\pip


[download.pytorch.org/whl/torch/](https://download.pytorch.org/whl/torch/)

离线安装 pytorch

下载好,pip install
```

指定目录安装虚拟环境
conda create --prefix=D:\envs\py1.2 python=3.7