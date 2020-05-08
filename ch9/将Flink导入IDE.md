# 将Flink导入IDE

[TOC]

以下各节描述了如何将Flink项目导入到IDE中以开发Flink本身。有关编写Flink程序的信息，请参阅[Java API](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/projectsetup/java_api_quickstart.html) 和[Scala API](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/projectsetup/scala_api_quickstart.html) 快速入门指南。

**注意：**每当您的IDE中无法正常工作时，请首先尝试使用Maven命令行（`mvn clean package -DskipTests`），因为它可能是您的IDE中存在错误或未正确设置。

## 制备

首先，请首先从我们的一个[存储库中](https://flink.apache.org/community.html#source-code)检出Flink源 ，例如

```
git clone https://github.com/apache/flink.git
```

## IntelliJ IDEA

有关如何设置IntelliJ IDEA IDE进行Flink内核开发的简要指南。由于Eclipse在混合Scala和Java项目方面存在问题，因此越来越多的贡献者正在迁移到IntelliJ IDEA。

以下文档描述了使用Flink源设置IntelliJ IDEA 2019.1.3（https://www.jetbrains.com/idea/download/）的步骤。

### 安装Scala插件

IntelliJ安装设置可用来安装Scala插件。如果尚未安装，请在导入Flink之前按照以下说明进行操作，以启用对Scala项目和文件的支持：

1. 转到IntelliJ插件设置（IntelliJ IDEA->首选项->插件），然后单击“安装Jetbrains插件...”。
2. 选择并安装“ Scala”插件。
3. 重新启动IntelliJ

### 导入Flink

1. 启动IntelliJ IDEA并选择“ New-> Project from Existing Sources”
2. 选择克隆的Flink存储库的根文件夹
3. 选择“从外部模型导入项目”，然后选择“ Maven”
4. 保留默认选项，然后依次单击“下一步”，直到到达“ SDK”部分。
5. 如果未列出SDK，请使用左上方的“ +”号创建一个。选择“ JDK”，选择JDK主目录，然后单击“确定”。选择最合适的JDK版本。注意：一个好的经验法则是选择与活动Maven配置文件匹配的JDK版本。
6. 单击“下一步”继续，直到完成导入。
7. 右键单击导入的Flink项目-> Maven->生成源和更新文件夹。请注意，这会将Flink库安装在本地Maven存储库中，默认情况下位于“ /home/$USER/.m2/repository/org/apache/flink/”。另外，`mvn clean package -DskipTests`也可以创建IDE运行所需的文件，但无需安装库。
8. 编译项目（Build-> Make Project）

### Java的Checkstyle

IntelliJ使用Checkstyle-IDEA插件在IDE中支持checkstyle。

1. 从IntelliJ插件存储库安装“ Checkstyle-IDEA”插件。
2. 通过转到设置->其他设置-> Checkstyle配置插件。
3. 将“扫描范围”设置为“仅Java源（包括测试）”。
4. 在“ Checkstyle版本”下拉菜单中选择*8.14*，然后单击“应用”。**此步骤很重要，请勿跳过！**
5. 在“配置文件”窗格中，使用加号图标添加新配置：
   1. 将“说明”设置为“链接”。
   2. 选择“使用本地Checkstyle文件”，然后将其指向 `"tools/maven/checkstyle.xml"`您的存储库中。
   3. 选中“相对于项目位置的存储”框，然后单击“下一步”。
   4. 将“ checkstyle.suppressions.file”属性值配置为 `"suppressions.xml"`，然后单击“下一步”，然后单击“完成”。
6. 选择“ Flink”作为唯一的活动配置文件，然后单击“ Apply”和“ OK”。
7. Checkstyle现在将在编辑器中针对任何违反Checkstyle的行为发出警告。

插件安装完成后，您可以直接`"tools/maven/checkstyle.xml"`转到方案下拉框旁边的设置->编辑器->代码样式-> Java->齿轮图标，直接导入。例如，这将自动调整导入布局。

您可以通过打开Checkstyle工具窗口并单击“检查模块”按钮来扫描整个模块。扫描不应报告任何错误。

**注意**一些模块没有完全被CheckStyle的，其中包括覆盖`flink-core`，`flink-optimizer`和`flink-runtime`。不过，请确保您在这些模块中添加/修改的代码仍符合checkstyle规则。

### 斯卡拉Checkstyle

通过选择设置->编辑器->检查，然后搜索“ Scala样式检查”，在Intellij中启用scalastyle。还要放置`"tools/maven/scalastyle-config.xml"`在`"/.idea"`或`"/project"`目录中。

### 常问问题

本节列出了开发人员过去使用IntelliJ时遇到的问题：

- 编译失败 `invalid flag: --add-expots=java.base/sun.net.util=ALL-UNNAMED`

这意味着，`java11`尽管使用了较旧的JDK ，IntelliJ仍激活了配置文件。打开Maven工具窗口（视图->工具窗口-> Maven），取消选中`java11`配置文件，然后重新导入项目。

- 编译失败 `cannot find symbol: symbol: method defineClass(...) location: class sun.misc.Unsafe`

这意味着IntelliJ正在为该项目使用JDK 11，但是您正在使用不支持Java 11的Flink版本。这通常在将IntelliJ设置为使用JDK 11并检出Flink的旧版本（<= 1.9）时发生。 ）。打开项目设置窗口（文件->项目结构->项目设置：项目），然后选择JDK 8作为项目SDK。如果要使用JDK 11，则可能必须在切换回新的Flink版本后恢复此状态。

- `NoClassDefFoundError`对于Flink类，示例失败。

这可能是由于将Flink依赖项设置为提供，导致它们未自动放置在类路径上。您可以在运行配置中勾选“包括具有'Provided'作用域的依赖关系”框，也可以创建一个调用`main()`示例方法的测试（`provided`依赖关系在测试类路径中可用）。

## 日食

**注意：**根据我们的经验，由于与Scala IDE 3.0.3捆绑在一起的旧Eclipse版本的缺陷或由于与Scala IDE 4.4.1捆绑的Scala版本的版本不兼容，此设置不适用于Flink。

**我们建议改为使用IntelliJ（请参见[上文](https://ci.apache.org/projects/flink/flink-docs-release-1.10/flinkDev/ide_setup.html#intellij-idea)）**

## PyCharm

有关如何为开发flink-python模块设置PyCharm的简要指南。

以下文档介绍了使用Flink Python源设置PyCharm 2019.1.3（https://www.jetbrains.com/pycharm/download/）的步骤。

### 导入flink-python

如果您位于PyCharm启动界面中：

1. 启动PyCharm，然后选择“打开”
2. 在克隆的Flink存储库中选择flink-python文件夹

如果您使用PyCharm打开了一个项目：

1. 选择“文件->打开”
2. 在克隆的Flink存储库中选择flink-python文件夹

### Checkstyle For Python

Apache Flink的Python代码检查样式应在项目中创建flake8外部工具。

1. 将flake8安装在使用的Python解释器中（请参阅（https://pypi.org/project/flake8/）。
2. 选择“ PyCharm->首选项...->工具->外部工具-> +（右侧页面的左下角）”，然后配置外部工具。
3. 将“名称”设置为“ flake8”。
4. 将“描述”设置为“代码样式检查”。
5. 将“程序”设置为Python解释器路径（例如/ usr / bin / python）。
6. 将“参数”设置为“ -m flake8 --config = tox.ini”
7. 将“工作目录”设置为“ $ ProjectFileDir $”

现在，您可以通过以下操作检查您的Python代码样式：“右键单击flink-python项目中的任何文件或文件夹->外部工具-> flake8”