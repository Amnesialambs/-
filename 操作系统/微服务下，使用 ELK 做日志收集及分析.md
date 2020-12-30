# 微服务下，使用 ELK 做日志收集及分析

ELK 分为：Elastic Search、Logstash 和 Kibana 三部分

## ELK 部署

#### Elastic Search 安装

+ 本次部署的目录为 /data/deploy/elk 下，首先需要下载：

```shell
cd /data/deploy/elk
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.3.tar.gz
```

如果目录不存在：

```shell
mkdir -p /data/deploy/elk
```

+ 解压到当前目录

```shell
tar -zxvf elasticsearch-6.4.3.tar.gz
```

+ 修改配置文件

```shell
cd elasticsearch-6.4.3/config
vim elasticsearch.yml
```

增加内容

> ```shell
> network.host: 0.0.0.0
> http.port: 9200
> http.cors.enabled: true
> http.cors.allow-origin: "*"
> ```

ps: Elastic Search启动：由于ES的启动不能用root账号直接启动，需要新创建用户，然后切换新用户去启动。命令如下：

```shell
-- 创建新用户及授权
# groupadd elsearch
# useradd elsearch -g elsearch -p elasticsearch
# cd /data/deploy/elk/
# chown -R elsearch:elsearch elasticsearch-6.4.3
-- 切换用户，启动
# su elasearch
# cd elasticsearch-6.4.3/bin
# sh elasticsearch &
```

启动过程中，会出现一些报错信息，如：

1、max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]

2、max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

解决问题（1）：将当前用户的软硬限制调大。

>```
># vim /etc/security/limits.conf
>-- 在后面增加一下配置后，保存退出
>es soft nofile 65535
>es hard nofile 65537
>-- 不需要重启，重新登录即生效
>-- 查看修改命名是否生效
># ulimit -n 65535
># ulimit -n
>-- 结果65535
># ulimit -H -n 65537
># ulimit -H -n
>-- 结果65537
>```

解决问题（2）：调大elasticsearch用户拥有的内存权限

>```
>-- 切换到root用户
># sysctl -w vm.max_map_count=262144
>-- 查看修改结果
># sysctl -a|grep vm.max_map_count
>-- 结果显示：vm.max_map_count = 262144
>
>-- 永久生效设置
># vim /etc/sysctl.conf
>-- 在文件最后增加以下内容，保存后退出：
>vm.max_map_count=262144
>```

解决以上问题后，再次启动：

>```
># su elasearch
># cd /data/deploy/elk/elasticsearch-6.4.3/bin/
># sh elasticsearch &
>```

启动成功后，访问：http://ip:9200，可以有json格式的返回信息，判断安装成功。

### elasticsearch-head 安装

* 安装nodejs

```shell
cd /opt
wget https://nodejs.org/dist/v10.9.0/node-v10.9.0-linux-x64.tar.gz #下载nodejs压缩包
tar -zxvf node-v10.9.0-linux-x64.tar.gz #解压压缩包
mv node-v10.9.0-linux-x64 /usr/local/nodejs #移动文件到/usr/local目录下，并将文件夹名称改为nodejs
ln -s /usr/local/nodejs/bin/node /usr/bin/node #创建软连接，让node命令全局生效
ln -s /usr/local/nodejs/bin/npm /usr/bin/npm #创建软连接，让npm命令全局生效
node -v #查看nodejs是否安装成功
npm -v 
```

+ 安装 git 并拉取 elasticSearch-head 

```shell
yum install –y git #安装git 安装过则更新
git --version #查看是否安装成功
git clone https://github.com/mobz/elasticsearch-head.git#从github上拉取elasticsearch-head代码
cd elasticsearch-head #进入elasticsearch-head文件夹
npm install cnpm -g --registry=https://registry.npm.taobao.org #因为npm安装非常非常慢，所以在这里先安装淘宝源地址
ln -s /usr/local/nodejs/bin/cnpm /usr/local/bin/cnpm #创建cnpm软链接，不然执行下面执行命令会报错
npm install -g cnpm --registry=https://registry.npm.taobao.org # 安装 cn
cnpm install #使用cnpm命令下载安装项目所需要的插件
vim _site/app.js #修改app.js 搜索localhost，将localhost修改为安装ElasticSearch服务器的ip，如下图
```

+ 安装 elasticsearch-head 依赖包

```shell
cd /elasticsearch-head/
npm install
```

* 修改 Gruntfile.js

```shell
vi Gruntfile.js
```

在connect-->server-->options下面添加：hostname:’*’，允许所有IP可以访问

![image-20201206111806224](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201206111806224.png)

+ 修改 elasticsearch-head 默认连接地址

```shell
cd /elasticsearch-head/_site/
vi app.js
```

将this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9200";中的localhost修改成你es的服务器地址，我的是：192.168.126.8:9200

![image-20201206112352031](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201206112352031.png)

+ 配置 elasticsearch 允许跨域

进入elasticsearch服务器，打开elasticsearch的配置文件elasticsearch.yml，在文件末尾追加下面两行代码即可：

http.cors.enabled: true

http.cors.allow-origin: "*"

+ 启动 elasticsearch-head 

```shell
cd /elasticsearch-head/
node_modules/grunt/bin/grunt server
```

```shell
Running "connect:server" (connect) task
Waiting forever...
Started connect web server on http://localhost:9100
```

如上，表示elasticsearch-head启动成功

### Kibana的安装

+ 下载安装包

>```shell
>-- 切换到root用户
>su
>-- 下载
>cd /data/deploy/elk/
>wget https://artifacts.elastic.co/downloads/kibana/kibana-6.4.2-linux-x86_64.tar.gz
>```

+ 解压配置：

>```shell
>tar -zxvf kibana-6.4.2-linux-x86_64.tar.gz
>cd kibana-6.4.2-linux-x86_64/config/
>vim kibana.yml
># -- 增加如下配置：
>server.port: 5601
>server.host: "0.0.0.0"
>elasticsearch.url: "http://localhost:9200"
>kibana.index: ".kibana"
>```

ps: 如：server.port: 5601 冒号后的空格必不可少

+ 启动Kibana

>```shell
>cd /data/deploy/elk/kibana-6.4.2-linux-x86_64/bin
>sh kibana &
>```

启动成功后，访问http://ip:5601，查看是否启动成功。



### Logstash 安装

+ 下载安装包

```
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.4.2.tar.gz
```

+ 解压

```shell
tar -zxvf logstash-6.4.2.tar.gz
cd logstash-6.4.2/bin
```

+ 在 bin 目录下 新增配置文件 logstash.conf

```shell
vim logstash.conf
```

配置内容

>```shell
>input {
>    # 从文件读取日志信息 输送到控制台  path要对应读取的目录
>    file {
>        path => "/home/elasticsearch/elasticsearch-6.4.3/logs/myes.log"
>        codec => "json" ## 以JSON格式读取日志
>        type => "elasticsearch"
>        start_position => "beginning"
>    }
>    tcp {
>      port => 5044
>      codec => json_lines
>    }
>}
>output {
>    # 标准输出 
>    # stdout {}
>    # 输出进行格式化，采用Ruby库来解析日志   
>    stdout { codec => rubydebug }
>    elasticsearch {
>        hosts => ["localhost:9200"]
>    }
>}
>```

+ 启动 logstash

```shell
cd /data/deploy/elk/logstash-6.4.2/bin
nohup sh logstash -f logstash.conf  &
```



ps : logstash 监控变动的日志。如果监控文件没变化，不会监控



### filebeat下载及配置

ps: 由于logstash只能收集本机日志，故在其他机器上搭建filbeat，将日志发送给logstash

+ 下载安装

```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.7.1-linux-x86_64.tar.gz
```



### Log4j向Logstash发送日志

+ 在logstash的配置文件中增加如下配置

>```shell
>input {
>  stdin {
>  }
>  tcp {
>    type => "log4j-test"
>    port => 4560
>  }
>}
>
>filter {
>  grok {
>    match => { "message" => "%{COMBINEDAPACHELOG}" }
>  }
>}
>
>output {
>  elasticsearch {
>        hosts => ["localhost:9200"]
>        index => "logstash-%{type}-%{+YYYY.MM.dd}"
>		document_type => "%{type}"
>  }
>  stdout { codec => rubydebug }
>}
>```

其中`input`下的`port`表示本机开放4560端口接收网络中其他主机应用程序中log4j发送过来的日志，也可以指定其他未被占用的端口，`type` 为接收的日志起的别名

`output`下的`elasticsearch`部分表示logstash将接收的日志发送给本机的elasticsearch，其端口为9200，`index` 表示生成索引的名称

这里的配置是服务器模式，也就是logstash作为日志服务器开放一个端口，网络中其他主机主动发送日志

+ 在应用中配置 log4j

log4j用SocketAppender将日志发送到指定的主机和端口，在log4j.xml中配置如下

>```shell
><?xml version="1.0" encoding="UTF-8"?>
><Configuration status="warn" name="MyApp" packages="">
>    <Appenders>
>        
>　　　　 <!-- 47.*.*.159为logstash主机外网IP，4560为logstash端口 -->
>        <Socket name="logstash-tcp" host="47.*.*.159" port="4560" protocol="TCP">
>            <PatternLayout pattern="${LOG_PATTERN}" />
>        </Socket>
>    </Appenders>
>    <Loggers>
>       <!-- 异步发送logstash -->
>        <!-- 如果使用<asyncRoot> 或 <asyncLogger>，includeLocation="true"是必须要设置才会有类路径等一些信息打印出来 -->
>        <AsyncLogger name="com.xinyartech" level="info" includeLocation="true" >
>            <appender-ref ref="logstash-tcp" />
>        </AsyncLogger>
>     
>        <Root level="INFO">
>            <AppenderRef ref="Console"/>
>            <AppenderRef ref="RollingFile"/>
>        </Root>  
>    </Loggers>
></Configuration>
>```

关于`SocketAppender`中的`RemoteHost`和`Port`等字段的含义参考`SocketAppender`的源码