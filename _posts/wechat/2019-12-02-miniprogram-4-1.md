---
layout: post
title: 【坚果商城实战系列学习】第 4-1 课：前后端交互之开篇
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 4-1 课：前后端交互之开篇
---

# 第 4-1 课：前后端交互之开篇

云开发已经告一段落了，现在我们需要前后端的数据对接工作了。很多时候在实际的开发中公司利用云开发可能只是部分功能，所以在这里编写的时候，我们处理的接口应该要做到通用。

## 1 全局通用配置

`client` 新建 `config.js` ，用来存放全局接口访问地址配置文件

```
const config = {
  cloud_route: 'index'
}

export { config }
```
`client`  新建 `utils/CloudRequest.js` ，后台路由请求公共工具类

```
import { config } from "../config.js"
class CloudRequest{

  constructor(){
    this.cloud_route = config.cloud_route
  }

  request(params){
    wx.cloud.callFunction({
      // 要调用的云函数名称
      name: this.cloud_route,
      // 传递给云函数的参数
      data: {
        // 要调用的路由的路径，传入准确路径或者通配符
        $url: params.url, 
        data: params.data 
      },
      success: res => {
        params.success(res)
      },
      fail: err => {
        console.log(err)
      }
    })
  }

  /*获得元素上的绑定的值*/
  getDataSet(event, key) {
    return event.currentTarget.dataset[key];
  }
  

}
export { CloudRequest }
```

> 后面的所有的 model 直接继承调用 request 就可以操作云函数了，在小程序的使用的形式很多时候就代码可以看起来更通俗易懂，constructor 初始化的就会执行，在这里我们全局之定义了一个路由，所以我放到后面配置文件，如果有几个路由这里大家 当前的类写一个判断路由的方法，每一个 model 在调用的时候传入一个类型做条件判断。在之前不知道大家怎么处理点击的返回的参数，不知道是不是每次点击都是在event 去获取，在这里我封装了一个公共的方法 ，getDataSet 方法大家只需要传入就能直接获取元素绑定的值，避免一次又一次的寻找复制粘贴。

## 2 代码分解

### 2.1 constructor 

```
constructor(){
    this.cloud_route = config.cloud_route
  }
```

*constructor*() 方法是类的默认方法,通过 new 命令生成对象实例时,自动调用该方法 。每次实例化去加载配置文件，`this.cloud_route` 将 `config ` 配置信息赋值。  

### 2.2 request 方法

```
  request(params){
    wx.cloud.callFunction({
      // 要调用的云函数名称
      name: this.cloud_route,
      // 传递给云函数的参数
      data: {
        // 要调用的路由的路径，传入准确路径或者通配符
        $url: params.url, 
        data: params.data 
      },
      success: res => {
        params.success(res)
      },
      fail: err => {
        console.log(err)
      }
    })
  }
```

与后台交互的时候我需要传入路由和参数（非必传）， `wx.cloud.callFunction` 官方提供的云函数调用方法函数，`$url` 因为我们后台采用的 `tcb-router` 所以每一个请求的函数需要参入路由在后台判断，这里的 `params.data ` 根据实际的情况进行传入。`params.success(res)`  将请求的返回数据返回调用方。其他类需要调用 `request` 继承 `CloudRequest` 通过继承使用。

- 无 data 调用

```
import { CloudRequest } from '../utils/cloud-request.js'
class TestModel extends CloudRequest {
  getTest(callBack) {
    this.request({
      url: "路由路径",
      success: res => {
        callBack(res)
      }
    })
  }
}
export { TestModel }
```

> 继承之后通过  this.request 直接使用, `CloudRequest` 中 params 对应参数如下

```
{
    url: "路由路径",
    success: res => {
    callBack(res)
    }
}
```

- 有 data 调用

```
import { CloudRequest } from '../utils/cloud-request.js'

class TestModel extends CloudRequest {
  getTest(callBack) {
    this.request({
      url: "路由路径",
      data:"{传入的数据}",
      success: res => {
        callBack(res)
      }
    })
  }
  
}

export { TestModel }
```

> data 使用json传入的形式，在后面的章节我会讲到，callBack 函数拿到数据这里不进行 在返回它的调用方。这里多次调用和返回可能有点绕，大家可以结合代码进行参看。

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)