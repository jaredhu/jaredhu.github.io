# 从源码构建Flink

本页介绍如何从源代码构建Flink 1.10.0。

[TOC]

## 建立链接

为了构建Flink，您需要源代码。无论是[下载一个释放源](https://flink.apache.org/downloads.html)或[克隆的Git仓库](https://github.com/apache/flink)。

另外，您需要**Maven 3**和**JDK**（Java开发工具包）。Flink **至少**需要**Java 8**才能构建。

*注意：Maven 3.3.x可以构建Flink，但是不能正确地遮盖某些依赖项。Maven 3.2.5会正确创建库。要构建单元测试，请使用Java 8u51或更高版本，以防止使用PowerMock运行器的单元测试失败。*

要从git克隆，请输入：

```
git clone https://github.com/apache/flink
```

构建Flink的最简单方法是运行：

```
mvn clean install -DskipTests
```

这指示[Maven](http://maven.apache.org/)（`mvn`）首先删除所有现有的内部版本（`clean`），然后创建一个新的Flink二进制文件（`install`）。

为了加快构建速度，您可以跳过测试，QA插件和JavaDocs：

```
mvn clean install -DskipTests -Dfast
```

## 建立PyFlink

如果要构建可用于pip安装的PyFlink软件包，则需要先构建Flink jar，如[Build Flink中所述](https://ci.apache.org/projects/flink/flink-docs-release-1.10/flinkDev/building.html#build-flink)。然后转到flink源代码的根目录，并运行以下命令来构建sdist包和wheel包：

```
cd flink-python; python setup.py sdist bdist_wheel
```

**注意**构建PyFlink需要Python版本（3.5、3.6或3.7）。

sdist和wheel软件包可以在下找到`./flink-python/dist/`。它们中的任何一个都可以用于pip安装，例如：

```
python -m pip install dist/*.tar.gz
```

## 依赖阴影

Flink [遮蔽了](https://maven.apache.org/plugins/maven-shade-plugin/)它使用的某些库，以避免与使用这些库的不同版本的用户程序发生版本冲突。带有阴影的库包括*Google Guava*，*Asm*，*Apache Curator*，*Apache HTTP Components*，*Netty*等。

依赖项着色机制最近在Maven中进行了更改，并要求用户根据自己的Maven版本来构建Flink稍有不同：

**Maven 3.1.x和3.2.x**`mvn clean install -DskipTests`在Flink代码库的根目录中 调用就足够了。

**Maven 3.3.x** 构建必须分两步完成：首先在基本目录中，然后在分发项目中：

```
mvn clean install -DskipTests
cd flink-dist
mvn clean install
```

*注意：*要检查您的Maven版本，请运行`mvn --version`。

[ 回到顶部](https://ci.apache.org/projects/flink/flink-docs-release-1.10/flinkDev/building.html#top)

## Hadoop版本

请参阅有关如何处理Hadoop类和版本的[Hadoop集成部分](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/hadoop.html)。

## Scala版本

仅使用Java API和库的**信息**用户可以*忽略*此部分。

Flink具有用[Scala](http://scala-lang.org/)编写的API，库和运行时模块。Scala API和库的用户可能必须将Flink的Scala版本与其项目的Scala版本进行匹配（因为Scala并非严格向后兼容）。

从1.7版开始，Flink使用Scala 2.11版（默认）和2.12版进行构建。

要针对Scala 2.12构建FLink，请发出以下命令：

```
mvn clean install -DskipTests -Dscala-2.12
```

[ 回到顶部](https://ci.apache.org/projects/flink/flink-docs-release-1.10/flinkDev/building.html#top)

## 加密文件系统

如果您的主目录已加密，则可能会遇到`java.io.IOException: File name too long`异常。某些加密的文件系统（如Ubuntu使用的encfs）不允许使用长文件名，这是导致此错误的原因。

解决方法是添加：

```
<args>
    <arg>-Xmax-classfile-name</arg>
    <arg>128</arg>
</args>
```

在`pom.xml`导致错误的模块文件的编译器配置中。例如，如果错误出现在`flink-yarn`模块中，则应将以上代码添加到的``标签下`scala-maven-plugin`。有关更多信息，请参[见此问题](https://issues.apache.org/jira/browse/FLINK-2003)。