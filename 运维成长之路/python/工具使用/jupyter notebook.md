pip install jupyter notebook

anaconda 自带jupyter，不需要安装

jupyter gpu kernel 设置
conda activate pyenv
conda install ipykernel


python -m ipykernel install --user --name=gpu2

jupyter notebook --allow-root --ip=""

虚拟环境
方法一：
在虚拟环境里，安装 jupyterlab
```
(base) conda activate pyenv
(pyenv)conda install jupyterlab
(pyenv)jupyter lab
```
方法二：
使用 conda base 里的 jupyter
```
(base) conda activate pyenv
(pyenv)conda install ipykernel
(pyenv) python -m ipykernel install --user --name=pyenv
```

```text
报错
AttributeError: partially initialized module 'charset_normalizer' has no attribute 'md__mypyc' (most likely due to a circular import)
安装
pip install cchardet
```
