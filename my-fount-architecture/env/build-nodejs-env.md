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

> 配置镜像仓库

```
npm config set registry https://registry.npm.taobao.org
```

验证镜像仓库命令
```
npm config get registry
```

### 注意事项 

> 1. npm warn package.json @1.0.0 no repository field.

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

> 2. npm WARN saveError ENOENT: no such file or directory, open '/home/incps/package.json'

项目目录下没有package.json这个文件。可以使用npm init -f命令生成一下，至于生成的package.json中缺少的字段你可以参照其他的模块的package.json文件填进去。至于package.json中的每个字段的值可以为“”。至于依赖项字段，以安装模块的时候使用-save参数就会自动写到文件中。

```
npm init -f
```