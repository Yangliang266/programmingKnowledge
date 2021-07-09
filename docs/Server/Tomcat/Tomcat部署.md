# 1.IntelliJ IDEA配置Tomcat服务器

步骤1：打开设置settings
在IDEA欢迎界面（打开IDEA进入项目之前或打开了一个工程后通过File -> close project都会进入此界面）点击底部的Configure下拉列表再点击Settings
或
File -> Settings快捷键：ctrl+alt+s

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t57.png)

步骤2：点击Plugins，在插件搜索框中输入tomcat进行搜索找到Tomcat and TomEE Integration插件，并确定其已经勾选，否则在下一步添加Tomcat Server时会没有这个选项

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t56.png)

步骤3:添加Tomcat服务器
Build,Execution,Deployment --> Application Servers --> 点击+，选择Tomcat Server

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t55.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t54.png)

此时会弹出Tomcat Server弹出框，Tomcat Home选择Tomcat安装目录即可，选择好后点击OK即可完成Tomcat配置

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t53.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t52.png)

# 2.创建动态web工程

步骤1：在IDEA欢迎界面点击Create New Project 创建新工程或File --> New --> Project
，这样创建工程向导页会打开

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t51.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t50.png)

步骤2：选择Java Enterprise --> 指定JDK --> 选择java EE版本 --> 选择配置好的应用服务器Application Server --> 在Additional Libraries and Frameworks下勾选Web Application复选框 --> 点击Next

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t49.png)

步骤3：指定工程名及路径，More Settings中的选项会根据Project name和location同步（默认即可），点击Finish完成创建

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t48.png)

创建完成的Project Structure如下图（和eclipse创建的工程并无太大区别）：

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t47.png)

- .idea文件夹和webapp.iml是IDEA自动创建的，包含了工程和模块的配置数据
- src文件夹是源码目录
- web文件夹相当于eclipse创建的web工程WebContent文件夹，包含了WEB-INF/web.xml及index.jsp
- External Libraries包含了JDK及Tomcat带的jsp-api、servlet-api jar文件

再贴上一张eclipse创建的web工程的目录结构图（可以对比一下）
主要的不同点是使用上述方法IDEA创建的web工程WEB-INF下没有lib目录

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t46.png)

步骤4：完善工程目录

- 添加WEB-INF/lib目录

方法一：点击WEB-INF，右击New --> Directory，directory name填写lib，拷贝项目所需的jar包到此目录，右击lib目录 --> Add as Library注意：这种方法如果你不拷贝jar包到lib下，右击时是没有Add as Library选项的

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t45.png)

这时会弹出Craete Library对话框，name填写lib即可，其它默认，点击OK确定

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t44.png)

添加完成可在Project Structure中的Libraries中看到，至于打开Project Structure，这里提供两种方法：File --> Project Structure（快捷键ctrl+alt+shift+s）或点击Navigation Bar中的Project Structure按钮（如下图）

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t43.png)

如果你没有这个Navigation Bar可在View中勾选Navigation Bar

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t42.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t41.png)

方法二：打开Project Structure --> 点击+选择java --> 在弹出的Select Library Files中在WEB-INF下创建lib目录选择并点击OK --> 在弹出的Choose Categories of Selected Files中选择Jar Directory点击OK --> 在弹出的Choose Modules中点击OK

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t40.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t39.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t38.png)

- 添加conf目录用于添加配置文件

方法一：右击项目New --> Directory --> directory name填写conf，点击OK --> 右击conf目录Mark Directory as --> Sources Root

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t37.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t36.png)

这样创建的conf source folder在Project Structure的Modules中可以看到

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t35.png)

方法二：在Project Structure的Modules中右击项目 --> New Folder --> Folder name填conf，点击OK --> 右击新建的conf --> Sources --> 点击底部的OK

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t34.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t33.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t32.png)

------

# 3.本地Tomcat运行应用

点击Navigation Bar上的运行按钮（快捷键shift+f10），debug快捷键shift+f9

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t31.png)

或
在左下角找到Application Servers（没有的话View --> Tool Windows --> Application Servers打开即可），点击run按钮

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t30.png)

运行成功后会默认打开Chrome浏览器访问[http://localhost](http://localhost/):8080/

运行后Run窗口如下图：
![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t29.png)

# 4.修改服务器配置

点击run configuration selector选择Edit Configurations，这时会打开Run/Debug Configurations窗口

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t28.png)

或
在Application Servers窗口右击选择Edit Configurations，这时会打开Edit Server Run Configurations窗口，这与上一个窗口有略微差别

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t27.png)

说明：在创建Web项目的时候如果手速比较快，没有添加Tomcat，可按如下方法设置

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t26.png)

点击+ --> Tomcat Server --> Local
![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t25.png)
点击Deployment选项卡 --> 点击+ --> 选择Artifact
![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t24.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t23.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat22.png)

# 5.将应用打成war包

步骤1：打开Project Structure --> 选择Artifacts --> 点击+ --> Web Application: Archive --> For 'webapp: war exploded'

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat21.png)

步骤2：生成manifest文件：点击Create Manifest且同意IDEA建议的位置 (web/META-INF/MANIFEST.MF)
![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat20.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat19.png)

步骤3：点击Project Structure对话框的OK按钮
步骤4：Build --> Build Artifacts --> 在弹出的Build Artifact弹出框选择webapp:war下的Bulid，点击

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat18.png)

至此，项目打包成功，可以在工程的out/artifacts/webapp_war下看到war包，IDEA默认以工程名+_war.war来命名

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat17.png)

也可以在磁盘上看到这个war包

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat16.png)

# 6.利用IDEA远程部署项目到Tomcat服务器

原来在eclipse下将项目导出为war包后往往需要将war包上传到服务器，进行部署，但IDEA提供了方便易用的远程部署方案，下面一起来看看把。

远程部署服务器ip：192.168.25.129
Tomcat版本：8.5.24
启动Tomcat后查看是否能正常访问

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat15.png)

- 服务器端的配置

修改Tomcat_HOME*T**o**m**c**a**t**H**O**M**E*/bin/catalina.sh，添加如下配置

```shell
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.port=1880"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.rmi.port=1880"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
CATALINA_OPTS="$CATALINA_OPTS -Djava.rmi.server.hostname=192.168.89.138"
CATALINA_OPTS="$CATALINA_OPTS -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5880"

export CATALINA_OPTS
```

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat14.png)

注意：hostname为你自己远程服务器的ip地址
实际上是打开jmx的1099端口，这可参考IDEA官网：[https://www.jetbrains.com/hel...](https://www.jetbrains.com/help/idea/2017.3/run-debug-configuration-tomcat-server.html)

- IDEA配置

步骤1：添加远程Tomcat服务器
Edit Configurations --> 点击+ --> Tomcat Server --> Remote

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat13.png)

步骤2：Remote Server配置

name随意，选择Deployment选项卡 --> 点击+ --> Artifact --> 选择生成的war包，点击OK

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat12.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat11.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat10.png)

切换到server选项卡，进行远程服务器的关键配置

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat9.png)

点击Host后的...会打开如下Deployment窗口

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat8.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat7.png)

测试连接若连接成功会出现以下的弹出框

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat6.png)

远程server设置完毕，点击Deployment窗口底部的OK即可

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat5.png)

接下里是一些剩余的配置：

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat4.png)

到这里就配置成功了。。。

- 远程部署运行测试

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat3.png)

或

![tomcat2.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/tomcat2.png)

![clipboard.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/t58.png)

到服务器上看一下

![tomcat1.png](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/tools/maven/tomcat/maven1.png)