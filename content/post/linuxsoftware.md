+++
title = "Linux software"
date = 2018-12-20T20:32:52+08:00
lastmod = 2018-12-20T20:32:52+08:00
categories = ["Linux"]
tags = ["linux", "tools"]
+++

# linux系统软件安装 & 环境配置

整合一下linux软件安装的笔记，原版是跟[这套课](https://coding.imooc.com/learn/list/96.html?distId=11b7b9&utm_source=fenxiao)（付费的）时做下的笔记，系统用的是CentOS 6.8，现在VPS上用了CentOS 7，有些命令使用方式变化了。另实操参考了[菜鸟教程-linux命令](http://www.runoob.com/linux/linux-command-manual.html)、鸟哥的linux私房菜（第四版）。

Redhat/CentOS体系使用rpm + yum方式；   
Debian/Ubuntu体系使用dpkg + apt-get方式。

## 一、 rpm
语法
```
rpm [-acdhilqRsv][-b<完成阶段><套间档>+][-e<套件挡>][-f<文件>+][-i<软件>][-p<软件>＋][-U<软件>][-vv][--addsign<软件>+][--allfiles][--allmatches][--badreloc][--buildroot<根目录>][--changelog][--checksig<软件>+][--clean][--dbpath<数据库目录>][--dump][--excludedocs][--excludepath<排除目录>][--force][--ftpproxy<主机名称或IP地址>][--ftpport<通信端口>][--help][--httpproxy<主机名称或IP地址>][--httpport<通信端口>][--ignorearch][--ignoreos][--ignoresize][--includedocs][--initdb][justdb][--nobulid][--nodeps][--nofiles][--nogpg][--nomd5][--nopgp][--noorder][--noscripts][--notriggers][--oldpackage][--percent][--pipe<执行指令>][--prefix<目的目录>][--provides][--queryformat<档头格式>][--querytags][--rcfile<配置档>][--rebulid<软件>][--rebuliddb][--recompile<软件>][--relocate<原目录>=<新目录>][--replacefiles][--replacepkgs][--requires][--resign<软件>+][--rmsource][--rmsource<文件>][--root<根目录>][--scripts][--setperms][--setugids][--short-circuit][--sign][--target=<安装平台>+][--test][--timecheck<检查秒数>][--triggeredby<软件>][--triggers][--verify][--version][--whatprovides<功能特性>][--whatrequires<功能特性>]
```
常用参数
```
-i  install
-h  显示安装进度
-v  显示指令执行过程
-U  升级指定的软件
-F  有则升级，无则不管

--prefix newpath 指定路径
--force 强制安装，谨慎使用

-q 查询
-a all
-i information 详细信息
-l list出所有的文件与目录所在完整文件名
-c 列出配置文件（/etc/目录下）
-d 列出说明文件（与man有关的）
-f 文件属于哪个软件

-e 删除

--rebuilddb 重建rpm数据库
```
### 常用方式
```
#安装示例
rpm -ihv yourrpmpacket
#安装CentOS 6的yum源
rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm --prefix /usr/local

#查看已安装的
rpm -qa
#结合grep使用查询JDK示例
rpm -qa | grep jdk
```

## 二、 yum

### 源配置
国外主机默认源就行，国内主机的话可以使用  
阿里云：  https://mirrors.aliyun.com/  重定向到https的地址了：https://opsx.alibaba.com/mirror   
网易云：  https://mirrors.163.com/   
清华云：  https://mirrors.tuna.tsinghua.edu.cn/centos/7/os/x86_64/

1. 源备份
```
sudo mv /etc/yum.repos.d/CentOS-Base.repo  /etc/yum.repos.d/CentOS-Base.repo.backup
```
2. 下载CentOS-Base.repos到/etc/yum.repos.d/
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```
3. 运行 `yum makecache` 生成缓存

or
#### Redhat源光盘
1. 将光盘rhel-server-6.3-i386-dvd.iso拷贝到linux的/tmp/目录下，并且设置其权限为777
2. 挂载ISO
```
mkdir -p /mnt/rhel
mount -t iso9660 -o loop /tmp/rhel-server-6.3-i386-dvd.iso  /mnt/rhel
```
3. 修改repo文件
`vim /etc/yum.repos.d/rhel-source.repo`
CentOS 目录一样
    ```
    [local]
    name=local
    baseurl=file:///mnt/rhel
    enabled=1
    gpgcheck=0
    ```
4. 修改yumRepo.py文件
`vim /usr/lib/python2.6/site-packages/yum/yumRepo.py`
查找
`remote = url + '/' +relative`
改成
`remote = url + '/local_yum_source' + relative`
5. `yum clean all`

现在即可使用本地的YUM进行安装软件

### 语法
`yum [options] [command] [package ...]`
### 常用命令
1. 列出可更新的软件：`yum check-update`
2. 更新所有软件：`yum update`
3. 安装指定软件：`yum install <package_name>`
4. 更新指定软件：`yum update <package_name>`
5. 列出可安裝软件：`yum list` 类似`rpm -qa`
列出可升级软件:`yum list updates `
6. `yum info`: 同上, 类似 `rpm -qai`
7. `yum providers`: 类似 `rpm -qf`
8. 删除软件包：`yum remove <package_name>`
9. 查找软件包：`yum search <keyword>`
10. 清除缓存:
`yum clean packages`: 清除缓存目录下的软件包     
`yum clean headers`: 清除缓存目录下的 headers     
`yum clean oldheaders`: 清除缓存目录下旧的 headers     
`yum clean, yum clean all (= yum clean packages; yum clean oldheaders)` :清除缓存目录下的软件包及旧的headers      

#### 使用示例
```
#查找源中可安装的jdk列表
yum search java | grep -i --color JDK
#安装git
yum -y install git
#已安装的 java
yum list installed |grep java

# 配置docker库
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```


## 三、Debian/Ubuuntu 体系下的 dpkg & apt-get

暂时换到了CentOS体系的系统了……
刚开始用的Ubuuntu, 可谁让是份教程就用CentOS作示例。

---
慕课上的免费linux课程：  
[linux权限管理](http://www.imooc.com/learn/481?distId=11b7b9&utm_source=fenxiao)   
[linux软件安装管理](http://www.imooc.com/learn/447?distId=11b7b9&utm_source=fenxiao)   
[linux达人]()   
[linux服务管理](http://www.imooc.com/learn/537?distId=11b7b9&utm_source=fenxiao)   
[iptables防火墙](http://www.imooc.com/learn/389?distId=11b7b9&utm_source=fenxiao)

以上全没看，滑稽，好吧，阅读型的学习者通过视频效率有点低, 后面改买书来看了。


默认端口：
mysql：3306
ngnix：80
ftp：20、21

# linux 常用软件的安装配置

CentOS 7 服务重启使用 `systemctl restart ?.service `,
服务启动文件在`/usr/lib/systemd/system/`。

## 1. git
#### 直接用`yum -y install git`
or
1. 下载   
1) `https://github.com/git/git/releases?after=v2.9.1`  
2) `wget https://github.com/git/git/archive/v2.8.0.tar.gz`

2. 安装依赖
`sudo yum -y install zlib-devel openssl-devel cpio expat-devel gettext-devel curl-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker`

### 查看
`git --version`

### 配置
1. 配置用户名(提交时引用)   
`git config --global user.name "{name}"`
2. 配置邮箱   
`git config --global user.email "{email}"`

3. kdiff3   
`git config --global merge.tool "kdiff3"`
4. 不管Windows/Unix换行符转换    
`git config --global core.autocrlf false`

5. 编码配置   

```
git config --global gui.encoding utf-8    
#避免git gui 中文乱码     
git config --global core.quotepath off    
#避免git status显示中文文件乱码   
windows   
git config --global core.ignorecase false
```
### 配置git ssh key pair(另一篇ssh也有提到)
1. `ssh-keygen -t rsa -C "@mail.com"`

2. `ssh-add ~/.ssh/id_rsa`
3. `cat ~/.ssh/id_rsa.pub` 查看公钥

`ssh-add` 出错Could not open a connection to your ……
执行`ssh-add ~/.ssh/rsa`成功
`ssh-add -l`有新加的rsa了

[Pro Git](https://git-scm.com/book/zh)

## 2. nginx

### 安装

1. 添加CentOS7的nginx源   
`rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm`
2. `yum install -y nginx`

or

1. 安装gcc   
`yum install gcc`   gcc -v 查询系统是否自带   
2. 安装pcre    
`yum install pcre-devel`    
3. 安装zlib    
`yum install zlib zlib-devel`   
4. 安装openssl   
`yum install openssl openssl-delel`     
合并后的命令：
`yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel`   
5. 下载源码包   
`wget http://nginx.org/download/nginx-1.10.2.tar.gz`    
`tar -zxvf nginx-1.10.2.tar.gz`   
6. Nginx安装   
1) 进入nginx执行  `./configure`   
注： 增加参数 `--prefix=/usr/nginx`   
`whereis ngnix` 查询
默认安装在/usr/local/nginx   
2) make
`make install`

### ngnix常用命令
测试配置文件：
安装路经下的`/nginx/sbin/nginx -t`    
启动命令：
安装路经下的`/nginx/sbin/nginx`   
停止命令：
安装路经下的`/nginx/sbin/nginx -s stop` or `nginx -s quit`   
重启命令：
安装路经下的`/nginx/sbin/nginx -s reload`

nginx路径加到path就不需要这么一串了，参考下面jdk。

查看进程命令：
`ps -ef | grep nginx`

平滑重启：
`kill -HUP [nginx 主进程号PID]`

### 增加防火墙访问权限
1. `sudo vim /etc/sysconfig/iptables`
2. `-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT`
3. 重启防火墙
`sudo service iptables restart`

### 反向代理、虚拟域名配置及测试验证
1. 编辑  
`sudo vim /usr/local/nginx/conf/nginx.conf`   
增加 `include vhost/*.conf`   
2. 在/usr/local/nginx/conf/目录新建vhost文件夹
即：`/usr/local/nginx/conf/vhost`   
3. 创建域名转发配置文件
```
liuyun.com.conf
blog.liuyuns.com.conf
img.liuyuns.com.conf
```
转发地址  
root (文件夹)    
proxy_pass (端口)

4. 启动/重启   
5. 访问


**配置hosts**

`sudo vim /etc/hosts`


## 3. jdk

安装
`yum  install  java-1.8.0-openjdk   java-1.8.0-openjdk-devel`

or

1. 查看已安装的jdk   
```
rpm -qa|grep java
rpm -qa|grep jdk
yum list installed |grep java
#查找源中可安装的列表
yum search java | grep -i --color JDK
#卸载
sudo yum remove ……
```

2. 下载   
wget ……   
赋予权限进行安装
```
sudo chmod 777 jdk-7u80-linux-x64.rpm
777权限：全开（用户、用户组、其他人，7：读写执行）
```
3. 安装
`sudo rpm -ivh jdk-7u80-linux-x64.rpm`    
4. 默认安装路径
`usr/java
例：/usr/java/jdk.1.7.0_80`


### 配置环境变量   
1.  `sudo vim /etc/profile`   
2.  添加

    ```
    export JAVA_HOME=/usr/java/jre-1.8.0-openjdk-1.8.0.121-0.b13.el7_3.x86_64
    export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ```  
3. 在export PATH中添加
    ```
    $JAVA_HOME/bin
    export PATH=$JAVA_HOME/bin:$PATH
    ```
4. 使配置生效
    `source /etc/profile`

### 查看
```
whereis java
echo $JAVA_HOME
java -version
```


## 4. tomcat
1. 下载  
`wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.35/bin/apache-tomcat-8.5.35.tar.gz`

2. 解压
`tar -zxvf apache-tomcat-7.0.73.tar.gz`
3. 配置环境变量   
  1) `sudo vim /etc/profile`    
  2) `export CATALINA_HOME=/dev`    
  3) 使配置生效  `source /etc/profile`
4. 配置UTF-8字符集   
1) 编辑`vim tomcat/conf/server.xml`   
2) 在配置8080默认端口位置，xml末尾加`URIEncoding="UTF-8"`

#### 验证
1.  启动
执行`sudo tomcat/bin/startup.sh`     
注: linux非root用户不能绑定到1024以下端口，非root使用80需曲线救国……iptables配置端口转发（没去验证，这里只是sudo验证了tomcat能正常起动，开放端口的事儿交给Nginx了）。
2.  访问

## 5. maven
1. 下载

2. 解压
`tar -zxvf apache-maven-3.0.5-bin.tar.gz`
3. 环境变量
```
export MAVEN_HOME=/develop/apache-maven-3.0.5
export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
```

### 验证
`mvn -version`
### 常用命令
```
mvn clean
mvn compile
mvn package
mvn clean package -Dmaven.test.skip=true
```


## 6. mysql

yum源获取：  `wget -i -c http://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm`   
yum源安装：  `rpm -ivh mysql80-community-release-el7-1.noarch.rpm`

1. 安装   
1) `sudo yum -y install mysql-server`   
2) `rpm -qa | grep mysql`  
3) 默认配置文件`/etc/my.cnf`  
2. 字符集配置    
1) `vim /etc/my.cnf`  
2) 添加配置，[mysqld]节点下

    ```
    default-character-set=utf8
    character-set-server=utf8
    collation-server=utf8_general_ci
    #linux下mysql安装完后是默认：表名区分大小写，列名不区分大小写； 0：区分大小写，1：不区分大小写
    #lower_case_table_names=1
    #设置最大连接数，默认为 151，MySQL服务器允许的最大连接数16384
    #max_connections=1000
    #[mysql]
    #default-character-set = utf8
    ```
3. 自启动配置    
1) 执行`chkconfig mysqld on`    
2) 执行`chkconfig --list mysqld`查看(2~5位on启用状态)
4. 防火墙配置      
`sudo vim /etc/sysconfig/iptables`    
`-A INPUT -p tcp --dport 3306 -j ACCEPT`    
`sudo service iptables restart`

### 服务启动
1. 启动mysqld服务
`sudo service mysqld start` 或
`/ect/rc.d/init.d/mysqld start`
2. 初始化环境配置  
默认未设置密码, `mysql -u root` 登陆服务器

##### 无法启动，mysql日志：
CentOS 7 下，mysql 8，关机重启服务器能自动启动，但关掉mysqld服务后重启不了（为什么要关掉？512M内存吃紧，用掉400M+，关掉后只有100M-），网上的中文方案试了个遍，暂时没解决。   
启动报错，最后查看到mysqld.log,   
关键错误如下:

> [ERROR] [MY-010735] [Server] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.   
……    
[ERROR] [MY-013129] [Server] A message intended for a client cannot be sent there as no client-session is attached. Therefore, we're sending the information to the error-log instead: MY-001146 - Table 'mysql.component' doesn't exist    
……      

 
根据信息执行`mysql_upgrade`

> mysql_upgrade: Got error: 2002: Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2) while connecting to the MySQL server
Upgrade process encountered error and will not continue.
……

另
改启动配置`/usr/lib/systemd/system/mysqld.service`里，
注释掉

```
#Restart=on-failure
#RestartPreventExitStatus=1
#PrivateTmp=false
```

嗯，以上都没用。centos7，mysql8.0，重启服务器能开机启动，关掉mysqld服务restart不了……


### mysql配置   
1. 查看当前用户
`select user, host, password from mysql.user`
2. 修改root密码

```
set password for root@localhost=password('{pwd}');
set password for root@127.0.0.1=password('{pwd}');
```
3. exit退出mysql
4. 重新登陆
`mysql -u root -p`
5. 输入密码
6. 删除匿名用户
查看是否存在匿名用户 `select user, host from mysql.user;`
删除匿名用户          `delete from mysql.user where user='';`
再次查看             
刷新                  `flush privileges;`
7. 插入新用户
`insert into mysql.user(Host, User, Password) values("[localhost]", "[username]", password("pwd"));`
8. 使生效
`flush privileges;`
9. 创建新database(databasename方括号外面加1左边的撇号)
`create database `[databasename]`default character set utf8 collate utf8_general_ci;`
10. 用户赋予本地访问的所有权限
`grant all privileges on [databasename].* to [username]@[localhost] identified by '[pwd]';`
11. 给用户开通外网的所有权限
```
grant all privileges on [databasename].* to '[username]'@'%' identified by '[pwd]';
grant [select, insert, update, delete] on [database].[table] to [who]@[where] identified by '[pwd]';
```

### 连接
1. ip
2. 客户端连接



## 7. vsftpd 文件服务器

### 安装
  1. `yum -y install vsftpd`
  2.  `rpm -qa|grep vsftpd`
  3. `/etc/vsftpd/vsftpd.conf`

### 创建虚拟用户
1. 在`/home`创建ftpfile:
`mkdir ftpfile`
2. 添加匿名用户
`useradd ftpuser -d /ftpfile -s /sbin/nologin`
3. 修改ftpfile权限
`chown -R ftpuser.ftpuser /ftpfile`
-R遍历 用户.用户组 to /ftpfile
4. 重设密码
`passwd ftpuser`

### 配置
1. `cd /etc/vsftpd`
2. `sudo vim chroot_list`
3. 把新增的虚拟用户加到配置文件
4. `sudo vim /etc/selinux/config`     
`SELINUX=disabled`    
`sudo setenforce 0`     
注：如果验证出现550拒绝访问执行
`sudo setsebbool _p ftp_home-dir 1`     
重启服务器

5. `sudo vim /etc/vsftpd/vsftpd.conf`

### 防火墙配置
1. `sudo vim /etc/sysconfig/iptables`
2. 添加规则
```
-A INPUT -p TCP --dport 61001:62000 -j ACCEPT
-A OUTPUT -p TCP --sport 61001:62000 -j ACCEPT
-A INPUT -p TCP --dport 20 -j ACCEPT
-A OUTPUT -p TCP --sport 20 -j ACCEPT
-A INPUT -p TCP --dport 21 -j ACCEPT
-A OUTPUT -p TCP --sport 21 -j ACCEPT
```
3. 重启防火墙
`sudo service iptable restart`


### 验证
1. 重启ftp服务
`sudo service vsftpd restart`
2. ftp://……
3. 用户名ftpuser密码password


## 使用sftp传输文件到虚机
远端    
pwd：当前目录信息  
ls：查看远端当前目录下的内容   

本地+前缀"l"，lcd、ll

下载远程文件 `get -r <remote_file_name> [local_file_name]`
上传本地文件到远程 `put -r local_directory_name`
-r递归的复制文件夹
