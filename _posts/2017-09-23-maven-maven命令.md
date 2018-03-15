---
layout:     post
title:      maven命令
date:  2017-09-23
category:   maven
tags:   [maven]
---
### Meavn的常用命令

1. mvn clean
2. mvn compile
3. mvn test-compile
4. mvn test  :执行测试程序
5. mvn package
6. mvn install
7. mvn site 生成站点
   ※注意：运行Maven命令时一定要进入pom.xml文件所在的目录！

### Meavn联网问题

1. meavn中仅仅定义了抽象的生命周期，具体的工作需要插件来完成，插件本身不在meavn的核心程序中
2. meavn首先去本地仓库主查找插件/jar包，核心仓库的位置为【Home】/.m2/repository
3. 如果找不到，会连接外网去中央仓库找。
4. 指定本地仓库：修改apache-maven-3.2.2\conf\settings.xml中的localRepository标签

### Meavn的典型的目录结构

    项目名称	 
    	|---src
    	|---|---main
    	|---|---|---java
    	|---|---|---resources
    	|---|---test
    	|---|---|---java
    	|---|---|---resources
    	|---pom.xml

这个项目结构是必须的，通过这个结构meavn找到各个源文件的位置，编译或者执行相应的程序

### POM

pom是项目对象模型

### 坐标

meavn通过坐标，来唯一定位一个meavn工程

    	groupId： //公司名称+项目名称
    	ArtifactId：//模块名称
    	Version:版本号  SNAPSHOT在项目中，表示是不稳定版本

Meavn坐标与本地仓库中的文件路径 的对应关系

meavn坐标对应的文件在仓库中的位置是：

    groupId/ArtifactId/Version/ArtifactId-Version.jar

### 仓库

- 本地仓库
- 远程仓库
  - 私服 ：nexus
  - 中央仓库
  - 中央仓库的镜像

仓库中 的内容

- Meavn自身需要的插件
- 第三方框架或工具 的jar包
- 我们自己开发发Meavn工程

mvn需要的jar包，先在本地仓库找，然后去中央仓库找，找不到就会报出异常。

但是如果jar包存在于其他仓库，可以通过以下方式配置，这样上面的步骤找不到后，还会去指定仓库在找一次

        <repositories>
            <repository>
                <id>java.net</id>
                <url>https://maven.java.net/content/repositories/public/</url>
            </repository>
        </repositories>

### 依赖·

- 对于我们自己编写的meavn工程，可以使用install，把项目安装到本地仓库中
- 依赖的范围
       <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.11</version>
          <!-- 使用scope表示依赖的范围 -->
          <scope>test</scope>
        </dependency>
- complie  主程序、测试程序都是可见的，参与打包，就是对于程序而言，全程都是有效的
- test        只有测试程序是可见的，测试程序是不参与打包和部署的的，只有test程序可以用，例如junit
- provided
  provide一般在web项目中使用，web项目最后运行在tomcat上，tomcat本身提供了一些jar包，如servlet-api，这些jar包在开发测试的过程中，没有部署服务器，那就是看不到的。因此需要引入，但是部署之后，因为服务器上还有一份jar，就冲突了，应该使用provided。

    provided对于主程序和测试程序都有效，不参与打包，部署的时候不可见。

- 依赖的传递性
  工程A导入B作为依赖jar包，那么B所以来的jar包会随着依赖的导入也传入到工程A中，就是说，工程A可以使用B所导入的所有的jar包。
  - 好处：重复的jar只需要引入一次就可以了
  - 注意：只有compile范围的依赖可以传递
- 依赖的排除
  依赖的传递性引入的jar包中的某些jar如果不想要的话，可以在<dependency>配置的时候使用<exclusions>排除掉，只需要指定groupId和artifactId，不论是什么版本都能够拿掉
      <dependency>
        <groupId>com.zview.meavn</groupId>
        <artifactId>meavn01</artifactId>
        <version>0.0.1</version>
        <exclusions>
          <exclusion>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
          </exclusion>
        </exclusions>
      </dependency>
- 依赖的原则
  当出现版本依赖冲突的时候，meavn通过两个原则来选择一个jar包
  - 路径最短者优先，就是依赖传递的次数越少的越优先
  - 路径相同时，先声明 的优先

### 生命周期

- 生命周期的顺序不能打乱
- 生命周期各个阶段的任务是通过插件来完成的
- 无论执行什么命令，都是从生命周期的最开始的位置开始执行到命令的位置
- 插件和目标
  - 生命周期的各个阶段仅仅是定义了要执行什么任务
  - 各个阶段和插件的目标是对应的
  - 相似的目标由同一个插件来执行比如test和test-compile都是meavn-compile-plugin来执行的

Mevan有三套独立的声明周期，每个声明周期又有不同的阶段

执行任何一个阶段的时候，它前面的所有阶段都会被运行， 例如我们运行 mvn install 的时候，代码会

被编译，测试，打包 。

1. Clean 生命周期

        ①pre-clean 执行一些需要在 clean 之前完成的工作

        ②clean 移除所有上一次构建生成的文件

        ③post-clean 执行一些需要在 clean 之后立刻完成的工作

1. Site 生命周期

       ①pre-site 执行一些需要在生成站点文档之前完成的工作 

       ②site 生成项目的站点文档 

       ③post-site 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 

       ④site-deploy 将生成的站点文档部署到特定的服务器上

      这里经常用到的是 site 阶段和 site-deploy 阶段，用以生成和发布 Maven 站点，这可是 Maven 相当强大

的功能， Manager 比较喜欢，文档及统计数据自动生成，很好看。

1. Default 生命周期
   Default 生命周期是 Maven 生命周期中最重要的一个，绝大部分工作都发生在这个生命周期中。这里只解释一些比较重要和常用的阶段：
   validate
   generate-sources
   process-sources
   generate-resources
   process-resources 复制并处理资源文件，至目标目录，准备打包。
   compile 编译项目的源代码。
   process-classes
   generate-test-sources
   process-test-sources
   generate-test-resources
   process-test-resources 复制并处理资源文件，至目标测试目录。
   test-compile 编译测试源代码。
   process-test-classes
   test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
   prepare-package
   package 接受编译好的代码，打包成可发布的格式，如 JAR。
   pre-integration-test
   integration-test
   post-integration-test
   verify
   install 将包安装至本地仓库，以让其它项目依赖。
   deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享或部署到服务器上运行。 

构建过程的各个环节

①清理：删除以前的编译结果，为重新编译做好准备。

②编译：将 Java 源程序编译为字节码文件。

③测试：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性。

④报告：在每一次测试后以标准的格式记录和展示测试结果。

⑤打包：将一个包含诸多文件的工程封装为一个压缩文件用于安装或部署。 Java 工程对应

工程对应 war 包。

⑥安装：在 Maven 环境下特指将打包的结果——jar 包或 war 包安装到本地仓库中。

⑦部署：将打包的结果部署到远程仓库或将 war 包部署到服务器上运行。 

使用properties

properties标签可以定义任何的变量，但是大多数时候还是使用它来处理版本号

    <properties>
      <!-- spring版本号 -->
      <spring.version>4.0.2.RELEASE</spring.version>
      <!-- mybatis版本号 -->
      <mybatis.version>3.2.6</mybatis.version>
      <!-- log4j日志文件管理包版本 -->
      <slf4j.version>1.7.7</slf4j.version>
      <log4j.version>1.2.17</log4j.version>
      <mockito.core.version>2.0.5-beta</mockito.core.version>
    </properties>
     <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-core</artifactId>
          <version>${spring.version}</version>
     </dependency>

### 继承

上面的依赖的传递性，如果不去重复的配置同一个jar包依赖，就能够保证整个工程中的jar包的版本号一致，但是对于不参与传递的作用域的jar包，就要一个一个去写了，这样不同 的meavn工程中 的jar包的版本号就不一样了，为了统一这一点，编写一个父meavn pom，让其他的meavn工程指定其为父工程就可以了。同时继承可以简化一些内容上的编写。

就是说将统一 的依赖提取到父工程中,这样子工程就能够直接引用父工程引入的依赖了。

子工程pom配置：

    <parent>
        <groupId>com.zview</groupId>
        <artifactId>meavn-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    
    <artifactId>meavn01</artifactId>

可以看到工程坐标只有artifactId，如果子工程语父工程本身就是同一个项目下的，那么子工程的groupid和version可以省略，因为和父工程一致。当然不是一个项目里面的这些都是可以写的。

这样父亲工程里面的依赖就是可以直接使用的。

- 依赖管理
  上面说的子类可以直接使用父类引入的依赖，但是如果父类并不想要引入，而仅仅是供给子类使用的话，那么父类中将这些依赖放在 <dependencyManagement> 中定义。子类需要引用 的时候使用<dependency> 正常引入，不需要指定版本号，也就是说，确保同一个工程里面的版本都是一样的
      父工程配置依赖管理器
      <dependencyManagement>
          <dependencies>
              <dependency>
                  <groupId>junit</groupId>
                  <artifactId>junit</artifactId>
                  <version>4.10</version>
                  <scope>test</scope>
              </dependency>
          </dependencies>
      </dependencyManagement>
      子工程引入依赖，不需要写版本号，而且scope也是继承过来了
      <dependencies>
        <dependency>
           <groupId>junit</groupId> 
           <artifactId>junit</artifactId>
        </dependency>
      </dependencies>
- 插件管理
  与依赖管理的逻辑完全相同
      父模块中定义插件
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
                              <phase>verify<phase>
                              <goals>
                                  <goal>jar-no-fork</goal>
                              </goals>
                          </exeuction>
                      </executions>
                  </plugin>
              </plugins>
          </pluginManagement>
      </build>
      
      子模块中调用插件
      <build>
          <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-source-plugin</artifactId>
              </plugin>
          </plugins>
      </build>

### 聚合

聚合

### 继承和聚合的关系

- 聚合是为了方便快速构件项目。对于聚合模块来说，它知道有哪些被聚合的模块，但那些模块不知道这个聚合模块的存在；
- 继承是为了消除重复配置。对于继承关系的父POM来说，它不知道有哪些子模块继承于它，但是子模块必须知道自己的父POM是什么。
- 在实际的项目中，一个POM可以既是聚合POM，又是父POM。
  如果父模块位于子模块们的上级，则不需要再额外配置relativePath，因为Maven默认能识别父模块的位置。

### meavn deploy

==自动化构建

### 其他

在IDE中使用meavn最好使用自己的meavn核心程序，指定自己 meavn仓库

- IDEA导入meavn工程
  File-new-Module Form existing source-选择pom文件即可
- mMaven 可以替我们自动的将当前 jar 包所依赖的其他所有 jar 包全部导入进来 ，因此有时候写了十几个依赖，实际上引入的jar包可能有几十个。例如:导入junit的依赖，同时也会引入hsmcredt-core的jar包，因为前者是依赖于后者的。

---

maven插件

一个插件有多个目标，下面的语法指定执行插件的某一个目标

    mvn [plugin-name]:[goal-name]

meavn有两种插件

  构建插件	在生成过程中执行，并在 pom.xml 中的<build/> 元素进行配置   
  报告插件	在网站生成期间执行，在 pom.xml 中的 <reporting/> 元素进行配置

插件在<pulgins>中指定

Pom基本配置项目

      <modelVersion>4.0.0</modelVersion>
      <groupId>...</groupId>
      <artifactId>...</artifactId>
      <version>...</version>
      <packaging>...</packaging>
      <dependencies>...</dependencies>
      <parent>...</parent>
      <dependencyManagement>...</dependencyManagement>
      <modules>...</modules>
      <properties>...</properties>

modelVersion：modelVersion 指定 pom.xml 符合哪个版本的描述符。maven 2 和 3 只能为 4.0.0

packaging：项目的类型，描述了项目打包后的输出，默认是 jar。常见的输出类型为：pom, jar, maven-plugin, war, ear, rar, par。

依赖配置

    <dependencies>
        <dependency>
         <groupId>org.apache.maven</groupId>
          <artifactId>maven-embedder</artifactId>
          <version>2.0</version>
          <type>jar</type>
          <scope>test</scope>
          <optional>true</optional>
          <exclusions>
            <exclusion>
              <groupId>org.apache.maven</groupId>
              <artifactId>maven-core</artifactId>
            </exclusion>
          </exclusions>
        </dependency>
        ...
      </dependencies>

type：对应 packaging 的类型，如果不使用 type 标签，maven 默认为 jar

scope：此元素指的是任务的类路径（编译和运行时，测试等）以及如何限制依赖关系的传递性

其他标签

`build`标签：下面的filename，表示构建最终打的包的名称




