# windows下pm2运行nuxt项目遇到的坑

>  nuxt/ pm2

## 1. 什么是nuxt ?
- 懒得讲了....(总之就是奔着SSR来的)

## 2. 什么是 pm2 ?
- 略..... (往下看吧)

## 3. nuxt工程安装过程中的选择

```
# nuxt 创建工程
# npx create-nuxt-app <项目名>

npx create-nuxt-app nuxtapp

```
### 3.1 我们看看安装时选择不同的选项,项目有什么不同

- 3.1.1 选择包管理器(Choose the package manager): npm- 略...

- 3.1.2 选择UI框架 

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/20191122112930.png)

  
`UI框架选择与否的差异`


![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/ps00001.png)

区别就在于plugins文件夹下的js文件,和nuxt.config.js中的配置不同(没什么影响)

- 3.1.3 选择自定义服务器框架

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/20191122113010.png)

  
`服务器框架选择与否的区别(区别很大)`

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/ps00002.png)

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/20191122131851.png)


区别就在于 
1. 安装了自定义服务器框架(例如koa), 会多一个server文件夹
2. package.json 中项目的启动方式不同(dev,start 命令)
3. 没有自定义服务器是靠nuxt start启动项目, 有自定义服务器是靠 node server/index.js 启动项目的(重点)

- 3.1.4 选择渲染模式

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/20191122113057.png)

`渲染模式选择的区别在项目中的体现`


![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/20191122132904.png)

主要体现在nuxt.config.js中mode的值(SSR: 'universal',SPA: 'spa'),目录结构上本人暂时没发现明显受影响的不同. 至于打包的区别在此不讲了...

### 3.2 以SSR模式打包,使用pm2 后台启动的坑

- 3.2.1 pm2 安装 (略)

- 3.2.2 pm2 后台启动nuxt

`咱们先看看坑在哪?`

在网上搜到的大部分命令无非是 pm2 start npm -- start <br>
但是这条命令并没有效果<br>
在网上搜了一圈有两个比较成功的案例
1. [Node pm2如何做进程管理Nuxt项目](https://www.xiangzongliang.com/blogContent?b=79)
2.  连接找不到了,通过安装node-cmd,用pm2 调用器脚本文件 运行npm start 命令

那么坑在哪?

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/20191122135143.png)

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/ps00003.png)

一堆报错+ 各种不如意
上面成功案例(1) 的缺点是随着nuxt版本的更新 nuxt在node_modules文件及文件夹名称变动可能带来的影响
<br>
使用node_cmd 那个案例有一个一直关不掉的黑窗口

### 3.3 解决办法

- 3.3.1 发现[1]网上的案例都没有安装自定义服务器框架


![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/1128831204.png) 

缺少server 文件夹(可能时他们在构建工程时没有选择服务器框架吧,而本人学习node时了解过koa,所以在构建工程时总不自觉的选择koa)<br>
server文件加下就一index.js,而且构建工程时选择自定义服务器框架的工程的package.json中的工程启动方式很引起了我的注意,<br>

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/20191122142053.png) 

于是我将server文件夹也整个拷贝了过去,然后配置了progresses.json <br>

```
pm2 start progresses.json

#progresses.json
{
    "name": "nuxtapp",
    "script": "./server/index.js",
    "cwd": "./",
    "watch": [
        ".nuxt",
        "server",
        "static"
    ],
    "ignore_watch": [
        "node_modules",
        "logs",
        "public"
    ],
    "watch_options": {
        "followSymlinks": false
    },
    "error_file": "./logs/app-err.log",
    "out_file": "./logs/app-out.log",
    "env": {
        "NODE_ENV": "production",
        "HOST": "127.0.0.1",
        "PORT": "3001"
    }
}
```
结果[1]:

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/ps00004.png) 


- 3.3.1 发现[2] 安装了自定义服务器框架打包结束时指定的有入口

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/20191122144955.png) 

除了[1],到底是哪一个呢?

![Image text](https://github.com/lys505735364/NUXT-PM2/blob/master/images/20191122145809.png) 



改配置运行试试....<br>

结果都不是!!!!!算了!!!

## 总结: 
> pm2 后台运行nuxt项目,项目构建时要引入一个自定义服务器框架,部署时将server文件夹也拷贝过去,pm2 运行server/index.js, 即可成功!




