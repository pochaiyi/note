# Maven

Maven 作用：管理依赖（版本），自动完成项目编译，测试，打包和部署。

# 约定配置

Maven 项目具有约定的目录结构，我们要尽可能地遵守这样的约定：

| 目录                                 | 目的                                                         |
| :----------------------------------- | :----------------------------------------------------------- |
| `${basedir}`                         | 存放 `pom.xml` 和所有子目录                                  |
| `${basedir}/src/main/java`           | 项目的 `java` 源代码                                         |
| `${basedir}/src/main/resources`      | 项目的资源，例如 `application.yaml`，`springmvc.xml`         |
| `${basedir}/src/test/java`           | 项目的测试类，例如 `Junit` 代码                              |
| `${basedir}/src/test/resources`      | 测试使用的资源                                               |
| `${basedir}/src/main/webapp/WEB-INF` | `web` 应用文件目录，`web` 项目的信息，例如 `web.xml`、本地图片、`jsp` 等 |
| `${basedir}/target`                  | 打包输出目录                                                 |
| `${basedir}/target/classes`          | 编译输出目录                                                 |
| `${basedir}/target/test-classes`     | 测试编译输出目录                                             |
| `Test.java`                          | Maven 只会自动运行符合该命名规则的测试类                     |
| `~/.m2/repository`                   | Maven 默认的本地仓库目录位置                                 |

# 安装配置

1. Maven 是基于 Java 的工具，需要提前配置 JDK 环境。

2. 下载 Maven 压缩包，解压至安装目录。

3. 设置 Maven 环境变量：

   ```
   M2_HOME = 安装目录
   
   eg：M2_HOME = D:\application\apache-maven-3.8.1
   ```

4. 验证是否成功：

   ```
   mvn -v

# POM 文件

POM（Project Object Model，项目对象模型）是 Maven 工程的基本工作单元，是一个 XML 文件，其包含项目的基本信息。

执行任务或目标时，Maven 会在当前目录中查找 POM 文件，获取所需的配置信息，然后执行目标。

POM 内可以指定以下配置：

- 项目版本
- 项目依赖
- 执行目标
- 插件
- 项目构建 profile
- 项目开发者列表
- 相关邮件列表信息

## 基本元素

| 结点         | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| project      | POM 文件的根标签，其它所有元素都在这个标签内。               |
| modelVersion | 模型版本，描述当前 POM 文件遵循哪种描述符规则。<br />Maven 2 和 Maven 3 都只能是 4.0.0 |

## 项目坐标

Maven 项目使用 GAV 坐标进行区分的，即 `groupId`，`artifactId`，`version`：

| 结点       | 描述                                                |
| :--------- | :-------------------------------------------------- |
| groupId    | 公司或组织的标识，配置时的路径也是据此生成。        |
| artifactId | POM 项目的唯一标识，一个 groupId 下可能有多个项目。 |
| version    | 项目的版本号。                                      |

## Super POM

所有 POM 文件都继承自 Super POM，无论是否显式定义。它包含一些可以被继承的默认设置，当 Maven 项目需要某些依赖时，它会到 Super POM 配置的默认仓库下载。

## POM 模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!--父项目坐标-->
    <parent>
        <groupId>x</groupId>
        <artifactId>x</artifactId>
        <version>x</version>
        <!--父项目pom文件的相对路径，默认值是../pom.xml。Maven按如下顺序寻找父项目pom文件：当前项目 -> 文件系统的relativePath路径 -> 本地仓库 -> 远程仓库-->
        <relativePath></relativePath>
    </parent>

    <!--POM模型版本-->
    <modelVersion>4.0.0</modelVersion>

    <!--当前Maven项目坐标-->
    <groupId>x</groupId>
    <artifactId>x</artifactId>
    <version>x</version>

    <!--项目产生的构件类型，如jar，war等，使用插件还可以创建其它的构件类型-->
    <packaging>jar</packaging>

    <!--项目名称，用于Maven产生的文档-->
    <name>xx</name>
    <!--项目主页的URL，用于Maven产生的文档-->
    <url>https://www.xxx.com</url>
    <!-- 项目的详细描述，用于Maven产生的文档-->
    <description>A maven project</description>
    <!--描述该项目构建环境的前提条件-->
    <prerequisites>
        <!--构建该项目或使用该插件所需要的Maven的最低版本 -->
        <maven>3.8.1</maven>
    </prerequisites>

    <!--以值替代名称，properties可以在整个POM中使用，也可以作为触发条件，格式是<proName>value</proName>-->
    <properties>
    </properties>

    <!--项目的所有依赖，这些依赖会从项目定义的仓库中下载-->
    <dependencies>
        <!--单个依赖-->
        <dependency>
            <groupId>x</groupId>
            <artifactId>x</artifactId>
            <version>x</version>
            <!--依赖类型，默认是jar-->
            <type>jar</type>
            <!--依赖范围，帮助决定在项目发布过程中哪些构件被包括进来：runtime，test-->
            <scope>test</scope>
            <!--从依赖的构件中排除的某些子构件，主要用于解决版本冲突问题 -->
            <exclusions>
                <exclusion>
                    <groupId>x</groupId>
                    <artifactId>x</artifactId>
                </exclusion>
            </exclusions>
            <!--可选依赖，阻断依赖的传递性-->
            <!--eg：项目B中把C依赖声明为可选，项目A依赖于B，但A还需要显式引用对C的依赖-->
            <optional>true</optional>
        </dependency>
    </dependencies>

    <!--继承该项目的所有子项目的默认依赖信息-->
    <!--这部分的依赖信息不会被立即解析，当子项目声明一个依赖，如果没有指定group ID和artifact ID以外的信息，则通过group ID和artifact ID获取这里的依赖信息-->
    <dependencyManagement>
        <dependencies>
            <dependency></dependency>
        </dependencies>
    </dependencyManagement>

    <!--构建项目需要的信息-->
    <build>
        <!--产生的构件的名称，默认是${artifactId}-${version}-->
        <finalName>projectName</finalName>
        <!--项目源码目录，该路径相对POM文件-->
        <sourceDirectory>src/main/java</sourceDirectory>
        <!--项目单元测试的源码目录，该路径相对POM文件-->
        <testSourceDirectory>src/test/java</testSourceDirectory>
        <!--编译的class文件的存放目录-->
        <outputDirectory>target/classes/</outputDirectory>
        <!--编译的测试class文件的存放目录-->
        <testOutputDirectory>target/test-classes</testOutputDirectory>
        <!--构建产生的所有文件的存放目录-->
        <directory>/dir</directory>
        <!--项目脚本源码目录，和源码目录不同：通常该目录的内容会被拷贝到输出目录，因为脚本是被解释的，而不是被编译的-->
        <scriptSourceDirectory>xxx</scriptSourceDirectory>

        <!--项目需要使用的所有构建扩展-->
        <!--顾名思义，构建拓展用于创建构件，可能是比较特别的构件类型-->
        <extensions>
            <!--单个构建扩展的信息-->
            <extension>
                <groupId>x</groupId>
                <artifactId>x</artifactId>
                <version>x</version>
            </extension>
        </extensions>

        <!--当项目没有规定目标时的默认值-->
        <defaultGoal></defaultGoal>

        <!--项目相关的所有资源路径列表，这些资源会被包含在最终的打包文件中-->
        <resources>
            <!--项目相关或测试相关的所有资源路径-->
            <resource>
                <!--资源的目标路径，该路径相对于target/classes目录-->
                <!--eg：如果想把资源放在特定的包 org.apache.maven.messages，就必须设置该元素为org/apache/maven/messages，但如果只是想把资源放到源码目录结构里，则不需要该配置-->
                <targetPath>x</targetPath>
                <!--是否使用参数值代替参数名，参数值取自properties元素或文件配置的属性，文件在filters元素里列出-->
                <filtering>false</filtering>
                <!--存放资源的目录，该路径相对于POM文件-->
                <directory>x</directory>
                <!--包含的模式列表，例如**/*.xml-->
                <includes>
                    <include>*.xml</include>
                </includes>
                <!--排除的模式列表，例如**/*.xml-->
                <excludes>
                    <exclude>*.xml</exclude>
                </excludes>
            </resource>
        </resources>

        <!--单元测试相关的所有资源路径列表，例如和单元测试相关的属性文件-->
        <testResources>
            <!--测试相关的所有资源路径，参见build/resources/resource的说明-->
            <testResource>
                <targetPath>x</targetPath>
                <filtering>false</filtering>
                <directory>x</directory>
                <includes>
                    <include>*.xml</include>
                </includes>
                <excludes>
                    <exclude>*.xml</exclude>
                </excludes>
            </testResource>
        </testResources>

        <!--当filtering打开时，使用到的过滤器属性文件列表-->
        <filters>
            <filter>x</filter>
        </filters>

        <!--使用的插件列表-->
        <plugins>
            <!--单个插件的信息-->
            <plugin>
                <groupId>x</groupId>
                <artifactId>x</artifactId>
                <version>x</version>
                <!--是否从该插件下载Maven扩展，例如打包和类型处理器，由于性能原因，只有在需要时才被设置成enabled-->
                <extensions>false</extensions>
                <!--构建生命周期中执行目标的配置列表-->
                <executions>
                    <!--单个执行目标的信息-->
                    <execution>
                        <!--执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标-->
                        <id>x</id>
                        <!--绑定目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段-->
                        <phase>x</phase>
                        <!--配置的执行目标-->
                        <goals></goals>
                        <!--是否被传播到子项目-->
                        <inherited>x</inherited>
                        <!--作为DOM对象的配置-->
                        <configuration></configuration>
                    </execution>
                </executions>
                <!--项目引入插件所需要的额外依赖-->
                <dependencies>
                    <dependency></dependency>
                </dependencies>
                <!--配置是否传播到子项目-->
                <inherited>false</inherited>
                <!--作为DOM对象的配置-->
                <configuration></configuration>
            </plugin>
        </plugins>

        <!--子项目可以引用的默认插件信息，该配置项直到被引用时才会被解析或绑定到生命周期，给定插件的任何本地配置都会覆盖这里的配置-->
        <pluginManagement>
            <!--使用的插件列表-->
            <plugins>
                <plugin>
                    <groupId>x</groupId>
                    <artifactId>x</artifactId>
                    <version>x</version>
                    <extensions>false</extensions>
                    <executions>
                        <execution>
                            <id>x</id>
                            <phase>x</phase>
                            <goals></goals>
                            <inherited>x</inherited>
                            <configuration></configuration>
                        </execution>
                    </executions>
                    <dependencies>
                        <dependency></dependency>
                    </dependencies>
                    <inherited>false</inherited>
                    <configuration></configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

# 生命周期

Maven 有以下三个标准的生命周期：

- `clean`：清理生命周期，移除上次构建生成的文件。
- `build`：构建生命周期，构建应用，其所有任务如上图所示。
- `site`：项目站点文档创建，创建新的报告文档、部署站点等。

每个生命周期都包含一系列阶段（phase），这些 phase 相当于 Maven 提供的的接口，phase 的实现由 Maven 插件完成。

我们在输入 mvn 命令的时候，例如 `mvn clean`，对应的就是 Clean 生命周期中的 clean 阶段，但是具体操作是由插件 `maven-clean-plugin` 实现的。

## 构件生命周期

Maven 使用构建生命周期表示一个项目的构建和发布过程，其大概由以下几个阶段序列组成：

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/maven-d66ed24a.png)

| 阶段          | 处理     | 描述                                                     |
| :------------ | :------- | :------------------------------------------------------- |
| 验证 validate | 验证项目 | 验证项目是否正确且所有必须信息是可用的                   |
| 编译 compile  | 执行编译 | 源代码编译在此阶段完成                                   |
| 测试 Test     | 测试     | 使用适当的单元测试框架（如 JUnit）运行测试               |
| 包装 package  | 打包     | 创建 JAR/WAR 包或其他在 pom.xml 中定义的类型             |
| 检查 verify   | 检查     | 对集成测试的结果进行检查                                 |
| 安装 install  | 安装     | 安装打包的项目到本地仓库，以供其他项目使用               |
| 部署 deploy   | 部署     | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程 |

**构建阶段由插件目标构成**

一个插件目标代表一个特定的任务（比构建阶段更精细），这有助于项目的构建和管理。这些目标可能被绑定到多个阶段或者不绑定。不绑定到任何构建阶段的目标可以在构建生命周期之外通过直接调用执行。这些目标的执行顺序取决于调用目标和构建阶段的顺序。

当通过 Maven 命令调用一个阶段时（`mvn compile`），该阶段以及前面的的所有阶段会被执行。

不同的 Maven 目标将根据打包的类型（JAR/WAR等），被绑定到不同的 Maven 生命周期阶段。

## 构件目标命令

* 清理

``` 
# 清理编译生成的target目录
maven clean
```

* 编译程序

```
# 编译程序并将编译结果和main/resource下的资源文件放置target/classes目录
maven compile
```

* 编译测试程序

```
# 编译测试程序并将编译结果和test/resource下的资源文件测试资源文件放置target/test-classes目录
maven test-complie
```

* 测试

```
# 测试失败会在target/surefire-reports目录生成测试报告
maven test
```

* 打包

```
# 根据pom.xml配置打包maven工程为jar/war,结果放置target目录
# 打包目标 src/main
maven package
```

* 安装

```
# 将当前项目构建输出到本地仓库，以后就可以通过坐标引用该构件
maven install
```

# 仓库

Maven 中的任何一个依赖、插件或者项目构建的输出，都可以称之为构件。Maven 仓库就是存放构件的地方，主要是 JAR 文件。Maven 仓库有三种类型：本地（local），中央（central），远程（remote）。

如果想知道构件的坐标以及它的迭代版本，可以在 https://mvnrepository.com 查询。

## 依赖搜索顺序

1. 搜索本地仓库。

2. 搜索中央仓库，如果未配置远程仓库，则抛出错误。

3. 按顺序搜索配置的远程仓库。

## 本地仓库

Maven 本地仓库在安装时不会创建，而是在第一次执行 maven 命令的时候才被创建。

运行 Maven 的时候，Maven 所需要的任何构件都是直接从本地仓库获取的。如果本地仓库没有，它会尝试从远程仓库下载构件至本地仓库，然后再使用本地仓库的构件。

默认情况，本地仓库被创建为 `%USER_HOME%\.m2\repository` 目录。可以通过修改 `%M2_HOME%\conf` 目录中的配置文件 `settings.xml`，自定义本地仓库的位置。

根据提示信息，在适当位置修改/添加标签：

```
<localRepository>D:\document\mvnRepository</localRepository>
```

## 中央仓库

Maven 中央仓库是由 Maven 社区提供的仓库，包含大量常用的构件。

Maven 默认配置中央仓库镜像，为加速下载，可以在 `settings.xml` 文件中添加国内镜像：

```
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>https://maven.aliyun.com/repository/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

还可以在 POM 文件中使用 `<repository>` 标签定义局部的远程仓库，但这实在没必要。

## 远程仓库

如果 Maven 在中央仓库中找不到依赖的构件，它会停止构建过程并输出错误信息到控制台。为避免这种情况，Maven 提供了远程仓库的概念，它是开发人员自己定制的仓库，包含所需要的代码库和构件。

# 依赖管理

Maven 的一个核心功能就是依赖管理。当我们处理多模块的项目，模块间的依赖关系就变得非常复杂，管理也变得很困难。针对此种情形，Maven 提供有高度控制的方法。

## 可传递性依赖发现

情形：A 依赖于 B，如果另外一个项目 C 想要使用 A ，那么 C 也需要使用 B。

Maven 可以避免自定义所有需要的库，通过读取 POM 文件，Maven 能找出所有项目之间的依赖关系。我们需要做的只是在每个项目中定义直接的依赖关系。其他的事情 Maven 会帮我们搞定。

通过可传递性的依赖，被包含的库会快速地增长。当有重复库时，还会发生版本冲突。Maven 提供有一些功能来控制可传递的依赖的程度。

| 功能     | 功能描述                                                     |
| :------- | :----------------------------------------------------------- |
| 依赖调节 | 决定当多个手动创建的版本同时出现时，哪个依赖版本将会被使用。 <br />如果两个依赖版本在依赖树里的深度是一样的时候，第一个被声明的依赖将会被使用。 |
| 依赖管理 | 直接的指定手动创建的某个版本被使用。<br />例如一个工程 C 的依赖管理模块包含工程 B，而 B 又依赖于 A， 那么 A 即可指定在 B 被引用时所使用的版本。 |
| 依赖范围 | 包含在构建过程每个阶段的依赖。                               |
| 依赖排除 | 任何可传递的依赖都可以通过 `exclusion` 元素被排除。          |
| 依赖可选 | 任何可传递的依赖都可以通过 `optional` 元素标记为可选的。<br />例如：A 依赖 B， B 依赖 C。因此，B 可以标记 C 为可选的， 这样 A 就不再引用 C。 |

## 依赖范围

传递依赖发现可以通过使用如下的依赖范围来得到限制：

| 范围     | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| 编译阶段 | 默认，相关依赖只在项目的类路径下有效。                       |
| 供应阶段 | 相关依赖是由运行时的 JDK 或者网络服务器提供的。              |
| 运行阶段 | 相关依赖在编译阶段不是必须的，但是在执行阶段是必须的。       |
| 测试阶段 | 相关依赖只在测试编译阶段和执行阶段有效。                     |
| 系统阶段 | 需要提供一个系统路径。                                       |
| 导入阶段 | 只当依赖被定义在 POM 时使用。<br />当前项目 POM 文件部分定义的依赖关系可以取代某特定的 POM。 |

## 依赖管理

通常情况，一个通用的项目有一系列的子项目。在这种情况，我们可以创建一个公共依赖的 POM 文件，其包含所有公共的依赖关系，我们称其为其他子项目 POM 的父 POM。 
