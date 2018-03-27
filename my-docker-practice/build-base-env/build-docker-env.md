# 构建docker的基础环境

在Ubuntu14.04为基础系统环境构建docker-ce的基础环境，根据具体步骤构建出本文。

### Ubuntu14.04下构建docker-ce环境

#### 移除原有的docker
sudo apt-get remove docker docker-engine docker.io

## 安装软件
sudo apt-get update  
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common    

## 添加官方PGP秘钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -  


## 添加apt安装配置
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"  

## 安装
sudo apt-get update  
sudo apt-get install docker-ce  

## 使用docker时取消sudo命令
sudo gpasswd -a username docker  
sudo service docker restart 
