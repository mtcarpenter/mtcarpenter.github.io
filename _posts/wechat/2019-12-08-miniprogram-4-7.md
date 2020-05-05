---
layout: post
title: 【坚果商城实战系列学习】第 4-7 课：前后端交互之订单实现  
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 4-7 课：前后端交互之订单实现  
---

# 第 4-7 课：前后端交互之订单实现  

## 1 逻辑处理

打开 `client` 新建 `models/orderModel.js` ,新增
```
import { CloudRequest } from '../utils/cloud-request.js'
class OrderModel extends CloudRequest {

    /**
     * 生成订单
     * @param {*} orderData 
     * @param {*} callBack 
     */
    creat(orderData, callBack) {
        this.request({
            url: "creatOrder",
            data: { orderData: orderData },
            success: res => {
                callBack(res)
            }
        })
    }

    /**
     * 根据订单id查询
     * @param {*} orderId 
     * @param {*} callBack 
     */
    getOrderById(orderId, callBack) {
        this.request({
            url: "getOrderById",
            data: { orderId: orderId },
            success: res => {
                callBack(res)
            }
        })
    }

}

export { OrderModel }
```

##  2 前台数据处理

回到我们 `pages/order/order.js`
```
// pages/order/order.js
import { CartModel } from '../../models/CartModel.js'
import {
  OrderModel
} from '../../models/OrdelModel.js'
import { ProductModel } from '../../models/productModel.js'
let cartmodel = new CartModel()
let productModel = new ProductModel()
let orderModel = new OrderModel()
Page({

  /**
   * 页面的初始数据
   */
  data: {
    address: [],
    products: [],
    account: [],
    orderStatus: 0,
    oldOrder: false,
    orderId: ''
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    this.data.account = options.account
    //购物车
    if (options.from == 'cart') {
      this.setData({
        products: cartmodel.getCartDataFromLocal(true),
        account: options.account,
        orderStatus: 0
      })
    } else if (options.from == 'product') {
      // let 获取商品信息
      productModel.getProductById(options.productId, res => {
        let product = res.result.data.data
        product.counts = parseInt(options.count)
        let newData = []
        newData.push(product);
        this.setData({
          products: newData,
          account: options.count * product.product_sell_price,
          orderStatus: 0
        })
      })
    }
    //旧订单
    else {
      orderModel.getOrderById(options.id, res => {
        let data = res.result.data.data
        let address = {}
        address.userName = data.buyer_name
        address.phone = data.buyer_phone
        address.detailAddress = data.buyer_address
        this.setData({
          orderId: data._id,
          address: address,
          products: data.orderdetail,
          account: data.order_amount,
          orderStatus: data.order_status,
          oldOrder: true
        })
      })
    }
  },

   //地址信息
  addressInfo: function (e) {
    if ("ok" == e.detail.status) {
      let address = {}
      let addressInfo = e.detail.addressInfo
      address.userName = addressInfo.userName
      address.phone = addressInfo.telNumber
      address.detailAddress = addressInfo.provinceName + addressInfo.cityName + addressInfo.countyName + addressInfo.detailInfo
      this.setData({
        address
      })
    }

  },
  // 提交订单
  confirm: function () {
    // 判断是否选择地址
    if (this.data.address.length == 0) {
      this._showToast('none', '请选择地址')
      return
    }
    wx.getSetting({
      success: (res) => {
        if (!res.authSetting['scope.userInfo']) {
          wx.showModal({
            title: '授权提示',
            content: '下单需要在个人中心授权！',
            success(res) {
              wx.switchTab({
                url: '/pages/my/my'
              })
            }
          })
        } else {
          // 判断是否是旧订单
          if (this.data.oldOrder) {
            this._showToast('none', '订单支付暂未开通！')
            return
          }
          // 地址拼接
          let orderData = {}
          orderData.address = this.data.address
          orderData.products = this.data.products
          orderData.account = this.data.account
          // 创建订单
          orderModel.creat(orderData, res => {
            if (res.result.code == 0) {
              this._showToast('none', '订单创建成功!')
              // 设置成就订单
              this.setData({
                oldOrder: true
              })
              let ids = []
              let products = this.data.products
              for (let i = 0; i < products.length; i++) {
                ids.push(products[i]._id);
              }
              cartmodel.delete(ids, res => { })
              wx.showModal({
                title: '支付提示',
                content: '支付暂未实现！',
                success(res) {
                  wx.switchTab({
                    url: '/pages/my/my'
                  })
                }
              })
            }
          })


        }
      }
    })


  },
  // 订单详情
  productDetail: function (event) {
    let product_id = orderModel.getDataSet(event, "id")
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + product_id,
    })
  },
  _showToast: function (type, msg) {
    wx.showToast({
      icon: type,
      title: msg
    })
  }
})
```

`pages/order/order.wxml`
```
<!--pages/order/order.wxml-->
<wxs module="filter" src="../../common/wxs/filters.wxs"></wxs>
<view class='container'>
  <!-- 地址 -->
  <view wx:if='{{address.length!=0}}'>
    <view wx:if="{{status!=0}}">
<address-comp addressInfo="{{address}}" slot="address"></address-comp>
    </view>
    <view wx:else>
      <img-btn-comp open-type="primary" bind:addressInfo="address">
        <address-comp addressInfo="{{address}}" slot="address"></address-comp>
      </img-btn-comp>
    </view>
  </view>

  <img-btn-comp open-type="primary" bind:addressInfo="addressInfo" wx:else>
    <view class='redirect-address' slot="address">
      <text class='redirect-text'>新增收货地址</text>
      <icon class='iconfont icon-webicon213'></icon>
    </view>
  </img-btn-comp>
  <view class='product-container'>
    <block wx:for="{{products}}" wx:key="">
      <view class="product-item">
        <view class="item-left"  data-id="{{item.product_id}}" bind:tap="productDetail">
          <image src="{{item.product_img}}"></image>
        </view>
        <view class="item-middle">
          <view>{{item.product_name}}</view>
          <view>￥{{item.product_sell_price==null?item.product_price:item.product_sell_price}}</view>
        </view>
        <view class="item-right">
          ×{{item.counts==null?item.product_count:item.counts}}
        </view>
      </view>
    </block>
  </view>
  <view class='order-container'>
    <view class='order-remarks'>
     <view class='details' wx:if="{{orderId!=null}}">
        <text>订单编号:</text>
        <text>{{orderId}}</text>
      </view>
      <view class='details'>
        <text>配送方式:</text>
        <text>快递 免邮</text>
      </view>
      <view class='details'>
        <text>优惠券:</text>
        <text>暂无可用</text>
      </view>
      <view class='details'>
        <text>配送费:</text>
        <text>¥0</text>
      </view>
    </view>
  </view>
  <view class='confirm-container'>
    <view class='total'>
      <text>合计:</text>
      <text>¥{{filter.toFix(account)}}</text>
    </view>
      <view class='confirm' data-key='{{item.key}}' bindtap='confirm'>
        <text>立即购买</text>
      </view>

  </view>

</view>
```
## 3 订单组件

`components/address/address.js`

```
// components/address/address.js
Component({
  /**
   * 组件的属性列表
   */
  properties: {
    addressInfo:Object
  },

  /**
   * 组件的初始数据
   */
  data: {

  },

  /**
   * 组件的方法列表
   */
  methods: {

  }
})

```

`components/address/index.wxml`

```
<!--components/address/address.wxml-->
<view class='container'>
  <view class='address-title'>
    <text class='name'>收货人：{{addressInfo.userName}}</text>
    <text class='phone'>{{addressInfo.phone}}</text>
  </view>
  <view class='address-content'>
    <text class='address'>收货地址：{{addressInfo.detailAddress}}</text>
    <icon class='iconfont icon-webicon213'></icon>
  </view>

</view>

```

`components/image-button/index.js`

```
// components/image-button/index.js
Component({
  /**
   * 组件的属性列表
   */
  options: {
    multipleSlots: true // 在组件定义时的选项中启用多slot支持
  },
  properties: {
    openType: {
      type: String
    },
    imageSrc: {
      type: String
    },
    bindgetuserinfo: {
      type: String
    }
  },
  /**
   * 组件的初始数据
   */
  data: {
    addressInfo:""
  },
  /**
   * 组件的方法列表
   */
  methods: {
    onGetUserInfo(event) {   
      if (this.data.openType == "getUserInfo"){             
       // if (event.detail.errMsg) {
        this.triggerEvent('getuserinfo', event.detail, {})
       // }
      }           
    }, 
    address(event){    
      if (this.data.openType == "primary") {
        this.addressInfo(event) 
      }      
    },
    addressInfo(event){      
      wx.chooseAddress({
        success: (res) => {
          this.setData({
            addressInfo: res
          })
        },
        fail:(err) =>{}, 
        complete:(e)=> {
          if (e.errMsg == "chooseAddress:ok") {
            this.triggerEvent('addressInfo', {
              addressInfo: this.data.addressInfo,
              status: "ok"
            }, {})
          } else {
            this.triggerEvent('addressInfo', {
              status: "error"
            }, {})
          }

        }
      })
    }
  }
})

```

`components/image-button/index.wxml`

```
<!--components/image-button/index.wxml-->
<button bind:getuserinfo="onGetUserInfo" bind:tap='address' open-type='{{openType}}'  plain='{{true}}' class="container" lang="zh_CN">
  <slot name="share"></slot>
  <slot name="address"></slot>
  <slot name="info"></slot>
</button>

```

## 4 代码解析

### 4.1 初始化数据

订单信息来源为购物车、商品详情、个人中心。个人中心为已经生成的订单，其它两个为还为提交的订单。来源购物车订单通过 `getCartDataFromLocal` 方法获取本地选中的商品，来源商品详情通过商品的 id 获取信息，进行数据的渲染，来源个人中心，则需要通过根据订单获取商品的信息。

```
 onLoad: function (options) {
    this.data.account = options.account
    //购物车
    if (options.from == 'cart') {
      this.setData({
        products: cartmodel.getCartDataFromLocal(true),
        account: options.account,
        orderStatus: 0
      })
      // 商品详情
    } else if (options.from == 'product') {
      //  获取商品信息
      productModel.getProductById(options.productId, res => {
        let product = res.result.data.data
        product.counts = parseInt(options.count)
        let newData = []
        newData.push(product);
        this.setData({
          products: newData,
          account: options.count * product.product_sell_price,
          orderStatus: 0
        })
      })
    }
    //旧订单
    else {
      orderModel.getOrderById(options.id, res => {
        let data = res.result.data.data
        let address = {}
        address.userName = data.buyer_name
        address.phone = data.buyer_phone
        address.detailAddress = data.buyer_address
        this.setData({
          orderId: data._id,
          address: address,
          products: data.orderdetail,
          account: data.order_amount,
          orderStatus: data.order_status,
          oldOrder: true
        })
      })
    }
   }, 
```

### 4.2 地址信息

这里的地址信息选择是微信的地址，通过调用组件 `image-button ` 实现，将获取的地址信息进行重新拼装。

```
//地址信息
  addressInfo: function (e) {
    if ("ok" == e.detail.status) {
      let address = {}
      let addressInfo = e.detail.addressInfo
      address.userName = addressInfo.userName
      address.phone = addressInfo.telNumber
      address.detailAddress = addressInfo.provinceName + addressInfo.cityName + addressInfo.countyName + addressInfo.detailInfo
      this.setData({
        address
      })
    }

  },
```

### 4.3 提交订单

提交订单首先下单之前必须选择地址，进行地址的检测，云开发订单操作需要用户授权，如果没有授权则跳转到个人中心提示用户授权。授权过的订单先判断当前订单信息是否旧订单，在上面获取订单信息设置过，不是旧订单进行数据拼接生成订单，如果是购物车的商品，生成订单成功删除购物车数据。

```
 // 提交订单
  confirm: function () {
    // 判断是否选择地址
    if (this.data.address.length == 0) {
      this._showToast('none', '请选择地址')
      return
    }
    wx.getSetting({
      success: (res) => {
        if (!res.authSetting['scope.userInfo']) {
          wx.showModal({
            title: '授权提示',
            content: '下单需要在个人中心授权！',
            success(res) {
              wx.switchTab({
                url: '/pages/my/my'
              })
            }
          })
        } else {
          // 判断是否是旧订单
          if (this.data.oldOrder) {
            this._showToast('none', '订单支付暂未开通！')
            return
          }
          // 地址拼接
          let orderData = {}
          orderData.address = this.data.address
          orderData.products = this.data.products
          orderData.account = this.data.account
          // 创建订单
          orderModel.creat(orderData, res => {
            if (res.result.code == 0) {
              this._showToast('none', '订单创建成功!')
              // 设置成就订单
              this.setData({
                oldOrder: true
              })
              let ids = []
              let products = this.data.products
              for (let i = 0; i < products.length; i++) {
                ids.push(products[i]._id);
              }
              cartmodel.delete(ids, res => { })
              wx.showModal({
                title: '支付提示',
                content: '支付暂未实现！',
                success(res) {
                  wx.switchTab({
                    url: '/pages/my/my'
                  })
                }
              })
            }
          })


        }
      }
    })


  },
  // 订单详情
  productDetail: function (event) {
    let product_id = orderModel.getDataSet(event, "id")
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + product_id,
    })
  },
  _showToast: function (type, msg) {
    wx.showToast({
      icon: type,
      title: msg
    })
  }
```

#### 4.4 订单详情

跟之前实现的一样

```
// 订单详情
  productDetail: function (event) {
    let product_id = orderModel.getDataSet(event, "id")
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + product_id,
    })
  }
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)