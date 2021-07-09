### 私服搭建setting配置

```xml
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  
  <localRepository>D:\repository</localRepository>

  <pluginGroups>
  </pluginGroups>

  <proxies>
  </proxies>

  <servers>
	<server>
      <id>release</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    
    <server>
      <id>snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>

  </servers>

  <mirrors>
	<mirror>
		<id>maven-public</id>
		<name>maven-public</name>
		<url>http://nexus.maven.com:8081/repository/maven-public/</url>
		<mirrorOf>central</mirrorOf>
	</mirror>
  </mirrors>

  <profiles>
    <profile>
      <id>default_profile</id>
      <repositories>
        <!--包含需要连接到远程仓库的信息 -->
        <repository>
          <!--远程仓库唯一标识 -->
          <id>maven-public</id>
          <!--远程仓库名称 -->
          <name>maven-public</name>
          <!--如何处理远程仓库里发布版本的下载 -->
          <releases>
            <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->
            <enabled>true</enabled>
            <!--该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。 -->
            <updatePolicy>never</updatePolicy>
            <!--当Maven验证构件校验文件失败时该怎么做-ignore（忽略），fail（失败），或者warn（警告）。 -->
            <checksumPolicy>warn</checksumPolicy>
          </releases>
          <!--如何处理远程仓库里快照版本的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的策略。例如，可能有人会决定只为开发目的开启对快照版本下载的支持。参见repositories/repository/releases元素 -->
          <snapshots>
            <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->
            <enabled>true</enabled>
            <!--该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。 -->
            <updatePolicy>always</updatePolicy>
            <!--当Maven验证构件校验文件失败时该怎么做-ignore（忽略），fail（失败），或者warn（警告）。 -->
            <checksumPolicy>warn</checksumPolicy>
          </snapshots>
          <!--远程仓库URL，按protocol://hostname/path形式 -->
          <url>http://nexus.maven.com:8081/repository/maven-public/</url>
          <!--用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。 -->
          <layout>default</layout>
        </repository>
      </repositories>
      
      <pluginRepositories>  
        <pluginRepository>  
          <id>maven-public</id>  
          <name>maven-public</name>
          <url>http://nexus.maven.com:8081/repository/maven-public/</url>  
          <releases>
            <enabled>true</enabled>  
          </releases>  
          <snapshots>  
            <enabled>true</enabled>  
          </snapshots>      
        </pluginRepository>  
      </pluginRepositories> 
  
    </profile>
  </profiles>

  <activeProfiles>
    <activeProfile>default_profile</activeProfile>
  </activeProfiles>
</settings>

```

