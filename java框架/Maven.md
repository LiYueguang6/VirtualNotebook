# Maven

## 一、简介

Maven是java项目的包管理工具，包括了Spring等常用的java依赖包，在使用时只要按照配置文件引用相应的包即可使用。

## 二、配置

1. 配置仓库



## 三、生命周期和插件

### 生命周期

maven的生命周期虽然存在，但是他自己什么都不会做，生命周期依赖插件来完成。使用```mvn clean install```来构建一个项目，会经历以下生命周期

- clean阶段，clean插件执行clean目标。清理上一次构建生成的所有文件。

- compile阶段，compiler插件执行compile目标。编译项目的源代码。

- process-test-resources阶段，resources插件执行testResources目标。复制和处理资源文件到target目录，准备打包。

- test-compile阶段，compiler插件执行testCompile目标。编译测试源代码。

- test阶段，surefire插件执行test目标。运行测试代码。

- package阶段，war插件执行war目标。打包成jar或者war或者其他格式的分发包。

- install阶段，install插件执行install目标。将打好的包安装到本地仓库，供其他项目使用。

	如果将install换成deploy的话，将会有下一个阶段

- deploy阶段，deploy插件执行deploy目标。将打好的包安装到远程仓库，供其他项目使用

另外还有site生命周期

- site   ：生成项目的站点文档；
- site-deploy   ：发布生成的站点文档

### 插件