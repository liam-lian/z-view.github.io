---
layout:     post
title:      Mybatis逆向工程
date:  2017-09-23
category:   Mybatis
tags:   [Mybatis]
---
## Mybatis 逆向工程（Mybatis Generator）

通过mybatis逆向工程可以快速的根据数据库表生成相应的Model、mapper接口、mapperXML文件，不需要在手动编写这些代码了。

生成过程需要以下的步骤

### 编写XML文件
---

xml文件主要作用是告诉MBG：

- How to connect to the database
- What objects to generate, and how to generate them
- What tables should be used for object generation

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE generatorConfiguration
            PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
            "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
    
    <generatorConfiguration>
        <!--导入属性配置-->
        <properties resource="generator.properties"></properties>
    
        <!--指定特定数据库的jdbc驱动jar包的位置-->
        <classPathEntry location="${jdbc.driverLocation}"/>
    
        <context id="DB2Tables" targetRuntime="MyBatis3">
    
            <!-- optional，旨在创建class时，对注释进行控制，这样不会生成大量注释 -->
            <commentGenerator>
                <property name="suppressDate" value="true"/>
                <property name="suppressAllComments" value="true"/>
            </commentGenerator>
    
            <!--jdbc的数据库连接 -->
            <jdbcConnection
                    driverClass="com.mysql.jdbc.Driver"
                    connectionURL="jdbc:mysql:///FORTEST?allowMultiQueries=true"
                    userId="root"
                    password="123321">
            </jdbcConnection>
    
    
            <!-- 非必需，类型处理器，在数据库类型和java类型之间的转换控制-->
            <javaTypeResolver>
                <property name="forceBigDecimals" value="false"/>
            </javaTypeResolver>
    
    
            <!-- Model模型生成器,用来生成含有主键key的类，记录类 以及查询Example类
                targetPackage     指定生成的model生成所在的包名
                targetProject     指定在该项目下所在的路径
            -->
            <javaModelGenerator targetPackage="com.zview.model"
                                targetProject="src/main/java">
    
                <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
                <property name="enableSubPackages" value="false"/>
                <!-- 是否对model添加 构造函数 -->
                <property name="constructorBased" value="true"/>
                <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
                <property name="trimStrings" value="true"/>
                <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
                <property name="immutable" value="false"/>
            </javaModelGenerator>
    
            <!--Mapper映射文件生成所在的目录 为每一个数据库的表生成对应的SqlMap文件 -->
            <sqlMapGenerator targetPackage="config.mapper"
                             targetProject="src/main/java">
                <property name="enableSubPackages" value="false"/>
            </sqlMapGenerator>
    
            <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码
                    type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
                    type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象
                    type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口
            -->
            <javaClientGenerator targetPackage="com.zview.dao"
                                 targetProject="src/main/java" type="XMLMAPPER">
                <property name="enableSubPackages" value="true"/>
            </javaClientGenerator>
    
    
            <table tableName="reguser" domainObjectName="User"
                   enableCountByExample="false" enableUpdateByExample="false"
                   enableDeleteByExample="false" enableSelectByExample="false"
                   selectByExampleQueryId="false">
            </table>
        </context>
    </generatorConfiguration>

### 运行
---

使用插件

    <build>
            <finalName>zsxt</finalName>
            <plugins>
                <plugin>
                    <groupId>org.mybatis.generator</groupId>
                    <artifactId>mybatis-generator-maven-plugin</artifactId>
                    <version>1.3.2</version>
                    <configuration>
                        <configurationFile>MBG.xml</configurationFile>
                        <verbose>true</verbose>
                        <overwrite>true</overwrite>
                    </configuration>
                </plugin>
            </plugins>
        </build>

之后添加一个添加一个“Run运行”选项

在meaven中输入：mybatis-generator:generate -e

运行，得出结果。

注意上面的插件中的configurationFile配置的是配置文件的位置，起始位置是项目的根目录。如果不去配置这个属性，默认的配置文件的位置是：





### XXExample
---

在使用 targetRuntime="MyBatis3”来生成代码的时候，会顺便产生一个名为XXExample的类，这个类主要用于各种复杂情况的查询，即指定sql语句的条件，主要是指定where后面的语句。

#### XXExample类的解析

     protected List<Criteria> oredCriteria;  
     //Criteria是where后面的条件，这个list指定的是多个相互之间是或关系的Criteria
     //Criteria 内部类，用来指定一个where条件，使用andXX方法来指定各种条件，
     //其中包含着Criterion的list，list中的各个条件之间是and的关系
     //Criterion 内部类，用来指定Criteria中的多个and只讲的一个个条件
    
    上面的三个类就是完成where条件的关键，其中Criteria之间是or的关系，Criterion之间是and的关闭，三者互相包含

除此之外XXExample类包含以下主要方法：

    example.setOrderByClause(String orderByClause);设置order by的语句(“字段名 ASC”);
    example.setDistinct(false)	去除重复，boolean型，true为选择不重复的记录
    new Example().createCriteria(); 
    //创建一个新的criteria，对于第一个创建的criteria会被默认的加入到List<Criteria> oredCriteria中，
    //后续的则需要通过or函数来手动添加

where后面的所有的条件都可以使用Criteria进行设置，包括大于小于等于，Between,in，模糊等等

    criteria.andXxxIn(List<？>)	添加xxx字段值在List<？>条件
    criteria.andXxxLike(“%”+value+”%”)	添加xxx字段值为value的模糊查询条件
    criteria.andXxxBetween(value1,value2)	添加xxx字段值在value1和value2之间条件

再次注意Criteria层面的都是and，or是在Example层面上的。

两个例子：

    ---例子1---
    studentExample studentExample=new studentExample();
    studentExample.Criteria criteria=studentExample.createCriteria();
    criteria.andIdLessThan(10086);
    criteria.andNameEqualTo("lz");
    studentExample.or().andCIdEqualTo(1);
    List list=studentMapper.selectByExample(studentExample);
    --例子2--
    studentExample studentExample=new studentExample();
    studentExample.Criteria criteria=studentExample.createCriteria();
    criteria.andIdLessThan(10086);
    criteria.andNameEqualTo("lz");
    studentExample.Criteria criteria1=studentExample.createCriteria();
    List list=studentMapper.selectByExample(studentExample);
    上面两个例子都是等价的，对应 的sql语句如下。
    select id, name, gender, addr, c_id from student WHERE ( id < ? and name = ? ) or( c_id = ? ) 

最后说明一下XXMapper的方法：

    int countByExample(studentExample example);
    
    int deleteByExample(studentExample example);
    
    int deleteByPrimaryKey(Integer id);
    
    int insert(student record);
    
    int insertSelective(student record);
    
    List<student> selectByExample(studentExample example);
    
    student selectByPrimaryKey(Integer id);
    
    int updateByExampleSelective(@Param("record") student record, @Param("example") studentExample example);
    
    int updateByExample(@Param("record") student record, @Param("example") studentExample example);
    
    int updateByPrimaryKeySelective(student record);
    
    int updateByPrimaryKey(student record);

这里面都非常简单，只需要考虑XXSelective的方法

int insertSelective(student record);   插入数值。直插入赋了值的域

int updateByPrimaryKeySelective(student record) ；//只更新赋了值的域，不会把record中的空也更新到数据库
