---
title: 使用jenkins自动打包iOS
date: 2016-03-30 09:16:35
tags: 持续集成, jenkins
categories: 测试, 持续集成
---

公司里android自动打包，以前渠道包自动打包都OK了，今天抽出时间弄一下iOS的，之前没做过iOS的，随手记录下来以备后续查看。
整个过程下来，感觉，用jenkins还是shell比较靠谱，使用插件各种问题。感觉这个任务完成后，自己的sed及正则水平提升了不少，同时，iOS打包过程中的各种坑都摸了一遍。
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

## 导出ipa包

```
xcodebuild -exportArchive -archivePath build/imoffice.xcarchive -exportPath build/imoffice_$build.ipa -exportFormat ipa -exportProvisioningProfile "imo_push_adhoc"

```

这一步就是把上一步生成的archive打包成ipa出来。

---
其实通过xcodebuild打包挺简单，比起jenkins xcode integeration插件来省事很多。

当然，坑也是遇到过不少的，后文后列出遇到的那些个坑。由于是公司项目，就不把完整的打包脚本列出来了，有需要的可以联系我。

下面是打包成功的截图，可以看到，失败了N多次之后才成功，也把后文说到的坑都填平了。
![jenkins](success.png)

# 安装xcode插件
jenkins的安装什么的很简单，不再多说，要想打包ipa,XCODE插件必不可少，在jenkins里这个插件的全名叫`Xcode integration`

好吧，实际上我用jenkins的插件打包时，一直没有成功，后面空下来继续完善这一部分，以及这个过程中遇到的坑。

# 构建过程中遇到的坑
* 不小心把开发者证书给回收掉了，当时iOS开发就炸开锅了，我到现在还没想起来我什么时候做过类似的操作。
* OSX下的sed命令，如果用`-i`去直接修改文件，会报错，OSX强制要求备份原文件，类似于 `sed -i '' "s/pattern/newPattern/" `这种，如果不备份就直接用''，可是，可是，我用这命令在osx下的命令行是OK的，jenkins中执行的时候，继续报错，那么只好用sed -e 先把结果缓存，然后再替换文件，删除文件。。。
* sed的正则表达式不是标准的正则，这个坑的我最苦，多少次我都以为是我的正则写的有问题，sed里除了.和* 其它符号均需要转义，比如说下面这个`[0-9]{6,}\([a-z]{4}[0-9]{1,}\)+`  这个在标准的正则里是没问题的，在sed里，()和{}都是不需要转义的，所以上面的正则在sed里应该这样写`[0-9]\{6,\}([a-z]\{4\}[0-9]\{1,\})\+`
* jenkins xcode integeration插件，这个也很坑，没有成功的打出来过包，后续继续研究
* 还是证书问题，明明我在命令行里可以成功的打包了，但是在jenkins里，却是不行的，因为，默认的，命令行下是当前的登陆用户，所使用的lib或者证书等等都在~/Library/下，而jenkins使用的是/Library/下的lib或者证书，所以此时的解决方案就是把用户目录下相关证书拷贝到根目录下`cp -R  ~/Library/MobileDevice/Provisioning\ Profiles/ /Library/MobileDevice/Provisioning\ Profiles/`

# Final Script

由于开发那边对源代码管理或者分支管理的不规范，描述文件经常更换，打包配置经常变，inhouse,adhoc都在主干分支上，真心的无力吐槽。想着要么就把这个打包过程写成可配置的吧。机缘巧合，在github上看到一个哥们写的脚本，拿过来用了一下，打出的ipa包安装时报duplicateIdentifier错误，于是就对之进行改进，并提交了一个pull request，也被原作者接受了,[github地址](https://github.com/qianyiliushang/BGIpaBuilder)，具体用法上面也有介绍，有问题联系我。

# 题外话

其实我还想了一个歪招，就是用xcode打包如果能成功的话，那么可以把这个配置文件拷贝出来，在用jenkins打包的时候，每次使用脚本去替换checkout出来的配置文件，这样一般来说是没有问题的。
另外，关于版本号以及环境的切换写简单的脚本就可以搞定了，这个是每个项目都不同的，难以写一个通用的脚本。
最后，可能在打包前unlock keychain,这里我写了一个脚本，希望能有所帮助

```shell
#!/bin/bash
keychain=~/Library/Keychains/login.keychain
#把下面这个换成你自己的
sign="iPhone Distribution: XXXXXXXXXX"
appName=imoffice

#mac用户密码
password="yourpassword"
security unlock-keychain -p "$password" "$keychain"
```

# 附1：SED命令与选项
附上sed相关的命令及选项，以后再写的时候就不用网上查了。
## SED命令


|   命令        | 功能           | 
|:--------------:|:-------------:| 
| a\     | 在当前行后添加一行或多行。多行时除最后一行外，每行末尾需用“\”续行 |
| c\     | 用此符号后的新文本替换当前行中的文本。多行时除最后一行外，每行末尾需用"\"续行      | 
| i\ | 在当前行之前插入文本。多行时除最后一行外，每行末尾需用"\"续行      | 
| d | 删除行|
|h  |   把模式空间里的内容复制到暂存缓冲区|
|H  |   把模式空间里的内容追加到暂存缓冲区|
|g  |   把暂存缓冲区里的内容复制到模式空间，覆盖原有的内容|
|l  |   列出非打印字符|
|p  |   打印行|
|n  |   读入下一输入行，并从下一条命令而不是第一条命令开始对其的处理|
|q  |   结束或退出sed|
|r  |    从文件中读取输入行|
|!  |   对所选行以外的所有行应用命令|
|s  |   用一个字符串替换另一个|
|g  |   在行内进行全局替换|
|w  |   将所选的行写入文件|
|x  |   交换暂存缓冲区与模式空间的内容|
|y  |   将字符替换为另一字符（不能对正则表达式使用y命令）|

## SED选项

|   选项  |   功能  |
|:-------:|:-------:|
|-e       | 进行多项编辑，即对输入行应用多条sed命令时使用|
|-n       | 取消默认的输出|
|-f       | 指定sed脚本的文件名|
|-i       | 表示将转换结果直接插入文件中（若不用-i，一般sed命令不会改变原文档里的内容，而只会输出到命令行。|
|-r       | 支持扩展正则+ ? () {}  |

看到上面的-r选项支持扩展的正则，我试了下，好像还是有点问题，后续再继续总结

# 附2：sed正则表达式与元字符

| 元字符   |   功能  |   示例  |
|:---------:|:------:|:------:|
|   ^   | 行首定位符 | /^my/  匹配所有以my开头的行|
|   $   | 行尾定位符 |  /my$/  匹配所有以my结尾的行|
|   .   |匹配除换行符以外的单个字符|/m..y/  匹配包含字母m，后跟两个任意字符，再跟字母y的行|
|   *   |匹配零个或多个前导字符|/my*/  匹配包含字母m,后跟零个或多个y字母的行|
|   []  |匹配指定字符组内的任一字符|/[Mm]y/  匹配包含My或my的行|
|  [^]  |匹配不在指定字符组内的任一字符|/[^Mm]y/  匹配包含y，但y之前的那个字符不是M或m的行|
| \\(..\\)|保存已匹配的字符|1,20s/\\(you\\)self/\1r/  标记元字符之间的模式，并将其保存为标签1，之后可以使用\1来引用它。最多可以定义9个标签，从左边开始编号，最左边的是第一个。此例中，对第1到第20行进行处理，you被保存为标签1，如果发现youself，则替换为your。|
|   &   |保存查找串以便在替换串中引用|s/my/##&##/  符号&代表查找串。my将被替换为##my##|
|   \\<  |词首定位符| /\<my/  匹配包含以my开头的单词的行|
|   \\>  |词尾定位符|/my\\>/  匹配包含以my结尾的单词的行|
|x\\{m\\}  |连续m个x|/9\\{5\\}/ 匹配包含连续5个9的行|
|x\\{m,\\} |至少m个x|/9\\{5,\\}/  匹配包含至少连续5个9的行|
|x\\{m,n\\}|至少m个，但不超过n个x| /9\\{5,7\\}/  匹配包含连续5到7个9的行|

最后，吐槽一下markdown的table，真的是反人类啊。

# 参考文档

* [SED单行脚本快速参考](http://sed.sourceforge.net/sed1line_zh-CN.html)
* [Sed 命令详解 正则表达式元字符](http://www.cnblogs.com/leaftime/p/3270257.html)
* [Building from the Command Line with Xcode FAQ](https://developer.apple.com/library/mac/technotes/tn2339/_index.html#//apple_ref/doc/uid/DTS40014588)


