---
layout: post
title: 【坚果商城实战系列学习】第 3-3 课：云开发之环境搭建
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 3-3 课：云开发之环境搭建
---

# 第 3-3 课：云开发之环境搭建

> 云开发之前需要掌握 ES6 中的 let 、const 、promise 、导入、导出、类，这些常见的特性。

## 1 环境搭建

小程序内提供了专门用于云函数调用的 API 。开发者可以在云函数内使用 wx-server-sdk 提供的 getWXContext 方法获取到每次调用的上下文（ Appid 、openid 等），无需维护复杂的鉴权机制，即可获取天然可信任的用户登录态（ openid ）。

新建云函数 `index`

右击 `index `打开命令窗口
安装 `wx-server-sdk`
```
npm install --save wx-server-sdk@latest
```
安装 `tcb-router`
```
npm install --save tcb-router
```
## 2 云函数入口文件编写

`index.js `

```
// 云函数入口文件
const cloud = require('wx-server-sdk')
const TcbRouter = require('tcb-router')

/ 云函数入口函数
exports.main = async (event, context) => {
  const app = new TcbRouter({ event });
  // app.use 表示该中间件会适用于所有的路由
  app.use(async (ctx, next) => {
    ctx.data = {};
    await next(); // 执行下一中间件
  });
  
  
  // 路由为字符串，该中间件只适用于 timer 路由
    app.router('timer', async (ctx, next) => {
        ctx.data.name = 'flytam';
        await next(); // 执行下一中间件
    }, async (ctx, next) => {
        ctx.data.sex = await new Promise(resolve => {
        // 等待500ms，再执行下一中间件
        setTimeout(() => {
            resolve('male');
        }, 500);
        });
        await next(); // 执行下一中间件
    }, async (ctx)=>  {
        ctx.data.city = 'Taishan';

        // ctx.body 返回数据到小程序端
        ctx.body = { code: 0, data: ctx.data };
    });  
        
     
   return app.serve();
}

```
-   将 `tcb-router` 中的 timer 作为测试。
  本地测试右击云函数 `index` 打开本地调试，勾选右边的本地调试，如果本地没有安装依赖本地调式会报错，需要安装本地依赖才能打开调式。
  后面所有的请求需要带上 `$url`，否则找不到路由。如下

![1563024084358](https://nux.oss-cn-beijing.aliyuncs.com/doc/065.png)

本地调试已经完成接下来我们将开始后台开发。

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)