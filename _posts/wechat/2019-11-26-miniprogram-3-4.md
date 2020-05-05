---
layout: post
title: 【坚果商城实战系列学习】第 3-4 课：云开发之配置文件和工具类
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 3-4 课：云开发之配置文件和工具类
---

# 第 3-4 课：云开发之配置文件和工具类

## 1 配置文件

数据库少了对集合的大量操作，对于集合的操作可能不会在同一个 serive ，所以我们使用全局常量方便后期维护。
云函数 `index` 中新建文件 `config/tableConfig.js`

```
// 集合名 
module.exports = {
  BANNER : 'banner', 
  THEME  : 'theme',
  PRODUCT: 'product',
  PRODUCT_THEME : 'product_theme',
  PRODUCT_CATEGORY : 'product_category',
  ORDER:"order"
}
```

##  2 返回结果工具类

在`云开发上手`的章节我为大家贴出一段官方的 demo，中有这么一句

```
 // ctx.body 返回数据到小程序端
  ctx.body = { code: 0, data: ctx.data };
```
如果我们有几十个操作我们的结果都需要复制几十个出来，如果后面的需求让我们 code 返回200，再返回一个 message ,那么我们就是的重新修改这几十行相同的代码，前面说过重复的代码，我们尽量优化，在这里我们自定义一个返回的工具类。
在云函数 `index` 中新建文件 `utils/ReturnUtil.js`
```
/**
 * 成功调用
 * @param {*} ctx
 * @retuen 
 */
const success = ctx => {
  return {
    code: 0,
    message: 'success',
    data: ctx.data
  }
}

/**
 * 调用失败 
 * @param {*} ctx
 * @param {*} msg
 * @retuen 
 */
const error = (ctx,msg) => {
  return {
    code: 400,
    message: msg,
    data: ctx.data
  }
}

module.exports={
  success,
  error
}
```
##  3 入口基本配置

```
// 云函数入口文件
const cloud = require('wx-server-sdk')
const TcbRouter = require('tcb-router');

cloud.init()


// 云函数入口函数
exports.main = async (event, context) => {
  const app = new TcbRouter({ event });
  // app.use 表示该中间件会适用于所有的路由
  app.use(async (ctx, next) => {
    ctx.data = {};
    await next(); // 执行下一中间件
  });
 
/***************************    首页   *****************************************/


/***************************    分类   *****************************************/


/***************************    商品信息   *************************************/  



/***************************    主题商品   *************************************/  



/***************************    订单   *****************************************/  


/***************************    测试   *****************************************/    


  return app.serve();
}
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)