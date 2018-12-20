+++
title= "SSH"
date= 2018-12-15T13:59:37+08:00
lastmod= 2018-12-15T13:59:37+08:00
categories= ["linux"]
tags= ["linux", "ssh", "Secure"]
+++

# SSH

**在补丁上总是将 SSH 程序包和需要的库保持为最新：**  
` yum update openssh-server openssh openssh-clients -y `

### 删除不安全文件

从系统上删除rlogin 和rsh二进制程序，并将它们替代为SSH的一个 symlink：

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

#### 另
 限制 SSH 将侦听和绑定到的可用接口，eg：

>  ListenAddress 192.168.100.17     
>  ListenAddress 209.64.100.15  

修改完执行：
`service sshd restart`  
centOS 7+ : `systemctl restart sshd.service`

---
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


`scp -P serverPort ~/.ssh/id_rsa.pub root@192.0.0.1:/home/user/.ssh/authorized_keys`   
//scopy a to b

//
**如果已经存在authorized_keys，需要将公钥追加至authorized_keys**
```
#server：服务器IP，root为对应用户目录
scp -P xxxxx ~/.ssh/id_rsa.pub server:/root/.ssh/tmp.pub
```
---
```
# 在服务器端执行
cat /root/.ssh/tmp.pub >> /root/.ssh/authorized_keys
```
---
```
chmod 600 .ssh/authorized_keys
```
##### 报错

> WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!  

删除~/.ssh/known_hosts文件
`rm ~/.ssh/known_hosts`

-----

## 趟坑记录
修改后xshell死活连不上，查日志：

> fatal: bad ownership or modes for chroot directory "/home/user"   
 
```
chown root:root /home/liu   
chmod g-w  /home/liu   
```
修改user的owner为root后，日志变为：

> error: /dev/pts/0: no such file or directory  
> error: open /dev/tty failed - can not set controlling tty: No such file or directory

然后找到这篇:[如何在Linux中设置SFTP监测](http://www.geekpills.com/operating-system/linux/how-to-setup-chroot-sftp-in-linux)   
sftp连接日志报：

> subsystem request for sftp  
> error: subsystem: cannot stat /usr/libexec/openssh/sftp-server: No such file or directory

```
Subsystem  internal-sftp  #指定sftp命令
```

#### 后续解决方案：
不使用ChrootDirectory。

**and better scheme：**  
chroot 必须包含支持用户会话所必需的文件和目录。
交互式会话，需要至少一个 shell，通常为 sh 和基本的 /dev 节点，例如 null、zero、stdin、stdout、stderr 和 tty 设备：   
`ls -l /dev/{null,zero,stdin,stdout,stderr,random,tty}`

```
mkdir -p /home/liu/dev  
chown liu:liu
cd /home/test/dev
mknod -m 666 null c 1 3
mknod -m 666 tty c 5 0
mknod -m 666 zero c 1 5
mknod -m 666 random c 1 8
```

为 SSH chroot 设置交互式shell
```
mkdir -p /home/liu/bin
cp -v /bin/bash /home/liu/bin
```
识别 bash 所需的共享库
```
ldd /bin/bash
mkdir -p /home/liu/lib64
cp -v /lib64/{libtinfo.so.5,libdl.so.2,libc.so.6,ld-linux-x86-64.so.2} /home/liu/lib64
```

more参照 [使用 chroot 限制 SSH 用户访问指定目录](www.tecmint.com/restrict-ssh-user-to-directory-using-chrooted-jail)

----

[SSH 安全性和配置入门](https://www.ibm.com/developerworks/cn/aix/library/au-sshsecurity)  
[VPS安全之SSH设置](https://blog.phpgao.com/vps_ssh.html)   
[How to Secure Your Server](https://www.linode.com/docs/security/securing-your-server)  
[Common Security of VPS ](https://blog.tankywoo.com/ops/2013/09/14/common-security-of-vps.html)
