# 安装jdk

### 方式一：手动解压安装包

+ 在usr 目录下新建 java 文件夹

```shell
cd /usr
mkdir java
cd java
```

+ 下载 jdk 1.8

```shell
wget http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz?AuthParam=1534129356_6b3ac55c6a38ba5a54c912855deb6a22

```

+ 解压安装包

```shell
tar -zxvf jdk-8u181-linux-x64.tar.gz
```

+ 修改系统配置

```shell
vi /etc/profile
```

配置内容：

```shell
#java
export JAVA_HOME=/usr/java/jdk1.8.0_181
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib
```

+ 验证：

```shell
java -version
```



### 方式二：yum 安装

+ 搜索 jdk 安转包

```shell
yum search java|grep jdk
```

+ 安装 jdk (默认路径: /url/lib/jvm )

```shell
yum install java-1.8.0-openjdk
```

+ 验证

```
java -version
```

