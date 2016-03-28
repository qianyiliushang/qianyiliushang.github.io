---
title: 使用jenkins自动打包iOS
date: 2016-03-25 09:16:35
tags: 持续集成
categories: 测试
---

公司里android自动打包，以前渠道包自动打包都OK了，今天抽出时间弄一下iOS的，之前没做过iOS的，随手记录下来以备后续查看。
整个过程大概包括xcodebuild命令、jenins中构建ipa、发布ipa到浦公英这几大步。
<!-- more -->

# xcodebuild打包

xcodebuild 命令是 Xcode Command Line Tools 的一部分。通过调用这个命令，可以完成 iOS 工程的编译，打包和签名过程。这个命令随着 Xcode 的版本不同使用方法上也会有所不同。打开命令行，调用以下命令查看使用方法`xcodebuild --help` ![xcodehelp](xcodehelp.png)
可以看到，xcodebuild的命令还是很多的，这里不一一讲解，有需要请移步[官方文档](https://developer.apple.com/library/mac/technotes/tn2339/_index.html#//apple_ref/doc/uid/DTS40014588)

## Archive工程
首先需要了解的是，archive 工程后，实际上我们是把整个工程编译，然后签名，变成了一个后缀名为 xcarchive 的文件。通过调用以下命令，我们将整个工程编译，签名，然后将生成的 xcarchive 文件放到工程根路径下的 build 文件夹里。
`xcodebuild -scheme imoffice -archivePath build/imoffice.xcarchive archive`<br>
其中`-scheme imoffice`其实就是工程名，根据各自的情况调整。
    
>以下命令可以列出项目的所有信息<br>
`xcodebuild -list -project <your_project_name>.xcodeproj`<br>
如下，我的工程执行以上命令
![projectinfo](projectinfo.png)
可以看到，最下面，列出了本项目的scheme




# 安装xcode插件
jenkins的安装什么的很简单，不再多说，要想打包ipa,XCODE插件必不可少，在jenkins里这个插件的全名叫`Xcode integration`

