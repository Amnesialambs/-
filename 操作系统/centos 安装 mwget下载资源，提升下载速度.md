# centos 安装 mwget

+ 安装mwget

```shell
wget http://jaist.dl.sourceforge.net/project/kmphpfm/mwget/0.1/mwget_0.1.0.orig.tar.bz2
```

+ 解压

```shell
tar -jxvf mwget_0.1.0.orig.tar.bz2
```

ps：解压的时候有可能会发生问题,需要先下载解压工具

```shell
yum install bzip2
bzip2 -d mwget_0.1.0.orig.tar.bz2
tar -jxvf mwget_0.1.0.orig.tar.bz2
```

+ 解压后切换进目录 执行 ./configure

```shell
./configure
```

ps：如果出现 error: C++ compiler cannot create executables 说明没有安装c++编译器 安装一个c++编译器就可以了

```shell
yum install gcc-c++
```

ps：如果出现缺失open-ssl,安装一个即可

```
yum install openssl-devel
```

ps：如果执行./configure 出现 configure: error: Your intltool is too old. You need intltool 0.35.0 or later.

需要安装0.35.0以上的版本

```
yum install intltool
```

+ 安装

```
make
make install
```

搞定，以后可以用mwget 来下载

