+++
title = "Hugo + Github Page 的Blog搭建"
date = 2018-11-29T23:40:57+08:00
lastmod = 2018-12-15T21:28:35+08:00
categories = ["Blog"]
tags = ["hugo", "blog", "github page", "DNS"]
+++

# Hugo + Github Page的Blog搭建

序：初写Blog，此文乃逗比版文风。
 
记下折腾Blog的过程：     
起因：经过一番对比（折腾）后，入手了这个**[VPS套餐](https://bwh8.net/cart.php?a=confproduct&i=0&aff=38411)**(不用看了，最便宜的套餐，大写的穷。11.03入手，12.13，该套餐已下架，最低的价格已翻倍)，搭了个梯子。  
继续，于是么想搭个blog，环比了一下初步定下了用**[Github Page](https://pages.github.com/)**(内容托管) + 
**[hugo](https://gohugo.io/)**(内容管理，同类型的还有Jekyll、hexo静态，更重量级的如wordpress为代表的cms系统) + 
**[DNSPod](https://www.dnspod.cn/)**(TX云下的域名解析，其他还有阿里云、七牛云等)( ~~域名还没入手~~ 嗯入手了一些免费域名)，结果最后是没用到VPS么，滑稽，后面看情况决定是不是迁移到自己的VPS。
补充下，图片暂时没分离，直接丢Blog里了，计划使用[七牛云](https://www.qiniu.com?code=3lfihxnbik1g2)，之前弄了个账号一直没用过。

选型之后就是撸起袖子开工了。  

## Hugo
**前情提要**：win7环境（其他OS仅供参考，推荐读官网教程，很详细，就是是英文的），hugo:v0.52。

[hugo官网](https://gohugo.io/)   
[官网教程](https://gohugo.io/getting-started) 先放前面。

### 第一回合：安装

先是搜索了中文教程，坑的满脸血，后面是啃官网教程，照做到安装完成配置环境变量。（1.下载，2.解压，3.环境变量）  

第一个要点来了：以**管理员身份**启动 `git bash` or `cmd` or `cmder`or……等CLI工具，cmd需要以**管理员**身份启动（**见鬼，18-12-01日记录验证，以普通身份也可以启动，前一天必须得管理员身份，中间重启过电脑，同一个用户账号配置没改过，环境抽风的情况最坑了**，12-09：啃官网发现有提到要用管理员，没解释原因，[具体在这里](https://gohugo.io/getting-started/installing/#technical-users)，第4条）。

很好，现在可以用`hugo server`来验证是否ok了，成功的话应该可以看到这样的消息：  

> $ hugo version  
> Hugo Static Site Generator v0.52 windows/amd64 

到此为止，hugo安装配置就搞定了，下面开始第二阶段：

### 第二回合：

hugo常用命令先来一打：

```
hugo [command] [-flags]

hugo help                   #万能的help啊

hugo                        #以当前目录作为输入目录来构建hugo站点

hugo new site dirName       #于dirName文件夹下创建新的站
hugo -t themename           #使用某主题
hugo server -D              #以本地hugo服务器启动，-D参数启用草稿（等效--buildDrafts）
                            #在localhost:1313查看

hugo server --watch=false   #停用热部署，或用 --disableLiveReload

hugo config | grep emoji    #查找配置（如表情配置）

hugo new hello.md           #在content目录下创建新的markdown内容文件，使用了archetypes下的模版

```


[hugo配置文件详解](https://gohugo.io/getting-started/configuration/)

重要的配置项

```
baseURL = "https://liuyunyishui.github.io/blog"    #对应Github Page的仓库地址，详见下文githubPage配置
languageCode = "zh-cn"
hasCJKLanguage =  true

publishDir="docs"           #发布到docs文件夹下，配合github page，默认是public

uglyURLs = true             #爬坑参数1，修改配置后必须Ctrl+c重启，
                            #强迫症专用参数，html后缀必须的，tags还没带上
                            #（因为这个开始自己整主题模版了）

#canonifyURLs=true          #使用基于baseURL的绝对路径
#relativeURLs = true        #使用（./）相对路径

enableEmoji = true          #启用表情，怎能少了我大滑稽神教，（没有滑稽）

disableKinds = ["RSS", "robotsTXT"] #2、别把home、page、section等给禁了，233，需重启

# 对应html>head>title
title = "流云的Blog"
theme = "liuyun"            #对应themes 下的主题文件夹名


#菜单，主题模版中定义的
[[menu.main]]
    url = "/"
    name = "Home"
    weight = 1

#填充模板中的.Site.Params变量
[params]                    
    author = "流云"
    description = "Personal blog"

[permalinks]
  post = "/:year/:month/:day/:title/"

```
:unamused:

三种类型的配置文件查找顺序（最左优先）
```
./config.toml
./config.yaml
./config.json
```

选好主题，下载到themes文件夹下，本处是在minimal主题的基础上改的，`hugo -t themename`使用主题(themename对应theme下的主题文件夹名)(主题选择可以丢到配置文件里，比CLI优先级高)。
不同主题需要的配置的属性不同，参考readme.md、exampleSite(不一定有)，有些主题需自行爬坑。

`hugo` build，hugo构建时不会删除之前生成的文件，需要手动删除旧的发布文件（默认是在`public/*`）
`rm -r public`（因github page改docs）；

`hugo server -D`跑起来，[默认地址](http://localhost:1313): `localhost:1313`；

*[Hugo30分钟搭建静态博客](https://m.linuxidc.com/Linux/2018-09/154467.htm)这个可以参考一下，发布时间是18年9月28。*


## GitHub Page

两个官网帮助文档索引：   
[GitHub页面基础](https://help.github.com/categories/github-pages-basics)   
[自定义GitHub页面](https://help.github.com/categories/customizing-github-pages)

github page 分用户&组织、项目两种类型，其中用户&组织这类需要把项目库名设为yourname.github.io。  
我这里用的是项目方式，对应地址为：https://liuyunyishui.github.io/blog 。     
**这里github page 给的url就是hugo中要配置的baseURL。**

#### 发布github page：
在project > settings  页面下的 Source项，
分为三种：

- master分支发布
- gh-pages分支发布
- master分支下的`/docs`目录发布

这里采用了第三种docs的方式。

## 域名 & DNS解析配置

**[自用namesilo管理面板](https://new.namesilo.com/account_domains?rid=01f8a17ax)**    
**[自用freenom管理面板](https://my.freenom.com/clientarea.php?action=domains)**    
**[自用dnspod管理面板](https://www.dnspod.cn/console/dashboard)**     
[namesilo域名管理界面](https://new.namesilo.com/account_domain_link_add.php?rid=01f8a17ax)（对于英语苦手来说真是难找）  
[namesilo帮助文档](https://new.namesilo.com/support?rid=01f8a17ax)  

### 域名

可以到**[freenom](https://www.freenom.com/zh/index.html?lang=zh)**（中文支持未建好）申请免费域名使用（只有.tk、.cf、.ml、.ga、.gq等后缀域名免费& 有使用权并没有所有权、转让权，可升级购买）（亲测有效，自备了梯子，多次申请后终于成功注册了几个备用，不使用梯子"IP was logod"，无法提交订单），    
或者以下域名商都是不错的选择，环比过了，价格角度推荐第一个，上面的freedom也提供付费域名，价格也很亲民。
以下都是国外的域名商，懒得备案的选择。     
[freenom域名价格表参考](https://www.freenom.com/en/pricechart.html)、[Namesilo折扣计划域名价格表参考](https://new.namesilo.com/account_discount_program.php/?rid=01f8a17ax)(参与折扣计划会弹窗:

> please confirm that you agree to the discount program term and conditions 
you will not be able to add account funds for an amount under $50 while you are in the discount program and for 30 days after you leave.           

google翻译如下：当您在折扣计划期间以及离开后30天内，您将无法为50美元以下的金额添加帐户资金)。

国外域名供应商：

- [Namesilo](https://new.namesilo.com/?rid=01f8a17ax)(支持支付宝付款，价格优势)

- [Namecheap](https://www.namecheap.com) & [enom](https://www.enom.com)

- [Name](https://www.name.com)

- [Godaddy](https://sg.godaddy.com)(世界重量级选手)

- [cn.resellerclub](https://cn.resellerclub.com/domain-names)(亚洲最大，支持中文)

- [domainsite](https://www.domaining.com/)

- [networksolutions](https://www.networksolutions.com/)

- [uniregistry](https://uniregistry.com)（支持中文）


###  DNSpod进行域名解析

通过上述网站或其他途径得到了自己的域名，下面是用dnspod进行域名解析，请参考[这个](https://support.dnspod.cn/Kb/showarticle/tsid/177)&[这个](https://support.dnspod.cn/Kb/showarticle/tsid/28)，这个可以做[索引](https://support.dnspod.cn/Kb/guide)。

DNSpod是腾讯云旗下的免费域名解析，免费的声称是10台服务器（腾讯的节操么你懂得，当然现在么和阿里云价格战玩的不亦乐乎）。

两个dnspod的免费DNS服务器地址：`f1g1ns1.dnspod.net`、`f1g1ns2.dnspod.net`


Github服务器IP地址是192.30.252.153和192.30.252.154。


配置[CloudFlare](https://www.cloudflare.com)以使用HTTPs，就不用DNSPod了。   
clyde.ns.cloudflare.com    
venus.ns.cloudflare.com    

设置CNAME

1. 在Github的网站目录下创建`CNAME`文件
2. 填自己的域名如`http://liuyun.com`

登录DNSPod，先添加域名，然后添加记录，设置如下

主机记录   | 记录类型     | 线路类型     | 记录值     | MX优先级     | TTL        
 ---       | ---          | ---          | ---        | ---          | ---          
 @         | CNAME        | 默认         |liuyunyishui.github.io. | - | 10           
 www       | CNAME        | 默认         |liuyunyishui.github.io. | - | 10           

sublime的Table Editor插件还没搞定，编辑个表格真是为难强迫症了。

### 尾

欢迎拍砖，包括错别字什么的在内……（评论系统没上线前是没法的，右上有联系方式），哦，补充以下，以下站点链接有尾巴，悉知：

- [搬瓦工](https://bwh8.net?a=confproduct&i=0&aff=38411)      
- [Namesilo](https://new.namesilo.com/?rid=01f8a17ax)


**注1：** 遇到问题建议去**[hugo官网教程](https://gohugo.io/getting-started)**找答案，如果和我一样是条英文咸鱼，那么Chrome>右键网页>translate To Chanise 可以帮到你（可能需要到浏览器高级设置里开启）,不习惯用Google Chrome？那么**[这个](https://translate.google.cn)**、**[这个](https://fanyi.baidu.com)**、还有**[这个](http://fanyi.youdao.com)**都可以帮到你，建议和原文对照，顺路学习英语。

**注2：** 什么？以上不能解决你的问题，那么万能的**[这个](https://www.google.com/ncr)**、以及**[备胎](https://www.baidu.com)**肯定能解决了。什么？打不开，那么看下**[ShodowSocks](https://github.com/shadowsocks/shadowsocks/wiki)** （备胎是没有结果的，神兽横行，暂时折腾的这个）or **[V2Ray](https://github.com/v2ray/v2ray-core)**
放在**[搬瓦工](https://bwh8.net?aff=38411)**或其他VPS上。