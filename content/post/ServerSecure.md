+++
title = "ServerSecure"
date = 2018-12-15T22:44:38+08:00
lastmod = 2018-12-15T22:44:38+08:00
categories = ["linux", "Secure"]
tags = ["linux", "Server", "Secure"]
+++

# 服务器安全

## 用户管理


#### 添加用户，设定密码
```
# just an example
useradd liu
passwd liu
```


#### 设置只允许wheel组切换root
添加用户到wheel组
`usermod -G wheel liu`      

设置只允许wheel组的帐号，        
`vim /etc/pam.d/su`
找到

> #auth            required        pam_wheel.so use_uid

去掉注释

---

`vim /etc/login.defs`   
添加

> SU_WHEEL_ONLY yes     

or      
`echo "SU_WHEEL_ONLY yes">>/etc/login.defs`

---

### 避免用户密码被篡改，锁定口令文件
```
chattr +i /etc/passwd;
chattr +i /etc/shadow;
chattr +i /etc/group;
chattr +i /etc/gshadow;
# ps: 锁定后，用户和组不能改动，需用户相关操作完成最后使用。
```


#### 添加用户到sudo list
方案一：
`usermod -a -G sudo liu`
(实操时:group 'sudo' does not exist,切换方案二)。

方案二：

a. 切换到root用户下

b. 添加sudo文件的写权限   `chmod 660 /etc/sudoers`

c. 编辑sudoers文件 ： `vi /etc/sudoers`

```
liu  ALL=(ALL)      ALL
or
liu   ALL=(ALL)      NOPASSWD:ALL
#   %liu            用户组
#   NOPASSWD:ALL    执行无需密码
```

d. sudo权限改回来`chmod 440 /etc/sudoers`

[介绍rsync用于liunx系统备份](http://www.linode.im/1573.html)

#### sftp用户
设置组sftp，这个组中的用户只能使用sftp，不能使用ssh，且sftp登录后只能在自己的home目录下活动
1. 创建sftp组
` groupadd sftp `

2. 创建一个sftp用户，名为mysftp
```
 useradd -g sftp -s /bin/false mysftp
 passwd mysftp
```
3. sftp组的用户的home目录统一指定到/home/sftp下
```
mkdir -p /home/sftp/mysftp
usermod -d /home/sftp/mysftp mysftp
```
4. 配置sshd_config
通过ChrootDirectory 实现

## SSH

**在补丁上总是将 SSH 程序包和需要的库保持为最新：**  
` yum update openssh-server openssh openssh-clients -y `

### 删除不安全文件

从系统上删除 rlogin 和 rsh 二进制程序，并将它们替代为 SSH 的一个 symlink：

```
# find /usr -name rsh
/usr/bin/rsh
# rm -f /usr/bin/rsh
# ln -s /usr/bin/ssh /usr/bin/rsh
```

### SSH服务的配置

SSH服务的配置文件位于`/etc/ssh/sshd_config`，安全设置都是围绕此文件展开。

备份： `cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup`

编辑： `vi /etc/ssh/sshd_config`

#### eg:
```
PermitRootLogin no       # 禁止ROOT用户登陆

# 限制用户登陆
AllowUsers fsmythe bnice swilson
DenyUsers jhacker joebadguy jripper
#Match Group sftp  # 匹配sftp组的用户，多个组逗号分隔
#Match User mysftp # 匹配用户，逗号分隔，按组匹配更灵活

#ChrootDirectory /home/%u        # 使用Chroot SSHD将用户局限于其自己的主目录
X11Forwarding no                # 转发禁掉
AllowTcpForwarding no

PermitEmptyPasswords no         # 禁止空密码登陆
IgnoreRhosts yes                # 禁用用户的 .rhosts文件
HostbasedAuthentication no      # 基于主机的身份验证禁掉
RhostsRSAAuthentication no

ClientAliveInterval 600 # (Set to 600 seconds = 10 minutes) 10分钟无操作自动掉线
ClientAliveCountMax 3   # 设为0会连不上，这里用默认值3

PasswordAuthentication no       # 密码认证关掉

#VerifyReverseMapping yes       # SSH版本验证（没有该属性）
UsePrivilegeSeparation yes      # 特权分割
StrictModes yes                 # 使用严格模式

RSAAuthentication yes           # 只有RSA认证
PubkeyAuthentication yes        # 公钥认证
#AuthorizedKeysFile      .ssh/authorized_keys   # 认证的公钥
#ChallengeResponseAuthentication no
#KerberosAuthentication no
#GSSAPIAuthentication yes  

#Subsystem       sftp    internal-sftp  # 使用sftp服务使用系统自带的internal-sftp
#ForceCommand    internal-sftp  #指定sftp命令
```

修改完执行：
`service sshd restart`

-----


### 指纹登陆
服务端建立文件夹存放rsa公钥：

```
mkdir /home/liu/.ssh
cd /home/liu
chown -R liu:liu .ssh
chmod 700 .ssh    # 赋权
```
---
##### 客户端：
客户端通过ssh-keygen生成RSA密钥，将公钥丢到服务器上进行认证。
`ssh-keygen -t rsa -b 4096 -C xxx` 生成密钥
```
ssh-keygen 参数
-t rsa  # RSA验证、DSA验证
-b      # 4096位
-C      # comment
```
将公钥拷贝至服务器对应用户的.ssh下，重命名为authorized_keys:  
`scp -P serverPort ~/.ssh/id_rsa.pub  root@192.0.0.1:/home/user/.ssh/authorized_keys`  

---
服务端赋权authorized_keys ：  
`chmod 600 .ssh/authorized_keys`

### 登陆IP限制

```  
  vim /etc/hosts.deny

  # add
  all:1.1.1.1
```
```  
  # make it work
  service network restart
```

### 检查登录日志

如果你的服务器一直很正常，那也可能不正常的表现，最好的办法就是定期查询ssh的登录日志，手动发现系统的异常！

```
# vim /etc/ssh/sshd_config
# add
LogLevel DEBUG

# 查看最近100条登录日志
tail -100 /var/log/secure

# 登录成功日志
who /var/log/wtmp

last
```

### fail2ban

这个小巧的软件可以代替你做很多事情，以暴力破解ssh密码为例，当我们安装fail2ban后，经过合理的配置，我们可以自动屏蔽某个攻击IP的所有请求。

项目地址：https://github.com/fail2ban/fail2ban

```
# install
# clone repo
git clone https://github.com/fail2ban/fail2ban.git
cd fail2ban
# do installation
python setup.py install
# copy to init.d
cp files/redhat-initd /etc/init.d/fail2ban
# make it executable
chmod 755 /etc/init.d/fail2ban
# run on boot
chkconfig --add fail2ban
chkconfig --level 35 fail2ban on

# config
vim /etc/fail2ban/jail.conf

# add

[ssh-iptables]
enabled  = true
filter   = sshd
action   = iptables[name=SSH, port=ssh, protocol=tcp]
           sendmail-whois[name=SSH, dest=root, sender=fail2ban@example.com, sendername="Fail2Ban"]
logpath  = /var/log/secure
maxretry = 2
[ssh-ddos]
enabled = true
filter  = sshd-ddos
action  = iptables[name=ssh-ddos, port=ssh,sftp protocol=tcp,udp]
logpath  = /var/log/messages
maxretry = 2

# mkdir
mkdir -p /var/run/fail2ban

# restart

service fail2ban restart

# check if the jail is working
service fail2ban status

# fail2ban-server (pid  5408) is running...
# Status
# |- Number of jail:    2
# `- Jail list:    ssh-ddos, ssh-iptables
```

内容来源：
[VPS安全之SSH设置](https://blog.phpgao.com/vps_ssh.html)

[SSH 安全性和配置入门](https://www.ibm.com/developerworks/cn/aix/library/au-sshsecurity)

[How to Secure Your Server](https://www.linode.com/docs/security/securing-your-server)

[Common Security of VPS ](https://blog.tankywoo.com/ops/2013/09/14/common-security-of-vps.html)


## 防火墙iptables

`vim /etc/sysconfig/tptables`

### 推荐的规则
**注意：所有链名必须大写，表名必须小写，动作必须大写，匹配必须小写**

```

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

```
iptables -L  #列出iptables规则
iptables -F  #清除iptables内置规则
iptables -X  #清除iptables自定义规则
# 可以加入-n查看

/etc/rc.d/init.d/iptables save  # 保存规则，只有保存规则后重启机器才会生效
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
