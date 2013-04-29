#Maven安装
Maven的两个核心概念：坐标和依赖。
##Maven坐标
通过一些元素进行定义，包括groupId、artifactId、version、packaging、classifier。
##依赖范围
依赖范围主要用来控制与三种classpath（编译classpath、测试classpath、运行classpath）的关系。

- compile:	编译依赖范围
- test:		测试依赖范围
- provided:	已提供依赖范围，例如Servlet-api，编译和测试需要，但运行时由容器提供，不需重复引入
- runtime:	运行时依赖
- system：	系统依赖范围；主要涉及本机系统绑定
- import:	导入依赖范围	
##在Windows上安装Maven
###检查JDK安装

	echo %JAVA_HOME%
	java -version		#查看Java版本
###下载Maven
###本地安装


1. 将压缩包解压到指定目录中，个人建议D:/java/apache-maven-3.0.5
2. 需要设置环境变量，将Maven安装配置到操作系统环境中
	
>
- 设置MAVEN_HOME，指向Maven安装目录：D:/java/apache-maven-3.0.5
- 修改PATH变量，在末尾添加";%MAVEN_HOME%/bin
检查设置
	
	echo %MAVEN_HOME%
	mvn -v               #查看Maven版本
### Maven设置


1. 全局设置，修改%MAVEN_HOME%/conf/settings.xml
2. 用户设置（在Linux多用户情况下），拷贝%MAVEN_HOME%/conf/settings.xml到～\.m2\settings.xml，修改此文件，在用户范围内定制Maven的行为。

### ～/.m2目录
1. 在用户目录下可以发现.m2文件夹。默认情况，该文件夹下放置了Maven本地仓库.m2/repository，所有Maven构件被存储在本地此目录下，以方便在多个项目间重用。
2. 在Linux多用户环境下，用户需要复制%MAVEN_HOME%/conf/settings.xml到～\.m2\settings.xml，这是一条最佳实践。
	
	mvn help:system		#打印当前Java系统属性和环境变量

###设置HTTP代理
编辑settings.xml，添加代理配置如下:

	<proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
##Eclipse的Maven插件m2eclipse
[m2eclipse插件](http://eclipse.org/m2e/download/ "m2eclipse插件")，其中模块Maven Integrateion for WTP（Optional），可以让Eclipse自动读取POM信息，并配置WTP项目（支撑WEB应用开发）。

*备注*：需要配置Eclipse，调整vm配置指向本机JDK

##Maven安装最佳实践
###设置MAVEN_OPTS环境变量
通常需要配置MAVEN_OPTS=-Xms128m -Xmx512m，因为Java默认最大内存往往不能够不满足Maven运行的需要，否则很容易得到java.lang.OutOfMemoryError.
###不要使用IDE内嵌的Maven
###配置maven-compile-plugin支持Java5，默认只支持编译Java1.3(pom.xml配置）

				<!-- compiler插件, 设定JDK版本 -->
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>3.0</version>
					<configuration>
						<source>1.5</source>
						<target>1.5</target>
						<showWarnings>true</showWarnings>
					</configuration>
				</plugin>
				
###Maven生成可执行Jar包(pom.xml配置）
maven有两种生成可执行jar包的插件，能够自动加载依赖包。分别为 maven-assembly-plugin 和appassembler-maven-plugin。 

- appassembler-maven-plugin 的优势是能够自动生成window和linux的启动脚本
- maven-assembly-plugin 生成jar包后需要执行 java -jar **.jar命令运行jar包。
### Maven项目骨架

- 根目录中放置pom.xml
- src/main/java目录中，放置项目的主代码
- src/main/resources目录中，放置项目的的资源文件，如配置文件*.ini/*.xml/*.propeties
- src/test/java目录中，放置项目的测试代码
###排除依赖(pom.xml配置）

		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.15</version>
			<exclusions>
				<exclusion>
					<groupId>javax.jms</groupId>
					<artifactId>jms</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jdmk</groupId>
					<artifactId>jmxtools</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jmx</groupId>
					<artifactId>jmxri</artifactId>
				</exclusion>
				<exclusion>
					<groupId>javax.mail</groupId>
					<artifactId>mail</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
###归类依赖(pom.xml配置）
	
	<properties>
		<!-- 主要依赖库的版本定义 -->
		<cxf.version>2.7.4</cxf.version>
	</properties>
	<!-- SOAP begin -->
	<dependency>
		<groupId>org.apache.cxf</groupId>
		<artifactId>cxf-rt-core</artifactId>
		<version>${cxf.version}</version>
		<exclusions>
			<!-- use javax.mail.mail instead -->
			<exclusion>
				<groupId>org.apache.geronimo.specs</groupId>
				<artifactId>geronimo-javamail_1.4_spec</artifactId>
			</exclusion>
			<!-- use javax.activation.activation instead -->
			<exclusion>
				<groupId>org.apache.geronimo.specs</groupId>
				<artifactId>geronimo-activation_1.1_spec</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
		<groupId>org.apache.cxf</groupId>
		<artifactId>cxf-rt-frontend-jaxws</artifactId>
		<version>${cxf.version}</version>
		<exclusions>
			<!-- see above -->
			<exclusion>
				<groupId>org.apache.geronimo.specs</groupId>
				<artifactId>geronimo-javamail_1.4_spec</artifactId>
			</exclusion>
			<exclusion>
				<groupId>org.apache.geronimo.specs</groupId>
				<artifactId>geronimo-activation_1.1_spec</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
		<groupId>org.apache.cxf</groupId>
		<artifactId>cxf-rt-transports-http</artifactId>
		<version>${cxf.version}</version>
	</dependency>
	<!-- SOAP end -->
	
###依赖优化（pom.xml配置)

	mvn dependency:list		#查看项目已解析依赖（Resolved Dependency）
	mvn dependency:tree		#查看当前项目的依赖树
	mvn dependency:analyze	#工具帮助分析当前项目的依赖，重点关注"Used undeclared dependencies"和"Unused declared dependencies"
	
##Maven常用命令
	
	mvn clean compile	#编译
	mvn clean test		#测试
	mvn clean package	#打包
	mvn clean install	#将打包安装到Maven库（本地库/私有库等）
