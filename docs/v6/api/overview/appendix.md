---
layout: docs
title: 附录
order: 100
---

<a id="appendix"></a>
# 附录

<a id="urlsafe-base64"></a>
## URL安全的Base64编码

URL安全的Base64编码适用于以URL方式传递Base64编码结果的场景。该编码方式的基本过程是先将内容以Base64格式编码为字符串，然后检查该结果字符串，将字符串中的加号`+`换成中划线`-`，并且将斜杠`/`换成下划线`_`，同时尾部保持填充等号`=`。

详细编码规范请参见[RFC4648](http://www.ietf.org/rfc/rfc4648.txt)标准中的相关描述。

<a id="domain-binding"></a>
## 域名绑定

每个空间都可以绑定一个到多个自定义域名，以便于更方便的访问资源。

比如`www.qiniu.com`的所有静态资源均存放于一个叫`qiniu-resources`的公开空间中。并将该空间绑定到一个二级域名`i1.qiniu.com`，那么如果要在一个HTML页面中引用该空间的`logo.png`资源，大概的写法如下：

```
<img source="http://i1.qiniu.com/logo.png"></img>
```

这样既可以在一定程度上隐藏正在使用七牛云存储的事实，但更大的好处是如果需要从一个云存储迁移到另一个云存储，只需要修改域名DNS的CNAME设置，而无需更新网页源代码。

<a id="qiniu-etag"></a>
## 七牛ETag算法

七牛的 `hash/etag` 算法是公开的。算法大体如下：

### 小于或等于4M的文件

```

1. 对文件内容做sha1计算；

  +---------------+
  |     <=4MB     |
  +---------------+
   \      |      /
    \   sha1()  /
     \    |    /
      \   V   /
    +--+-----+
    |1B| 20B |              2. 在sha1值（20字节）前拼上单个字节，值为0x16；
    +--+-----+
     |  |
     |  \--- 文件内容的sha1值 
     |
     \------ 固定为0x16

3. 对拼接好的21字节的二进制数据做url_safe_base64计算，所得结果即为ETag值。

```

### 大于4M的文件

```

1. 对文件内容按4M大小切块；
2. 对每个块做sha1计算；

          +----------+----------+-------
          |    4MB   |   4MB    | ...
          +----------+----------+-------
           \    |    |   |     /
            \ sha1() | sha1() /
             \  |    |   |   /
              \ V    |   V  /
            +--+-----+-----+-------
            |1B| 20B | 20B | ...
            +--+-----+-----+-------
             |  |     |
             |  |     \---- 第二块的sha1值，类推
             |  |
             |  \---------- 第一块的sha1值 
             |
             \------------- 固定为0x96

3. 将所有sha1值按切块顺序拼接，并在最前面拼上单个字节，值为0x96；
4. 对拼接好的二进制数据做再做sha1计算；
5. 对前一步的sha1值做url_safe_base64计算，所得结果即为ETag值。

```

### FAQ

1. 为何需要公开 `hash/etag` 算法？这个和 “消重” 问题有关，详细见：[如何避免用户上传相同的文件](http://kb.qiniu.com/53tubk96)。  

2. 为何在 sha1 值前面加一个字节的标记位(0x16或0x96）？  

0x16 = 22，而 2^22 = 4M。所以前面的 `0x16` 其实是文件按 4M 分块的意思。  
0x96 = 0x80 | 0x16。其中的 `0x80` 表示这个文件是大文件（有多个分块），hash 值也经过了2重的 sha1 计算。  

### 相关工具

[qetag](https://github.com/qiniu/qetag) 是一个计算文件在七牛云存储上的 hash 值（也是文件下载时的 etag 值）的实用程序。
