# Maven打包的类型？

以下是几种常见的打包形式：

1、`jar` (Java Archive)

- **用途**：用于包含 Java **类文件**和**其他资源**（如属性文件、配置文件等）的库项目。
- **特点**：
  - 可以被其他项目作为依赖引用。
  - 适合创建独立的应用程序或可重用的组件。
- **生成文件**：`.jar`文件。

2、`war` (Web Application Archive)

- **用途**：专为 Web 应用程序设计，包含了 Servlets、JSP 页面、静态资源（如 HTML、CSS、JavaScript）、以及其他必要的配置文件。
- **特点**：
  - 部署在应用服务器上，如 Apache Tomcat、JBoss 等。
  - 包含一个特殊的目录结构，例如`WEB-INF/classes`和`WEB-INF/lib`。
- **生成文件**：`.war`文件。

\3. `pom` (Project Object Model)

- **用途**：不是实际的二进制打包格式，而是用来表示一个多模块项目的父 POM。
- **特点**：
  - 不会产生任何输出文件，除非指定了具体的构建目标。
  - 定义了一组共享的配置信息给子模块使用。
  - 有助于管理大型、复杂的企业级项目。
- **生成文件**：无直接生成的文件，但会生成元数据（如`.pom`文件），用于描述项目及其依赖关系。

4、 `maven-plugin`

- **用途**：在自定义插件时，需要指定打包类型为 maven-plugin**。**最后生成的是 jar 包。

为了指定打包类型，需要在项目的`pom.xml`文件中定义`<packaging>`元素。例如：

```XML
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    ...
    <packaging>jar</packaging>
    ...
</project>
```

> 附：
>
> 1、与 packaging 标签相对的标签是 type，表示引入依赖的类型，例如 pom 类型、jar 类型，默认为 jar 类型。
>
> 
>
> 2、jar 和 war 的比较：
>
> ![img](https://i-blog.csdnimg.cn/direct/2105c709da0340f4895be03494e4227e.png)
>
> 
>
> 3、Maven 打包文件的来源？[14]
>
> 虽然 `src/main` 是放置源代码和资源的地方，但**实际打包的是经过编译和处理后存放在** **`target`** **目录下的文件**。`target` 目录是构建工具用来存放所有中间产物和最终产物的地方，在每次构建时可能会被清理或更新。



# Maven打的jar包有各种依赖吗？

Maven 构建的 JAR 文件是否包含依赖项取决于如何配置构建过程。Maven 提供了几种不同的方式来打包项目，每种方式对依赖项的处理不同：

**1、普通 JAR**

这是默认的打包方式，当运行 `mvn clean package` 命令时，Maven 会编译代码并将其打包成一个 JAR 文件，但**不会**将项目的依赖项包含在这个 JAR 文件中。这意味着生成的 JAR 文件只包含您的项目代码和资源文件。

**使用场景**：运行环境已经提供了所需依赖，如 big-marketing 项目就使用 Docker 提供了 MySQL、Redis 等。

**2、带有依赖的 JAR（Fat/Uber JAR）**

有时候可能想要创建一个**包含所有依赖项的 JAR 文件**，这样就**可以直接运行**而不需要额外的类路径设置。为了实现这一点，可以使用 Maven 的插件，例如 `maven-shade-plugin` 或 `maven-assembly-plugin`。

**(1) 使用** **`maven-shade-plugin`** **创建 Fat JAR：**

```XML
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.4</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**(2) 使用** **`maven-assembly-plugin`** **创建 Fat JAR：**

```XML
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.3.0</version>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
                <archive>
                    <manifest>
                        <mainClass>com.example.MainClass</mainClass>
                    </manifest>
                </archive>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**使用场景**：当需要一个独立的、可执行的 JAR 文件时。更确切地说，是自定义的归档文件。



# Maven打包的插件有几个？

- `maven-jar-plugin:jar`：将普通项目或模块打成 jar 包
- `maven-war-plugin:war`：将 JaveWeb 项目或模块打成 war 包
- `maven-shade-plugin:shade`：**在 jar 目标打包的基础上**，将 compile 和 runtime 的依赖打进 fat jar
- `maven-assembly-plugin:single`：个性化打包，用户可以自定义**打包的类型**、**打包的文件**、**打包的目录结构等**。
- `spring-boot-maven-plugin:repackage`：**在 jar 目标打包的基础上**，将 compile、runtime 和 provided 的依赖打进 fat jar。

## 演示：

### **1、maven-jar-plugin:jar**

- 在父工程 maven-package-demo 中创建一个子模块 jar-project
- 在 pom 文件中指定打包方式 packaging 为 jar（默认为 jar，所以可不指定）
- 执行打包命令 mvn org.apache.maven.plugins:maven-jar-plugin:jar -pl :jar-project

执行流程如下：

```Bash
xxx@xxxdeMacBook-Air maven-package-demo % mvn org.apache.maven.plugins:maven-jar-plugin:jar -pl :jar-project
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------< cn.myphoenix:jar-project >----------------------
[INFO] Building jar-project 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- jar:3.4.1:jar (default-cli) @ jar-project ---
[WARNING] JAR will be empty - no content was marked for inclusion!
[INFO] Building jar: /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/jar-project/target/jar-project-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.313 s
[INFO] Finished at: 2024-12-24T21:06:43+08:00
[INFO] ------------------------------------------------------------------------
```



### **2、maven-war-plugin:war**

- 在父工程 maven-package-demo 中创建一个子模块 war-project
- 在 pom 文件中指定打包方式 packaging 为 war
- 执行打包命令 mvn org.apache.maven.plugins:maven-war-plugin:war -pl :war-project

执行流程如下：

```Bash
xxx@xxxdeMacBook-Air maven-package-demo % mvn org.apache.maven.plugins:maven-war-plugin:war -pl :war-project
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------< cn.myphoenix:war-project >----------------------
[INFO] Building war-project 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ war ]---------------------------------
[INFO] 
[INFO] --- war:3.4.0:war (default-cli) @ war-project ---
[INFO] Packaging webapp
[INFO] Assembling webapp [war-project] in [/Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/war-project/target/war-project-1.0-SNAPSHOT]
[INFO] Processing war project
[INFO] Copying webapp resources [/Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/war-project/src/main/webapp]
[INFO] Building war: /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/war-project/target/war-project-1.0-SNAPSHOT.war
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.355 s
[INFO] Finished at: 2024-12-24T21:14:44+08:00
[INFO] ------------------------------------------------------------------------
```

> 附：另外，我还将 war 的 packaging 声明删掉了，发现依然能够成功将模块打成 war 包。



### **3、maven-shade-plugin:shade**

- 在父工程 maven-package-demo 中创建一个子模块 shade-project
- 在 pom 文件中引入 maven-shade-plugin 插件，并将其 shade 目标绑定到 package阶段
- 在 pom 文件中引入 5 种依赖范围的依赖
- 执行打包命令 `mvn package -pl :shade-project`

pom 文件中引入 maven-shade-plugin，并将 shade 目标绑定到 package 阶段

```XML
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.6.0</version>
        </plugin>
    </plugins>
</build>
```

pom 文件中引入的 5 种依赖范围的依赖，观察最终打成的 fat jar 中包含哪种类型的依赖：

```XML
<dependencies>
    <!-- compile 的依赖-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.16</version>
        <scope>compile</scope>
    </dependency>
    <!-- test 的依赖-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>3.8.1</version>
        <scope>test</scope>
    </dependency>
    <!-- provided 的依赖-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>3.0-alpha-1</version>
        <scope>provided</scope>
    </dependency>
    <!-- runtime 的依赖-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
        <scope>runtime</scope>
    </dependency>
    <!-- system 的依赖-->
    <dependency>
        <groupId>cn.myphoenix</groupId>
        <artifactId>big-marketing-wheel</artifactId>
        <version>1.0-SNAPSHOT</version>
        <scope>system</scope>
        <systemPath>/Users/xxx/desktop/big-marketing-wheel-1.0-SNAPSHOT.jar</systemPath>
    </dependency>
</dependencies>
```

命令的执行流程如下：

```Bash
xxx@xxxdeMacBook-Air maven-package-demo % mvn package -pl shade-project
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------------< cn.myphoenix:shade-project >---------------------
[INFO] Building shade-project 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[WARNING] The artifact mysql:mysql-connector-java:jar:8.0.33 has been relocated to com.mysql:mysql-connector-j:jar:8.0.33: MySQL Connector/J artifacts moved to reverse-DNS compliant Maven 2+ coordinates.
[INFO] 
[INFO] --- resources:3.3.1:resources (default-resources) @ shade-project ---
[INFO] Copying 0 resource from src/main/resources to target/classes
[INFO] 
[INFO] --- compiler:3.13.0:compile (default-compile) @ shade-project ---
[INFO] Recompiling the module because of changed source code.
[INFO] Compiling 1 source file with javac [debug target 8] to target/classes
[INFO] 
[INFO] --- resources:3.3.1:testResources (default-testResources) @ shade-project ---
[INFO] skip non existing resourceDirectory /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/shade-project/src/test/resources
[INFO] 
[INFO] --- compiler:3.13.0:testCompile (default-testCompile) @ shade-project ---
[INFO] Recompiling the module because of changed dependency.
[INFO] 
[INFO] --- surefire:3.2.5:test (default-test) @ shade-project ---
[INFO] 
[INFO] --- jar:3.4.1:jar (default-jar) @ shade-project ---
[INFO] Building jar: /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/shade-project/target/shade-project-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- shade:3.6.0:shade (default) @ shade-project ---
[INFO] Including org.mybatis:mybatis:jar:3.5.16 in the shaded jar.
[INFO] Including com.mysql:mysql-connector-j:jar:8.0.33 in the shaded jar.
[INFO] Including com.google.protobuf:protobuf-java:jar:3.21.9 in the shaded jar.
[INFO] Dependency-reduced POM written at: /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/shade-project/dependency-reduced-pom.xml
[WARNING] mybatis-3.5.16.jar, mysql-connector-j-8.0.33.jar, protobuf-java-3.21.9.jar, shade-project-1.0-SNAPSHOT.jar define 1 overlapping resource: 
[WARNING]   - META-INF/MANIFEST.MF
[WARNING] maven-shade-plugin has detected that some files are
[WARNING] present in two or more JARs. When this happens, only one
[WARNING] single version of the file is copied to the uber jar.
[WARNING] Usually this is not harmful and you can skip these warnings,
[WARNING] otherwise try to manually exclude artifacts based on
[WARNING] mvn dependency:tree -Ddetail=true and the above output.
[WARNING] See https://maven.apache.org/plugins/maven-shade-plugin/
[INFO] Replacing original artifact with shaded artifact.
[INFO] Replacing /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/shade-project/target/shade-project-1.0-SNAPSHOT.jar with /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/shade-project/target/shade-project-1.0-SNAPSHOT-shaded.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.254 s
[INFO] Finished at: 2024-12-25T15:05:30+08:00
[INFO] ------------------------------------------------------------------------
```

打成了两个 jar 包：

- shade-project-1.0-SNAPSHOT.jar：shade 目标打成的包含依赖的 fat jar
- original-shade-project-1.0-SNAPSHOT.jar：原本的 jar 目标打成的 jar 包

![img](https://i-blog.csdnimg.cn/direct/9082d27175ea41a29fa0ef82e28a907c.png)

如标红处所示，将 compile 范围的 MyBatis 依赖和 runtime 范围的 MySQL 依赖打到了 fat jar 中。

> 附：
>
> `maven-shade-plugin` 的 `shade` 目标通常需要在 Maven 的 `package` 阶段执行，这意味着它依赖于之前阶段生成的主要构件（通常是 JAR 文件）。具体来说，在 `shade` 操作之前，Maven 必须已经构建并打包了项目的主类文件和资源文件到一个标准的 JAR 文件中。这是因为它的工作原理是基于这个主 JAR 文件，将所有依赖项合并到其中。
>
> 
>
> **一、为什么** **`shade`** **需要先执行** **`jar`****？**
>
> - **主要构件的存在**：`maven-shade-plugin` 需要有一个基础的 JAR 文件来作为起点。这个 JAR 文件包含了编译后的项目代码和资源文件。如果 `maven-jar-plugin` 没有正确配置或没有被执行，那么就不会生成这个必要的主 JAR 文件，从而导致 `shade` 操作失败。
> - **生命周期顺序**：Maven 的生命周期是有序的，每个阶段都有其特定的任务。`package` 阶段的任务就是创建可分发格式的包（如 JAR、WAR 等），而 `maven-shade-plugin` 的 `shade` 目标则是对这些包进行进一步处理。因此，自然地，`shade` 应该在 `package` 阶段及之后执行，以确保它能够操作到已经生成好的主 JAR 文件。
> - **避免重复工作**：直接从命令行调用 `shade` 可能会导致一些不必要的重复工作或者跳过某些重要的构建步骤。通过将 `shade` 绑定到 `package` 阶段，可以保证整个构建过程按照预期的顺序进行，不会遗漏任何关键步骤。
>
> 
>
> **二、如何执行 shade 目标？**
>
> - **配置插件绑定**：在 `pom.xml` 中明确指定 `maven-shade-plugin` 的 `shade` 目标绑定到 `package` 阶段，如下所示：
>   - ```XML
>     <build>
>         <plugins>
>             <plugin>
>                 <groupId>org.apache.maven.plugins</groupId>
>                 <artifactId>maven-shade-plugin</artifactId>
>                 <version>3.2.4</version> <!-- 使用最新版本 -->
>                 <executions>
>                     <execution>
>                         <phase>package</phase> <!-- 绑定到 package 阶段 -->
>                         <goals>
>                             <goal>shade</goal>
>                         </goals>
>                     </execution>
>                 </executions>
>             </plugin>
>         </plugins>
>     </build>
>     ```
>
>   - 
> - **运行完整的生命周期命令**：使用像 `mvn clean package` 这样的命令来触发完整的构建过程，而不是直接尝试运行 `shade` 目标。这样可以确保所有必要的前期工作都已完成，并且 `shade` 操作可以顺利进行。
>
> 
>
> **三、shade 目标会将哪些 scope 的依赖打进 fat jar？**
>
> - 默认情况下，`maven-shade-plugin` 会将 `compile` 和 `runtime` 范围的依赖打包进 Fat JAR 中，而不会包含 `provided`、`test` 和 `system` 范围的依赖。上面实验也验证了这一点。



### **4、maven-assembly-plugin**

> **实验设计**：
>
> 1、创建子模块如下图所示：
>
> ![img](https://i-blog.csdnimg.cn/direct/2e39a6cb65e24ee7820e2d8ded704ed0.png)
>
> 
>
> 2、**最终实现**：
>
> - 利用 single 目标生成一个名为 assembly-project-demo.zip 的压缩包
> - 将构建过程中生成的 jar 包打包进 assembly-project-demo.zip 的根目录下
> - 将 /resources/mybatis 目录下的 MyBatis 的 xml 配置文件打包进压缩包的 config/mybatis 目录下
> - 将 /resources 目录下的 application.properties 打包进压缩包的 config 目录下
> - 将 /resources/images 目录下的所有图片打包进压缩包的 config/images 目录下

**使用 assembly 插件的步骤：**

- 构思确定打包的类型、需要打包的文件、最终包的目录结构
- 在 pom 文件中引入 maven-assembly-plugin 插件。建议将 single 目标绑定到某一阶段如 package，当然也可以直接执行插件目标。
- 编写配置文件
  - 需要打包的文件，如 jar 包、配置文件、资源文件等
  - 打包文件的放置目录，就是把某个文件放到最终包的某个目录下
- 执行打包

**实验步骤：**

- 在父工程 maven-package-demo 中创建一个子模块 assembly-project
- 创建配置文件 src/main/assembly/assembly_config.xml，在配置文件配置打包的依赖、文件、jar 包等
- 在 pom 文件中引入 maven-assembly-plugin 插件，并将其 single 目标绑定到 package阶段
- 在 pom 文件中引入 4 种依赖范围的依赖
- 执行打包命令 `mvn package -pl :assembly-project`

配置文件 assembly_config.xml 的内容如下：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<assembly>
    <!-- 最终打包文件的后缀，格式为 assembly-project-demo -->
    <id>demo</id>

    <!-- 最终打包成一个用于发布的zip文件 -->
    <formats>
        <format>zip</format>
    </formats>

    <!-- 把项目中依赖的 jar 包打进 assembly-project-demo.zip 压缩包的 lib 目录下 -->
    <dependencySets>
        <!-- 打包 runtime 类型的依赖 -->
        <dependencySet>
            <!-- 不使用项目的 artifact -->
            <useProjectArtifact>false</useProjectArtifact>
            <!-- 打包进 zip 文件下的 lib 目录中  -->
            <outputDirectory>lib</outputDirectory>
            <!-- 第三方 jar 不要解压 -->
            <unpack>false</unpack>
        </dependencySet>
    </dependencySets>

    <!-- 把项目中的文件打包进 assembly-project-demo.zip 压缩包 -->
    <fileSets>
        <!-- target 目录下的 jar 包打包进 assembly-project-demo.zip 压缩包的根目录下 -->
        <fileSet>
            <directory>target</directory>
            <outputDirectory>/</outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
        </fileSet>

        <!-- /resources/mybatis 目录下的 MyBatis 的 xml 配置文件打包进 assembly-project-demo.zip 压缩包的 config/mybatis 目录中 -->
        <fileSet>
            <!-- 文件原始路径 -->
            <directory>${project.basedir}/src/main/resources/mybatis</directory>
            <!-- 文件打包目标路径 -->
            <outputDirectory>/config/mybatis</outputDirectory>
            <includes>
                <!-- 直接指定所有文件 -->
                <include>*.*</include>
            </includes>
        </fileSet>

        <!-- /resources 目录下的 application.properties 打包进 assembly-project-demo.zip 压缩包的 config 目录中 -->
        <fileSet>
            <directory>${project.basedir}/src/main/resources</directory>
            <outputDirectory>/config</outputDirectory>
            <includes>
                <include>*.properties</include>
            </includes>
        </fileSet>

        <!-- /resources/images 目录下的所有图片打包进 assembly-project-demo.zip 压缩包的 config/images 目录中 -->
        <fileSet>
            <directory>${project.build.directory}</directory>
            <outputDirectory>/config/images</outputDirectory>
            <includes>
                <include>*.jeg</include>
                <include>*.jpeg</include>
                <include>*.png</include>
            </includes>
        </fileSet>

    </fileSets>
</assembly>
```

pom 文件的主要配置如下：

```XML
<!-- 添加 4 种依赖范围的依赖 -->
<dependencies>
    <!-- compile 的依赖-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.16</version>
        <scope>compile</scope>
    </dependency>
    <!-- test 的依赖-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>3.8.1</version>
        <scope>test</scope>
    </dependency>
    <!-- provided 的依赖-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>3.0-alpha-1</version>
        <scope>provided</scope>
    </dependency>
    <!-- runtime 的依赖-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>

<!-- 配置 assembly 插件-->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.2.0</version>
            <configuration>
                <!-- 配置打包后的文件前缀名 -->
                <finalName>assembly-project</finalName>
                <descriptors>
                    <!-- 指定配置文件的路径 -->
                    <descriptor>src/main/assembly/assembly_config.xml</descriptor>
                </descriptors>
            </configuration>
            <!-- 配置 assembly 的 goal -->
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**开始实验：**

执行命令后，控制台打印如下：

```Bash
xxx@xxxdeMacBook-Air maven-package-demo % mvn package -pl assembly-project
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------------< cn.myphoenix:assembly-project >--------------------
[INFO] Building assembly-project 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[WARNING] The artifact mysql:mysql-connector-java:jar:8.0.33 has been relocated to com.mysql:mysql-connector-j:jar:8.0.33: MySQL Connector/J artifacts moved to reverse-DNS compliant Maven 2+ coordinates.
[INFO] 
[INFO] --- resources:3.3.1:resources (default-resources) @ assembly-project ---
[INFO] Copying 8 resources from src/main/resources to target/classes
[INFO] 
[INFO] --- compiler:3.13.0:compile (default-compile) @ assembly-project ---
[INFO] Recompiling the module because of changed source code.
[INFO] Compiling 2 source files with javac [debug target 8] to target/classes
[INFO] 
[INFO] --- resources:3.3.1:testResources (default-testResources) @ assembly-project ---
[INFO] skip non existing resourceDirectory /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/assembly-project/src/test/resources
[INFO] 
[INFO] --- compiler:3.13.0:testCompile (default-testCompile) @ assembly-project ---
[INFO] Recompiling the module because of changed dependency.
[INFO] 
[INFO] --- surefire:3.2.5:test (default-test) @ assembly-project ---
[INFO] 
[INFO] --- jar:3.4.1:jar (default-jar) @ assembly-project ---
[INFO] Building jar: /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/assembly-project/target/assembly-project-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- assembly:3.2.0:single (make-assembly) @ assembly-project ---
[WARNING]  Parameter 'finalName' is read-only, must not be used in configuration
[INFO] Reading assembly descriptor: src/main/assembly/assembly_config.xml
[WARNING] The assembly descriptor contains a *nix-specific root-relative reference (starting with slash). This is not portable and might fail on Windows: /
[WARNING] The assembly descriptor contains a *nix-specific root-relative reference (starting with slash). This is not portable and might fail on Windows: /config/mybatis
[WARNING] The assembly descriptor contains a *nix-specific root-relative reference (starting with slash). This is not portable and might fail on Windows: /config
[WARNING] The assembly descriptor contains a *nix-specific root-relative reference (starting with slash). This is not portable and might fail on Windows: /config/images
[INFO] Building zip: /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/assembly-project/target/assembly-project-demo.zip
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.367 s
[INFO] Finished at: 2024-12-25T17:01:21+08:00
[INFO] ------------------------------------------------------------------------
```

**说明**：由绿色标注可以看到，assembly 的 single 目标在执行时，先读取了配置文件，然后构建了 zip 文件。

解压打成的 assembly-project-demo.zip 压缩包如下所示，可以看到和设计的目录结构一致：

![img](https://i-blog.csdnimg.cn/direct/b301f14a4ae74b74a7fdc5f62d3d3e3b.png)

**说明**：从 lib 目录可知，assembly:single 默认会将依赖范围是 compile 和 runtime 的依赖打进最终的归档文件。

**附**：

我搜索的资料，说 maven-assembly-plugin 使用最多。比如大数据项目中往往有很多 shell 脚本、SQL 脚本、.properties 及 .xml 配置文件等，采用 assembly 插件可以自定义**打包类型**、**打包的文件**、**打包的目录结构**。

- assembly 插件的主要作用：允许用户将项目输出项与其依赖项、模块、站点文档、脚本或其他文件等一起组装成一个可分发的归档文件。换句话说，用户可以个性化、选择性、定制化的打包。
- 最常用的标签：dependencySet 用来打包依赖；fileSet 用来打包文件。

参考：

- https://maven.apache.org/plugins/maven-assembly-plugin/?spm=5176.28103460.0.0.2e935d27YS2Bey
- https://maven.apache.org/plugins/maven-assembly-plugin/assembly.html
- https://zhuanlan.zhihu.com/p/690199623
- https://juejin.cn/post/6844904095245942798?searchId=20241225160224C56599BB59BAE97F66E5



### **5、spring-boot-maven-plugin**

**实验步骤：**

- 在父工程 maven-package-demo 中创建一个子模块 boot-maven-plugin-project
- 在 pom 文件中引入 spring-boot-maven-plugin 插件
- 在 pom 文件中引入 5 种依赖范围的依赖
- 执行打包命令 `mvn spring-boot:repackage -pl boot-maven-plugin-project`

pom 文件中配置如下：

```XML
<properties>
    <main-class>com.myphoenix.MyApplication</main-class>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.7.5</version>
            <configuration>
                <mainClass>${main-class}</mainClass>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

说明：

- 标红：repackage 目标的执行需要借助于 maven-jar-plugin:jar。在此必须指定 repackage，然后执行命令 `mvn package`后会执行 repackage 目标。
- 标绿：spring-boot-maven-plugin:repackage 的初衷是打一个可以运行的 fat jar，因此，建议配置好主程序类。

执行流程如下所示：

```Bash
xxx@xxxdeMacBook-Air maven-package-demo % mvn package -pl boot-maven-plugin-project
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------------< cn.myphoenix:boot-maven-plugin-project >---------------
[INFO] Building boot-maven-plugin-project 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[WARNING] The artifact mysql:mysql-connector-java:jar:8.0.33 has been relocated to com.mysql:mysql-connector-j:jar:8.0.33: MySQL Connector/J artifacts moved to reverse-DNS compliant Maven 2+ coordinates.
[INFO] 
[INFO] --- resources:3.3.1:resources (default-resources) @ boot-maven-plugin-project ---
[INFO] Copying 0 resource from src/main/resources to target/classes
[INFO] 
[INFO] --- compiler:3.13.0:compile (default-compile) @ boot-maven-plugin-project ---
[INFO] Recompiling the module because of changed source code.
[INFO] Compiling 3 source files with javac [debug target 8] to target/classes
[INFO] 
[INFO] --- resources:3.3.1:testResources (default-testResources) @ boot-maven-plugin-project ---
[INFO] skip non existing resourceDirectory /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/boot-maven-plugin-project/src/test/resources
[INFO] 
[INFO] --- compiler:3.13.0:testCompile (default-testCompile) @ boot-maven-plugin-project ---
[INFO] Recompiling the module because of changed dependency.
[INFO] 
[INFO] --- surefire:3.2.5:test (default-test) @ boot-maven-plugin-project ---
[INFO] 
[INFO] --- jar:3.4.1:jar (default-jar) @ boot-maven-plugin-project ---
[INFO] Building jar: /Users/xxx/Code_Workspace/IDEA_Workspace/maven_project/maven-package-demo/boot-maven-plugin-project/target/boot-maven-plugin-project-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot:2.7.5:repackage (default) @ boot-maven-plugin-project ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.928 s
[INFO] Finished at: 2024-12-25T18:20:09+08:00
[INFO] ------------------------------------------------------------------------
```

如下图所示，一共生成了两个包：

- boot-maven-plugin-project-1.0-SNAPSHOT.jar：spring-boot-maven-plugin 打的 fat jar 包
- boot-maven-plugin-project-1.0-SNAPSHOT.jar.original：maven-jar-plugin 打的普通 jar 包

![img](https://i-blog.csdnimg.cn/direct/17e2c17e2c9e42ca8983261dc90713a7.png)

打成的 fat jar 解压后结构如下图所示：

![img](https://i-blog.csdnimg.cn/direct/9d84dbba68794576b24dc2fe878f5ac4.png)

说明：将依赖范围是 compile、runtime 和 provided 的依赖打包了，但 test 和 system 的依赖没有打包。

> 思考：既然 Maven 提供了多个打包插件，而且功能十分强大，为啥 Spring 官方还会再提供一个 spring-boot-maven-plugin 插件？
>
> 理解：即使 Maven 官方提供了多个插件，但是默认与生命周期绑定的只有 maven-jar-plugin，它的 jar 目标绑定到了 package 阶段上，只能打普通的 jar 包，无法直接运行。Spring Boot 倡导的一个理念是约定大于配置，其提供好了一些默认的配置。repackage 承袭了 Maven 的默认绑定，并且增强了 jar 目标。



# 本地文件jar包如何在pom文件里配置？

该题目的意思是，jar 包在自己的机器上，但是不在 Maven 的本地仓库中。所以，不能直接在 pom 文件中引入这个 jar 包。

**解决方案**：

**方案一：**

- 使用 scope 标签声明此 jar 包为系统级依赖
- 使用 systemPath 标签指定 jar 包在本地机器上的路径

**方案二：**

使用`mvn install:install-file`命令将JAR包安装到本地Maven仓库。你需要提供JAR包的路径、组ID（groupId）、构件ID（artifactId）和版本号（version）。例如：

```Bash
mvn install:install-file -Dfile=<path-to-file> -DgroupId=<group-id> -DartifactId=<artifact-id> -Dversion=<version> -Dpackaging=jar
```

> 附：Maven 的设计理念是**所有依赖都应当通过仓库来管理**，使用系统路径依赖项偏离了这一原则，可能会导致构建行为不可预测。

**演示一：**

创建一个项目，编写两个类，将项目打成 jar 包：

![img](https://i-blog.csdnimg.cn/direct/b5a83e37bf274a7295c34471693ab4c0.png)

得到 jar 包 big-marketing-wheel-1.0-SNAPSHOT.jar，将此 jar 包放置到某个路径下，例如桌面 /Users/xxx/desktop/big-marketing-wheel-1.0-SNAPSHOT.jar。

可以看到，这个 jar 包并没有在本地仓库中，那么我们应该如何在项目中引入此 jar 包呢？

上面已经给出了解决方案。pom 文件配置如下：

```XML
<dependency>
    <groupId>cn.myphoenix</groupId>
    <artifactId>big-marketing-wheel</artifactId>
    <version>1.0-SNAPSHOT</version>
    <scope>system</scope>
    <systemPath>/Users/xxx/desktop/big-marketing-wheel-1.0-SNAPSHOT.jar</systemPath>
</dependency>
```

如下图所示，成功引入了位于桌面的 jar 包：

![img](https://i-blog.csdnimg.cn/direct/73b4a06ae8d64c8d8796a952ee9d9556.png)

**演示二：**

将桌面上的 jar 包安装到本地仓库，然后直接在项目中导入本地仓库的 jar 包。

初始本地仓库中没有此 jar 包：

![img](https://i-blog.csdnimg.cn/direct/a5cf869bb303440684cf4135be4618cb.png)

执行如下命令，将桌面上的 jar 包安装到本地仓库：

```Bash
mvn install:install-file -Dfile=/Users/xxx/desktop/big-marketing-wheel-1.0-SNAPSHOT.jar -DgroupId=cn.myphoenix -DartifactId=big-marketing-wheel -Dversion=1.0-SNAPSHOT -Dpackaging=jar
```

执行流程如下：

```Bash
xxx@xxxdeMacBook-Air sky-takeout % mvn install:install-file -Dfile=/Users/xxx/desktop/big-marketing-wheel-1.0-SNAPSHOT.jar -DgroupId=cn.myphoenix -DartifactId=big-marketing-wheel -Dversion=1.0-SNAPSHOT -Dpackaging=jar
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------< cn.myphoenix:sky-takeout >----------------------
[INFO] Building sky-takeout 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- install:3.1.2:install-file (default-cli) @ sky-takeout ---
[INFO] Installing /Users/xxx/desktop/big-marketing-wheel-1.0-SNAPSHOT.jar to /Users/xxx/softwares/developer/apache-maven-3.9.9/repository/cn/myphoenix/big-marketing-wheel/1.0-SNAPSHOT/big-marketing-wheel-1.0-SNAPSHOT.jar
[INFO] Installing /var/folders/lw/_dqb_2nj2pzg16m_c06g9mf80000gn/T/big-marketing-wheel-1.0-SNAPSHOT3384356351638361494.pom to /Users/xxx/softwares/developer/apache-maven-3.9.9/repository/cn/myphoenix/big-marketing-wheel/1.0-SNAPSHOT/big-marketing-wheel-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.238 s
[INFO] Finished at: 2024-12-24T20:36:09+08:00
[INFO] ------------------------------------------------------------------------
```

如下图所示，成功将桌面上的 jar 包安装到本地仓库：

![img](https://i-blog.csdnimg.cn/direct/4c52635dd8c745d5a7bafc9dbe2648a8.png)