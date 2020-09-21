# maven相关学习

### pom.xml相关配置

~~~xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.9.9</version>
        </dependency>
    </dependencies>
</dependencyManagement>
~~~

场景：当项目中用到多个jar包，间接引用到jackson-core包，这个时候需要统一管理此jar包版本，可以使用<dependencyManagement>标签。

~~~xml
<dependency>
    <groupId>com.qcloud</groupId>
    <artifactId>cos_api</artifactId>
    <version>${qcloud.cos_api.version}</version>
    <exclusions>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>
~~~

场景：当项目中多个jar包，产生类冲突，可以使用 <exclusions>标签移除没有使用的依赖jar包。



~~~xml
 <dependency>
            <groupId>ICEPDF</groupId>
            <artifactId>core</artifactId>
            <version>6.1.2</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/lib/icepdf-core.jar</systemPath>
        </dependency>
~~~

场景：依赖本地第三方jar



~~~xml
<dependencies>
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
<build>
	<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
	</plugins>
</build>

~~~

<dependencies>依赖jar包的标签，<build><plugins>依赖工具包。



### springboot打包外部依赖包

~~~xml
<resources>
			<resource>
				<directory>${project.basedir}/src/main/resources/lib</directory>
				<targetPath>BOOT-INF/lib/</targetPath>
				<includes>
					<include>**/*.jar</include>
				</includes>
			</resource>
			<resource>
				<directory>src/main/resources</directory>
				<targetPath>BOOT-INF/classes/</targetPath>
			</resource>
		</resources>
~~~



### maven上传本地仓库及远程库

~~~shell
mvn install:install-file -Dfile=C:/Users/PF0W8JF8/workspace/api_V2.2.7/src/main/webapp/WEB-INF/lib/szcaservice.jar -DgroupId=com.shzhca -DartifactId=szcaservice -Dversion=2.7 -Dpackaging=jar 

mvn deploy:deploy-file -DgroupId=com.shzhca -DartifactId=szcaservice -Dversion=2.7 -Dpackaging=jar -Dfile=C:/Users/PF0W8JF8/workspace/api_V2.2.7/src/main/webapp/WEB-INF/lib/szcaservice.jar -Durl=http://127.0.0.1:8080/nexus/content/repositories/kk_repostiory/ -DrepositoryId=kk_repostiory 
~~~

mvn install：安装到本地仓库，mvn deploy：部署到远程私服仓库

