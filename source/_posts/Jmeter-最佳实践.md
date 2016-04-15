---
title: Jmeter 最佳实践
date: 2016-04-11 15:29:38
tags: jmeter, 性能
categories: jmeter, 性能
---

其实很久之前就在用jmeter做性能测试了，最近换了新工作，接了新的性能测试任务，jmeter好久不用了，一切又要重新开始。之前有朋友说他用jmeter，并发数一直上不去，不用想就是没有做好优化，所以我就想着把之前的积累记录下来，以最少的客户端资源去测更多的并发。
<!-- more -->

# From Jmeter official user manual

Jmeter官方已经给出了最佳实践方案了，[原文链接点我](http://jmeter.apache.org/usermanual/best-practices.html)
+ 使用最新版本的Jmeter(写本文时最新稳定版本为2.13)
+ 使用合适的线程数
硬件资源(施压端)和测试计划的设计都会影响到并发的线程数，无论使用何种压测工具，如果并发数设置的不合理，都会得到错误的或者不准确的测试结果。如果需要大规模的迸发测试，可以考虑使用分布式并以NON-GUI模式运行Jmeter
+ 正确的管理cookie
+ 正确管理认证信息
+ 使用HTTP(S)录制脚本
+ 用户变量
  - 通过文本来保存用户名密码，并以逗号分隔，将文本同测试计划放在同一目录下
  - 在测试计划中添加CSV DataSet配置元素
  - 用变量名去替换取样器中的数据
+ 减少资源消耗
    - 使用non-GUI模式：`jmeter -n -t test.jmx -l test.jtl`
    - 尽量少的使用监听器；如果像上面的脚本那样使用了`-l`选项，可以删除所有的监听器
    - 不要使用`View Result Tree`和`View Result Table`，仅当你在调试脚本的时候使用他们
    - 使用循环去运行同一个`Sampler`而不是使用大量类似的`Sampler`(使用变量去区分这些samplers)
    - 不要使用`function mode`
    - 使用`CSV`而不是`xml`作为输出
    - 仅保存你需要的数据
    - 尽量少的使用`Assertion`
    - 使用最高效的脚本语言
Jmeter官方的用户手册就只有上述几条最佳实践的建议，当然，这对于我们的实际操作来说是远远不够的。

# From BlazeMeter

BlazeMeter给了一些其它额外的建议，[原文在这里](https://guide.blazemeter.com/hc/en-us/articles/207421405-JMeter-Best-Practices)

* 不要使用GUI模式去运行你的脚本
    在高压下，Jmeter GUI会消耗大量的内存资源，除非你只是在调试脚本或者做小规模的并发测试
* 限制每台客户机的线程数在300以内
    当然，这个并不是绝对的，取决于你的客户机性能。你并不用担心大规模并发，使用分布式测试，每台客户机300个线程，那台2台客户机就有600个线程了
* 禁用`View Result Tree`监听器
    这会消耗大量的内存，或者你只勾选`Errors`(仅错误日志)
    ![error](error.png)
* 禁用所有的图像展示
    这个也会消耗大量的内存，你可以通过JTLs查看所有的实时图像
* 监控日志

当然，这里的好多建议都是针对BlazeMeter提出来的，我选了一部分Jmeter也通用的。从以上可以看出，正式的压测一定要禁用所有带图像展示的监听器。

# 配置优化jmeter.properties

配置文件里明确说明不要直接修改jmeter.properties，可以创建用户自己的属性文件

```text
#                      THIS FILE SHOULD NOT BE MODIFIED
#
# This avoids having to re-apply the modifications when upgrading JMeter
# Instead only user.properties should be modified:
# 1/ copy the property you want to modify to user.properties from jmeter.properties
# 2/ Change its value there
#
```

这里我通过拷贝jmeter.properties创建一个叫user.properties的配置文件
其中可以修改的参数很多，这里的大多数参数并不需要去修改，我们拣重要的说
* Remote hosts and RMI configuration
    这个是在分布式运行的时候，把客户机的ip及端口号配置在这里
   `remote_hosts=10.0.0.158, 10.0.0.140,localhost`
   这里有几点需要注意
   1. 关闭防火墙
   2. 所有的客户端都在同一个子网内
   3. server也必须在同一子网内如果使用192.x.x.x或者10.x.x.x这样的IP地址，如果server没有使用192或者10这样的IP地址，（server同client不在同一子网内）将不会有任何问题
   4. 确保Jmeter可以访问到server
   5. 确保各系统的Jmeter版本保持一致，不同版本的Jmeter将不能很好的工作
* Results file configuration
    这个是测试结果的配置，相对来说比较重要，具体如下：

    ```
    #使用csv格式保存测试结果
    jmeter.save.saveservice.output_format=csv
    jmeter.save.saveservice.data_type=false
    jmeter.save.saveservice.label=true
    jmeter.save.saveservice.response_code=true
    jmeter.save.saveservice.response_data.on_error=false
    jmeter.save.saveservice.response_message=false
    jmeter.save.saveservice.thread_name=true
    jmeter.save.saveservice.time=true
    jmeter.save.saveservice.subresults=false
    jmeter.save.saveservice.assertions=false
    jmeter.save.saveservice.latency=true
    jmeter.save.saveservice.connect_time=true
    jmeter.save.saveservice.timestamp_format=HH:mm:ss
    jmeter.save.saveservice.default_delimiter=;
    jmeter.save.saveservice.print_field_names=true
    ```

* Summariser
    生成测试结果概要（仅non-GUI模式有效）
    ```

    summariser.interval=10
    summariser.log=true
    summariser.out=true
    ```


# Jmeter命令行参数

最后，需要了解一下通过non-GUI模式运行jmeter有哪些选项

```
    -h, --help
                print usage information and exit
　　　　　　　　　#打印帮助信息　
        -v, --version
                print the version information and exit
　　　　　　　　　 #打印版本信息
        -p, --propfile {argument}
                the jmeter property file to use
　　　　　　　　　 #运行时指定property文件，默认是使用JMETER_HOME/bin目录下的jmeter.properties，如果用户自定义有其它的配置，在这里加上
　　　　　　　　　 #用法如下： -p user.properties
        -q, --addprop {argument}
                additional property file(s)
　　　　　　　　　 #其它配置文件，如JVM参数等等
        -t, --testfile {argument}
                the jmeter test(.jmx) file to run
　　　　　　　　  #要运行的jmeter脚本
        -j, --jmeterlogfile {argument}
                the jmeter log file
　　　　　　　　  #指定记录jmeter log的文件，默认为jmeter.log
        -l, --logfile {argument}
                the file to log samples to
　　　　　　　　  #记录采样器Log的文件
        -n, --nongui
                run JMeter in nongui mode
　　　　　　　　  #以nongui模式运行jmeter
        -s, --server
                run the JMeter server
　　　　　　　　  #运行JMeter server
        -H, --proxyHost {argument}
                Set a proxy server for JMeter to use
　　　　　　　　  #代理服务器地址
        -P, --proxyPort {argument}
                Set proxy server port for JMeter to use
　　　　　　　　  #代理服务器端口
        -u, --username {argument}
                Set username for proxy server that JMeter is to use
　　　　　　　　  #代理服务器的用户名
        -a, --password {argument}
                Set password for proxy server that JMeter is to use
　　　　　　　　  #代理服务器用户名对应的密码
        -J, --jmeterproperty {argument}={value}
                Define additional JMeter properties
　　　　　　　　  #定义额外的Jmeter属性
        -G, --globalproperty (argument)[=(value)]
                Define Global properties (sent to servers)
                e.g. -Gport=123
                 or -Gglobal.properties
　　　　　　　　  #定义发送给server的全局属性
　　　　　　　　　#如：-Gport=123 或者-Gglobal.properties（指定监听server的端口）
        -D, --systemproperty {argument}={value}
                Define additional System properties
　　　　　　　　  #定义系统属性
        -S, --systemPropertyFile {filename}
                a property file to be added as System properties
　　　　　　　　　#通过指定的property文件定义系统属性
        -L, --loglevel {argument}={value}
                Define loglevel: [category=]level 
                e.g. jorphan=INFO or jmeter.util=DEBUG
　　　　　　　　  #定义日志等级
        -r, --runremote (non-GUI only)
                Start remote servers (as defined by the jmeter property remote_hosts)
　　　　　　　　  #启动远程server（在jmeter property中定义好的remote_hosts），公在non-gui模式下此参数才生效
        -R, --remotestart  server1,... (non-GUI only)
                Start these remote servers (overrides remote_hosts)
　　　　　　　　  #启动远程server（如果使用此参数，将会忽略jmeter property中定义的remote_hosts）
        -d, --homedir {argument}
                the jmeter home directory to use
                #Jmeter运行的主目录
        -X, --remoteexit
                Exit the remote servers at end of test (non-GUI)
　　　　　　　　  #测试结束时，退出（在non-gui模式下）
```

# 使用non-gui模式进行分布式测试

这个可以参考我之前写的一篇博客[Jmeter以non-gui模式进行分布式测试](http://www.cnblogs.com/qianyiliushang/p/4381639.html)
在实际使用过程中还遇到一些坑，这里也一并记录下来
* 启动slave报rmi的错误

```
Server failed to start: java.rmi.RemoteException: Cannot start. localhost.localdomain is a loopback address.
An error occurred: Cannot start. localhost.localdomain is a loopback address.
```

解决方案是在启动命令后加上`-Djava.rmi.server.hostname=192.168.16.210`或者修改`jmeter-server`，直接在启动脚本里修改`RMI_HOST_DEF=-Djava.rmi.server.hostname=192.168.16.210`

# 使用influxdb和granfa实时展示测试结果

前面的配置中，我们有讲到`Summariser`相关的配置，这个其实就是以non-gui模式运行是，控制台输出的测试结果![Summariser](summariser.png)
这一部分后面会单独写文章进行分析
