---
layout: post
title: 【坚果商城实战系列学习】第 3-1 课：数据库设计
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 3-1 课：数据库设计
---

# 第 3-1 课：数据库设计

## 1 banner ：轮播

| 字段名      | 数据类型 | 默认值 | 含义                     | 备注               |
| ----------- | -------- | ------ | ------------------------ | ------------------ |
| id          | string   |        | 主键                     |                    |
| name        | string   |        | Banner名称，通常作为标识 |                    |
| description | string   |        | 描述                     |                    |
| image       | string   |        | 主图                     |                    |
| show        | int      |        | 是否显示                 | 0：显示，1：不显示 |
| product_id  | String   |        | 商品id                   |                    |
| update_time | date     |        | 创建时间                 |                    |

## 2 theme ：主题

| 字段名         | 数据类型 | 默认值 | 含义     | 备注 |
| -------------- | -------- | ------ | -------- | ---- |
| id             | string   |        | 主键     |      |
| thme_name      | string   |        | 专题名   |      |
| thme_type      | string   |        | 编号     |      |
| theme_icon     | string   |        | 专题小图 |      |
| theme_head_img | string   |        | 专题图   |      |

## 3 product_category ：商品分类

| 字段名        | 数据类型 | 默认值 | 含义     | 备注 |
| ------------- | -------- | ------ | -------- | ---- |
| category_id   | string   |        | 主键     |      |
| category_name | string   |        | 类目名字 |      |
| category_type | int      |        | 类目编号 |      |
| create_time   | date     |        | 创建时间 |      |
| update_time   | date     |        | 修改时间 |      |
## 4 product ：商品信息

| 字段名              | 数据类型 | 默认值 | 含义     | 备注          |
| ------------------- | -------- | ------ | -------- | ------------- |
| product_id          | string   |        | 主键     |               |
| product_name        | String   |        | 商品名   |               |
| product_img         | string   |        | 商品主图 |               |
| product_price       | double   |        | 商品名   | 价格,单位：分 |
| product_stock       | long     |        | 商品数量 |               |
| product_description | string   |        | 商品描述 |               |
| product_status      | int      |        | 商品状态 |               |
| category_type       | int      |        | 分类     |               |
| create_time         | date     |        | 创建时间 |               |
| update_time         | date     |        | 更新时间 |               |

## 5 product_theme : 专题商品

| 字段名     | 数据类型 | 默认值 | 含义     | 备注 |
| ---------- | -------- | ------ | -------- | ---- |
| theme_id   | string   |        | 主键     |      |
| theme_type | string   |        | 专题编号 |      |
| product_id | string   |        | 商品id   |      |

## 6 order ：订单

| 字段名        | 数据类型 | 默认值 | 含义     | 备注                                                    |
| ------------- | -------- | ------ | -------- | ------------------------------------------------------- |
| order_id      | string   |        | 订单主键 |                                                         |
| buyer_openid  | string   |        | 订单id   |                                                         |
| buyer_name    | string   |        | 买家姓名 |                                                         |
| buyer_phone   | string   |        | 买家电话 |                                                         |
| buyer_address | string   |        | 买家地址 |                                                         |
| order_amount  | double   |        | 总价     |                                                         |
| order_status  | int      |        | 订单状态 | 1:未支付， 2：已支付，3：已发货 , 4: 已支付，但库存不足 |
| create_time   | date     |        | 创建时间 |                                                         |
| update_time   | date     |        | 更新时间 |                                                         |

## 7 order_detail : 订单详情

| 字段名        | 数据类型 | 默认值 | 含义     | 备注 |
| ------------- | -------- | ------ | -------- | ---- |
| detail_id     | string   |        | 主键     |      |
| order_id      | string   |        | 订单id   |      |
| product_id    | string   |        | 商品id   |      |
| product_name  | string   |        | 商品名   |      |
| product_price | string   |        | 商品价格 |      |
| product_count | int      |        | 商品数量 |      |
| product_img   | stirng   |        | 商品图片 |      |
| create_time   | date     |        | 创建时间 |      |
| update_time   | date     |        | 更新时间 |      |

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)