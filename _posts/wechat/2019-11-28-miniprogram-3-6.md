---
layout: post
title: 【坚果商城实战系列学习】第 3-6 课：云开发之首页数据实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 3-6 课：云开发之首页数据实现
---

# 第 3-6 课：云开发之首页数据实现

## 1 集合处理

在实际的开发中，大家尽量不要吧所有的数据返回给前台，很多时候我们需要几个字段返回了几十个这样是非常不友好的，还有我们什么都返回别人很容易就知道我们的后台实现，避免不必要的破坏。
在[官方文档](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-server-api/database/collection.field.html)中过滤字段采用 `filed` 方法，如下：

- Collection.field / Query.field / Document.field
指定返回结果中记录需返回的字段
- 方法签名如下：
function field(definition: object): Collection | Query | Document
方法接受一个必填字段用于指定需返回的字段
- 示例代码
返回 description, done 和 progress 三个字段：

```
const cloud = require('wx-server-sdk')
cloud.init()
const db = cloud.database()
exports.main = async (event, context) => {
  try {
    return await db.collection('todos').field({
      description: true,
      done: true,
      progress: true
    }).get()
  } catch(e) {
    console.error(e)
  }
}
```
> 设置字段为 true 则是返回字段，需要注意的是自增 id ，如果不需要返回需要 ` _id : false `，默认会返回，需要手动处理。

首页包括涉及到三个集合处理轮播、主题、商品信息。
在 `index`  云函数新建夹 `fields` ,处理集合返回前台的字段。
在 `fields` 文件夹新建 `bannerField.js`
```
// banner 指定返回结果中记录需返回的字段
module.exports = {
  BANNERFIELD: {
    name: true,
    description: true,
    image: true,
    product_id:true
  }
}
```
在 `fields ` 文件夹新建 `themeField.js`
```
// theme 指定返回结果中记录需返回的字段 _id需要特殊处理
module.exports = {
  THEMEFIELD: {
    theme_icon: true,
    theme_name: true,
    theme_type: true,
    _id: false 
  }
}
```
在 `fields` 文件夹新建 `productField.js`
```
// product 指定返回结果中记录需返回的字段
module.exports = {
  PRODUCTFIELD: {
    product_name: true,
    product_img: true,
    product_price: true,
    product_sell_price: true,
    product_stock: true,
    product_description:true
  }
}
```

##  2 业务层实现

在 `index` 云函数新建夹 `service` ,存放业务逻辑相关代码
在 `service` 文件夹新建 `bannerService.js`

```
// 导入数据库操作公共方法
const model = require('../models/BaseModel.js')
// 全局集合名称
const { BANNER } = require('../config/tableConfig.js')
// 返回字段处理
const { BANNERFIELD } = require('../fields/bannerField.js')
/**
 * 获取首页轮播
 * @return 
 */
const getBanner = ()=>{
  // 查询需要前天显示的轮播
  let options = {show : true}
  // 传递参数   对应的BaseModel的方法名称
  return model.query(BANNER, BANNERFIELD, options )
}
// 导出
module.exports = {
  getBanner
}
```
- 在 vscode 大家鼠标执行相应的方法上面，会提示参数位置，在实际的开发大家可以使用 vscode + 微信开发者工具一起开发，这样更能提高效率。
在 `service` 文件夹新建 `themeService.js`
```
// 引入BaseModel 集合名  字段过滤
const model = require('../models/BaseModel.js')
const { THEME } = require('../config/tableConfig.js')
const { THEMEFIELD } = require('../fields/themeField.js')

/**
 * 获取主题列表
 * @return 
 */
const getTheme = () => {
  return model.query( THEME, THEMEFIELD )
}


module.exports = {
  getTheme
}
```
只有导出的代码，我们导入才不会报错，这里实现是使用的 ES6 相关特性，大家如果不是特别理解，在回顾之前的再回头巩固下。
在 `service` 文件夹新建 `productService.js`
```
const model = require('../models/BaseModel.js')
const { PRODUCT  } = require('../config/tableConfig.js')
const { PRODUCTFIELD } = require('../fields/productField.js')

/**
 * 获取商品
 * @param options 条件
 * @param page    
 * @param size
 * @return 
 */
const getProduct = (options , page = 0, size = 10 , order = {} ) => {
  // 查询条件
  options.product_status = 1
  // 排序条件 根据需要调正优化
  order.name = 'creat_time'
  order.orderBy= 'asc'
  return model.query(PRODUCT, PRODUCTFIELD, options, page,size,order)
}


module.exports = {
  getProduct
}
```
对于操作的可能相对其他调用的会更多，所以这里在业务层也增加不少的条件，方便灵活调用，方法复用。

## 3 入口文件实现

在前面我们已经编写入口文件的基本实现，现在我们继续编写，引入业务层文件

```
// 返回工具类
const returnUtil = require('utils/ReturnUtil.js')
// 轮播业务层
const banner = require('service/bannerService.js')
// 主题业务层
const theme  = require('service/themeService.js')
// 商品信息业务层
const product = require('service/productService.js')
```
编写首页交互代码
```
/***************************    首页   *****************************************/
  // 获取轮播
  app.router('getBanner', async (ctx, next) => {
     // banner.getBanner() 取出数据，然后对数据进行重新拼装
    ctx.data = await _bannerItem(banner.getBanner())
    ctx.body = await returnUtil.success(ctx)
    await next();
  })
  // 获取主题
  app.router('getTheme',async (ctx,next) => {
    ctx.data = await _themeItem(theme.getTheme())
    ctx.body = await returnUtil.success(ctx)
    await next()
  })
  // 获取最新前5商品
  app.router('getProductNew',async (ctx,next) => {
    // product.getProduct({},0,4)  {}：没有条件直接控{}，0，4 分页查询 
    ctx.data = await _productItem(product.getProduct({},0,4))
    ctx.body = await returnUtil.success(ctx)
    await next()
  })

// 轮播图片地址拼接
  function _bannerItem(data){
    return new Promise((resolve,reject) => {
      data.then(res=>{
        res.data.forEach(data => {
          data.image = 'cloud://release-prod.7265-release-prod'+data.image
        })
        resolve(res)
      })
    }) 
  }
// 主题图片地址拼接
  function _themeItem(data){
    return new Promise((resolve,reject) => {
      data.then(res=>{
        res.data.forEach(data => {
          data.theme_icon = 'cloud://release-prod.7265-release-prod'+data.theme_icon
        })
        resolve(res)
      })
    }) 
  }
 // 多个商品图片地址拼接
  function _productItem(data){
    return new Promise((resolve,reject) => {
      data.then(res=>{
        res.data.forEach(data => {
          data.product_img = 'cloud://release-prod.7265-release-prod'+data.product_img
        })
        resolve(res)
      })
    }) 
  }
  
```
> 在数据库中图片并没有存取访问的地址，因此在我们取出数据之后需要手动拼接，每一个方法前缀为`_`，表示只在当前类和当前js调用。returnUtil.success() 对数据做统一的返回格式处理。

如果前面编写完成了，大家对编写每一个路由进行查询，查看是否有遗漏的地方。

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)