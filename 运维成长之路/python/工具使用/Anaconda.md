
改源 .condarc



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