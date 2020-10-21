### 父控pom文件

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>


  <groupId>com.gpmall.parent</groupId>
  <artifactId>gpmall-parent</artifactId>
  <version>1.0-SNAPSHOT</version>

  <!-- 所有的父级项目的packing都为pom，packing默认是jar类型，如果不作配置，maven会将该项目打成jar包-->
  <packaging>pom</packaging>
  <name>gpmall-parent</name>

  <!-- 依赖包所有的文件版本在此定义，后续引用如果需要更改版本需要自定义版本，可覆盖这里定义的版本 -->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring-boot.version>2.1.6.RELEASE</spring-boot.version>
    <dubbo.version>2.7.3</dubbo.version>
    <dubbo-spring-boot.version>2.7.3</dubbo-spring-boot.version>
    <curator.famework.version>4.2.0</curator.famework.version>
    <mybatis.version>2.1.0</mybatis.version>
    <tkmybatis.version>3.4.2</tkmybatis.version>
    <tkmybatis.springboot.version>2.1.5</tkmybatis.springboot.version>
    <druid.version>1.1.19</druid.version>
    <gpmall.commons>1.0-SNAPSHOT</gpmall.commons>
    <mapstruct.version>1.3.0.Final</mapstruct.version>
    <fastjson.version>1.2.56</fastjson.version>
    <jwt.version>3.8.1</jwt.version>
    <user-api.version>0.0.1-SNAPSHOT</user-api.version>
    <joda-time.version>2.10.3</joda-time.version>
    <redisson.version>3.11.1</redisson.version>
    <pagehelper.version>5.1.10</pagehelper.version>
    <pagehelper.sprinboot.version>1.2.12</pagehelper.sprinboot.version>
    <swagger-starter.version>1.8.0.RELEASE</swagger-starter.version>
    <validation-api.version>2.0.1.Final</validation-api.version>
    <hibernate-validator.version>6.1.0.Final</hibernate-validator.version>
    <javax.mail.version>1.6.2</javax.mail.version>
  </properties>

  <!-- 私服，负责管理构件的发布 -->
  <distributionManagement>
	<!-- release发布仓库 属性和setting对应 -->
    <repository>
      <id>releases</id>
      <name>Nexus Release Repository</name>
      <url>http://nexus.maven.com:8081/repository/maven-releases/</url>
    </repository>

	<!-- snapshot快照仓库 属性和setting对应 -->
    <snapshotRepository>
      <id>snapshots</id>
      <name>Nexus Snapshot Repository</name>
      <url>http://nexus.maven.com:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
  </distributionManagement>


  <!-- 项目中多个模块间公共依赖的版本号、scope的控制,统一版本号 -->
  <dependencyManagement>

<!-- 
    1. 若 dependencies 里的 dependency 自己没有声明 version 元素，那么maven 就会 到 depenManagement 里去找有		没有该 artifactId 和 groupId 进行过版本声明
    2. 若存在，则继承它，若没有则报错，你必须为dependency声明一个version
    若 dependencies 中的 dependency 声明了version，则 dependencyManagement 中的声明无效 
-->
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>${spring-boot.version}</version>
        <type>pom</type>

		<!-- compile在编译与打包时都会存储进去 -->
		<!-- provided在执行（mvn package）进行打包时不会加入 -->
		<!-- test在编译与打包的时候都不会使用这个依赖 -->
		<!-- runtime在运行的时候才会依赖，在编译的时候不会依赖 -->
		<!-- import从其它的pom文件中导入依赖设置 -->
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <version>${dubbo.version}</version>

		<!-- 排斥列出的依赖 -->
        <exclusions>
          <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
          </exclusion>
          <exclusion>
            <artifactId>log4j</artifactId>
            <groupId>log4j</groupId>
          </exclusion>
          <exclusion>
            <artifactId>slf4j-api</artifactId>
            <groupId>org.slf4j</groupId>
          </exclusion>
          <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
          </exclusion>
        </exclusions>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!-- Aapche Dubbo  -->
      <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-bom</artifactId>
        <version>${dubbo.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
        <version>${dubbo.version}</version>
        <exclusions>
          <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring</artifactId>
          </exclusion>
          <exclusion>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
          </exclusion>
          <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
          </exclusion>
          <exclusion>
            <artifactId>spring-context</artifactId>
            <groupId>org.springframework</groupId>
          </exclusion>
          <exclusion>
            <groupId>org.yaml</groupId>
            <artifactId>snakeyaml</artifactId>
          </exclusion>
        </exclusions>
      </dependency>
      <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>${dubbo-spring-boot.version}</version>
      </dependency>
      <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>${mybatis.version}</version>
        <exclusions>
          <exclusion>
            <artifactId>spring-boot-starter</artifactId>
            <groupId>org.springframework.boot</groupId>
          </exclusion>
        </exclusions>
      </dependency>
      <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper</artifactId>
        <version>${tkmybatis.version}</version>
      </dependency>
      <!--mapper-->
      <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper-spring-boot-starter</artifactId>
        <version>${tkmybatis.springboot.version}</version>
      </dependency>
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>${druid.version}</version>
      </dependency>
      <dependency>
        <groupId>com.auth0</groupId>
        <artifactId>java-jwt</artifactId>
        <version>${jwt.version}</version>
      </dependency>
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>${fastjson.version}</version>
      </dependency>
      <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct-jdk8</artifactId>
        <version>${mapstruct.version}</version>
      </dependency>
      <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct-processor</artifactId>
        <version>${mapstruct.version}</version>
      </dependency>
      <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
        <version>${joda-time.version}</version>
      </dependency>
      <dependency>
        <groupId>org.redisson</groupId>
        <artifactId>redisson</artifactId>
        <version>${redisson.version}</version>
      </dependency>

      <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>${pagehelper.version}</version>
      </dependency>
      <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper-spring-boot-autoconfigure</artifactId>
        <version>${pagehelper.sprinboot.version}</version>
      </dependency>
      <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper-spring-boot-starter</artifactId>
        <version>${pagehelper.sprinboot.version}</version>
      </dependency>
      <!--swagger在线文档工具-->
      <dependency>
        <groupId>com.spring4all</groupId>
        <artifactId>swagger-spring-boot-starter</artifactId>
        <version>${swagger-starter.version}</version>
      </dependency>
      <dependency>
        <groupId>javax.validation</groupId>
        <artifactId>validation-api</artifactId>
        <version>${validation-api.version}</version>
      </dependency>

      <dependency>
        <groupId>org.hibernate.validator</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>${hibernate-validator.version}</version>
      </dependency>

      <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>${curator.famework.version}</version>
      </dependency>

      <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-recipes</artifactId>
        <version>${curator.famework.version}</version>
      </dependency>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
        <version>${spring-boot.version}</version>
      </dependency>
      <dependency>
        <groupId>javax.mail</groupId>
        <artifactId>javax.mail-api</artifactId>
        <version>${javax.mail.version}</version>
      </dependency>
      <dependency>
        <groupId>com.sun.mail</groupId>
        <artifactId>javax.mail</artifactId>
        <version>${javax.mail.version}</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <build>
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-maven-plugin</artifactId>
          <version>${spring-boot.version}</version>
        </plugin>
        <plugin>
          <groupId>org.mybatis.generator</groupId>
          <artifactId>mybatis-generator-maven-plugin</artifactId>
          <version>1.3.7</version>
          <dependencies>
            <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>8.0.16</version>
            </dependency>
            <dependency>
              <groupId>org.mybatis.generator</groupId>
              <artifactId>mybatis-generator-core</artifactId>
              <version>1.3.7</version>
            </dependency>
            <dependency>
              <groupId>tk.mybatis</groupId>
              <artifactId>mapper</artifactId>
              <version>${tkmybatis.version}</version>
            </dependency>

          </dependencies>
          <executions>
            <execution>
              <id>Generate MyBatis Artifacts</id>
              <phase>package</phase>
              <goals>
                <goal>generate</goal>
              </goals>
            </execution>
          </executions>
          <configuration>
            <!--允许移动生成的文件 -->
            <verbose>true</verbose>
            <!-- 是否覆盖 -->
            <overwrite>false</overwrite>
            <!-- 自动生成的配置 -->
            <configurationFile>src/main/resources/mybatis-generater.xml</configurationFile>
          </configuration>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```



### maven项目 主pom

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.itcrazy.alanmall.user</groupId>
  <artifactId>user-service</artifactId>
    
  <!--所有的父级项目的packing都为pom，packing默认是jar类型，如果不作配置，maven会将该项目打成jar包 -->
  <packaging>pom</packaging>
  <version>1.0-SNAPSHOT</version>
    
  <!--通过modules标签将项目的所有子项目引用进来，在build父级项目时，会根据子模块的相互依赖关系整理一个build顺序，然后依次build。-->
  <modules>
    <module>user-api</module>
    <module>user-provider</module>
  </modules>

  <name>user-service</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>


</project>

```

### maven项目 子pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>alanmall-parent</artifactId>
        <groupId>com.itcrazy.alanmall.parent</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <groupId>com.itcrazy.alanmall.user</groupId>
    <artifactId>user-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>user-provider</name>
    
    <!-- 子类项目的packing值只能是war或者jar -->
    <packaging>jar</packaging>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <artifactId>user-api</artifactId>
            <groupId>com.itcrazy.alanmall.user</groupId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <artifactId>common-tools</artifactId>
            <groupId>com.itcrazy.alanmall.common</groupId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!--mapstruct-->
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct-jdk8</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct-processor</artifactId>
        </dependency>

        <!--mybatis-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.7</version>
        </dependency>

        <!--dubbo-->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
        </dependency>

        <!--zookeeper-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.10.0</version>
        </dependency>

        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>

        <!--JSON-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.7</version>
                <configuration>
                    <configurationFile>src/main/resources/generator-config.xml</configurationFile>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                </configuration>
                <executions>
                    <execution>
                        <id>generate</id>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```



### maven resourses 标签

构建Maven项目的时候，如果没有进行特殊的配置，Maven会默认按照标准的目录结构查找和处理各种类型文件。

1. > src/main/java和src/test/java

    这两个目录中的所有*.java文件会分别在compile和test-compile阶段被编译，编译结果分别放到了target/classes和targe/test-classes目录中，但是这两个目录中的其他文件都会被忽略掉。如下图:src/main/java/com/demo.xml文件在编译时被忽略(src/test/java截图略)

<img src="https://raw.githubusercontent.com/Yangliang266/images/master/img/21550029-af9c1de58b93bd6f.png.webp" alt="21550029-af9c1de58b93bd6f.png" style="zoom: 67%;" />



2. > src/main/resouces和src/test/resources

    如上图可看出这两个目录中的文件也会分别被复制到target/classes和target/test-classes目录中。



3. > target/classes

    打包插件默认会把这个目录中的所有内容打入到jar包或者war包中。



#### 资源文件的配置

资源文件是Java代码中要使用的文件。代码在执行的时候会到指定位置去查找这些文件。前面已经说了Maven默认的处理方式，但是有时候我们需要进行自定义的配置。

有时候有些配置文件通常与.java文件一起放在src/main/java目录（如mybatis或hibernate的表映射文件）。有的时候还希望把其他目录中的资源也复制到classes目录中。



#### 解决办法

为了使JAR插件正确地捆绑资源，您可以在Pom.xml文件中,在<build></build>节点中配置resources节点



#### 配置resources节点

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   ...
   ...

   <build>
      ...
      ...
      <resources>
         <resource>
            <directory>src/main/java</directory>
            <tartgetPath>/plexus</targetPath>
            <includes>
               <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
         </resource>
         <resource>
            <directory>${basedir}/src/main/resources</directory>
            <excludes>
               <exclude>/**</exclude>
            </excludes>
         </resource>
      </resources>
      
      <testResources>
      	..
      </testResources>
   </build>

</project>
```



resources:是资源元素的列表，每个元素描述与此项目关联的文件的内容和位置。

targetPath:指定资源文件编译后放置的目录,根目录是target/classes,如上则在目录target/classes/plexus下。

filtering:为真或假，表示是否要为此资源启用筛选。

directory:此元素的值定义在何处找到资源。构建的默认目录是${basedir}/src/main/resources。

include:一组文件模式，指定要包含的文件作为指定目录下的资源,使用*作为通配符。

exclude:与include相同的结构，但指定忽略哪些文件。

testResources: testResources元素块包含testResource元素。它们的定义类似于资源元素，但在测试阶段自然会使用它们。