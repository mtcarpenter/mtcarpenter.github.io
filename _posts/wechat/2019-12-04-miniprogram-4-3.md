---
layout: post
title: 【坚果商城实战系列学习】第 4-3 课：前后端交互之分类实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 4-3 课：前后端交互之分类实现
---

# 第 4-3 课：前后端交互之分类实现

## 1 逻辑处理

`client` 新建 `models/CategoryModel.js`
```
import { CloudRequest } from '../utils/cloud-request.js'
class CategoryModel extends CloudRequest {
    /**
     * 获取分类
     * @param {*} callBack 
     */
    getCateGory(callBack){
        this.request({
            url: "getCategoryMenu",
            success: res => {
              callBack(res)
            }
        })
    }
    
    /**
     * 根据商品类型获取商品
     * @param {*} category_type 
     * @param {*} callBack 
     */
    getCateGoryProduct(category_type,callBack){
        this.request({
            url: "getCategoryProduct",
            data:category_type,
            success: res => {
              callBack(res)
            }
        })
    }

}
export { CategoryModel }
```

##  2 前台数据处理

回到我们 `pages/category/category.js`
```
// pages/category/category.js
import { CategoryModel } from '../../models/CategoryModel.js'
let category = new CategoryModel()
Page({

  /**
   * 页面的初始数据
   */
  data: {
    menuCategories: [
                    { category_name:'坚果炒货',category_type:1},
                    { category_name: '休闲零食', category_type: 2 },
                    { category_name: '饼干蛋糕', category_type: 3 },
                    { category_name: '蜜饯果干', category_type: 4 },
                    { category_name: '肉干肉脯', category_type: 5 },
                    ],
    menuSelect:1,
    menuNmae:'',
    products:[]
  },

   /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    this._init()
  },
  // 初始化数据
  _init:function(){
    category.getCateGory(res=>{
      let menuCategories = res.result.data.data
      let menuSelect =  menuCategories[0].category_type
      let menuNmae = menuCategories[0].category_name
      this.setData({
        menuCategories,
        menuSelect,
        menuNmae       
      })
      this._getCateGory(menuSelect)
    })
   
  },
  // 菜单切换
  menu:function(e){
    let index = category.getDataSet(event, "index")
    let menuCategories = this.data.menuCategories
    let menuSelect = menuCategories[index].category_type
    let menuNmae = menuCategories[index].category_name
    this._getCateGory(menuSelect)
    this.setData({
      menuSelect,
      menuNmae
    })
  },
  // 跳转商品详情
  productDetails:function(e){
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + e.detail.productId,
    })
  },
  _getCateGory:function(type){
    category.getCateGoryProduct(type, data => {
      this.setData({
        products: data.result.data.data
      })
    })
  }
})
```
> menuCategories : 这个一般不是经常改变的默认给出初始值，方便提前显示给前台页面

`pages/category/category.wxml`
```
<!-- pages/category/category.wxml -->
<view class='container'>
  <!-- 分类左边选择区域 -->
  <scroll-view class='left-container' scroll-y="true">
    <block wx:for="{{menuCategories}}" wx:key="key">
      <view class="categoryBar {{ menuSelect==item.category_type?'active':''}}" data-id='{{item.category_type}}' data-index='{{index}}' bind:tap="menu">
        <text>{{item.category_name}}</text>
      </view>
    </block>
  </scroll-view>
  <!-- 分类右边选择区域 -->
  <scroll-view class='right-container' scroll-y="true">
    <!-- 主题宣传图 -->
    <view class='introduce-image'>
      <image src='../../images/temp/category.png'></image>
    </view>
    <view class='category-name'>
      <text>{{menuNmae}}</text>
    </view>
    <view class='product-container'>
      <block wx:for="{{products}}" wx:key="key">
        <category-comp product="{{item}}" bind:productDetails="productDetails"></category-comp>
      </block>
    </view>
  </scroll-view>
</view>
```
> wxml ：当前页面和组件通信，见前后端交互之首页实现

` components/behaviors/product-behavior.js`
> 商品信息 Behavior 公用，跟上一个章节一样，这里就不在贴出代码

`components/category/index.wxml`
```
<!--components/category/index.wxml-->
<view class='container' >
  <view class='product-img' bind:tap='productDetails'>
    <image src='{{product.product_img}}'></image>
  </view>
  <view class='product-name'>
    <text>{{product.product_name}}</text>
  </view>
</view>
```
> 样式在之前静态页面已经完成

## 3 代码分解

整个章节从初始数据、页面加载数据、菜单切换、商品详情跳转的流程为大家讲解。

### 3.1 初始化数据

分类的菜单和主题的菜单一开始我都是给出默认值，在获取数据库的数据，每一个分类打击了之后需要改变状态和名称 ，这里的 `menuSelect`  和 `menuNmae` 则处理我们菜单交互的过程，`menuSelect` 的值默认为第一个分类>

```
  menuCategories: [
                    { category_name:'坚果炒货',category_type:1},
                    { category_name: '休闲零食', category_type: 2 },
                    { category_name: '饼干蛋糕', category_type: 3 },
                    { category_name: '蜜饯果干', category_type: 4 },
                    { category_name: '肉干肉脯', category_type: 5 },
                    ],
    menuSelect:1, // 页面加载默认第一个选中
    menuNmae:'',
    products:[]
```

### 3.2 页面加载数据

` category.getCateGory()` 获取初始化数据，将左边的分类、选择的分类、和分类名重新赋值，初始化数据为下面我们注释的 5 步。 

```
 category.getCateGory(res=>{
      let menuCategories = res.result.data.data
      let menuSelect =  menuCategories[0].category_type
      let menuNmae = menuCategories[0].category_name
      this.setData({
        menuCategories,
        menuSelect,
        menuNmae       
      })
      this._getCateGory(menuSelect)
    })
```

### 3.3 菜单切换

菜单切换是根据页面上的绑定的值判断

```
 // 菜单切换
  menu:function(e){
    // 取出 wxml 菜单绑定的 index 值
    let index = category.getDataSet(event, "index")
    // 赋值页面分类的 data 值
    let menuCategories = this.data.menuCategories
    // 取出当前选中的分类类型
    let menuSelect = menuCategories[index].category_type
    // 取出当前分类的名称
    let menuNmae = menuCategories[index].category_name
    // 获取选中分类的数据
    this._getCateGory(menuSelect)
    // 改变 data 的值
    this.setData({
      menuSelect,
      menuNmae
    })
  },
```

### 3.4 跳转商品详情

在这里跳转商品详情 跟之前的基本类似 productDetails 获取组件的传值

```
 // 跳转商品详情
  productDetails:function(e){
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + e.detail.productId,
    })
    
  },
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)

