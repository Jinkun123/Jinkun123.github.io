---
title: Vue3+vite基础搭建
date: 2021-09-17 14:40:31
categories:
    - Vue3
tags: [vue,vite]
---



# 前言

作为一个小前端，基础菜的一笔，不过还是要跟随技术得发展和学习，此篇为初学篇，大佬勿喷！！！


Vue3.0 的创建，我们可以使用vue-CLI,WebPack,这两种工具我们在Vue2x中经常使用到，这里我们就不再多讲， 此次我们重点讲述**vite工具**创建Vue3.0项目：


#### 什么是Vite?

1. Vite是Vue开发者开发的一款意图取代WebPack的工具。
2. 其原理是利用ES6的import会发送请求去加载文件的特性，拦截这些请求，做一些预编译，省去WebPack冗长的打包时间。

废话不多说，现在开始：

##### 1. 安装Vite:

通过cmd命令行工具，在你需要创建Vue3.0项目饿文件夹中执行以下命令：
```
npm install -g create-vite-app
```

##### 2.利用Vite创建Vue3.0项目：

1. 首先创建项目名称，在上一步完成之后接着执行如下命令：create-vite-app projectName   注意这里的projectName 是你的项目名称，你可以随意取，但是名称里最好不要有大写字母，cmd命令行里创建带有大写字母的文件时，会有错误提示。

2. 当文见创建好之后，会提示你执行：
       
        
           cd  projectName 

           npm install  

           npm run dev
            
##### 3.安装依赖运行项目：

1. 完成上一步操作之后，我们就要开始安装Vue项目运行的依赖了，执行命令语句  cd  projectName   进入你刚刚创建好的项目文件夹下，因为接下来的项目依赖都会安装到这个文件夹之下。
2. 在命令行中执行：npm install   开始安装依赖，这个过程可能需要几分钟，安装成功。


##### 4.开始运行项目：

在命令行中执行：
`npm run dev   `
运行成功后会提示你在浏览器中执行 localhost:3000    即可访问创建好的Vue项目工程。

目录大概如下：

![](https://cdn.JsDelivr.net/gh/Jinkun123/blog_imgs/2020-01-12/catalogue.png)


##### Vite配置，插件ui引入

**Vite简单配置一下，更详细得请百度官网**

```
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
const path = require('path')
console.log(path.resolve(__dirname, './src'))
// https://vitejs.dev/config/
export default defineConfig({
  // vite.config.js # or vite.config.ts
  plugins: [vue()],
  "resolve.alias": {
    '@': path.resolve(__dirname, 'src'),
    'view': path.resolve(__dirname, 'src/view'),
    'com': path.resolve(__dirname, 'src/components'),
    'api': path.resolve(__dirname, 'src/api'),
    'utils': path.resolve(__dirname, 'src/utils'),
  },
  server:{
    hostname: '0.0.0.0', // 默认是 localhost
    port: '8080', // 默认是 3000 端口
    open: true, // 浏览器自动打开
    https: false, // 是否开启 https
    ssr: false, // 服务端渲染
    base: './', // 生产环境下的公共路径
    outDir: 'dist', // 打包构建输出路径，默认 dist ，如果路径存在，构建之前会被删除
    proxy: { // 本地开发环境通过代理实现跨域，生产环境使用 nginx 转发
      '/api': {
        target: 'http://poetry.apiopen.top', // 后端服务实际地址
        secure: false,  // 如果是https接口，需要配置这个参数
        ws: true,//是否代理websockets
        changeOrigin: true,
        rewrite: path => path.replace(/^\/api/, '')
      }
    }
  }
})


```

**element引用**

elementui官方  为配合vue3 推出element-plus
输入：

```
npm install element-plus --save
```

**main.js全局引用，局部安装详见官方文档**

```
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'  //引入路由
import axios from 'axios' //引入axios
import store from './store' //引入Vuex
import ElementPlus from 'element-plus' //引入elementui plus
import 'element-plus/dist/index.css'  //引入elementui css
const app=createApp(App); //创建APP实例
app.config.globalProperties.$axops=axios
// base api
console.log(import.meta.env.VITE_NODE_ENV);
app
.use(router)
.use(store)
.use(ElementPlus)
.mount('#app')
```

`Vue2 和 Vue3 全局引入稍有不同 `


##### 5.环境变量配置

创建 .env.development (本地开发环境)  .env.production(生产环境)

`(注意环境变量 名字需以 VITE_ 为开头 vite 要求识别)`


和vue2一样 修改 package.json 文件 scripts


```
"scripts": {
    "dev": "vite",
    "test": "vite --mode development",
    "prod": "vite --mode production",
    "build": "vite build --mode production"
  },

```





