# Vue 前台项目

## 项目结构

### node_modules

项目依赖文件夹

### public

一般放置一些静态资源（图片），需要注意，放在public文件夹中的静态资源，webpack进行打包时，会原封不动的打包到dist文件夹中。

### src文件夹

#### asset

一般放置静态文件（一般当值多个组件共用的静态资源），需要注意，放置在assets文件夹里， 静态资源，在webpack打包的时候，webpack会把静态资源当做一个模块，打包到JS文件中。

#### components

一般放置的是非路由组件（全局组件）

#### App.vue

全局唯一的根组件（.vue）

#### main,js

入口文件，整个程序最先运行的文件

### babel.config.js

配置文件，babel相关

### package.json

项目的身份证，说明项目的具体信息，有哪些依赖，怎么运行

### package-lock.json

缓存文件

## 小细节

### 运行项目时自动打开网页

在package.json文件中的

```
    "serve": "vue-cli-service serve",
改为
    "serve": "vue-cli-service serve --open",

```

### 关闭eslint校验功能关闭

### src文件夹简写形式配置别名

## 项目路由分析

### 路由组件

Home首页路由组件

Search路由组件

login路由

Register组件

### 非路由组件

header，（在首页，在搜索页）

footer （在首页，在搜索页）

## 安装Less环境

```
cnpm install --save-dev less-loader less
```

## 搭建路由

```
 cnpm install --save vue-router
```

## 关于路由传参

### params只能使用name,不能用path

### 为什么params传参时要占位？

不占位也是可以接受到数据的，那为什么还要占位？

#### 原因

params分为两种，

一种是在url显示参数

这种保密性较好，但是刷新后数据会消失

```
    path:'/search',
```

一种不显示参数

这种保密性较差，但是刷新不会出现问题，

```
    path:'/search/:keyword/:test?'
```

**值得注意的是，如果不显示参数形式的params后面不加问号时，路由会出现问题，即不显示当前的路由**

### 如何指定params参数可传可不传

在配置路由的时候，在占位的后面加上一个问号

## 接口拿数据

code字段200为正常

## axios二次封装

### 为什么要二次封装？

请求拦截器，响应拦截器

### API文件夹通常是axios封装

## 接口统一管理

小项目：完全可以在组件的声明周期中发请求

大项目：为了便于统一管理，可以将接口封装入一个模块中

## 跨域

什么是跨域：协议，域名，端口号的不同请求，称之为跨域问题

## nprogress进度条

## vuex模块化

将多模块的SAGM 单独放置，最后使用modules开发，这里需要使用命名空间

## 节流与防抖

> 正常情况下，用户慢慢操作，每一个事件都会被触发
>
> 特殊情况下，只有部分的被触发，
>
> 这是因为用户的操作太快，导致浏览器无法正常运行

## 防抖

前面的所有的触发被取消，最后一次执行在规定的时间之后才被触发，也就是说连续快速的触发，只会触发一次

## 节流

在规定的时间范围内不会重复触发回调，只有大于这个时间段才会触发回调把频繁触发更改为少量触发

## lodash  _.throttle(节流)

## 跳转路由

使用router-link跳转路由时，出现了卡段的现象，是因为在使用router-link时，由于嵌套结构，会同时构造非常非常多的路由

## 节省性能

因为向typeNav这种被复用的组件，会多次请求数据

这样的话可以在app实例化时获取数据

## 如nextTick等生命钩子函数，不一定一定要在外层使用，也可以在函数内部通过this.$nexTick调用

## nextTick可以保证页面中所以的结构都是有的

与需要DOM存在的插件一起使用

## 合并对象

