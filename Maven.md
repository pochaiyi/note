# Maven

Maven 是 Apache 管理的开源的项目管理工具，主要用于 Java 平台的**项目构建**，**依赖管理**和**项目信息管理**。所谓构建，就是编译、测试、打包等工作，这类工作简单繁琐，理应用自动化工具完成。

# 初步认识

## 安装软件

Maven 使用 Java 开发，运行它需要 JDK 环境。

```
java --version
```

再从[官网](https://maven.apache.org/download.cgi)下载匹配当前操作系统的压缩包，解压得到安装目录。

配置系统环境变量 `M2_HOME=安装目录路径`，把 `%M2_HOME%\bin` 加到 PATH 以在命令行运行 mvn 命令。

检验 Maven 是否安全成功，随便运行一个 mvn 命令。

```
mvn --version
```

Maven 第一次下载构件的时候，会自动创建一个目录用来存放构件，这个目录就是本地仓库，这样以后就能重用这些构件。默认每个用户都有一个本地仓库，路径 *~/.m2/repository*。

到此，Maven 安装完成，以上步骤对 Windows 或类 UNIX 系统都适用。

## 安装目录

```
M2_HOME/
	- bin/
	- boot/
	- conf/
	- lib/
	- LICENSE
	- NOTICE
	- README.txt
```

*bin/* 存放各种平台 mvn 脚本，脚本运行时首先配置 Java 命令，准备 classpath 和 Java 系统属性，然后执行 Java 命令。因此，执行 mvn 命令本质是运行 Java 命令；

*boot/* 只有一个文件 *plexus-classworlds-2.2.3.jar*，这是一个类加载器框架，Maven 用它加载自带的类库；

*conf/* 包含文件 *settings.xml*，这是 Maven 配置文件，用于全局调整 Maven 行为；

*lib/* 包含 Maven 运行需要的各种类库；

*LICENSE* 记录 Maven 软件许可证，*NOTICE* 记录 Maven 包含的第三方软件，*README* 简要介绍。

## 初步使用

### POM 文档

POM（Project Object Model，项目对象模型）是 Maven 项目的核心，所有 Maven 项目都有 *pom.xml*，该文档定义项目基本信息，描述项目如何构建、声明项目依赖，等等。

以下是 POM 范例：第一行 XML 头指定文档的版本和编码方式；然后是 `project` 元素，它是所有 POM 元素的根元素，其中声明了一些 POM 相关的命名空间和 xsd 元素，这些属性非必要，用于帮助第三方工具快速编辑 POM 文档；元素 `modelVersion` 指定当前 POM 模型的版本，Maven 3 该值只能是 4.0.0；元素 `name` 声明一个对用户友好的项目名称，非必须，但推荐，以便信息交流。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <name>Hello World Project</name>
	...
</project>
```

### 项目代码

Maven 区分项目的主代码和测试代码，因为主代码会被打包，测试代码只在运行测试时有用。按照约定，主代码应该位于 *src/main/java* 目录。编写完主代码之后，执行 `mvn clean compile` 命令，该命令首先会删除输出目录 *target/*，然后编译主代码到 *target/classes*。

> *target/* 是 Maven 约定的输出目录，默认项目的所有输出都在这里。

### 测试代码

按照约定，测试代码应该位于 *src/test/java* 目录，Junit 单元测试编写完后，执行 `mvn clean test`，该命令首先删除输出目录，然后编译测试代码到 *src/test/java*，最后运行测试程序并返回测试结果。

### 打包安装

打包项目，执行 `mvn clean package`，默认 `jar` 类型，打包结果位于 *target/*。

如果需要，执行 `mvn clean install`，该命令会打包项目并安装到本地仓库，以供其它项目使用。

## 其它经验

### 用户范围配置

默认情况，目录 *~/.m2* 除了 *repository* 仓库之外没有其它的文件或目录，但很多时候，用户会把 *settings.xml* 文件复制到这里以对 Maven 进行局部配置，这些配置只在当前用户生效。

### 慎用内嵌 Maven

多数 IDE 默认集成 Maven，尽量别用它们，因为内嵌 Maven 通常版本比较新，较不稳定。与本地 Maven 大概率版本不同，容易导致构建行为不一致。

### 环境变量 MAVEN_OPTS

已知 mvn 命令本质是运行 Java 命令，既然如此，那么 Java 命令的运行参数自然也对 mvn 命令有效。此时，可以使用环境变量 MAVEN_OPTS 设置 Java 命令运行参数，运行 mvn 命令时将自动应用这些参数。

示例，设置 `MAVEN_OPTS=-Xms128m-Xmx512m`，项目较大时 Maven 生成站点会占用大量内存，JVM 默认内存大小容易发送 OOM 异常。

# Maven 依赖

## 构件坐标

Maven 定义了一条规则：世界上任何一个构件都能通过 Maven 坐标唯一标识，以下是坐标的元素说明：

* groupId：必须，当前 Maven 项目所属的实际项目，而非公司或组织，通常反向域名；
* artifactId：必须，实际项目的模块，常以 groupId 作为前缀；
* version：必须，当前项目的版本；
* packaging：可选，默认 `packaging`，还有 `war`、`pom`。打包方式影响项目构建的生命周期，同时决定输出构件的文件拓展名；
* classifier：用于帮助定义构建输出的附属构件，该功能由插件完成，用户不能定义这个元素。

任何 Maven 项目，包括个人项目，全都需要 Maven 坐标。

## 依赖配置

### 整体模板

以下是一个依赖的声明可以包含的所有元素：groupId、artifactId、version 坐标元素；type 依赖类型，对应项目打包方式，默认 `jar`；scope 依赖范围；exclusions 排除的传递性依赖。

```
<project>
    ...
    <dependencies>
        <dependency>
            <groupId>...</groupId>
            <artifactId>...</artifactId>
            <version>...</version>
            <type>...</type>
            <scope>...</scope>
            <optional>...</optional>
            <exclusions>
                <exclusion>
                    <groupId>...</groupId>
                    <artifactId>...</artifactId>
                </exclusion>
                ...
            </exclusions>
        </dependency>
        ...
    </dependencies>
    ...
</project>
```

### 依赖范围 scope

Maven 编译项目主代码时使用一套 classpath，编译和运行测试代码时使用另一套 classpath，而实际运行项目时又使用一套 classpath。

依赖范围，便是用来控制依赖会在哪个 classpath 生效，Maven 有以下几种依赖范围：

* `compile`：默认，编译范围，对 3 种 classpath 有效；
* `test`：测试范围，只对测试 classpath 有效，比如 `JUnit`；
* `provided`：已提供范围，对于编译、测试有效，比如 `servlet-api`；
* `runtime`：运行时范围，对于测试、运行有效，比如 JDBC 驱动；
* `system`：系统范围，对于编译、测试有效，使用时需用 `systemPath` 元素指定依赖文件的路径，因为这类依赖不由 Maven 仓库解析，且往往与本机系统绑定，容易造成构建不可移植，慎用；
* `import`：导入范围，用于引入 `dependencyManagement` 信息，对 classpath 没有影响。

### 依赖传递

声明的依赖可能依赖于其它构件，Maven 将会解析所有直接依赖 POM 信息，把那些必要的间接依赖，以传递性依赖的方式引入当前项目。

除了控制与 3 种 classpath 的关系，依赖范围 scope 还影响着间接依赖是否会作为传递性依赖被引入，以及这个间接依赖的依赖范围，如下所示。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/maven-1140e677.png)

对于间接依赖，如果两条依赖路径分别包含同一个构件的不同版本，Maven 将会选择哪个版本呢？依赖路径长度不同时，引入较短的那个；长度相同时，引入先申明的那个。

### 可选依赖 optional

元素 `dependency` 有一个子元素 `optional`，设置它的值为 `true`，表示这个依赖仅在当前项目有效，不会作为传递性依赖被其它项目间接引入。

```
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    ＜version＞1.1.1＜/version＞
    <optional>true</optional>
</dependency>
```

### 排除依赖 exclusions

声明依赖时如果想要排除某些间接依赖，让它们在依赖传递中不被引入，使用 `exclusions` 和 `exclusion` 元素指定这些间接依赖，如下所示。

```
<dependency>
    <groupId>com.juvenxu.mvnbook</groupId>
    <artifactId>project-b</artifactId>
    ＜version＞1.0.0＜/version＞
    <exclusions>
        <exclusion>
            <groupId>com.juvenxu.mvnbook</groupId>
            <artifactId>project-c</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## 依赖分析

已知 Maven 会自动解析项目的直接依赖和传递性依赖，判断依赖的范围，甚至调节一些依赖冲突，确保任何构件只有一个版本被引入。做完这些之后，最终得到的依赖被称为已解析依赖（Resolved Dependency）。

列出当前项目所有已解析依赖，同时显示依赖范围。

```
mvn dependency:list
```

把直接依赖作为顶层，以树状图形式表现依赖的关系。

```
mvn dependency:tree
```

如果需要，可以使用以下命令帮助分析当前项目的依赖，返回的内容包括：使用到但未显式声明的依赖（Used undeclared dependencies）、未使用但显式声明的依赖（Unused declared dependencies）。

```
mvn dependency:analyze
```

虽然以上命令可以帮助分析项目依赖，但如果真的发生冲突，使用 IDEA 插件 `maven-helper` 更方便。

# Maven 仓库

## 认识仓库

Maven 世界的任何依赖、插件或项目输出，都被称为构件。依赖的坐标是构件的逻辑表现方式，构件的物理表现方式是文件，仓库就是存放这些文件的地方。所以，Maven 只需解析依赖的坐标信息，就能从仓库获取对应的文件并引入，这使多个项目能共用同一个构件文件，提高效率，节省空间。

## 仓库布局

每个构件都有一个唯一坐标，Maven 不是直接把所有构件放到仓库目录下边，而是根据坐标信息为构件定义相对仓库的存储路径。这是 Maven 仓库默认的布局方式，通常不会修改。

比如，构件 `log4j:log4j:1.2.15`，它在仓库内的存储路径是 *log4j/log4j/1.2.15/log4j-1.2.15.jar*，能够发现存储路径与坐标的对应方式大致是 *groupId/artifactId/version/artifactId-version.packaging*，其中 groupId 点号分隔符会被替换成文件分隔符，当然，还有还有一些额外处理，比如文件拓展名、附属构件等。

## 仓库分类

### 运行过程

Maven 项目没有类似 *lib/* 这样用来存放依赖的目录，编译或测试时，Maven 基于坐标信息把解析到的依赖从本地仓库添加到 classpath。

Maven 仓库可以分为两种：本地仓库和远程仓库。Maven 根据坐标从仓库找构件，首先查看本地仓库，有则直接使用，没有或者需要检查更新，就要搜索所有配置的远程仓库，找到之后，再下载到本地仓库使用。如果两种仓库都找不到目标构件，Maven 将会报错。

### 本地仓库

前面知识，每个用户默认有一个本地仓库 *~/.m2/repository*，用户能在这使用 *settings.xml* 局部配置 Maven，如果想要修改本地仓库的位置，可以修改相关配置，如下所示。

```
<settings>
	<localRepository>D:\data\mvn-repo</localRepository>
</settings>
```

用户的本地仓库初始并不存在，仅当 Maven 第一次使用它时才会进行创建。

构件进入本地仓库的方式有两种：其一是 Maven 自动从远程仓库下载；其二是执行 `mvn install` 手动安装本地项目到本地仓库。插件 `install` 的 `install` 目标会把项目的构建输出安装到本地仓库。

### 远程仓库

本地仓库服务于本地 Maven 项目，而远程仓库专门服务本地仓库。本地仓库默认为空，它的所有构件都是从远程仓库下载。每个用户只有一个本地仓库，但能配置多个远程仓库。

#### 中央仓库

Maven 默认配有一个远程仓库，即中央仓库，它由 Maven 官方维护，包含世界上绝大多数开源 Java 构件，以及源码、作者、许可证等信息。*$M2_HOME/lib/maven-model-builder-3.0.jar* 包含中央仓库的配置信息，具体文件是 *org/apache/maven/model/pom-4.0.0.xml*，这是超级 POM，它被所有 Maven 项目继承，Maven 项目的默认行为以及那些**约定**基本都在这声明。

以下是具体的中央仓库配置，它的 ID 是 central，名称 Maven Repository Switchboard，使用 default 默认仓库布局，注意 `snapshots` 元素，其子元素 `enabled` 值为 `false`，表示不从该仓库下载快照版本的构件。

```
<repositories>
    <repository>
        <id>central</id>
        <name>Central Repository</name>
        <url>https://repo.maven.apache.org/maven2</url>
        <layout>default</layout>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```

#### 配置仓库

默认的中央仓库可能无法满足需求，此时，可用 POM 文档 `repositories` 元素来配置其它远程仓库，其下使用 `repository` 子元素配置一个或多个远程仓库。仓库 ID 是唯一标识，如果使用 `central` 作为 ID，将会覆盖中央仓库的配置。

```
<project>
    <repositories>
        <repository>
            <id>jboss</id>
            <name>JBoss Repository</name>
            <url>...</url>
            <layout>default</layout>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>
```

对于元素 `releases` 和 `snapshots`，子元素 `enabled` 控制是否从当前仓库下载发布版本、快照版本的构件。另外还有 `updatePolicy` 和 `checksumPolicy` 两个子元素，前者控制快照构件检查更新的频率，默认 `daily` 每天检查一次，`never` 从不检查，`always` 每次构建都检查，`interval:x` 每隔 x 分钟检查一次。后者是校验和验证策略，仓库下载构件的时候，还会下载对应的校验和文件以验证文件完整性，验证失败该怎么办呢？默认 `warn` 输出警告信息，`fail` 构建失败，`ignore` 忽略错误。

```
<snapshots>
    <enabled>false</enabled>
    <updatePolicy>daily</updatePolicy>
    <checksumPolicy>ignore</checksumPolicy>
</snapshots>
```

#### 远程认证

多数远程仓库可以直接访问，但有些仓库基于安全考虑，需要提供认证信息才能访问。为了让 Maven 能够访问这些远程仓库，需要配置一些认证信息。与仓库不同，认证信息只能在 *settings.xml* 配置，因为 POM 往往会被提交到版本仓库，不安全。子元素 `id` 用于匹配设置的远程仓库，`username` 和 `password` 则是认证信息。

```
<servers>
    <server>
        <id>deploymentRepo</id>
        <username>repouser</username>
        <password>repopwd</password>
    </server>
</servers>
```

#### 部署构件

除了安装到本地仓库，Maven 项目还能部署到远程仓库，从而分享给更多的人。首先要修改 *pom.xml* 文件，使用元素 `distributionManagement`、`repository` 和 `snapshotRepository` 配置远程仓库。

```
<distributionManagement>
    <repository>
        <id>...</id>
        <name>...</name>
        <url>...</url>
    </repository>
    <snapshotRepository>
        <id>...</id>
        <name>...</name>
        <url>...</url>
    </snapshotRepository>
</distributionManagement>
```

然后，执行 `mvn clean deploy` 命令，Maven 首先构建项目，然后把输出的构件部署到远程仓库，如果当前项目是快照版本，则部署到 snapshotRepository，否则部署到 repository。

#### 私服代理

私服是一种架设在局域网内的特殊远程仓库，其实就是对互联网上的远程仓库的代理，它能节省外网带宽，加速构件下载，便于部署第三方构件，提高稳定性等。可以使用 Nexus 搭建，暂不赘述。

## 快照版本

Maven 构件分为发布版本和快照版本，前者稳定，后者随时更新。如果声明的构件是快照版本，Maven 将会根据策略定时检查该构件是否有更新的版本，有则下载并使用。

生成快照版本的构件时，Maven 会自动为其打上时间戳，比如 `2.1-20091214.221414-13`，这个字符串从年精确到秒，最后 13 表示构件第 13 次更新。通过时间戳，Maven 能够比较构件的发布顺序。所以，发布版本的构件只对应一个文件，而快照版本可能对应一群以时间戳结尾的文件。

默认情况，Maven 每天检查一次快照构件的更新，可在配置仓库时通过 `updatePolicy` 元素修改检查频率。

## 仓库解析机制

Maven 如何从仓库解析项目的依赖呢？以下是解析过程的大致步骤：

1. 如果依赖的范围是 `system`，委托给本地文件系统进行解析；
2. 根据坐标算出仓库路径，尝试从本地仓库获取，找到目标构件则解析成功；
3. 从本地仓库找不到目标构件时，如果声明的版本是显式的发布版本，比如 1.0、2.1-beta-1 等，则遍历所有远程仓库，寻找目标构件，然后下载并使用；
4. 如果版本是 RELEASE 或 LATEST，读取所有远程仓库的元数据 *groupId/artifactId/maven-metadata.xml*，将其与本地仓库对应元数据结合，算出实际版本，再回到第 2 步使用该版本寻找构件；
5. 如果版本是 SNAPSHOT，读取所有远程仓库的元数据 *groupId/artifactId/version/maven-metadata.xml*，将其与本地仓库对应元数据结合，算出最新版本，再回到第 2 步使用该版本寻找构件；
6. 如果最后得到的版本是时间戳，比如 `1.4.1-20091104.121450-121`，则复制该时间戳格式的文件为非时间戳格式，比如 SNAPSHOT，然后使用该非时间戳格式的构件。

解析还会受到 `enabled` 元素的影响，只有仓库允许才能访问；如果依赖的版本不明确，比如 RELEASE、LATEST 和 SNAPSHOT，元素 `updatePolicy` 控制 Maven 检查更新版本的频率，用户还能从命令行使用参数 `-U`，强制检查更新。

以下是发布版本的构件的元数据，RELEASE、LATEST 分别表示最新发布版本和最新版本（包括快照），所谓最新便是根据这些数据比较而来。对于下例，RELEASE 是 1.4.0，LATEST 是 1.4.2-SNAPSHOT。

> 对于 Maven 3，插件的版本默认是 RELEASE。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/maven-bf4d46a2.png" style="zoom:80%;" />

以下是快照版本的构件的元数据，子元素 `timestamp` 和 `buildNumber` 分别表示快照的时间戳和构建序号，基于它们能够比较各个快照的先后顺序。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/maven-59ab9b2b.png" style="zoom:80%;" />

最后，需要明白，仓库元数据并非永远正确，构件无法解析或解析出错时，可能就是元数据出错，这时需要手动或使用工具（比如 Nexus）对其进行修复。

## 仓库镜像

如果一个仓库能提供另一个仓库的所有内容，那就认为前者是后者的镜像。因为通信因素，地区镜像的访问速度往往比中央仓库快得多。以下修改 *settings.xml*，配置 Maven 使用国内镜像替代中央仓库。

```
<settings>
    <mirrors>
        <mirror>
            <id>maven.net.cn</id>
            <name>one of the central mirrors in China</name>
            <url>http://maven.net.cn/content/groups/public/</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
        ...
    </mirrors>
</settings>
```

元素 `mirrorOf` 值为 `central`，表示该配置是中央仓库的镜像，**任何**访问中央仓库的请求都会转至该镜像。其余几个元素与普通仓库配置无异，如果镜像仓库需要认证，还需基于 `id` 配置认证信息。

为了满足一些复杂需求，Maven 还支持更高级的镜像配置，比如 `<mirrorOf>*</mirrorOf>` 表示匹配所有远程仓库，`<mirrorOf>Repo1,Repo2</mirrorOf>` 匹配仓库 Repo1和 Repo2。

# 生命周期和插件

## 认识生命周期

项目的构建本身就有生命周期，因为无论是否使用构建工具，项目都需编译、测试、部署等工作，生命周期会定义执行这些工作的顺序和方式。如果没有统一标准，各项目的生命周期将有很大差异，每次构建新项目都要重新组织生命周期，太过麻烦，不易迁移。

基于众多项目和构建工具的经验，Maven 提出一套完善、易拓展的生命周期，其中包含清理、初始化、编译、测试、打包、验证、部署、生成站点等几乎所有构建步骤，足以映射绝大多数项目的构建。

## 三套生命周期

Maven 生命周期不是一个整体，而是 3 个相互独立生命周期，包括 clean 清理项目，default 构建项目，site 建立项目站点。每个生命周期有多个阶段（phase），这些阶段在周期内有先后顺序，且后面阶段依赖前面阶段。

### 调用阶段

用户与 Maven 最直接的交互方式就是调用生命周期的阶段。调用某个阶段，Maven 将从该周期内的第一个阶段开始，依次执行直到执行完目标阶段。这不会触发其它生命周期的任何阶段。

示例：

* `mvn clean`：调用 clean 生命周期 clean 阶段，实际会执行 pre-clean 和 clean 阶段；
* `mvn test`：调用 default 生命周期 test 阶段，实际会执行 validate 至 test 所有阶段；
* `mvn clean install`：调用 clean 周期 clean 阶段和 default 周期 install 阶段。

### Site 周期

基于 POM 信息生成和发布站点，包含 4 个阶段。

1. pre-site：生成站点之前需要做的工作；
2. site：生成项目站点文档；
3. post-site：生成站点之后需要做的工作；
4. site-deploy：发布生成的项目站点到服务器。

### Clean 周期

清理项目的输出，包含 3 个阶段。

1. pre-clean：清理之前需要做的工作；
2. clean：清理上一次构建产生的文件；
3. post-clean：清理之后需要做的工作。

### Default 周期

核心周期，定义构建项目需要执行的所有步骤，阶段很多，这里只说明重要部分。

1. validate
2. initialize
3. generate-sources
4. process-sources：处理项目主资源文件，通常是对 *src/main/resources* 内容进行变量替换等工作，然后将其复制到项目输出的主 classpath 目录；
5. generate-resources
6. process-resources
7. compile：编译项目的主代码；
8. process-classes
9. generate-test-sources
10. process-test-sources：处理项目测试资源文件；
11. generate-test-resources
12. process-test-resources
13. test-compile：编译项目的测试代码；
14. process-test-classes
15. test：运行单元测试，测试代码不会被打包或部署；
16. prepare-package
17. package：把编译好的代码打包成可发布格式，比如 `jar`；
18. pre-integration-test
19. integration-test
20. ·post-integration-test
21. verify
22. install：把包安装到本地仓库；
23. deploy：把包复制到远程仓库。

## 插件目标

生命周期只是一个概念，具体的行为由 Maven 插件实现。这是模板设计模式，生命周期定义框架结构，插件执行实际行为，保证既有拓展空间，又能固定流程结构。所以，Maven 核心非常小，它只在需要时下载插件。

很多功能的逻辑有重叠，包含很多重复代码，为了复用这些代码，Maven 把多个功能集成到一个插件。比如依赖相关的插件 `maven-dependency-plugin`，它能分析项目依赖、找出潜在无用的依赖，以及显示依赖树等。对于插件来说，每一个功能就是一个插件目标，目标才是执行任务的角色。

命令 `mvn dependency:list` 列出项目所有依赖，便是使用 `maven-dependency-plugin` 插件，`dependency` 是插件前缀，`list` 是插件目标，这是一种通用写法，后面详述。

## 绑定目标

生命周期的阶段能与插件的目标相互绑定，这样就能通过调用阶段来运行目标，从而构建项目。

**内置绑定**

为了让用户几乎不用任何配置就能构建 Maven 项目，默认会为一些重要的生命周期阶段绑定插件目标，因此用户能直接调用阶段来构建项目，默认绑定的都是 Maven 官方插件。

其余没有默认绑定的阶段，调用它们不会有任何实际行为。比如 clean 生命周期，它有 3 个阶段，只有 clean 阶段与 `maven-clean-plugin` 插件 clean 目标绑定，另外两个阶段没有默认绑定。

**定制绑定**

除了内置绑定，Maven 允许用户把其它插件的目标绑定到指定生命周期的阶段，拓展构建的功能。以下配置绑定插件 `maven-source-plugin` 目标 `jar-no-fork` 到 verify 阶段，该目录能生成源码 `jar` 包。

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.1.1</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

元素 `plugins` 和 `plugin` 声明插件，groupId 是 org.apache.maven.plugins，说明这是官方插件，artifactId 是 maven-source-plugin，version 是 2.1.1。然后，使用 `executions` 子元素配置插件的执行，子元素 `phase` 指定阶段，`goals` 指定目标，这样就把 `jar-no-fork` 与 verify 阶段绑定。

部分插件的目标在编写时设有默认绑定阶段，这种目标即使不配置阶段也能运行，使用 maven-help-plugin 查看该插件的详细信息，字段 Bound to phase 表示其默认的绑定阶段。

如果有多个插件目标绑定到同一个阶段，这些目标的执行顺序与目标的声明顺序一致。

## 插件配置

插件目标绑定之后，用户还能为插件配置参数，进一步调整目标的执行。几乎所有 Maven 插件目标都会提供一些可配置参数，通过命令行或 POM 等方式设置这些参数。

### 命令行配置

很多插件目标的参数支持从命令行设置，以下为 `maven-surefire-plugin` 设置 `maven.test.skip=true`，表示该目标跳过执行测试阶段。`-D` 是 Java 参数，用于命令行设置 Java 系统属性，Maven 准备插件的时候会检查系统属性，然后根据属性值调整相关配置。

```
mvn install -D maven.test.skip=true
```

### POM 插件配置

命令行设置的参数只对本次运行有效，但有的参数比较稳定，它的值可能从项目创建到发布都不会改变，这时可在 POM 插件声明的地方进行配置，以后项目使用该插件时将自动应用这些参数。

示例，配置 `maven-compiler-plugin` 插件，指定源代码和字节码的版本，这个配置很常见。

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.1</version>
            <configuration>
                <source>8</source>
                <target>8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

与上面示例等效的做法是设置 POM 属性，因为超级 POM 使用这两属性设置 `source` 和 `target`。

```
<properties>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
</properties>
```

### POM 任务配置

用户还可以对插件的任务进行单独配置。比如 `maven-antrun-plugin`，它的目标 run 用于调用 Ant 任务，以下配置绑定 `maven-antrun-plugin:run` 到 validate 阶段。`configuration` 元素位于 `execution` 下面，说明配置单个任务，而非插件整体。这里 ant-validate 配置了一个 echo Ant 任务，运行时向命令行打印一段文字。

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>1.3</version>
            <executions>
                <execution>
                    <id>ant-validate</id>
                    <phase>validate</phase>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    <configuration>
                        <tasks>
                            <echo>I'm bound to validate phase.</echo>
                        </tasks>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 直接调用插件

除了调用生命周期的阶段来运行插件目标，Maven 还支持从命令行直接调用插件的目标，毕竟不是所有目标都会绑定到生命周期。

执行 `mvn -h` 查看 mvn 帮助信息，返回的第一行是 mvn 使用格式，表示 mvn 后面可以跟一个或多个 goal。

```
usage: mvn [options] [<goal(s)>] [<phase(s)>]
...
```

示例，命令 `mvn dependency:list` 运行 `maven-dependency-plugin` 的 `list` 目标，它会列出当前项目的所有依赖，该目标未与任何阶段绑定。

## 插件解析机制

### 插件远程仓库

与依赖一样，插件构件同样基于坐标保存 Maven 仓库，需要的时候，Maven 首先查找本地仓库，找不到再尝试从远程仓库下载并使用。

但是，远程仓库区分依赖仓库和插件仓库，前面 `<repositories>` 元素配置的是依赖远程仓库，Maven 不会从这下载插件，插件远程仓库使用 `pluginRepositories` 和 `pluginRepositorie` 元素配置。以下是插件远程仓库默认的配置，与依赖远程仓库相似，且也是使用中央仓库。

```
<project>
<pluginRepositories>
    <pluginRepository>
        <id>central</id>
        <name>Maven Plugin Repository</name>
        <url>http://repo1.maven.org/maven2</url>
        <layout>default</layout>
        <releases>
            <updatePolicy>never</updatePolicy>
        </releases>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>
</project>
```

### 默认官方分组

解析插件的时候，如果发现其没有声明 groupId，Maven 默认填充 org.apache.maven.plugins，即认为这个插件是官方插件，然后再尝试解析。所以，官方插件的声明可以省略 `groupId`，这种特性弊大于利。

```
<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.1</version>
        </plugin>
    </plugins>
</build>
```

### 解析插件版本

同样为了简化配置和使用，如果插件声明没有提供版本，Maven 将会进行自动解析。首先，超级 POM 已为核心插件设定了默认版本，因此，即便没有版本，核心插件也能正常使用。

对于非核心插件，Maven 首先会归并所有仓库的元数据，算出 RELEASE、LATEST，Maven 3 选择 RELEASE 作为默认版本，具体步骤参考仓库的解析机制。以下是插件元数据的片段。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/maven-e1ad3504.png" style="zoom:80%;" />

尽管如此，版本不固定就存在不稳定，所以 Maven 会为所有核心插件显式设置版本。

### 解析插件前缀

命令 `mvn dependency:list` 列出项目依赖，`list` 表示目标，`dependency` 为什么能表示一个插件呢？这其实是插件前缀，本节讲述 Maven 如何把插件前缀解析为插件。

插件前缀与 `groupId:artifactId` 对应，这类匹配关系存放在仓库元数据 *groupId/maven-metadata.xml*，如下所示。但这只有 artifactId 与 prefix，没有 groupId，稍后解释。Mavne 可以在这根据前缀查找 artifactId。

<img src="C:\Users\28371\AppData\Roaming\Typora\typora-user-images\image-20240311150513957.png" alt="image-20240311150513957" style="zoom:80%;" />

插件大多位于 [Apache](http://repo1.maven.org/maven2/org/apache/maven/plugins/) 和[ Codehaus MoJo](http://repository.codehaus.org/org/codehaus/mojo/)，所以 Maven 默认会从这两仓库搜索插件，Apache 的 groupId 是 org.apache.maven.plugins，MoJo 的 groupId 是 org.codehaus.mojo。

如有需要，配置 *settings.xml* 让 Maven 搜索其它 groupId 上的插件仓库元数据，groupId 原来在这设置。由此可知相同仓库的插件的 groupId 相同。

```
<settings>
    <pluginGroups>
        <pluginGroup>com.your.plugins</pluginGroup>
    </pluginGroups>
</settings>
```

总结，如果 Maven 遇到 `dependency:list` 这种命令，首先基于给定 groupId 搜索所有插件仓库的元数据，根据前缀找到 artifactId，然后再使用上一节的方法解析插件版本。如果解析失败，报错。

# 聚合与继承

## 聚合

为了功能划分、复用和管理代码，较大的项目可能划分为多个模块，每个模块是一个 Maven 项目，Maven 允许把多个模块聚合到一起，进行统一管理和构建。

Maven 需要一个专门的项目来聚合真实模块，这个项目称为聚合模块。聚合模块只有一个 POM 文件，没有任何代码或者资源。以下是聚合模块 POM，没什么特别，但打包方式必须设为 `pom`，使用 `modules` 和 `module`  元素声明模块，`module` 指定模块目录的相对位置，默认处于聚合模块子目录。

```
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-aggregator</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <name>Account Aggregator</name>

    <modules>
        <module>account-email</module>
        <module>account-persist</module>
    </modules>
</project>
```

聚合模块运行 `mvn clean install`，输出以下信息。Maven 会解析聚合模块 POM，分析将要构建的模块，计算出一个反应堆构建顺序，根据反应堆依次构建各个模块。反应堆是所有模块组成的构建结构。可以看到，输出的是项目名称，而非 artifactId 属性，所以 `name` 元素能帮助理解。最后有一个报告，包括各模块是否构建成功、花费时间，以及整体构建花费的时间、使用的内存等。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/maven-4618ddcd.png" style="zoom:80%;" />

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/maven-6c556684.png" style="zoom:80%;" />

## 继承

相同项目的模块往往有许多相同配置，这些配置在各个模块反复声明，应该使用**继承**进行简化。

### 继承元素

类似 Java 类，Maven 项目允许继承，子模块会继承父模块的许多元素，以下列举它们：

* groupId：项目组 ID
* version：项目版本
* properties：自定义 Maven 属性
* dependencies：依赖配置
* dependencyManagement：依赖管理配置
* repositories：仓库配置
* build：源码目录配置、输出目录配置、插件配置、插件管理配置等
* distributionManagement：部署配置
* issueManagement：缺陷跟踪系统信息
* ciManagement：持续集成系统信息
* scm：版本控制系统信息
* mailingLists：邮件列表信息
* reporting：项目的报告输出目录配置、报告插件配置等
* description：项目描述信息
* organization：组织信息

* inceptionYear：创始年份
* url：项目 URL 地址
* developers：开发者信息
* contributors：贡献者信息

### 创建模块

#### 父模块

Maven 父模块与普通项目没有区别，但它的打包方式必须设为 `pom`。注意，这里声明的依赖、插件、属性等元素会被子模块完全继承，所以只应在此声明公共内容，否则会导致项目急剧增大。

由于父模块只是为了帮助消除配置的重复，所以它不包含除了 POM 以外的项目文件。

#### 子模块

使用元素 `parent` 声明继承父模块，相同项目模块的坐标比较一致，子模块往往只需定义 artifactId 元素。子元素 `relativePath` 指定父模块 POM 相对位置，如果找不到则搜索本地仓库，默认 *../pom.xml*，所以常把子模块放到父模块目录之下，既方便配置，又能显出父子关系。另外，本地声明的元素会覆盖从父模块继承的相同元素。

```
<parent>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>ccount-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <relativePath>../account-parent/pom.xml</relativePath>
</parent>
<artifactId>account-email</artifactId>
<name>Account Email</name>
```

### 依赖管理

继承 `dependencies` 元素会直接为子模块引入依赖，这种配置适合公共依赖。`dependencyManagement` 既能让子模块继承父模块的依赖配置，又能保证子模块依赖使用的灵活性。

`dependencyManagement` 不会为当前模块引入实际依赖，但它能约束 `dependencies` 下的依赖。比如，如果某个依赖有约束配置，子模块声明该依赖时只需 `groupId` 和 `artifactId`，POM 默认使用约束的版本和范围，当然还可以自定义覆盖约束配置。以下是父模块 POM。

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

以下是子模块 POM，看起来并没有节省太多配置，但把依赖的版本和范围集中管理，减少冲突，方便修改。

```
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```

除了继承，POM 还能使用依赖声明引入 `dependencyManagement` 配置，如下所示。注意，这种依赖的依赖范围必须是 `import`。

```
<dependencies>
    <dependency>
        <groupId>com.juvenxu.mvnbook.account</groupId>
        <artifactId>account-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
        <packaging>pom</packaging>
        <scope>import</scope>
    </dependency>
</dependencies>
```

### 插件管理

同样，插件也能进行约束配置，使用 `pluginManagement` 元素，如下所示。

```
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>2.1.1</version>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

# 灵活的构建

## Maven 属性

Maven 具有属性功能，便于获取项目有关的值和常用的值，这些属性有的内置，有的自定义，分为 6 种：

* 自定义属性：使用 `properties` 元素定义，POM 文档内通过 `${属性名字}` 引用；

  ```
  <project>
      <properties>
          <spring-cloud.version>Hoxton.SR12</spring-cloud.version>
      </properties>
  
      <dependencyManagement>
          <dependencies>
              <dependency>
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-dependencies</artifactId>
                  <version>${spring-cloud.version}</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
          </dependencies>
      </dependencyManagement>
  </project>
  ```

* 内置属性：常用的有两个，*${basedir}* 项目根目录，*${version}* 项目版本；

* POM 属性：引用 POM 元素的值，例如 *${project.artifactId}* 对应 *\<project>\<artifactId>* 元素的值；
  * `${project.build.directory}`：构建输出目录，默认 *target/*；
  * `${project.build.sourceDirectory}`：主代码目录，默认 *src/main/java/*；
  * `${project.build.testSourceDirectory}`：测试代码目录，默认 *src/test/java/*；
  * `${project.outputDirectory}`：主代码编译输出目录，默认 *target/classes/*；
  * `${project.testOutputDirectory}`：测试代码编译输出目录，默认 *target/test-classes/*；
  * `${project.groupId}`：项目 groupId；
  * `${project.artifactId}`：项目 artifactId；
  * `${project.version}`：项目 version；
  * `${project.build.finalName}`：输出文件名字，默认 *${project.artifactId}-${project.version}*。

* Settings 属性：引用 *settings.xml* 中 XML 元素的值，比如 *${settings.localRepository}* 表示本地仓库地址；
* Java 系统属性：引用 Java 系统属性，比如 *${user.home}* 表示用户目录；
* 环境变量属性：引用系统环境变量，比如 ${env.JAVA_HOME} 表示 JAVA_HOME 环境变量的值。
