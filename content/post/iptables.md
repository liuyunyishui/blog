+++
title = "Iptables"
date = 2018-12-15T23:09:12+08:00
lastmod = 2018-12-15T23:09:12+08:00
categories = ["linux"]
tags = ["linux", "Secure", "iptables"]
+++

# 防火墙iptables

#### 更新iptables：
`yum update -y iptables`

**iptables** 是基于内核的防火墙，功能非常强大，iptables内置了filter，nat和mangle三张表。

**filter** 负责过滤数据包，包括的规则链有，INPUT，OUTPUT和FORWARD；

**nat** 则涉及到网络地址转换，包括的规则链有，PREROUTING，POSTROUTING 和 OUTPUT；

**mangle** 表则主要应用在修改数据包内容上，用来做流量整形的，默认的规则链有：INPUT，OUTPUT，NAT，POSTROUTING，PREROUTING；

**INPUT** 匹配目的IP是本机的数据包；  
**FORWARD** 匹配流经本机的数据包；   
**PREROUTING** 用来修改目的地址用来做DNAT；   
**POSTROUTING** 用来修改源地址用来做SNAT。

### 推荐的规则
**注意：所有链名必须大写，表名必须小写，动作必须大写，匹配必须小写**

```
*filter

#  Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
-A INPUT -i lo -j ACCEPT
#-A OUTPUT -o lo -j ACCEPT
-A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT


#  Allow all outbound traffic - you can modify this to only allow certain traffic
-A OUTPUT -j ACCEPT

#  Accept all established inbound connections
-A INPUT  -m state --state ESTABLISHED,RELATED -j ACCEPT
#-A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#  Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT

#  DNS(not need this rule, as we only accept the input with [ESTABLISHED,RELATED])
#-A INPUT  -p udp --dport 53 -j ACCEPT
#  DNS
#-A OUTPUT -p udp --dport 53 -j ACCEPT

#  Allow ping
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
#-A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT
#-A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT

#  Allow SSH connections
#  The -dport number should be the same port number you set in sshd_config
-A INPUT -p tcp -m state --state NEW --dport xxx -j ACCEPT

#  DDoS
-A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT

#  Log iptables denied calls
-A INPUT  -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7
-A OUTPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

# Allow shadowsocks connections
#  The -dport number should be the same port number you set in config.json
-A INPUT -p tcp --dport xxx -j ACCEPT

#  Allow ports for testing
#-A INPUT -p tcp --dport 8080:8090 -j ACCEPT

#  Allow ports for MOSH (mobile shell)
#-A INPUT -p udp --dport 60000:61000 -j ACCEPT

#  Reject all other inbound - default deny unless explicitly allowed policy
# 从上往下过滤，放前面会墙掉所有input，即使后面有符合的accept规则，问我怎么知道的，把自己墙外面过，捂脸
-A INPUT -j REJECT   
-A FORWARD -j REJECT
#-A OUTPUT -j DROP

COMMIT
```
让防火墙规则生效：`service iptables restart`/CentOS 7+ :`systemctl restart iptables.service `    `systemctl list-unit-files|grep iptables `    
查看防火墙规则：`iptables -L`

接下来我们需要设置下只要开机防火墙就开启服务：   
`systemctl enabled iptables.service`

### iptables的语法
```
iptables -A INPUT -j ACCEPT
-A --append # 向规则链中添加一条规则，默认被添加到末尾
iptables -D INPUT -j ACCEPT
-D --delete # 删条规则链

-A 向规则链中添加一条规则，默认被添加到末尾
-D 从规则链中删除规则，可以指定序号或者匹配的规则来删除
-E 重命名自定义链
-R 进行规则替换
-I 插入一条规则，默认被插入到首部
-F 清空所选的链，重启后恢复
-N 新建用户自定义的规则链
-X 删除用户自定义的规则链

-t 指定要操作的表，默认是filter
-p 用来指定协议可以是tcp，udp，icmp等也可以是数字的协议号，
-s 指定源地址
-d 指定目的地址
-i 进入接口
-o 流出接口
-j 采取的动作，ACCEPT，DROP，SNAT源地址转换，DNAT目标地址转换，REDIRECT端口重定向，RETURN返回主链继续匹配，MASQUERADE地址伪装，MARK打标签
--sport 源端口
--dport 目的端口，端口必须和协议一起来配合使用

-P 定义预设规则(Policy)

语法: iptables -P [INPUT,OUTPUT,FORWARD] [ACCEPT,DROP]

iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```
**注意：所有链名必须大写，表明必须小写，动作必须大写，匹配必须小写**
### iptables常用操作
```
iptables -L  #列出iptables规则
iptables -F  #清除iptables内置规则
iptables -X  #清除iptables自定义规则
# 可以加入-n查看

#####
/etc/rc.d/init.d/iptables save  # 保存规则，只有保存规则后重启机器才不会生效
/etc/init.d/iptables status     # 查看规则
/etc/init.d/iptables stop       # 停用防火墙
chkconfig –level 35 iptables off  # 停止防火墙服务
```

### iptables规则的导出和还原
```
iptables-save > somefile
iptables-restore < somefile
```
[Java电商网站开发](https://coding.imooc.com/learn/list/96.html?distId=11b7b9&utm_source=fenxiao)

[VPS安全之防火墙设置](https://blog.phpgao.com/vps_iptables.html)

[iptables详解](http://blog.51cto.com/yijiu/1356254)
