---
layout: post
title: 【坚果商城实战系列学习】第 3-8 课：云开发之商品信息和主题商品数据实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 3-8 课：云开发之商品信息和主题商品数据实现
---

# 第 3-8 课：云开发之商品信息和主题商品数据实现

> 因为前面做了大量的铺垫，越到后面我实现起来越简单，商品信息和主题商品目前只有两个路由，我就放在一遍文章里面写。

## 1 集合处理

在 `fields` 文件夹新建 `productThemeField.js`

```
//  指定返回结果中记录需返回的字段
module.exports = {
    PRODUCTTHEMEFIELD: {
       theme_type: true,
       product_theme: true,
       _id: false
    }
  }
```

## 2 业务层实现

`service/productService.js`
```
// 头部引入
const { PRODUCTTHEMEFIELD } = require('../fields/productThemeField.js')
/**
 * 获取单个商品
 * @param product_id 条件
 * @return 
 */
const getProductById = (product_id) => {
  return model.findById(PRODUCT, PRODUCTFIELD, product_id)
}
/**
 * 获取商品主题
 * @param product_theme 条件
 * @return 
 */
const getThemeProduct = (product_theme) => {
  let options = {product_theme:product_theme}
  return  model.query(PRODUCT, PRODUCTFIELD, options)
}
```

## 3 入口文件实现

```
/***************************    商品信息   *****************************************/  
// 获取商品信息
app.router('getProductById', async (ctx,next) =>{
  let product_id =  event.data.product_id
  ctx.data = await _productImg(product.getProductById(product_id))
  ctx.body = await returnUtil.success(ctx)
  await next()
})


/***************************    主题商品   *****************************************/  
// 获取主题商品列表
app.router('getThemeProduct', async (ctx,next) =>{
  // 前台传入主题类型    
  let theme_type = event.data.theme_type
  ctx.data = await _productItem(product.getThemeProduct(theme_type))
  ctx.body = await returnUtil.success(ctx)
  await next()
})
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)