# 构建容器之jenkins环境

### 安装jenkins

启动一个jenkins容器

docker run -d --name jenkins -p 18081:8080 -v /home/incps/devsoft/dockerWork/jenkins\_home:/home/jenkins\_home jenkins

查看jenkins服务   docker ps \| grep jenkins

启动服务端 。localhost:8081/jenkins

去容器内部找密码

进入容器：docker exec -it jenkins bash

执行：cat /var/jenkins\_home/secrets/initialAdminPassword  红框即为jenkins初始密码

输入密码之后，重启docker镜像  docker restart   {CONTAINER ID}

### 配置Jenkins

参见　[https://blog.csdn.net/m0\_37106742/article/details/78815753](https://blog.csdn.net/m0_37106742/article/details/78815753)



打开浏览器，最好使用谷歌，或者火狐。在地址栏输入http://IP:8080/jenkins

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170601153503805-1152040019.png)

这里需要输入密码，密码按照提示寻找，正常情况下应该是在：/root/.jenkins/secrets/initialAdminPassword

输完密码点击 Continue 按钮进行下一步![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170601154122930-79347306.png)

左边是默认安装，右边是自定义安装，本地实验不需要特殊需求，所以选择默认安装了。

附张正在装的图，页面还是蛮精美的

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170601154429446-1590176109.png)

如果无法安装，就是你的网络有问题，自己先把网络搞定了在说吧！

安装完成后输入自己的用户名，密码等个人信息。然后Save and Finish一下，之后就可以启动Jenkins了

![](http://images2015.cnblogs.com/blog/1131126/201706/1131126-20170601155929524-305250595.png)

　　至此Jenkins 安装完成。

