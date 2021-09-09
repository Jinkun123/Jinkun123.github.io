---
title: sourcetree
date: 2021-05-04 21:08:04
categories:
    - Git
tags: [Git]
---
# sourcetree是免费的Git客户端，如何利用它从gitlab上拉取下代码呢？步骤如下：

1. 下载并安装git；
2. 运行git,生成秘钥，

    在桌面右键打开Git bash Here输入命令为：ssh-keygen -t rsa
秘钥生成的目录在你系统盘用户目录下的\.ssh\id_rsa.pub。(以前生成过，请删除在从新生成,避免错误)


3. 在自己的git服务器上绑定自己git公钥;（绑定操作：Settings --> SSH Keys --> Add key（打开本地公钥文件粘贴里面所有内容））


4. 利用sourcetree拉取代码。点击"工具-->选项-->一般"，注意以下4个部分的设置
    1. 首先在选项里面填写拉去时所需信息，在一般和验证
    2. 填写完毕，点击文件clone（具体详细看图解）

![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/Sourcetree.png)
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/Sourcetree2.png)
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/Sourcetree4.png)
![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/Sourcetree3.png)