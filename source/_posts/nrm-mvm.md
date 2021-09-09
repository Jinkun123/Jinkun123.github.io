---
title: Node 版本管理  nvm nrm
date: 2021-09-09 23:20:21
categories:
    - Node
tags: [nvm,nrm]
---

# 前言

作为一个小前端，是否因为搭建环境烦恼过，是否因为 npm 等国外镜像踩坑过，不要怕，接下来跟着我一步步搭建适合自己的开发环境吧！！！

## node
这个不用说了吧，我们经常和他打交道，无论是 gulp、webpack和parcel等打包工具，还是各种脚手架的工具，都离不开node环境的支持，接下来我就介绍一下我常用的一些工具和模块。

## nvm

管理node版本，通过nvm我们可以同时安装/切换不同的node版本不过,nvm不支持window版本,但是有替代方案,就是nvm-window,具体为什么nvm为何不支持windows平台?这里就不做谈论了…


>  ps: nvm-window 下载链接，如果网速快就不需要在这里下载了，github 下载链接，建议下载nvm-setup.zip会帮你配置好环境变量


#### 安装
如果没什么特别要求，无脑下一步即可


1. 如果之前已安装node,作者的建议是卸载原有的node版本,避免发生冲突
2. 配置 setting.txt 文件,主要是配置为国内镜像源镜像源
 配置文件在：C:\Users\用户名\AppData\Roaming\nvm 下（如果和我一下，无脑下一步的话）
 
 
```
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

>ps:如果遇到Powershell下禁止执行脚本的问题,请用管理员打开Powershell执行set-ExecutionPolicy RemoteSigned，选择 y 即可
 


常用命令

1. 可列出已安装的 node 版本 nvm list/nvm ls

        nvm ls
    
2. 安装指定版本的 node nvm install[arch]

        nvm install latest # latest表示安装最新版
   
>ps: arch 为可选的平台架构项（32 位/64 位），，默认为系统平台对应的版本，若设置为 all，则同时安装两个版本。

3. 卸载指定版本 nvm uninstall

        nvm uninstall 13.6.0 # latest表示安装最新版

4. 设置镜像源 nvm node_mirror

    * 设置node镜像源

           nvm node_mirror https://npm.taobao.org/mirrors/node/
           
    * 设置npm镜像源
    
           nvm npm_mirror https://npm.taobao.org/mirrors/node/

##     nrm

众所周知的一点，npm 日常会挂掉，还时不时丢包，所以我们需要一款切换源的工具，来帮我们解决这个问题。

>ps: 虽然可以手动切换源，但是相对来讲还是比较麻烦的，所以推荐使用工具来帮我们完成这件事


#### 安裝

`
npm install -g nrm
`

#### 常用命令


1. 列出当前支持切换的源
```
nrm ls* npm -------- https://registry.npmjs.org/
yarn ------- https://registry.yarnpkg.com/
cnpm ------- http://r.cnpmjs.org/
taobao ----- https://registry.npm.taobao.org/
nj --------- https://registry.nodejitsu.com/
npmMirror -- https://skimdb.npmjs.com/registry/
edunpm ----- http://registry.enpmjs.org/

```
2. 使用 taobao 源作为默认的 npm 源
```
nrm use taobao    Registry has been set to: https://registry.npm.taobao.org/
```
3. 测试源速度
      * 测试一个源
        `nrm test npm`
      * 测试所有源
        `nrm test`
4. 访问源的主页
  `nrm home taobao`
  
>ps: 此命令会在默认浏览器中打开淘宝源的主页：https://registry.npm.taobao.org/


5. 添加/刪除 一个源


* 添加源：nrm add[home]，home 参数主要用于访问源的主页（可选）
`nrm add gating http://npm.gatings.com/  http://gatings.cn/`

* 刪除源：nrm del
`nrm del gating`

>ps: nrm del 命令不能删除 nrm 自己内置的源。


  



