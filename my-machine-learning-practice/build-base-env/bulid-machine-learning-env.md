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



conda create -n py36 python=3.6

source activate py36
pip install ipykernel
python -m ipykernel install --name py36
source deactivate py36
使用jupyter kernelspec list查看安装的内核和位置

python -m ipykernel install --user --name py36


```

> 安装 ipykernel 内核

```
pip install ipykernel
```

* 出现如下情况，原因是默认的IPython5.x支持python2.7
  ```
   IPython 6.0+ does not support Python 2.6, 2.7, 3.0, 3.1, or 3.2.     
   When using Python 2.7, please install IPython 5.x LTS Long Term Support version.
   python2.7 ipython Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-Vxrpi9/ipython/
  ```
* 解决方案
  ```
  # 下载ipython5.4
  wget https://pypi.python.org/packages/f7/48/5702699caf20208d61a92157c01d1eb281093e3e02e9bcd4b5031ccea6a1/ipython-5.4.1-py2-none-any.whl#md5=2b801f50b5e82a3fabca42b661568bf5
  wangpeng@ubuntu:~$ sudo pip install ipython-5.4.1-py2-none-any.whl
  # 安装ipython5.4
  pip install ipython-5.4.1-py2-none-any.whl
  # 安装 ipykernel
  pip install ipykernel
  ```

##### 安装ipykernel到user

```
python -m ipykernel install --user
```

成功提示信息

```
Installed kernelspec python2 in /home/incps/.local/share/jupyter/kernels/python2
```

### 

### 

### 4.3 运行 jupyter notebook

```
incps@incpshome:~/jupyterhome$ jupyter notebook
```



