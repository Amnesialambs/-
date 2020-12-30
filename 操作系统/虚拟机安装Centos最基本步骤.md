# 虚拟机安装Centos最基本步骤

> 人生得意须尽欢，莫使金樽空对月。
>
> 天生我才必有用，千金散尽还复来。



## 步骤一：安装 centos 系统

基本上就是点点下一步结束。



## 步骤二：设置网络

#### 查看本机ip

```shell
ifconfig
```

可能会提示命令不存在；

输入 ip

![image-20201201134048120](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201201134048120.png)

​       可以看到 OBJECT 的值  

```shell
  ip address 
```

![image-20201201134428004](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201201134428004.png)



#### 修改网卡名称

* 切换至 网卡配置文件

```shell
cd /etc/sysconfig/network-scripts
```



![image-20201201135557559](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201201135557559.png)

* 编辑配置文件

```shell
vim ifcfg-ens33
```

修改属性：

name=eth0

device=eth0

保存

* 重命名网卡配置文件 ifcfg-ens33 为 ifcfg-eth0

```shell
mv ifcfg-ens33 ifcfg-eth0
```

* 编辑 /etc/default/grub 并加入“net.ifnames=0 biosdevname=0 ”到GRUBCMDLINELINUX变量

```shell
[root@localhost network-scripts]# vi /etc/default/grub

GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto net.ifnames=0 biosdevname=0 rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

* 运行命令grub2-mkconfig -o /boot/grub2/grub.cfg 来重新生成GRUB配置并更新内核参数

```shell
[root@localhost network-scripts]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-514.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-514.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-b7f83ca165964a47b8b283907b126140
Found initrd image: /boot/initramfs-0-rescue-b7f83ca165964a47b8b283907b126140.img
done
```

* 重启系统

```shell
[root@localhost network-scripts]# reboot
```



#### ip 设置

* 切换至 网卡配置文件目录

```shell
cd /etc/sysconfig/network-scripts
```

* 编辑配置文件：前一步将 ifcfg-ens33 修改为 ifcfg-eth0

```
vi ifcfg-eth0
```

* 修改一下属性：

> ``` 
> ONBOOT=yes #开机启动
> BOOTPROTO=static #静态IP
> IPADDR=192.168.22.110 #本机地址
> NETMASK=255.255.255.0 #子网掩码
> GATEWAY=192.168.22.2 #默认网关
> DNS1=192.168.22.2 #DNS 配置
> ```



![在这里插入图片描述](https://img-blog.csdnimg.cn/2019030514590019.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pzbmh1eA==,size_16,color_FFFFFF,t_70)



* 重启网络服务

```shell
service network restart
```



* 重启服务

```shell
reboot
```



* 另外补充一些NAT模式的知识，图片来自尚学堂

![](https://img-blog.csdnimg.cn/20190305150911715.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pzbmh1eA==,size_16,color_FFFFFF,t_70)



## 步骤三：配置防火墙

#### 关闭 firewall 防火墙

```shell
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```

#### 安装配置 iptables 

* **安装 iptables**

```shell
yum -y install iptables-services
```

+ **修改防火墙配置，增加对3306 端口规则**

```shell
vi /etc/sysconfig/iptables
```

+ **增加规则**

```shell
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
```

+ **重启防火墙**

```shell
systemctl restart iptables.service #重启防火墙使配置生效
```

+ **设置防火墙开机启动**

```shell
systemctl enable iptables.service #设置防火墙开机启动
```

+ **重启服务器**

```shell
reboot
```





