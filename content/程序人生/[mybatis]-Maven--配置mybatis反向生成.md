---
title: 配置mybatis反向生成
---
1 配置pom.xml 

新建Maven项目，在pom.xml文件中配置
```xml
<build> 
 <plugins> 
 <plugin> 
 <groupId>org.mybatis.generator</groupId> 
 <artifactId>mybatis-generator-maven-plugin</artifactId> 
 <version>1.3.2</version> 
 <configuration> 
 <verbose>true</verbose> 
 <overwrite>true</overwrite> 
 </configuration> 
</plugin> 
 </plugins> 
 </build> 
```
配置generatorConfig.xml 

生成的xml存放在src\main\resources路径下
```xml 
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE generatorConfiguration 
 PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" 
 "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd"> 

<generatorConfiguration> 

<!-- 数据库驱动--> 

 <classPathEntry location="mysql-connector-java-5.1.25-bin.jar"/> 
 <context id="DB2Tables" targetRuntime="MyBatis3"> 

 <commentGenerator> 
 <property name="suppressDate" value="true"/> 
 <!-- 是否去除自动生成的注释 true：是 ： false:否 --> 
 <property name="suppressAllComments" value="true"/> 

 </commentGenerator> 

 <!--数据库链接URL，用户名、密码 --> 

 <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://192.168.1.100:3306/XMAN" userId="root" password="yunji123"> 
 </jdbcConnection> 

 <javaTypeResolver> 
 <property name="forceBigDecimals" value="false"/> 
 </javaTypeResolver> 

 <!-- 生成模型的包名和位置--> 
 <javaModelGenerator targetPackage="mybatis.pojo" targetProject="src"> 
 <property name="enableSubPackages" value="true"/> 
 <property name="trimStrings" value="true"/> 
 </javaModelGenerator> 

 <!-- 生成映射文件的包名和位置--> 
 <sqlMapGenerator targetPackage="mybatis.mapping" targetProject="src"> 
 <property name="enableSubPackages" value="true"/> 
 </sqlMapGenerator> 

 <!-- 生成DAO的包名和位置--> 
 <javaClientGenerator type="XMLMAPPER" targetPackage="mybatis.dao" targetProject="src"> 
 <property name="enableSubPackages" value="true"/> 
 </javaClientGenerator> 

 <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名--> 
 <table tableName="tb_config" domainObjectName="Config" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>

 </context> 

</generatorConfiguration> 
```
3.执行maven命令 mvn mybatis-generator:generate 生成Mybatis文件 

编写好的项目：[https://git.oschina.net/yihyforever/mybatis-generator](https://git.oschina.net/yihyforever/mybatis-generator)
