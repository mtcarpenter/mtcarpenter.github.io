---
layout: post
title: 【坚果商城实战系列学习】第 4-5 课：前后端交互之购物车实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 4-5 课：前后端交互之购物车实现
---

# 第 4-5 课：前后端交互之购物车实现

## 1 逻辑处理

`client` 新建 `models/CartModel.js`
```
import { CloudRequest } from '../utils/cloud-request.js'
class CartModel extends CloudRequest {
    _storageKeyName = 'cart';
    constructor() {
        super()
    };

    /**
     * 获取购物车数据
     * @param {*} callBack 
     */
    getCartData(callBack) {
        callBack(this.getCartDataFromLocal())
    }

    /**
     * 添加到购物车
     * @param {*} item 
     * @param {*} counts 
     * @param {*} callBack 
     */
    add(item, counts, callBack) {
        callBack(this._localAdd(item, counts))
    }

    /**
     * 增加商品数量
     * @param {*} id 
     * @param {*} callBack 
     */
    addCounts(id, callBack) {
        this._changeCounts(id, 1)
        callBack()
    }
    /**
     * 减少数量
     * @param {*} id 
     * @param {*} callBack 
     */
    cutCounts(id, callBack) {
        this._changeCounts(id, -1)
        callBack()
    }

    /**
     * 删除商品
     * @param {*} ids 
     * @param {*} callBack 
     */
    delete(ids, callBack) {
        callBack(this._delete(ids))
    }

    /********************* 下面本地数据  ***************************/

    /*本地缓存 保存／更新*/
    localSetStorageSync(data) {
        wx.setStorageSync(this._storageKeyName, data);
    }

    /**
     * 加入购物车
     * @param {*} item 商品
     * @param {*} counts 数量
     */
    _localAdd(item, counts) {
        let cartData = this.getCartDataFromLocal();
        if (!cartData) {
            cartData = []
        }
        let isproduct = this._checkProduct(item._id, cartData)
        //新商品
        if (isproduct.index == -1) {
            item.counts = counts
            item.selectStatus = true  //默认在购物车中为选中状态
            cartData.push(item)
        }
        //已有商品
        else {
            cartData[isproduct.index].counts += counts
        }
        this.localSetStorageSync(cartData)  //更新本地缓存
        return cartData;

    }

    getCartTotalCounts() {
        let data = this.getCartDataFromLocal()
        let counts = 0
        for (let i = 0; i < data.length; i++) {
            if (data[i].selectStatus) {
                counts++
            }
        }
        return counts
    };




    /*
    * 修改商品数目
    * params:
    * id - {int} 商品id
    * counts -{int} 数目
    * */
    _changeCounts(id, counts) {
        let cartData = this.getCartDataFromLocal()
        let hasInfo = this._checkProduct(id, cartData)
        if (hasInfo.index != -1) {
            if (hasInfo.data.counts >= 1) {
                cartData[hasInfo.index].counts += counts
            }
        }
        this.localSetStorageSync(cartData);  //更新本地缓存
    };

    /*
    * 获取购物车
    * param
    * flag - {boolean} 是否过滤掉不下单的商品
    */
    getCartDataFromLocal(flag) {
        let res = wx.getStorageSync(this._storageKeyName);
        if (!res) {
            res = []
        }
        //在下单的时候过滤不下单的商品，
        if (flag) {
            let newRes = [];
            for (let i = 0; i < res.length; i++) {
                if (res[i].selectStatus) {
                    newRes.push(res[i])
                }
            }
            res = newRes;
        }

        return res;
    };

    /*购物车中是否已经存在该商品*/
    _checkProduct(id, arr) {
        let item, result = { index: -1 };
        for (let i = 0; i < arr.length; i++) {
            item = arr[i];
            if (item._id == id) {
                result = {
                    index: i,
                    data: item
                }
                break;
            }
        }
        return result;
    }

    /*
   * 删除某些商品
   */
    _delete(ids) {
        if (!(ids instanceof Array)) {
            ids = [ids];
        }
        let cartData = this.getCartDataFromLocal()
        for (let i = 0; i < ids.length; i++) {
            let hasInfo = this._checkProduct(ids[i], cartData)
            if (hasInfo.index != -1) {
                cartData.splice(hasInfo.index, 1)  //删除数组某一项
            }
        }
        this.localSetStorageSync(cartData)
    }

}
export { CartModel }
```
> 在这里采用本地缓存购物车，在设计的时候我采用跟之前的 model 相同的实现逻辑，只是在数据的处理没有请求后台，计算我们临时需要把购物车数据放在后台，我们也只是小动，只需要把 callBack 的回调换成 this.requset 修改一些数据的变化。

## 2 前台数据处理 

回到我们 `pages/cart/cart.js`

```
// pages/cart/cart.js
import { CartModel } from '../../models/CartModel.js'
let cartmodel = new CartModel()
Page({

  /**
   * 页面的初始数据
   */
  data: {
    cartData: [],
    selectedCounts: 0, //总的商品数
    selectedCheckCounts: 0, //总的商品数  
    account: 0
  },

  /**
   * 生命周期函数--监听页面显示
   */
  onShow: function () {
    this._init()
  },
  // 初始化数据
  _init: function () {
    cartmodel.getCartData(res => {
      this.setData({
        cartData: res,
        account: this._totalAccountAndCounts(res).account,
        selectedCheckCounts: this._totalAccountAndCounts(res).selectedCheckCounts
      })

    })
  },
  // 更新商品数量
  changeCounts: function (event) {
    let id = cartmodel.getDataSet(event, 'id')
    let type = cartmodel.getDataSet(event, 'type')
    let index = this._getProductIndexById(id)
    counts = 1
    if (type == 'add') {
      cartmodel.addCounts(id, res => { })
    } else {
      counts = -1
      cartmodel.cutCounts(id, res => { })
    }
    //更新商品页面
    this.data.cartData[index].counts += counts
    this._resetCartData()
  },
  /*更新购物车商品数据*/
  _resetCartData: function () {
    let newData = this._totalAccountAndCounts(this.data.cartData) /*重新计算总金额和商品总数*/
    this.setData({
      account: newData.account,
      selectedCounts: newData.selectedCounts,
      selectedCheckCounts: newData.selectedCheckCounts,
      cartData: this.data.cartData
    })
  },
  /*离开页面时，更新本地缓存*/
  onHide: function () {
    cartmodel.localSetStorageSync(this.data.cartData)
  },
  /*
  * 计算总金额和选择的商品总数
  * */
  _totalAccountAndCounts: function (data) {
    let len = data.length
    let account = 0
    let selectedCounts = 0
    let selectedCheckCounts = 0
    let multiple = 100
    for (let i = 0; i < len; i++) {
      //避免 0.05 + 0.01 = 0.060 000 000 000 000 005 的问题，乘以 100 *100
      if (data[i].selectStatus) {
        account += data[i].counts * multiple * Number(data[i].product_sell_price) * multiple;
        selectedCounts += data[i].counts
        selectedCheckCounts++
      }
    }
    return {
      selectedCounts: selectedCounts,
      selectedCheckCounts: selectedCheckCounts,
      account: account / (multiple * multiple)
    }
  },
  /*根据商品id得到 商品所在下标*/
  _getProductIndexById: function (id) {
    let data = this.data.cartData
    let len = data.length
    for (let i = 0; i < len; i++) {
      if (data[i]._id == id) {
        return i
      }
    }
  },
  /*删除商品*/
  delete: function (event) {
    let id = cartmodel.getDataSet(event, 'id')
    let index = this._getProductIndexById(id)
    this.data.cartData.splice(index, 1)//删除某一项商品
    this._resetCartData()
    cartmodel.delete(id, res => { })  //内存中删除该商品
  },

  /*选择商品*/
  toggleSelect: function (event) {
    let id = cartmodel.getDataSet(event, 'id')
    let status = cartmodel.getDataSet(event, 'status')
    let index = this._getProductIndexById(id)
    this.data.cartData[index].selectStatus = !status
    this._resetCartData()
  },
  /*全选*/
  checkall: function (event) {
    let status = cartmodel.getDataSet(event, 'status') == 'true'
    let data = this.data.cartData
    for (let i = 0; i < data.length; i++) {
      data[i].selectStatus = !status
    }
    this._resetCartData()
  },
  /*提交订单*/
  confirm: function () {
    wx.navigateTo({
      url: '/pages/order/order?account=' + this.data.account + '&from=cart'
    })
  },
  // 订单详情
  productDetail: function (event) {
    let product_id = cartmodel.getDataSet(event, "id")
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + product_id,
    })
  }

})


```
> - _resetCartData ： 每次对于购物车的操作，我们都需要重计算商品的数量价格
> - _totalAccountAndCounts  ： 计算总金额和选择的商品总数 ，对于金钱的计算一定要精确 否则就会造成不必要的损失
> - _getProductIndexById  ： 为了方便处理，我们根据商品的id，拿到数组的下表就能直接修改状态等
> - 购物车的逻辑相对 比较复杂 业务的需求不一样 可能很多是是实现也是不一样的 大致的思路差不多，这里大家把代码跑起来 


`pages/cart/cart.wxml`
```
<!--pages/cart/cart.wxml-->
<view class='container'>
  <view class='cart-container' wx:for="{{cartData}}" wx:key="index">
    <!-- 商品左边 -->
    <view class='cart-left-container' bindtap="toggleSelect" data-id="{{item._id}}" data-status="{{item.selectStatus}}" >
        <view class="cart-select {{item.selectStatus?'selectActive':''}}"   >
            <icon  class='iconfont iconiconfontcheck'></icon>
        </view>   
    </view>
     <!-- 商品图片 -->
    <view class='cart-middle-container' data-id="{{item._id}}" bind:tap="productDetail">
        <image src="{{item.product_img}}"></image>
    </view>
    <view class='cart-right-container'>
        <view class='product-basic'> 
          <view class='product-title'>
            <text>{{item.product_name}}</text>
          </view>
          <view class='product-price'>
            <text>￥{{item.product_sell_price}}</text>
          </view>
        </view>  
        <view class='edit-contianer'>
           <view class='edit-num'>
              <icon class="iconfont {{item.counts==1?'disabled':''}} iconjian  "  bind:tap="{{item.counts==1?'':'changeCounts'}}" data-id="{{item._id}}" data-type="cut"></icon>
              <view class='num'> <text>{{item.counts}}</text></view>
              <icon class='iconfont allow iconjia1' bindtap="changeCounts" data-id="{{item._id}}" data-type="add"></icon>
           </view>
           <view class='delete' data-id="{{item._id}}" bindtap="delete">
            <icon class='iconfont iconshanchu'></icon>
           </view>
        </view>
    </view>
  </view>

 <!-- 全选  wx:if="{{checkall?'selectActive':''}}" -->
    <view class='bottom-container'> 
      <view class='all-select'>
        <view class="all-select-icon {{selectedCheckCounts==cartData.length?'selectActive':''}}"  bind:tap='checkall'   data-status="{{selectedCheckCounts==cartData.length?'true':'false'}}" >
          <icon  class='iconfont iconiconfontcheck'></icon>
        </view>
        <view class='all-select-text'>
            <text >全选</text>
        </view>
      </view>
      <view class='total-container' >
        <view class='total-price'>
          <text>合计:</text>        
          <text class='price-symbol' >￥</text>
          <text class='price' >{{account}}</text>
        </view>
        <view class='accounts' bind:tap="confirm">
          <text>结算</text>
        </view>      
      </view>
    </view>
</view>
```
> 购物车这块相比其他的功能要复杂的多，所以大家在写代码的时候需要每一个方法是怎么实现的 ，一个一个方法去理解，不要一看就这么复杂，然后不知所措，在每一个 js 中带有`_`下划线的开头的都是在当前类和 js 中调用，因为博客不必视频所以很多东西没有办法一点一点的贴出来。更多的时候需要一点点的思考和理解。

## 3 代码解析

### 3.1 model 层解析

在这里我们主要有获取购物车数据、添加到购物车、增加商品数量、减少数量、删除商品功能，首先在 model 编写他们的方法。

```
   /**
     * 获取购物车数据
     * @param {*} callBack 
     */
    getCartData(callBack) {
       
    }

    /**
     * 添加到购物车
     * @param {*} item 
     * @param {*} counts 
     * @param {*} callBack 
     */
    add(item, counts, callBack) {
        
    }

    /**
     * 增加商品数量
     * @param {*} id 
     * @param {*} counts 
     * @param {*} callBack 
     */
    addCounts(id, callBack) {
      
    }
    /**
     * 减少数量
     * @param {*} id 
     * @param {*} counts 
     * @param {*} callBack 
     */
    cutCounts(id, callBack) {
        
    }

    /**
     * 删除商品
     * @param {*} ids 
     * @param {*} callBack 
     */
    delete(ids, callBack) {
       
    }
```

#### 3.1.1 获取购物车数据

为了接口后面的更好的使用，这里我任然采用获取后台数据的形式，采用 callBack 将处理的数据返回调用者。 将本地购物车数据的抽离 `getCartDataFromLocal()` 方法 ，本地的数据使用 flag 标识是否筛选。 

```
    /**
     * 获取购物车数据
     * @param {*} callBack 
     */
    getCartData(callBack) {
        callBack(this.getCartDataFromLocal())
    }
    
   /*
    * 获取购物车
    * param
    * flag - {boolean} 是否过滤掉不下单的商品
    */
    getCartDataFromLocal(flag) {
        // 获取本地数据购物车
        let res = wx.getStorageSync(this._storageKeyName);
        if (!res) {
            res = []
        }
        //在下单的时候过滤不下单的商品，
        if (flag) {
            let newRes = [];
            for (let i = 0; i < res.length; i++) {
                if (res[i].selectStatus) {
                    newRes.push(res[i])
                }
            }
            res = newRes;
        }

        return res;
    };  
```

#### 3.1.2 添加到购物车 

添加商品需要传入商品数据和商品数量，添加购物车的时候需要获取本地数据，判断本地是否有数据，如果为空则需要一个空数组否则直接添加会报错。检查当前商品是否最新，如果是最新的商品设置数量和选择的状态（默认新商品选中），否则就直接修改就商品的数量，更新本地缓存，返回数据。

```
    /**
     * 添加到购物车
     * @param {*} item 
     * @param {*} counts 
     * @param {*} callBack 
     */
    add(item, counts, callBack) {
        callBack(this._localAdd(item, counts))
    }
    
    /**
     * 加入购物车
     * @param {*} item 商品
     * @param {*} counts 数量
     */
    _localAdd(item, counts) {
        // 获取本地数据
        let cartData = this.getCartDataFromLocal()
        // 判断是否有数据
        if (!cartData) {
            cartData = []
        }
        // 判断是新商品
        let isproduct = this._checkProduct(item._id, cartData)
        //新商品
        if (isproduct.index == -1) {
        	// 商品数量
            item.counts = counts
            //默认在购物车中为选中状态
            item.selectStatus = true
            // 添加到数组
            cartData.push(item)
        }
        //已有商品 新旧数量相加
        else {
            cartData[isproduct.index].counts += counts
        }
        //更新本地缓存
        this.localSetStorageSync(cartData)  
        return cartData;

    }
    
    
      /*购物车中是否已经存在该商品*/
    _checkProduct(id, arr) {
        let item, result = { index: -1 };
        for (let i = 0; i < arr.length; i++) {
            item = arr[i];
            if (item._id == id) {
                result = {
                    index: i,
                    data: item
                }
                break;
            }
        }
        return result;
    }
    
```

#### 3.1.3 修改商品数量 

增加商品数量和减少商品数量，每次点击的数量只能增加和减少 1 ，通过 `_changeCounts` 方法判断商品是否存在，修改数量，进行本地缓存更新。

```
     /**
     * 增加商品数量
     * @param {*} id 
     * @param {*} callBack 
     */
    addCounts(id, callBack) {
        this._changeCounts(id, 1)
        callBack()
    }
    /**
     * 减少商品数量
     * @param {*} id 
     * @param {*} callBack 
     */
    cutCounts(id, callBack) {
        this._changeCounts(id, -1)
        callBack()
    }
	
    
   /*
    * 修改商品数目
    * params:
    * id - {int} 商品id
    * counts - {int} 数目
    * */
    _changeCounts(id, counts) {
    	// 获取商=购物车数据
        let cartData = this.getCartDataFromLocal()
        // 检查商品是是否存在
        let hasInfo = this._checkProduct(id, cartData)
        // hasInfo.index 初始值为 -1  
        if (hasInfo.index != -1) {
            if (hasInfo.data.counts >= 1) {
                cartData[hasInfo.index].counts += counts
            }
        }
         //更新本地缓存
        this.localSetStorageSync(cartData); 
    };
```

#### 3.1.4 商品删除

商品的删除主要通过商品的 id ，这里采用数组的形式这样一个或者多个都能直接删除。

```
     /**
     * 删除商品
     * @param {*} ids 
     * @param {*} callBack 
     */
    delete(ids, callBack) {
        callBack(this._delete(ids))
    }
    
   /*
   * 删除某些商品
   */
    _delete(ids) {
        if (!(ids instanceof Array)) {
            ids = [ids];
        }
        let cartData = this.getCartDataFromLocal()
        for (let i = 0; i < ids.length; i++) {
            let hasInfo = this._checkProduct(ids[i], cartData)
            if (hasInfo.index != -1) {
                cartData.splice(hasInfo.index, 1)  //删除数组某一项
            }
        }
        this.localSetStorageSync(cartData)
    }
```

### 3.2 cart.js 数据分析

#### 3.2.1 页面初始化

页面初始化数据购车的显示的数据，选中的金额，选中的个数。通过 `cartmodel` 获取数据，金额的计算和商品的数量 `_totalAccountAndCounts` 方法，对于价格我们在实际的处理过程中需要特别注意，因为精度的丢失会影响整个价格的不一致。

```
// cart.js
// 获取购物车数据
cartmodel.getCartData(res => {
  this.setData({
    // 订单数据
    cartData: res,
    // 金额的计算
    account: this._totalAccountAndCounts(res).account,
    // 选中的商品数
    selectedCheckCounts: this._totalAccountAndCounts(res).selectedCheckCounts
  })
})

/*
* 计算总金额和选择的商品总数
* */
_totalAccountAndCounts: function (data) {
  let len = data.length
  let account = 0
  let selectedCounts = 0
  let selectedCheckCounts = 0
  let multiple = 100
  for (let i = 0; i < len; i++) {
    //避免 0.05 + 0.01 = 0.060 000 000 000 000 005 的问题，乘以 100 *100
    if (data[i].selectStatus) {
      account += data[i].counts * multiple * Number(data[i].product_sell_price) * multiple;
      selectedCounts += data[i].counts
      selectedCheckCounts++
    }
  }
  return {
    selectedCounts: selectedCounts,
    selectedCheckCounts: selectedCheckCounts,
    account: account / (multiple * multiple)
  }
},
```

#### 3.2.2 商品数量的修改

商品数量的修改分为二步，第一步直接修改 data 的数据，第二步修改缓存的数据，每次修改数据之后则需要重新计算。每次 `wxml` 需要传入商品的 id 和 商品操作 type ,通过 `_getProductIndexById` 方法获取当前商品在数组的中的下标，直接修改数组的数量，也需要重置本地缓存保证数据的一致性，修改数据完成通过 `_resetCartData` 方法重新计算。

```
  // 更新商品数量
  changeCounts: function (event) {
    // 获取修改数量的 id
    let id = cartmodel.getDataSet(event, 'id')
    // type ： 增加  减少
    let type = cartmodel.getDataSet(event, 'type')
    // 获取商品在数组的下标
    let index = this._getProductIndexById(id)
    // 默认为 1 
    let counts = 1
    if (type == 'add') {
      cartmodel.addCounts(id, res => { })
    } else {
      counts = -1
      cartmodel.cutCounts(id, res => { })
    }
    //更新商品页面
    this.data.cartData[index].counts += counts
    // 更新购物车
    this._resetCartData()
  },
  /*更新购物车商品数据*/
  _resetCartData: function () {
    let newData = this._totalAccountAndCounts(this.data.cartData) /*重新计算总金额和商品总数*/
    this.setData({
      account: newData.account,
      selectedCounts: newData.selectedCounts,
      selectedCheckCounts: newData.selectedCheckCounts,
      cartData: this.data.cartData
    })
  },
   /*根据商品id得到 商品所在下标*/
  _getProductIndexById: function (id) {
    // data 数据
    let data = this.data.cartData
    // 数组的长度
    let len = data.length
    for (let i = 0; i < len; i++) {
      if (data[i]._id == id) {
        return i
      }
    }
  },
```

#### 3.2.3 商品删除

获取商品的 id ，通过 `cartmodel.delete` 修改内存中缓存的数据，`_getProductIndexById` 方法获取 data 的购物车数据，直接从当前的数组移除，移除完成通过 `_resetCartData` 方法重新计算。

```
 /*删除商品*/
  delete: function (event) {
    let id = cartmodel.getDataSet(event, 'id')
    let index = this._getProductIndexById(id)
    this.data.cartData.splice(index, 1)//删除某一项商品
    this._resetCartData()
    cartmodel.delete(id, res => { })  //内存中删除该商品
  },
```

#### 3.2.4 选择商品 

获取商品的 id 和 商品的状态 ，通过下标直接修改本地的选择状态，每次操作完成都需要调用 `_resetCartData` 方法重新计算购物车。

```
 /*选择商品*/
  toggleSelect: function (event) {
    // 获取商品 id
    let id = cartmodel.getDataSet(event, 'id')
    // 是否选择的状态
    let status = cartmodel.getDataSet(event, 'status')
    // 获取数组下标
    let index = this._getProductIndexById(id)
    // 当前状态取反
    this.data.cartData[index].selectStatus = !status
    // 重新计算购物车
    this._resetCartData()
  },
```

#### 3.2.5 购物车全选

购物车全选首先判断是否选中，每次取反之后在重新计算购物车。

```
 /*全选*/
  checkall: function (event) {
     // 判断是否全选
    let status = cartmodel.getDataSet(event, 'status') == 'true'
    // data 数据
    let data = this.data.cartData
    for (let i = 0; i < data.length; i++) {
      data[i].selectStatus = !status
    }
    // 重新计算
    this._resetCartData()
  },
```

#### 3.2.6 提交订单

提交订单的过程我们需要将总价格，这样在订单实现的过程中则不需要重新计算。

```
 /*提交订单*/
  confirm: function () {
    wx.navigateTo({
      url: '/pages/order/order?account=' + this.data.account + '&from=cart'
    })
  },
  // 订单详情
  productDetail: function (event) {
    let product_id = cartmodel.getDataSet(event, "id")
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + product_id,
    })
  }
```

#### 3.2.6 订单详情

点击购物车的图片，获取商品的 id ，跳转商品详情。

```
  // 订单详情
  productDetail: function (event) {
    let product_id = cartmodel.getDataSet(event, "id")
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + product_id,
    })
  }
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)