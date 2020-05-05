---
layout: post
title: 【坚果商城实战系列学习】第 3-2 课：云开发开篇
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 3-2 课：云开发开篇
---

# 第 3-2 课：云开发开篇

## 1 初识云开发

云开发是微信团队和腾讯云团队共同研发的一套小程序基础能力，简言之就是：云能力将会成为小程序的基础能力。
小程序云开发目前提供三大基础能力支持：
- 云函数：在云端运行的代码，微信私有协议天然鉴权，开发者只需编写业务逻辑代码
- 数据库：一个既可在小程序前端操作，也能在云函数中读写的  JSON 数据库
- 文件存储：在小程序前端直接上传/下载云端文件，在云开发控制台可视化管理

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/046.png)

## 2 云开发怎么开发

程序写出来和写好程序是两件事情，在开始的章节官方的 demo，数据库操作是前台操作的，但是在是实际的开发中，数据库放在前台操作是不可取的，服务器的代码和本地小程序的代码是分开部署的，服务器的代码是无法直接查看到，在实际的开发中我们所有的数据操作全部放在服务器。大家调试了官方的 demo ，是不是发现每一个功能我们都需要一个云函数，这样让我们很难维护，这个肯定不是我们想要的结果，只要觉得不好维护和麻烦的事情，肯定有办法解决的。
小程序云函数高级玩法，在掘金开发者大会上，提到一种云函数的用法，我们可以将相同的一些操作，比如用户管理、支付逻辑，按照业务的相似性，归类到一个云函数里，这样比较方便管理、排查问题以及逻辑的共享。甚至如果你的小程序的后台逻辑不复杂，请求量不是特别大，完全可以在云函数里面做一个单一的微服务，根据路由来处理任务。
下面三幅图可以看出：
![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/047.png)
比如这里就是传统的云函数用法，一个云函数处理一个任务，高度解耦。
![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/048.png)
第二幅架构图就是尝试将请求归类，一个云函数处理某一类的请求，比如有专门负责处理用户的，或者专门处理支付的云函数。
![1](https://nux.oss-cn-beijing.aliyuncs.com/doc/049.png)
最后一幅图显示这里只有一个云函数，云函数里有一个分派任务的路由管理，将不同的任务分配给不同的本地函数处理。
想实现成最后一幅图，我们的使用咱们腾讯云  Tencent Cloud Base 团队开发了 [tcb-router](https://github.com/TencentCloudBase/tcb-router)，云函数路由管理库。大家点击 tcb-router 进去看看，如果之前做过 node + koa 的同学应该很熟悉，官方例子如下：

```
// 云函数的 index.js
const TcbRouter = require('./router');

exports.main = (event, context) => {
    const app = new TcbRouter({ event });
  
    // app.use 表示该中间件会适用于所有的路由
    app.use(async (ctx, next) => {
        ctx.data = {};
        await next(); // 执行下一中间件
    });

    // 路由为数组表示，该中间件适用于 user 和 timer 两个路由
    app.router(['user', 'timer'], async (ctx, next) => {
        ctx.data.company = 'Tencent';
        await next(); // 执行下一中间件
    });

    // 路由为字符串，该中间件只适用于 user 路由
    app.router('user', async (ctx, next) => {
        ctx.data.name = 'heyli';
        await next(); // 执行下一中间件
    }, async (ctx, next) => {
        ctx.data.sex = 'male';
        await next(); // 执行下一中间件
    }, async (ctx) => {
        ctx.data.city = 'Foshan';
        // ctx.body 返回数据到小程序端
        ctx.body = { code: 0, data: ctx.data};
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
tips: 小程序云函数的 node 环境默认支持 async/await 语法，推荐涉及到的异步操作时像 demo 中那样使用
小程序端
```
// 调用名为 router 的云函数，路由名为 user
wx.cloud.callFunction({
    // 要调用的云函数名称
    name: "router",
    // 传递给云函数的参数
    data: {
        $url: "user", // 要调用的路由的路径，传入准确路径或者通配符*
        other: "xxx"
    }
});
```

## 3 坚果商城云开发起步

为了方便演示，我为大家准备了演示的图片和数据，进入微信开发者工具中的`云开发`,在存储中新建下面几个文件夹，把对应的图片上传到后台。
之前我们的数据库设计，现在我们就在数据库一一把集合创建，小程序数据库使用 mongdb，不像mysql先把字段添加进去，在这里我们直接按着名字导入我给大家准备的数据，订单和订单详情在实际操作才会产生数据。
暂时我们只使用一个云函数实现我们后台的操作，下面是此我们云函数的文件介绍：
大家上传部署的时候如果出现同步不上去，一是再次操作，二是右击云函数文件夹先同步本地云函数在上传。

## 4 数据导入

数据存放在 nux-shop 中的 data 文件夹下 dababase 文件，在导入我们数据前我们也要在云开发控制面板中的数据库，新建集合在导入对应文件下的数据，如下图：

![1563608303097](https://nux.oss-cn-beijing.aliyuncs.com/doc/066.png)

图片数据上传，在控制面版中的存储，新建 banner 、product 、theme 三个文件夹，将 nux-shop 的 data 下的 images 根据对应文件夹上传

![1563608428278](https://nux.oss-cn-beijing.aliyuncs.com/doc/067.png)

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)