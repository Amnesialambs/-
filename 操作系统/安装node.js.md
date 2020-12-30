# 安装 node.js



### 方式一：

+ 下载安装包

```shell
wget https://nodejs.org/dist/v14.15.1/node-v14.15.1.tar.gz		
```

* 解压

```shell
tar -zxvf node-v14.15.1.tar.gz
```

+ 配置

```shell
ln -s /usr/local/nodejs/bin/node /usr/bin/node #创建软连接，让node命令全局生效
ln -s /usr/local/nodejs/bin/npm /usr/bin/npm #创建软连接，让npm命令全局生效
```

+ 验证

```shell
node -v 
npm -v
```



### 方式二：

```shell
curl --silent --location https://rpm.nodesource.com/setup_10.x | sudo bash -
yum -y install nodejs
```

