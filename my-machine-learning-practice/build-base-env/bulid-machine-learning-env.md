# 构建机器学习基础环境

## 1.安装pip3

```bash
sudo apt-get install python3-pip
```

## 2. pip安装虚拟环境

```bash
sudo pip install virtualenv
```

## 3. 虚拟环境分别配置python2和python3环境

### 3.1 配置python2环境

```bash
# 安装配置环境
sudo virtualenv -p /usr/bin/python2.7 py2env
# 执行环境
sudo source ./py2env/bin/activate
```

### 3.2 配置python3.5环境

```bash
# 安装配置环境
sudo virtualenv -p /usr/bin/python3.5 py3env
# 执行环境
sudo source ./py3env/bin/activate
```

## 4.Anaconda环境安装及配置

### 4.1下载anaconda

```
https://www.anaconda.com/download/
```

### 4.2 安装anaconda

在linux下的命令行安装anaconda的时候，注意要选择添加环境变量，要不然无法在命令行中使用conda命令，参见下图

![](/my-machine-learning-practice/build-base-env/anaconda-steup.png)

#### 4.2.1使用conda创建jupyter notebook的 虚拟环境核心

> 创建虚拟环境

```
# 创建虚拟环境
conda create -n mypy27 python=2.7
# 开启虚拟环境
source activate mypy27
# 关闭虚拟环境
source deactivate mypy27
```

```
IPython 6.0+ does not support Python 2.6, 2.7, 3.0, 3.1, or 3.2.     
When using Python 2.7, please install IPython 5.x LTS Long Term Support version.
python2.7 ipython Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-Vxrpi9/ipython/
```

```
pip install ipython-5.4.1-py2-none-any.whl

pip install ipykernel
```

### 

### 

### 4.3 运行 jupyter notebook

```
incps@incpshome:~/jupyterhome$ jupyter notebook
```



