# Docker创建JIRA 7.2.7中文破解版

## 1. 介绍

JIRA是Atlassian公司出品的项目与事务跟踪工具，被广泛应用于缺陷跟踪、客户服务、需求收集、流程审批、任务跟踪、项目跟踪和敏捷管理等工作领域。  
  JIRA中配置灵活、功能全面、部署简单、扩展丰富，其超过150项特性得到了全球115个国家超过19,000家客户的认可。  
  官网地址[https://www.atlassian.com/](https://www.atlassian.com/)

## 2. 安装docker版本的Jira7.2.7破解版

### 2.1 创建MySQL 5.6的容器

```
sudo docker run -d \
    --name=mysql-db \
    --hostname=mysql-db \
    -p 20010:3306 \
    -e MYSQL_ROOT_PASSWORD=123456 \
    -e MYSQL_DATABASE=jira \
    -e MYSQL_USER=jira \
    -e MYSQL_PASSWORD=jira \
    idoall/mysql:5.6
```

### 2.1 创建JIRA 7.2.7容器

```
sudo docker run -d \
    --name jira \
    --hostname jira \
    --link mysql-db:mysql \
    -p 20011:8085 \
    -p 20012:8080 \
    -p 20013:8443 \
    -p 20014:8090 \
    -p 20015:22 \
    idoall/ubuntu16.04-jira:7.2.7
```

## 3. 配置JIRA

### 3.1 进入Jira首页

```
http://localhost:20012
```

### 3.2 选择JIRA的安装选项

> 选择 I‘ll set it up mysqlf

![](/my-practice-architect-roadmap/build-jira-soft/jira-01.png)

### 3.2 选择数据库的安装选项

> 选择 My Own Database选项，并填写如下信息内容

![](/my-practice-architect-roadmap/build-jira-soft/jira-02.png)

> 然后执行Test Connection

![](/my-practice-architect-roadmap/build-jira-soft/jira-03.png)

> 测试数据库连接上后，点击Next，开始在数据库创建jira需要的表。

![](/my-practice-architect-roadmap/build-jira-soft/jira-04.png)

### 3.3 Set up application properties

输入你的Application Title，选择Mode，输入Base URL，点击Next

> Application Title 建议不要输入中文，我这里测试出错  
> Mode中，我们在此使用的是Private模式，在这个模式下，用户的创建需要由管理员创建。而在Public模式下，用户是可以自己进行注册

![](/my-practice-architect-roadmap/build-jira-soft/jira-05.png)

### 3.4 Specify your license key

> 在申请New Evaluation License的时候，选择如下，然后点击Generate License，会申请个临时30天的许可证

![](/my-practice-architect-roadmap/build-jira-soft/jira-06.png)

> 申请完毕后点击【Yes】，将会迁移到本地安装的Jira页面中

![](/my-practice-architect-roadmap/build-jira-soft/jira-07.png)

> 点击下一步，进行管理员账户的设置

![](/my-practice-architect-roadmap/build-jira-soft/jira-08.png)

### 3.5 Set up administrator account

输入你的全称：root、邮件：xxx@xxx.com、用户名：root、密码：123456，点击Next

![](/my-practice-architect-roadmap/build-jira-soft/jira-9.png)

> 如果出现错误，重新执行4.1

### 3.6 Set up email notifications

稍后对邮件服务器进行配置

![](/my-practice-architect-roadmap/build-jira-soft/jira-10.png)

### 3.7 Welcome to JIRA!

![](/my-practice-architect-roadmap/build-jira-soft/jira-11.png)

## 4. 创建一个新的工程


### 3.1 Create a new project


### 2.1 dockerFile位置

github地址为 [https://github.com/idoall/docker/blob/master/\_del\_jira/README.md.bak](https://github.com/idoall/docker/blob/master/_del_jira/README.md.bak)

