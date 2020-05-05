---
layout: post
title: 【坚果商城实战系列学习】第 3-7 课：云开发之分类数据实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 3-7 课：云开发之分类数据实现
---

# 第 3-7 课：云开发之分类数据实现

## 1 集合处理

在 `fields` 文件夹新建 `productCategoryField.js`

```
//  指定返回结果中记录需返回的字段
module.exports = {
    PRODUCT_CATEGORY_FIELD: {
      category_name: true,
      category_type: true,
      _id:false
    }
  }
```
> 在前面的章节，我也提到过，`_id` ,不需要返回咱们的手动写，否则会返回给前台

## 2 业务层实现 

`service/productService.js`

```
// 在原来的上面增加 PRODUCT_CATEGORY
const { PRODUCT ,PRODUCT_CATEGORY } = require('../config/tableConfig.js')
// 新增分类字段过滤
const { PRODUCT_CATEGORY_FIELD } = require('../fields/productCategoryField.js')
/**
 * 获取商品分类
 * @return 
 */
const getCategoryMenu = () =>{
  return model.query(PRODUCT_CATEGORY,PRODUCT_CATEGORY_FIELD)
}

/**
 * 根据商品分类获取商品
 * @param {*} options 
 */
const getCategoryProduct = (options) => {
  options.product_status = 0 
  return model.query(PRODUCT, PRODUCTFIELD, options)
}

```


**三、入口文件实现**

```
/***************************    分类   *****************************************/
  // 获取分类
  app.router('getCategoryMenu', async (ctx,next) =>{
    ctx.data = await product.getCategoryMenu()
    ctx.body = await returnUtil.success(ctx)
    await next()
  })
  // 获取分类商品
  app.router('getCategoryProduct', async (ctx , next) => {
    let options = {}
    // ctx.data 前台传过来的category_type
    options.category_type = event.data
    ctx.data = await _productItem(product.getCategoryProduct(options))
    ctx.body = await returnUtil.success(ctx)
    await next()
  })
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)