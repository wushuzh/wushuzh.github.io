+++
date = "2017-11-24T11:37:35+08:00"
title = "Java 开发环境"
showonlyimage = false
image = "/img/blog/setup-java-dev-env/dev.png"
draft = false
weight = 110
+++

设定 JDK 和 Intellij 集成开发环境
<!--more-->

### JDK 和 IDE

{{< highlight console >}}
$ sudo pacman -S jdk8-openjdk
$ source /etc/profile
# put this into ~/.bashrc and source
$ export JAVA_HOME=/usr/lib/jvm/default
$ archlinux-java status
Available Java environments:
  java-8-openjdk (default)

$ cower -u intellij-idea-ce

$ sudo pacman -S gradle

{{< /highlight >}}

启动初始化设置，安装常用插件，新建或导入 maven/gradle 项目。设定项目的 SDK 为 /usr/lib/jvm/default-runtime 即可。 

- [IdeaVim](http://plugins.jetbrains.com/plugin/164) 

### CA

有时候开发需要向本地/内网的私有仓库下载或是提交软件模块，这就需要在开发环境中导入内部自签名的证书。考虑到各种软件和不同操作系统为支持私有证书各不相同的导入配置操作、证书维护和未来的培训成本，这可能是一件得不偿失的事情。

但工程师的主要任务是尽快搭建其开发环境，比如 Archlinux 使用 ca-certificates 将自有 crt 格式证书放在指定目录，运行指令即可。


{{< highlight console >}}

$ keytool -list -keystore ${JAVA_HOME}/jre/lib/security/cacerts
...
Your keystore contains 140 entries
...
$ sudo mv any.crt /etc/ca-certificates/trust-source/anchors/
$ sudo trust extract-compat
$ keytool -list -keystore ${JAVA_HOME}/jre/lib/security/cacerts
...
Your keystore contains 143 entries
...

{{< /highlight >}}

针对用户申请材料，CA 为其制作证书。同时它也会广而告之“证书失效列表” (CRL)。证书

### Tomcat

安装并配置 tomcat 运行管理权限，之后才能通过脚本/IDE 管理其容器内的各个应用。

{{< highlight console >}}
$ sudo pacman -S tomcat8
$ sudo vi /etc/tomcat8/tomcat-users.xml
# uncomment and modify role and user section

$ sudo gpasswd -a wushuzh tomcat8

{{< /highlight >}}

> 将当前用户加入 tomcat8 用户组，以便可以读取配置文件。要重新启动 IDE 登录让新用户组生效

下面的设定开放了如下角色: 运行 tomcat 脚本管理 manager-script，建立两个用户，并设定好他们的密码。

{{< highlight xml >}}
<role rolename="tomcat"/>
<role rolename="manager-script"/>
<user username="tomcat" password="[CHANGE_ME]" roles="tomcat"/>
<user username="manager" password="[CHANGE_ME]" roles="tomcat,manager-script"/>
{{< /highlight >}}

配置完毕，尝试首次启动 tomcat8 服务，并使用浏览器查看 localhost:8080 ，应该看到的是花花绿绿的恭喜信息。
{{< highlight console >}}
$ systemctl status tomcat8
$ sudo systemctl start tomcat8
# check with chrome with localhost:8080
$ sudo systemctl stop tomcat8

{{< /highlight >}}

查看 catalina.sh 的运行选项支持 Java Platform Debugger Architecture (JPDA)，在 Intellij 中配置

TODO: 查看 Intellij 的两个 tomcat 插件

### Docker

参考文档

> - Archlinux wiki 
    - [OpenSSL](https://wiki.archlinux.org/index.php/OpenSSL)
    - [Tomcat](https://wiki.archlinux.org/index.php/Tomcat)
> - Jan Steffens (2014-12-11) [ca-certificates update](https://www.archlinux.org/news/ca-certificates-update/)
> - Apache Tomcat 8 [Manager App HOW-TO](http://tomcat.apache.org/tomcat-8.0-doc/manager-howto.html#Configuring_Manager_Application_Access)
> - Olaf Kock (2015-05-01) [Whats the difference between service tomcat start/stop and ./catalina.sh run/stop](https://stackoverflow.com/a/29992541/4393386)
> - markiewb (2017-04-06) [IDEA: Build your own Tomcat integration for the free IntelliJ IDEA Community Edition](https://benkiew.wordpress.com/2017/06/04/idea-build-your-own-tomcat-integration-for-the-free-intellij-idea-community-edition/)
> - [Deploying a web app into an app server container](https://www.jetbrains.com/help/idea/2017.2/deploying-a-web-app-into-an-app-server-container.html)

封面图片来自 [Java Developer Joke](https://dribbble.com/shots/3530278-Java-Developer-Joke) <a href="https://dribbble.com/browserling"><i class="fa fa-dribbble" aria-hidden="true"></i> Browserling</a>
