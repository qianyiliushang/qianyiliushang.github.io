---
title: Gradle Basic
date: 2016-03-17 10:44:26
tags: gradle
categories: gradle
---


# 为什么会有这样一篇博客
* 其实这几年工作中，项目构建包括依赖管理，用的最多的还是maven，最近参与了一个新项目，也在github上找了些想关的工具，发现在很多项目(java)都仅使用gradle构建项目，而我们的android项目也是用gradle来构建，虽然在自动化打渠道包时对gradle有所了解，但是仅限于基本使用，趁着新项目的开始，好好学习一下gradle的原理。
* 另一方面，现在非常流行的DSL语言，在gradle中也会用到，这也算是一个开始吧。
* 之前用过的很多东西，后面再用的时候，还是需要翻文档，也许真的是老了，刚好最近折腾了github pages，所以开始培养写博客的习惯，不为其它，只为以后再查阅的时候方便一点。
* 最后，感觉用gradle比maven的逼格更高，哇哈哈。。

<!-- more -->

# 关于gradle
## 什么是gradle
下面是来自度娘的解释
>Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化建构工具。它使用一种基于Groovy的特定领域语言(DSL)来声明项目设置，抛弃了基于XML的各种繁琐配置。

## 为什么是gadle
>Gradle makes the impossible possible, the possible easy and the easy elegant.<br>Gradle让一切不可能以一种更简单和优雅的方式变得可能。

* 统一的跨平台构建
* 能集成一切你能想到的工具
* 健壮的依赖管理
* 完全可编程的构建
* 最快的构建工具
* 构建（打包）报告

那么，gradle相对于我们以前使用的maven有什么优势呢？请看官方的对比[gradle vs maven](http://gradle.org/maven_vs_gradle/)

# 开始使用gradle

## 安装gradle

这个相对比较简单，所有平台安装方法都一样，下载gradle安装包，解压，配置环境变量，然后测试安装是否成功
```
⇒  gradle -version

------------------------------------------------------------
Gradle 2.10
------------------------------------------------------------

Build time:   2015-12-21 21:15:04 UTC
Build number: none
Revision:     276bdcded730f53aa8c11b479986aafa58e124a6

Groovy:       2.4.4
Ant:          Apache Ant(TM) version 1.9.3 compiled on December 23 2013
JVM:          1.8.0_65 (Oracle Corporation 25.65-b01)
OS:           Mac OS X 10.11.3 x86_64
```

## gradle任务
使用命令`gradle taskName`即可执行指定的任务

### 不执行特定的任务

如果在执行某一任务不想执行其依赖的某一任务时，使用如下命令`gradle taskName -x dependTaskName`

### 模糊的任务名

如果你的任务名比较长，而又不想打那么多字，可以尝试打出任务名的一部分试试
有以下任务
```
task dist << {
    println 'building the distribution'
}
```

执行`gradle d`看看会有什么样的输出
```
FAILURE: Build failed with an exception.

* What went wrong:
Task 'd' is ambiguous in root project 'openApiTest'. Candidates are: 'dependencies', 'dist'.

* Try:
Run gradle tasks to get a list of available tasks. Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 3.922 secs
```
可以看到，报错了，因为d不能唯一确定一个任务，因为gradle识别出有两个任务`dependencies`和`dist`
再试一下`gradle di`看看
```
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 3.723 secs
```
成功了

### 选择执行哪一个构建
假如项目根目录下有`subdir/hello.gradle`文件
文件内容如下
```
task hello << {
    println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
}
```

        ⇒  gradle -q -b subdir/hello.gradle hello
        using build file 'hello.gradle' in 'subdir'.

### 获取项目列表
    ⇒  gradle projects
    :projects
    
    ------------------------------------------------------------
    Root project - The open api test project
    ------------------------------------------------------------
    
    Root project 'openApiTest' - The open api test project
    No sub-projects
    
    To see a list of the tasks of a project, run gradle <project-path>:tasks
    For example, try running gradle :tasks
    
    BUILD SUCCESSFUL
    
    Total time: 3.98 secs

可以使用`description = 'The open api test project'`来给项目添加描述内容

### 获取任务列表

```
⇒   gradle -q tasks

------------------------------------------------------------
All tasks runnable from root project - The open api test project
------------------------------------------------------------

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
classes - Assembles classes 'main'.
clean - Deletes the build directory.
jar - Assembles a jar archive containing the main classes.
testClasses - Assembles classes 'test'.

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Documentation tasks
-------------------
javadoc - Generates Javadoc API documentation for the main source code.

Help tasks
----------
components - Displays the components produced by root project 'openApiTest'. [incubating]
dependencies - Displays all dependencies declared in root project 'openApiTest'.
dependencyInsight - Displays the insight into a specific dependency in root project 'openApiTest'.
help - Displays a help message.
projects - Displays the sub-projects of root project 'openApiTest'.
properties - Displays the properties of root project 'openApiTest'.
tasks - Displays the tasks runnable from root project 'openApiTest'.

Verification tasks
------------------
check - Runs all checks.
test - Runs the unit tests.

Other tasks
-----------
dist

Rules
-----
Pattern: clean<TaskName>: Cleans the output files of a task.
Pattern: build<ConfigurationName>: Assembles the artifacts of a configuration.
Pattern: upload<ConfigurationName>: Assembles and uploads the artifacts belonging to a configuration.

To see all tasks and more detail, run with --all.
chenpengpeng@localhost:~/git-repo/openApiTest|
```
感觉这个列表很有帮助，按照任务的类型对任务进行分类，下篇专门分析一下这些任务

### 获取任务的详细信息
```
⇒  gradle -q help --task init
Detailed task information for init

Path
     :init

Type
     InitBuild (org.gradle.buildinit.tasks.InitBuild)

Options
     --type     Set type of build to create.
                Available values are:
                     basic
                     groovy-library
                     java-library
                     pom
                     scala-library

Description
     Initializes a new Gradle build. [incubating]
```

### 获取项目依赖

```
⇒  gradle -q dependencies

------------------------------------------------------------
Root project - The open api test project
------------------------------------------------------------

archives - Configuration for archive artifacts.
No dependencies

compile - Compile classpath for source set 'main'.
\--- com.jayway.restassured:rest-assured:2.9.0
     +--- org.codehaus.groovy:groovy:2.4.4
     +--- org.codehaus.groovy:groovy-xml:2.4.4
     |    \--- org.codehaus.groovy:groovy:2.4.4
     +--- org.apache.httpcomponents:httpclient:4.5.1
     |    +--- org.apache.httpcomponents:httpcore:4.4.3
     |    +--- commons-logging:commons-logging:1.2
     |    \--- commons-codec:commons-codec:1.9
     +--- org.apache.httpcomponents:httpmime:4.5.1
     |    \--- org.apache.httpcomponents:httpclient:4.5.1 (*)
     +--- org.hamcrest:hamcrest-core:1.3
     +--- org.hamcrest:hamcrest-library:1.3
     |    \--- org.hamcrest:hamcrest-core:1.3
     +--- org.ccil.cowan.tagsoup:tagsoup:1.2.1
     +--- com.jayway.restassured:json-path:2.9.0
     |    +--- org.codehaus.groovy:groovy-json:2.4.4
     |    |    \--- org.codehaus.groovy:groovy:2.4.4
     |    +--- org.codehaus.groovy:groovy:2.4.4
     |    \--- com.jayway.restassured:rest-assured-common:2.9.0
     |         +--- org.codehaus.groovy:groovy:2.4.4
     |         \--- org.apache.commons:commons-lang3:3.3.2
     \--- com.jayway.restassured:xml-path:2.9.0
          +--- org.codehaus.groovy:groovy-xml:2.4.4 (*)
          +--- org.codehaus.groovy:groovy:2.4.4
          +--- com.jayway.restassured:rest-assured
          ....
          ....
```
### 获取项目属性

```
⇒  gradle properties
:properties

------------------------------------------------------------
Root project - The open api test project
------------------------------------------------------------

allprojects: [root project 'openApiTest']
ant: org.gradle.api.internal.project.DefaultAntBuilder@25b52284
antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@245ec1a6
archivesBaseName: openApiTest
artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@782be4eb
asDynamicObject: org.gradle.api.internal.ExtensibleDynamicObject@38792286
assemble: task ':assemble'
baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@34d4860f
binaries: [classes 'main', classes 'test']
build: task ':build'
buildDependents: task ':buildDependents'
buildDir: /Users/chenpengpeng/git-repo/openApiTest/build
buildFile: /Users/chenpengpeng/git-repo/openApiTest/build.gradle
buildNeeded: task ':buildNeeded'
buildScriptSource: org.gradle.groovy.scripts.UriScriptSource@41fe8e5f
buildTasks: [build]
buildscript: org.gradle.api.internal.initialization.DefaultScriptHandler@3062f9f4
check: task ':check'
childProjects: {}
class: class org.gradle.api.internal.project.DefaultProject_Decorated
classLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@2016f509
classes: task ':classes'
clean: task ':clean'
compile: task ':compile'
compileJava: task ':compileJava'
compileTest: task ':compileTest'
compileTestJava: task ':compileTestJava'
components: [org.gradle.api.internal.java.JavaLibrary@6f1a80fb]
configurationActions: org.gradle.configuration.project.DefaultProjectConfigurationActionContainer@5a237731
configurations: [configuration ':archives', configuration ':compile', configuration ':default', configuration ':runtime', configuration ':testCompile', configuration ':testRuntime']
convention: org.gradle.api.internal.plugins.DefaultConvention@7d2998d8
defaultArtifacts: org.gradle.api.internal.plugins.DefaultArtifactPublicationSet_Decorated@6a0094c9
defaultTasks: []
dependencies: org.gradle.api.internal.artifacts.dsl.dependencies.DefaultDependencyHandler_Decorated@51a6cc2a
dependencyCacheDir: /Users/chenpengpeng/git-repo/openApiTest/build/dependency-cache
dependencyCacheDirName: dependency-cache
depth: 0
description: The open api test project
dist: task ':dist'
distsDir: /Users/chenpengpeng/git-repo/openApiTest/build/distributions
distsDirName: distributions
docsDir: /Users/chenpengpeng/git-repo/openApiTest/build/docs
docsDirName: docs
ext: org.gradle.api.internal.plugins.DefaultExtraPropertiesExtension@10fda3d0
extensions: org.gradle.api.internal.plugins.DefaultConvention@7d2998d8
fileOperations: org.gradle.api.internal.file.DefaultFileOperations@2123064f
fileResolver: org.gradle.api.internal.file.BaseDirFileResolver@4f6b687e
gradle: build 'openApiTest'
group: com.imo
inheritedScope: org.gradle.api.internal.ExtensibleDynamicObject$InheritedDynamicObject@28cb3a25
jar: task ':jar'
javadoc: task ':javadoc'
libsDir: /Users/chenpengpeng/git-repo/openApiTest/build/libs
libsDirName: libs
logger: org.gradle.api.logging.Logging$LoggerImpl@5555ffcf
logging: org.gradle.logging.internal.DefaultLoggingManager@6cfd9a54
modelRegistry: org.gradle.model.internal.registry.DefaultModelRegistry@78c1372d
module: org.gradle.api.internal.artifacts.ProjectBackedModule@38f2e97e
name: openApiTest
parent: null
parentIdentifier: null
path: :
plugins: [org.gradle.api.plugins.HelpTasksPlugin@7c8326a4, org.gradle.language.base.plugins.LifecycleBasePlugin@67001148, org.gradle.api.plugins.BasePlugin@2b4c3c29, org.gradle.api.plugins.ReportingBasePlugin@6a10b263, org.gradle.language.base.plugins.LanguageBasePlugin@769a58e5, org.gradle.api.plugins.LegacyJavaComponentPlugin@5af97169, org.gradle.api.plugins.JavaBasePlugin@14e2e1c3, org.gradle.api.plugins.JavaPlugin@55e3d6c3]
processOperations: org.gradle.api.internal.file.DefaultFileOperations@2123064f
processResources: task ':processResources'
processTestResources: task ':processTestResources'
project: root project 'openApiTest'
projectDir: /Users/chenpengpeng/git-repo/openApiTest
projectEvaluationBroadcaster: ProjectEvaluationListener broadcast
projectEvaluator: org.gradle.configuration.project.LifecycleProjectEvaluator@9aa2002
projectRegistry: org.gradle.api.internal.project.DefaultProjectRegistry@73fb1d7f
properties: {...}
rebuildTasks: [clean, build]
reporting: org.gradle.api.reporting.ReportingExtension_Decorated@73d4066e
reportsDir: /Users/chenpengpeng/git-repo/openApiTest/build/reports
repositories: [org.gradle.api.internal.artifacts.repositories.DefaultMavenArtifactRepository_Decorated@25d2f66]
resources: org.gradle.api.internal.resources.DefaultResourceHandler@5a2fa51f
rootDir: /Users/chenpengpeng/git-repo/openApiTest
rootProject: root project 'openApiTest'
runtimeClasspath: file collection
scriptHandlerFactory: org.gradle.api.internal.initialization.DefaultScriptHandlerFactory@71945bc0
scriptPluginFactory: org.gradle.configuration.DefaultScriptPluginFactory@22a0d4ea
serviceRegistryFactory: org.gradle.internal.service.scopes.ProjectScopeServices$5@49ede9c7
services: ProjectScopeServices
sourceCompatibility: 1.8
sourceSets: [source set 'main', source set 'test']
sources: [source set 'main', source set 'test']
standardOutputCapture: org.gradle.logging.internal.DefaultLoggingManager@6cfd9a54
state: project state 'EXECUTED'
status: integration
subprojects: []
targetCompatibility: 1.8
tasks: [task ':assemble', task ':build', task ':buildDependents', task ':buildNeeded', task ':check', task ':classes', task ':clean', task ':compile', task ':compileJava', task ':compileTest', task ':compileTestJava', task ':dist', task ':jar', task ':javadoc', task ':processResources', task ':processTestResources', task ':properties', task ':test', task ':testClasses', task ':tester']
test: task ':test'
testClasses: task ':testClasses'
testReportDir: /Users/chenpengpeng/git-repo/openApiTest/build/reports/tests
testReportDirName: tests
testResultsDir: /Users/chenpengpeng/git-repo/openApiTest/build/test-results
testResultsDirName: test-results
tester: task ':tester'
version: 1.0-SNAPSHOT

BUILD SUCCESSFUL

Total time: 2.068 secs
```

### 记录构建过程中的各操作的时间
执行以下命令`gradle clean build --profile`
在build目录下会生成reports,如下
![profile-report](profile-report.png)
