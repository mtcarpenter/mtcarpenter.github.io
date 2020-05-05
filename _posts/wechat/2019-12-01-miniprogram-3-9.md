---
layout: post
title: 【坚果商城实战系列学习】第 3-9 课：云开发之订单品数据实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 3-9 课：云开发之订单品数据实现
---

# 第 3-9 课：云开发之订单品数据实现

## 1 集合处理

在 `fields` 文件夹新建 `orderField.js`
```
// order 指定返回结果中记录需返回的字段
module.exports = {
    ORDERFIELD: {
        buyer_name: true,
        buyer_phone: true,
        buyer_address: true,
        order_amount: true,
        orderdetail: true,
        create_time: true,
        order_status: true
    }
}
```

## 2 业务层实现

> 在开始的时候，订单和订单详情是分别存取的 每一个订单详情都关联了订单号，我采用的通过 `node-uuid` 生成ID的形式，后面再优化的时候，`mongodb` 非关系数据库是比较灵活的，直接把订单详情放入订单中，也更好相对比较号操作。下面的内容没有移除的原因，想告诉大家在日常使用的过程中，我们在云函数中是可以安装其他依赖的，云函数在安装的时候记得添加 `--save` ,写入到 package.json  不然云上是读取不到本来依赖的。

订单 id 这里我采用自己生成，因为订单详情也需要使用订单 id ，在云函数`index`打开客户端安装`node-uuid`依赖，在命令窗口输入以下命令：

```
npm install --save  node-uuid
```
`node-uuid`使用比较简单,如下
```
引入
var uuid = require('node-uuid');
// V1 是根据时间戳生成。 
var uid = uuid.v1();
// V4 是随机数生成。 
var uidv4 = uuid.v4();
```
在我们实际的开发选择 v1 ，安装之后我们继续我们订单业务层编写

> 注意：上面的`node-uuid`代码中已经移除

`service/orderService.js`

```
const model = require('../models/BaseModel.js')
const { ORDER } = require('../config/tableConfig.js')
const { ORDERFIELD } = require('../fields/orderField.js')

//orderData,userInfo
const create = (orderData, userInfo) => {
    let orderdetailS = []
    // 添加订单详情
    let create_time = new Date()
    let update_time = new Date()
    for (let product of orderData.products) {
        let params_order_detail = {
            product_id: product._id,
            product_name: product.product_name,
            product_price: product.product_sell_price,
            product_count: product.counts,
            product_img: product.product_img,
            create_time: create_time,
            update_time: update_time
        }
        orderdetailS.push(params_order_detail)

    }
    // 订单信息
    let params_order = {
        buyer_openid: userInfo.openId,
        buyer_name: orderData.address.userName,
        buyer_phone: orderData.address.phone,
        buyer_address: orderData.address.detailAddress,
        order_amount: orderData.account,
        order_status: 0,// 默认未付款
        create_time: new Date(),
        update_time: new Date(),
        orderdetail: orderdetailS
    }

    // 订单生成
    let order = model.add(ORDER, params_order);
    return order
}
/**
 * 根据订单id获取订单信息
 * @param {*} orderId 
 */
const getOrderById = (orderId) => {
    return model.findById(ORDER, ORDERFIELD, orderId)
}

/**
 * 根据用户openid获取信息
 * @param {*} userInfo 
 */
const getOrderList = (userInfo, page = 0, size = 20, order = {}) => {
    order.name = 'create_time'
    order.orderBy = 'desc'
    let options = { buyer_openid: userInfo.openId }
    return model.query(ORDER, ORDERFIELD, options, page, size, order)
}



module.exports = {
    create,
    getOrderById,
    getOrderList
}
```
> 订单处理的数据比较多，在这里我们通过前台的数据解析出来，放进我们的自己的集合中，作为演示我是之前通过前台取值存进去，如果在真实的案例中，大家需要通过从后台取出数据与前台对比，以免前台数据传入错误。

## 3  入口文件实现

头部引入

```
const order = require('service/orderService.js')
```
```
  /***************************    订单   *****************************************/
  // 生成订单
  app.router('creatOrder', async (ctx, next) => {
    //event.data.orderData,event.userInfo
    ctx.data = await order.create(event.data.orderData, event.userInfo)
    ctx.body = await returnUtil.success(ctx)
    await next()
  })

  // 根据订单获取信息
  app.router('getOrderById', async (ctx, next) => {
    let orderId = event.data.orderId
    ctx.data = await order.getOrderById(orderId)
    ctx.body = await returnUtil.success(ctx)
    await next()
  })

  // 获取订单信息
  app.router('getOrderList', async (ctx, next) => {
    ctx.data = await order.getOrderList(event.userInfo)
    ctx.body = await returnUtil.success(ctx)
    await next()
  })

```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)