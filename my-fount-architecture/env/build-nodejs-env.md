# 构建nodejs基础环境


### 安装nvm管理nodejs的版本


```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.29.0/install.sh | bash
```

> 查看 bashrc环境

```
cat ~/.bashrc|grep nvm
```

> 执行 bashrc环境可用

```
source ~/.bashrc
```

> 查看　nvm　版本
```
nvm --version
```
### 安装nodejs 8.11.1

```
nvm install 8.11.1
```
执行后如下显示
```
Downloading https://nodejs.org/dist/v8.11.1/node-v8.11.1-linux-x64.tar.xz...
######################################################################## 100.0%
WARNING: checksums are currently disabled for node.js v4.0 and later
Now using node v8.11.1 (npm v5.6.0)
```

### 注意事项 

#### npm warn package.json @1.0.0 no repository field.
看字面意思大概是package.json里缺少repository字段，也就是说缺少项目的仓库字段
```
{
    ...
    "repository": {
        "type": "git",
        "url": "http://baidu.com"
    },
    ...
}
```
但作为测试项目或者练习用，只需在package.json里面做如下配置即可:
```
{
    ...
    "private": true,
    ...
}
```
以这种方式把项目声明为私有。