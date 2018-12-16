+++
title= "markdown基本语法"
date= 2018-11-29T23:40:57+08:00
categories= ["markdown"]
tags=["markdown"]
+++

# markdown基本语法

**序：**初写Blog，本篇试用极简风格，原版是参考互联网（多篇，具体来源已不可考），给自己做的MarkDown参考书，标点不齐全。


## 一、标题

 `#`: `#`*(1~6)后 空格 + title  (1~6级标题)
  
## 二、分割线
`>=3`的 `-`or`*` 换行  
eg:     
 分割线  `---`
 ---
 or     
 分割线 `***`
***

## 三、文本

**无空格**

**加粗** ：
2个`*`包裹

*斜体*：
1个`*`包裹

***斜体加粗***：
3个`*`包裹

~~删除线~~：
2个`~` 包裹

`代码` :

`单行代码`：`包裹

```
  代码块：``` 换行包裹
```

****
 
> 引用：

> 文字前用 `>` 可嵌套


* 无序列表：
`+` `-` `+` `*` 空格 内容

0. 有序列表：
1. 数字`. `空格 内容

列表嵌套：
上下之间用Tab键（标准语法是空格）


## 四、超链接&图片

#### 超链接：
\[显示文本](超链接地址 "alt")
alt可加可不加,
`{:target="_blank"}`加上打开新链接(hugo里不支持)    

\[Google](https://google.com/ncr "google.com")  
[Google](https://google.com/ncr "google.com")

#### 图片：
\!\[图片title](URI 空格 "图片alt")
title可加可不加

\!\[Google](url "Google.com")

![Google](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1544295905848&di=4cc90e1b21d647ed6f3f46ae503e8fab&imgtype=0&src=http%3A%2F%2Fa.zdmimg.com%2F201509%2F02%2F55e64ead49ddb4580.png_a200.jpg "Google.com")


## 五、表格
```
表头|表头       
 -  | -     
内容|内容       
内容|内容       
```

表头|表头
--- |---
内容|内容
内容|内容

第二行分割表头和内容。
`-`一个就行，为了对齐，可以多加，hugo里得三个;

文字默认居左

- `-`两边加：表示文字居中；
- `-`右边加：表示文字居右。

注：原生的语法两边都要用 `|` 包起来，此处省略.
