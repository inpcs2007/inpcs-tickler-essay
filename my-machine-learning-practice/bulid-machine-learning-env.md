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



