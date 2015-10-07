# 微信服务器配置

微信公众平台的开发的条件是微信公众号以及有公网IP的服务器。

## 微信公众号申请

微信公众号可以直接到[微信公众号官网](https://mp.weixin.qq.com/)进行申请，个人只能申请订阅号，订阅号能够使用的接口有限。

## 公网IP的服务器

服务器方面可以去[阿里云](http://www.aliyun.com/)申请一个服务器，普通的配置只需要几十块一个月。如果实在没钱，你可以去新浪云（SAE）或百度云（BAE）上申请免费的一个应用服务器，具体可以百度。服务器的系统建议选择CENTOS6.5，不预装任何软件。

有了服务器后，我们还需要配置一些基本的开发环境。

### 安装SUN JDK 1.7

我选择的是纯净版的CENTOS6.5系统，没有预装任何软件。

去Oracle网站下载[jdk7.0的rpm包](http://www.oracle.com/technetwork/java/javase/downloads/jdk-7u2-download-1377129.html)，下载之后用scp命令上传到服务器上

```
scp /Users/yurongchan/TEMP/jdk-7u2-linux-i586.rpm root@130.21.122.223:/chenyr/deploy/tomcat6044/webapps
```

用rpm命令安装JDK：

```
rpm -ivh  jdk-7u2-linux-i586.rpm 
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


