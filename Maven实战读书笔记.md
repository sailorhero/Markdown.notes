#Maven简介
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

##Maven仓库
坐标和依赖是任何一个构件在Maven世界中的逻辑表示方式，构件的物理表示方式是文件，Maven通过仓库来统一管理这些文件。

-仓库布局 大致为：groupId/artifactId/version/artifactId-version.packaging

-仓库分类 对Maven来说，只分为本地参考和远程仓库。Maven查找构件次序：优先本地仓库，本地无则去远程仓库查找。*私服是一种特殊的远程仓库，为节省带宽和时间，应在局域网内假设一个私服*

###本地仓库
自定义本地仓库目录位置，例如C盘空间不足等。
修改～/.m2/setting.xml，设置localRespository元素的值

	<localRepository>D:/Java/apache-maven-3.0.5/local_repos</localRepository>
默认情况下，～/.m2/settings.xml文件不存在，用户需要从Maven的安装位置，复制一份%MAVEN_HOME%/conf/settings.xml拷贝再进行编辑。

### 从仓库解析依赖的机制

1. 当依赖范围是system时，Maven直接从本地文件系统解析构件
2. 根据依赖坐标计算仓库路径后，优先尝试直接从本地仓库寻找构件
3. 本地不存在，如果依赖版本是显式的发布版本构件，如1.2、1.2-beta-1等，则遍历所有远程仓库
4. 如果依赖的版本是RELEASE或者LATEST，则基于更新策略读取所有远程仓库的元数据groupId/artifactId/maven-metadata.xml,将其与本地仓库的对应元数据合并后，计算出RELEASE或者LATEST真实值，然后基于这个真实值检查本地和远程仓库
5. 如果依赖的版本是SNAPSHOT，则基于更新策略读取所有远程仓库的元数据groupId/artifactId/version/maven-metadata.xml,与本地元数据合并后得到最新快照版本的值，然后检查本地仓库，或者从远处仓库下载
6. 如果最后解析得到的构件版本是时间戳格式的快照，如1.4.1-20091104.121450-121,则复制其时间戳格式的文件至非时间戳格式，如SNAPSHOT，并使用该非时间戳格式的构件。

*当依赖的版本不明晰的时候，如RELEASE/LATEST/SNAPSHOT，Maven需要基于更新远程仓库的更新策略来检查更新*

### 仓库搜索服务
使用Maven进行日常开发，常见问题就是如何寻找需要的依赖，我们可能只知道需要使用类库的项目名称，但添加Maven依赖需要提供确切的Maven坐标；我们可以通过使用仓库搜索服务来根据关键字得到Maven坐标。

1. [MVNrepository](http://mvnrepository.com/)
2. [Sonatype Nexus](http://repository.sonatype.org/)

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

###私服Nexus

####私服的好处

- 节省外网带宽
- 加速Maven构建
- 部署自有及第三方构件，如Oracle的JDBC驱动
- 提高稳定性，增强控制

####部署至远程仓库

- 修改settings.xml，添加远程仓库认证

    	<servers>  
    	  <server>  
    		<id>nexus-releases</id>  
    		<username>admin</username>  
    		<password>xxxxxx</password>  
    	  </server>  
    	  <server>  
    		<id>nexus-snapshots</id>  
    		<username>admin</username>  
    		<password>xxxxxx</password>  
    	  </server> 
    	  <server>  
    		<id>thirdparty</id>  
    		<username>admin</username>  
    		<password>xxxxxxx</password>  
    	  </server> 
    	</servers>	

- 修改pom.xml配置，指定部署的远程仓库位置

    	<project>  
    	...  
    	<distributionManagement>  
      	<repository>  
    	<id>nexus-releases</id>  
    	  <name>Nexus Release Repository</name>  
       	   <url>http://127.0.0.1:8080/nexus/content/repositories/releases/</url>  
      	</repository>  
      	<snapshotRepository>  
    	<id>nexus-snapshots</id>  
       	 <name>Nexus Snapshot Repository</name>  
    	<url>http://127.0.0.1:8080/nexus/content/repositories/snapshots/</url>  
      	</snapshotRepository>  
    	</distributionManagement>  
    	...  
    	</project>
	
## Maven生命周期和插件
除了坐标、依赖以及仓库外，Maven另外两个核心概念是生命周期和插件。

### Maven的生命周期
Maven的生命周期就是对所有的构建过程进行抽象和统一。
>生命周期包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有构建步骤。
>Maven生命周期是抽象的，生命周期本身不做任何实际的工作，实际任务都交由插件来完成。

#### 三套生命周期
Maven拥有三套相互独立的生命周期，分别为clean,default和site.

####clean生命周期：	目的是清理项目,包含三个阶段
1. pre-clean		执行清理前需要完成的工作
2. clean     		清理上一次构建生成的文件
3. post-clean		执行清理后需要完成的工作

#### default生命周期：	目的是构建项目
定义真正构建时所需要执行的所有步骤，重要阶段如下：

1. validate
2. initialize
3. generate-sources
4. process-sources   #处理项目主资源文件。一般对src/main/resources目录的内容进行变量替换等工作后，复制到项目输出的主classpath目录中。
5. generate-resources
6. process-resources 
7. compile			#编译项目的主源码，一般是编译src/main/java目录下的Java文件到项目输出的主classpath目录中
8. process-classes
9. generate-test-sources  #处理项目测试资源文件，一般对src/test/resources目录下内容进行变量替换等工作后，复制到输出的测试classpath目录中。
10. generate-test-resources
11. process-test-resources
12. test-compile    #编译项目测试代码。一般编译src/test/java目录下的Java文件到项目输出的测试classpath目录中。
13. process-test-classes
14. test            #使用单元测试框架运行测试，测试代码不会被打包或部署
15. prepare-package
16. package         #接受编译好的代码，打包成可发布的格式
17. pre-integration-test
18. integration-test
19. post-integration-test
20. verify
21. install         #将包安装到Maven本地仓库
22. deploy          #将最终的包复制到远程仓库，供其他开发人员和Maven项目使用
详细信息，可参阅 [Maven官方的解释](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

#### site生命周期
目的是建立和发布项目的站点，Maven能够基于POM所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息。

1. pre-site
2. site        #生成项目站点文档
3. post-site
4. site-deploy #将生成的项目站点发布到服务器上。

![内置绑定关系1](Maven实战读书笔记.images/内置绑定关系1.png) 

![内置绑定关系2](Maven实战读书笔记.images/内置绑定关系2.png) 

### 插件配置

1. 命令行插件配置
例如针对测试插件maven-surefire-plugin，提供了maven.test.skip参数

    mvn install -Dmaven.test.skip=true 	#跳过测试

2. POM中插件全局配置
例如配置maven-compiler-plugin，编译java1.5版本的源文件，生成与jvm 1.5兼容的字节码文件。

### 在线插件信息
1. [Apache官方插件](http://maven.apache.org/plugins/index.html)
### Maven插件的聚合与继承的关系
多模块Maven项目中，聚合与继承是两个概念，聚合是为了方便快速构建多个项目，继承是为了消除重复配置。

聚合模块知道那些模块被聚合，但被聚合的模块不知道这个聚合模块的存在。

继承关系的父POM，它不知道哪些子模块继承于它，但子模块必须知道自己的父POM是什么。

聚合POM与继承关系中的父POM的packaging属性必须是pom，同时，聚合模块与继承关系中的父模块除了POM之外，都没有实际的内容。如图：

![聚合与继承POM关系](Maven实战读书笔记.images/聚合与继承POM关系.png) 

## Maven常用命令
	
	mvn clean compile	#编译
	mvn clean test		#测试
	mvn clean package	#打包
	mvn clean install	#将打包安装到Maven库（本地库）
	mvn clean deploy    #将项目勾结输出的构件部署到远程仓库

	
