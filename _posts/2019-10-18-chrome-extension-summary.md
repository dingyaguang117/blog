---
layout: post
title:  "MindHero项目总结 -- Chrome插件开发"
date:   2019-10-18 15:04:02 +0800
---


Chrome插件开发介绍
----

Chrome 插件开发主要还是基于Web的技术栈，主要的脚本运行环境就是如下图所示：

![](/assets/img/2019-10-18-chrome-1.png)

1. 常驻后台的 background.js
2. 点击地址栏右侧插件按钮会展示的 popup.html
3. 自动注入正常网页页面的 content script。

除此之外还有另外几种不太常用的类型：

4. 注入正常网页页面的 injected script（和 content script 的区别是是与页面是同一个 JS Runtime，即：可以访问所有的全局变量）
5. devtools js（开发者面板）
6. 覆盖默认的Chrome页面，比如: Newtab、下载页


项目结构设计
----

MindHero 插件主要功能包括：

1. 页面浏览记录统计 （background js）
2. Newtab页面替换 （独立页面）
3. Popup页面 （独立特免）
4. 网页内提醒 （content script）

所以基本上有4个主要的运行脚本，background 除了指定js作为入口也支持html page作为入口， 考虑到有多个的页面开发，所以统一采用的Vue多页面构建方式：

```
├── package.json
.
. 省略常规构建配置
.
├── scripts                       // 一些自动化脚本
├── src
│   ├── assets                    // 静态资源
│   ├── chrome                    // 需要原样拷贝到插件目录的资源
│   │   ├── manifest.json
│   │   └── resource
│   ├── common                    // 公共组件
│   │   ├── API.js
│   │   ├── DB.j                  // IndexedDB 框架 dexie.js 封装
│   │   ├── Model.js              // 数据结构定义
│   │   ├── Storage.js            // 对存储API进行封装，支持开发环境调试
│   │   ├── ChromeUtil.js         // 对Chrome API的封装， Promise化+支持开发环境调试
│   │   ├── EventBus.js
│   │   ├── config.js
│   │   .
│   │   .
│   └── pages             // 页面目录，每个目录会单独编译成 [directory-name].html
│      ├── background                    // background 页面
│      │   ├── background.html           // html 模板
│      │   ├── background.js             // entry: load and run all modules
│      │   └── modules
│      │       ├── AudioManager.js       // 背景音乐模块
│      │       ├── Runtime.js            // Runtime模块包含了所有运行时的配置参数和核心数据                         
│      │       ├── WebBlocker.js         // 阻挡网站模块
│      │       ├── WebtimeTracker.js     // 网站使用统计模块
│      │       .
│      │       .
│      ├── newtab                       // newtab 页面
│      │   ├── newtab.html              // html 模板
│      │   ├── newtab.js                // entry
│      │   ├── App.vue                  // 跟组件
│      │   ├── router.js                // 路由
│      │   ├── components               // vue 组件
│      │   .
│      │   .
│      └── popup
│      │   ├── popup.html              // html 模板
│      │   ├── popup.js                // entry
│      │   ├── App.vue                 // 根组件
│      │   .
│      │   .
└── static

```



一些 Best Practice
----


Chrome插件与web程序有一些不同的地方，比如:  

   1. 插件一般可以离线运行 
   2. 插件（可以）有常驻后台的程序。

所以也诞生了很多的新问题。在相当长的一段时间里，半天重构半天开发成为标准的开发节奏。

**1. 数据存储**

  - 一般的web前端程序只需要在页面 mounted 或者 created 的时候从服务器加载数据即可，如果支持离线访问那么数据就需要本地存储。我在这里选择的是chrome插件提供的 chrome.storage 存储能力， 最起初的好处是可以直接利用Google账户同步。
  - 为了支持在调试阶段使用存储能力，我将  chrome.storage 封装成存储模块，在调试阶段退化为使用 localStorage。 

**2. 数据共享**

  - 后台Runtime的数据很多是需要和页面共享的，比如：当前的番茄工作法状态；当前的背景音乐状态；实时的行为统计数据；用户的自定义设置。这里的数据同步架构如何设计？我的解决方式是使用共享内存，运行时数据放在background的Runtime里面，在启动插件的时候从本地load，其他页面可以在通过 getBackground().Runtime 引用运行时数据。
 


**3. 其他**

  - html-webpack-plugin在多页面中有性能问题，据说在4.0以后修复了，如果无法升级请使用 html-webpack-plugin-for-multihtml

 - 如何同步用户设置？ 如果不自己做后端服务的话，可是使用Chrome提供的 chrome.storage.sync 系列api。这个api提供了自动同步功能，可以在登录了google账号的chrome之间同步数据。

**to be continued...**
