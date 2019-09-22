# Mybatis理解

#### springboot集成Mybatis、Mybatis-generator

1. pom.xml配置

   ~~~xml
   <plugin>
   	            <groupId>org.mybatis.generator</groupId>
   	            <artifactId>mybatis-generator-maven-plugin</artifactId>
   	            <version>1.3.2</version>
   	            <executions>
   	                <execution>
   	                    <id>Generate MyBatis Artifacts</id>
   	                    <phase>deploy</phase>
   	                    <goals>
   	                        <goal>generate</goal>
   	                    </goals>
   	                </execution>
   	            </executions>
   	            <configuration>
   	                <!-- generator 工具配置文件的位置 -->
   	                <configurationFile>src/main/resources/mybatis-generator/generator-config.xml</configurationFile>
   	                <verbose>true</verbose>
   	                <overwrite>true</overwrite>
   	            </configuration>
   	            <dependencies>
   	                <dependency>
   	                    <groupId>mysql</groupId>
   	                    <artifactId>mysql-connector-java</artifactId>
                        <!-- 注意版本号，跟安装的sql版本号一致 -->
   	                    <version>8.0.16</version>  
   	                </dependency>
   	                <dependency>
   	                    <groupId>org.mybatis.generator</groupId>
   	                    <artifactId>mybatis-generator-core</artifactId>
   	                    <version>1.3.2</version>
   	                </dependency>
   	            </dependencies>
   	        </plugin>
   ~~~

   

2. generator.xml配置

   ~~~xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE generatorConfiguration
           PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
           "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
   <!-- 配置生成器 -->
   <generatorConfiguration>
       <!--执行generator插件生成文件的命令： call mvn mybatis-generator:generate -e -->
       <!-- 引入配置文件 -->
       <properties resource="mybatis-generator/mybatis-generator-init.properties"/>
       <!--classPathEntry:数据库的JDBC驱动,换成你自己的驱动位置 可选 -->
       <!--<classPathEntry location="D:\generator_mybatis\mysql-connector-java-5.1.24-bin.jar" /> -->
   
       <!-- 一个数据库一个context -->
       <!--defaultModelType="flat" 大数据字段，不分表 -->
       <context id="MysqlTables" targetRuntime="MyBatis3Simple" defaultModelType="flat">
           <!-- 自动识别数据库关键字，默认false，如果设置为true，根据SqlReservedWords中定义的关键字列表；
           一般保留默认值，遇到数据库关键字（Java关键字），使用columnOverride覆盖 -->
           <property name="autoDelimitKeywords" value="true" />
           <!-- 生成的Java文件的编码 -->
           <property name="javaFileEncoding" value="utf-8" />
           <!-- beginningDelimiter和endingDelimiter：指明数据库的用于标记数据库对象名的符号，比如ORACLE就是双引号，MYSQL默认是`反引号； -->
           <property name="beginningDelimiter" value="`" />
           <property name="endingDelimiter" value="`" />
   
           <!-- 格式化java代码 -->
           <property name="javaFormatter" value="org.mybatis.generator.api.dom.DefaultJavaFormatter"/>
           <!-- 格式化XML代码 -->
           <property name="xmlFormatter" value="org.mybatis.generator.api.dom.DefaultXmlFormatter"/>
           <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
   
           <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
   
           <!-- 注释 -->
           <commentGenerator >
               <property name="suppressAllComments" value="true"/><!-- 是否取消注释 -->
               <property name="suppressDate" value="true" /> <!-- 是否生成注释代时间戳-->
           </commentGenerator>
   
           <!-- jdbc连接 -->
           <jdbcConnection driverClass="${jdbc_driver}" connectionURL="${jdbc_url}" userId="${jdbc_user}" password="${jdbc_password}" />
           <!-- 类型转换 -->
           <javaTypeResolver>
               <!-- 是否使用bigDecimal， false可自动转化以下类型（Long, Integer, Short, etc.） -->
               <property name="forceBigDecimals" value="false"/>
           </javaTypeResolver>
   
           <!-- 生成实体类地址 -->
           <javaModelGenerator targetPackage="com.example.demo.dao.entity" targetProject="${project}" >
               <property name="enableSubPackages" value="false"/>
               <property name="trimStrings" value="true"/>
           </javaModelGenerator>
           <!-- 生成mapxml文件 -->
           <sqlMapGenerator targetPackage="mappers" targetProject="${resources}" >
               <property name="enableSubPackages" value="false" />
           </sqlMapGenerator>
           <!-- 生成mapxml对应client，也就是接口dao -->
           <javaClientGenerator targetPackage="com.example.demo.dao" targetProject="${project}" type="XMLMAPPER" >
               <property name="enableSubPackages" value="false" />
           </javaClientGenerator>
           <!-- table可以有多个,每个数据库中的表都可以写一个table，tableName表示要匹配的数据库表,也可以在tableName属性中通过使用%通配符来匹配所有数据库表,只有匹配的表才会自动生成文件 -->
           <table tableName="d_account" domainObjectName="Account" enableUpdateByExample="true" enableDeleteByExample="true" enableSelectByExample="true" selectByExampleQueryId="true">
               <property name="useActualColumnNames" value="false" />
               <!-- 数据库表主键 -->
               <generatedKey column="id" sqlStatement="Mysql" identity="true" />
           </table>
           
       </context>
   </generatorConfiguration>
   ~~~

   

3. mybatis-generator-init.properties配置

   ~~~
   #Mybatis Generator configuration
   #dao类和实体类路径
   project =src/main/java
   #mapper映射文件路径
   resources=src/main/resources
   
   jdbc_driver =com.mysql.jdbc.Driver
   jdbc_url=jdbc:mysql://127.0.0.1:3306/demo
   jdbc_user=root
   jdbc_password=password
   ~~~

   

4. 执行mvn指令自动生成对应实体类、xml文件

~~~
pom.xml文件所在的目录执行如下命令:
mvn mybatis-generator:generate
~~~

