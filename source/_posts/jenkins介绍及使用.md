---
title: jenkins介绍及使用
date: 2018-04-19 14:47:04
tags: 持续集成,CI,自动化
categories: CI,持续集成
---

本文将简单介绍持续集成，jenkins在持续集成中的作用，从jenkins的安装、配置介绍jenkins的基本使用，并用一个java web项目演示demo

<!-- more -->

## 什么是jenkins

> Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

简单来说就是一个开源的自动化服务器，用来自动化运行编译、测试、交付或者部署软件的各种任务

## 持续集成

在讲jenkins之前，先了解一下什么是持续集成(Continuous Integration)

> 持续集成是一种软件开发实践，即团队开发成员经常集成他们的工作，通过每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试）来验证，从而尽早地发现集成错误。

* 传统软件开发，整合过程通常在每个人完成工作之后、在项目结束阶段进行。

![traditional software dev & deliver](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/cd-oldway.png)

* 持续集成&持续交付

![CI&CD](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/ci-cd.png)

* Devops

![devops](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/devops-newway.png)

## Jenkins能做什么

**Automation everything.**

由于jenkins是基于java开发的开源软件，所以在各平台都可以使用。所有可以通过命令行完成的任务(编译、构建、自动化测试、部署)都可以使用jenkins来做，再加上丰富的触发策略(手动、定时、hook)和插件系统，可以很好的扮演CI Server的角色。

## Jenkins 安装

硬件要求如下

> * 256M内存
> * 1G的磁盘空间(如果用docker运行的话建议最小磁盘为10G)
>
> 以上是最低要求，推荐的配置为：
>
> * 1G以上的内存
> * 50G以上的磁盘空间

软件要求 ：Java8 - JRE或者JDK都可以

* 安装jdk

  > 去[Oracle官网](http://www.oracle.com/technetwork/java/javase/overview/index.html)下载对应操作系统的java版本(建议java8版本)，此处以通用的Linux版本为例
  >
  > 解压`tar zxvfjdk-8u101-linux-x64.tar.gz `
  >
  > 设置环境变量

  ```shell
  #编辑/etc/profile文件，在文件结尾处添加
  export JAVA_HOME=/home/apps/jdk1.8.0_101
  export PATH=$PATH:$JAVA_HOME/bin
  #编辑完成后运行以下命令使配置生效
  source /etc/profile
  #然后执行
  java -version
  #看到下面的信息就说明java安装成功
  #java version "1.8.0_101"
  #Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
  #Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
  ```

  下载运行jenkins

  jenkins主要有两种启动方式(当然也可以通过docker去运行，不在此文讨论范围内)，下面会分别介绍。

  [下载链接&官网](https://jenkins.io/)

  *TIPS：在运行jenkins前最好先设置JENKINS_HOME环境变量，默认目录为`~/.jenkins`避免后期因为磁盘空间问题做jenkins主目录迁移*

  - 使用内嵌的servlet(netty)独立运行

    ```shell
    #默认端口为8080，为避免端口冲突，在启动时可以手动指定
    java -jar jenkins.war --httpPort=8888
    ```

    启动后，记住控制台里的密码(见下图)，后面初始化jenkins需要用到。如果没记下来可以去`$JENKINS_HOME/secrets/initialAdminPassword`里找到

    ![jenkins init password](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/jenkins-init-password.png)

    然后用浏览器访问jenkins`yourhost:yourport`我这里是`http://192.168.1.69:8888/`

  - 使用tomcat运行jenkins

    1. [下载tomcat](http://tomcat.apache.org/download-80.cgi#8.0.51)
    2. 解压tomcat，并把jenkins.war拷贝到`tomcat_home/webapps/`目录下
    3. 进入`tomcat_home/bin`目录，运行`sh startup.sh`启动tomcat

    **TIPS**

    > 如果启动失败，检查一下bin目录下的所有.sh文件是否有可执行文件，如果没有，手动添加权限`chmod +x ./*.sh`
    >
    > Tomcat默认端口也是8080，如果有端口冲突，修改`tomcat_home/conf/server.xml`找到

    ```xml
    <Connector port="8080" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
    ```

    > 将8080改成要使用的新端口，如8888

    4. 用浏览器访问`yourhost:port/jenkins`即可

## 初始化jenkins

不论用上面哪种方式运行jenkins，打开页面后都是需要输入管理员密码以解锁jenkins，按照提示输入即可

![jenkins init password](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/jenkins-password.png)

* 安装插件

选择推荐的插件即可，其它要用的插件可以后续再单独安装

![install plugins](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/WX20180503-105913@2x.png)

* 设置管理员账号

  按照页面提示输入即可



## 创建任务

一般都是选择构建自由风格的项目，此处以编译java web项目为例

* 输入项目名称后进入项目配置

* 源码配置

  jenkins默认支持svn和git，这里使用git来管理源码，一般建议使用ssh协议访问git，此处演示使用http协议

  ![jenkins-git-config](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/project-git-config.png)

  在Repository URL处填写项目的git仓库地址

  Credentials处选择git服务器的凭证，如果没有就点击add添加即可

  Branches to build选择要构建的分支，默认使用master分支

* 构建

  点击添加构建步骤，选择`Execute shell `，接着输入以下命令`gradle clean build`然后点击保存

> 其它参数都没有配置不代码不重要，比如可以通过参数化构建来动态选择分支、根据实际情况选择不同的构建策略、构建环境初始化等

## 执行构建

找到刚才创建的项目，点击`立即构建`看看会发生什么。

项目执行后发现前面有个红色的圆点，说明此次构建失败，点击那个红色的圆圈看看

![jenkins-console-log](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/jenkins-console-log.png)

可以看到执行命令的时候发现gradle命令找不到，下面来解决这个问题

* [下载gradle](https://gradle.org/)

* 在jenkins服务器环境变量里加上gradle(参考java配置)

* 检查gradle安装成功`gradle -version`

  ```shell
  Welcome to Gradle 4.7!

  Here are the highlights of this release:
   - Incremental annotation processing
   - JDK 10 support
   - Grouped non-interactive console logs
   - Failed tests are re-run first for quicker feedback

  For more details see https://docs.gradle.org/4.7/release-notes.html


  ------------------------------------------------------------
  Gradle 4.7
  ------------------------------------------------------------

  Build time:   2018-04-18 09:09:12 UTC
  Revision:     b9a962bf70638332300e7f810689cb2febbd4a6c

  Groovy:       2.4.12
  Ant:          Apache Ant(TM) version 1.9.9 compiled on February 2 2017
  JVM:          1.8.0_101 (Oracle Corporation 25.101-b13)
  OS:           Linux 4.4.0-87-generic amd64
  ```

* 回到jeknins，进入系统管理-全局工具配置-Gradle，不要点自动安装，手动填写GRADLE_HOME即可

  ![jenkins-gradle](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/jenkins-gradle.png)

* 然后进入系统管理-系统设置-全局属性-Tool Locations-添加，如下图

  ​	![jenkins-tool-locations](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/jenkins-tool-location.png)

配置成功后，再重新编译一下刚才失败的job，会发现刚才的红点变成蓝色，说明构建成功。

## 归档构建产出

上面构建执行完毕后，如何获取我们要的构建产出物(artifact)呢

点击刚才创建的demo项目，进入配置页，在**构建后操作**下点击`增加构建后操作步骤`然后选择`Arhcive the artifacts`，用于存档的文件这里填上要归档的文件路径或者正则表达式，如`**/*.war`(表示把所有目录下以.war结尾的文件都归档)，然后再重新构建这个job,构建成功后可以看到页面上有显示最后一次成功构建结果

![jenkins-archive](http://o86koa12m.bkt.clouddn.com/image/blog/jenkins/jenkins_archive.png)

## 理解JEKINS_HOME

JENKINS_HOME下的主要目录如下

```shell
|-- fingerprints
|-- jobs
|-- logs
|-- nodes
|-- plugins
|-- secrets
|-- updates
|-- userContent
|-- users
|-- war
|-- workflow-libs
`-- workspace
```



其中我们主要关注的两个目录分别是`jobs`和`workspace`

```shell
./jobs/
`-- jenkins_test
    |-- builds
    |   |-- 1
    |   |-- 10
    |   |-- 2
    |   |-- 3
    |   |-- 4
    |   |-- 5
    |   |-- 6
    |   |-- 7
    |   |-- 8
    |   |-- 9
    |   |-- lastFailedBuild -> 8
    |   |-- lastStableBuild -> 10
    |   |-- lastSuccessfulBuild -> 10
    |   `-- lastUnsuccessfulBuild -> 8
    |-- lastStable -> builds/lastStableBuild
    `-- lastSuccessful -> builds/lastSuccessfulBuild
```

job目录列出了在jenkins上创建的所有任务，同时每个任务下builds目录记录了所有的构建记录、最后一次失败、成功、稳定的构建

```shell
./workspace/
|-- jenkins_test
|   |-- build
|   |-- db
|   |-- gradle
|   |-- protos
|   `-- src
`-- jenkins_test@tmp
```

而`workspace`目录下则是所有项目的工作(构建)空间，包括从git上拉下来的源码、构建产出物等都在对应的job目录里。

大家注意一下上面有一个jekins_test@tmp目录，这个是当同一个job并发构建时所使用的工作空间

## 其它

Jenkins仅是一个CI平台，上面只是以构建web应用为例，其它类型的项目也都可以完成，比如构建安卓项目(需要服务器上安装android sdk)，构建iOS项目(需要jenkins部署在MacOS上或者添加MacOS为slave节点)，构建FED前端项目(需要nodejs)等等，具体job的构建过程都有所不同，有的一行命令就可以搞定，有的可能需要写shell脚本或者其它脚本来辅助完成。

对于构建后的操作也不仅仅局限于归档，比如可以把移动端项目的产出物上传的蒲公英或者其它移动应用分发平台并显示二维码，以方便测试或者分发，还可以把构建产出物部署或者更新到其它服务器上(发布)，或者把artifact上传到nexus、dockerhub等等。

当然，jenkins也不是自动化构建的唯一选择，类型的产品还有jetbrain公司的[team city](https://www.jetbrains.com/teamcity/)、[travis ci](https://travis-ci.org/)、[gitlab ci](https://about.gitlab.com/)等等。

随着jenkins2.0的发布，pipeline和blueocean是jenkins未来一个阶段的主要功能，后续再写专题分别介绍pipelince和blueocean