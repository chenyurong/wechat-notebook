# 配置微信开发服务器

上次讲了微信公众号的分类以及权限，今天讲一下如何配置一个能接收微信服务器消息的服务器。

## 什么是微信开发者

微信开发者指的是开发微信公众号的开发人员。要进行微信公众号的开发必须进行开发者认证成为开发者之后才能进行微信公众号的开发。

每个公众号都有一个开发者中心菜单，只要在其中填写了正确的校验资料，就可以成功开通开发模式，利用微信提供的接口进行高级功能的开发。

微信开发者认证的原理是当你填写完资料后，微信服务器会向你填写的URL地址发送一个请求，并校验返回值是否与你填写的一致，从而确定你是否拥有这台服务器的控制权。

所以简单地说，你要写的其实就是一个接收HTTP请求的后台接口程序，而至于你用什么语言编写，这都没有什么问题。在这个系列的文章中，我使用的是Java语言。

下面就针对Java语言，我们对服务器进行一些基本的配置。

## 服务器配置

### 安装SUN JDK 1.7

我选择的是纯净版的CENTOS6.5系统，没有预装任何软件。

去[Oracle网站](http://www.oracle.com/technetwork/java/javase/archive-139210.html)下载，下载之后用scp命令上传到服务器上

```
scp /Users/yurongchan/TEMP/jdk-7u2-linux-i586.rpm root@130.21.122.223:/chenyr/deploy/tomcat6044/webapps
```

改变文件权限，并用rpm命令安装JDK：

```
chmod +x jdk-7u67-linux-x64.rpm
rpm -ivh jdk-7u67-linux-x64.rpm
```

JDK默认安装在`/usr/java`目录下

打开`/etc/profile`文件，配置JAVA环境变量：

```
# set the JAVA
JAVA_HOME=/usr/java/jdk1.7.0_67
JRE_HOME= $JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/rt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

### Tomcat 服务器

将下载好的tomcat.zip包上传到服务器上并解压。

在终端中定位到tomcat目录下，输入：

`sudo chmod 755 ./bin/*.sh`

运行以下命令启动Tomcat：

`sudo sh bin/startup.sh`

访问tomcat默认页面看tomcat是否配置完成

`http://serverip:port`

到这里我们已经部署好了一个可以部署Java Web应用的服务器，下篇我们将介绍如何创建一个能接收微信服务器消息的应用，并用这个应用开通开发者模式。


