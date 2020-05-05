---
layout: post
title: 【坚果商城实战系列学习】第 2-1 课：商城项目搭建
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 2-1 课：商城项目搭建
---

# 第 2-1 课：商城项目搭建

> 在搭建项目前，根据自己需要下载本系列文章的源代码
>
> 本项目地址：<https://github.com/gurenla/nux-shop>  

## 1 准备工作

已经申请小程序，获取小程序 AppID 在[小程序管理后台](https://mp.weixin.qq.com/)中， 设置的开发设置 下可以获取微信小程序 AppID 。

## 2 新建项目

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/032.png)

这里我们已经不需要官方的模板，将其官方给的图片和模板删除。

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/033.png)

项目重命名，文件夹和 `project.config.json` 对应即可

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/034.png)

## 3 `app.json` 配置文件修改 

修改 `app.json` 全局的默认窗口配置

```
"window": {
    "backgroundColor": "#F6F6F6",
    "backgroundTextStyle": "light",
    "navigationBarBackgroundColor": "#F6F6F6",
    "navigationBarTitleText": "坚果商城",
    "navigationBarTextStyle": "black"
  }
```

自定义 `tabBar` ，坚果商城目前主要的有首页、分类、购物车、个人中心。在配置 `tabBar` 首先我们需要新建我们每一个要指向 `tabBar`  的页面。在 `pages` 写好页面路径列表 ，保存微信开发者工具自动帮我们生成

```
  "pages": [
    "pages/index/index",
    "pages/category/category",
    "pages/cart/cart",
    "pages/my/my"
  ],
```

`tabBar ` 所需要的图片存放 `pages/images/tabBar` 文件中。

```
"window": {
   ....
},
"tabBar": {
    "color": "#7F8389",
    "selectedColor": "#FF6200",
    "backgroundColor": "#fff",
    "borderStyle": "black",
    "list": [
      {
        "pagePath": "pages/index/index",
        "iconPath": "images/tabBar/home.png",
        "selectedIconPath": "images/tabBar/home@select.png",
        "text": "首页"
      },
      {
        "pagePath": "pages/category/category",
        "iconPath": "images/tabBar/category.png",
        "selectedIconPath": "images/tabBar/category@select.png",
        "text": "分类"
      },
      {
        "pagePath": "pages/cart/cart",
        "iconPath": "images/tabBar/cart.png",
        "selectedIconPath": "images/tabBar/cart@select.png",
        "text": "购物车"
      },
      {
        "pagePath": "pages/my/my",
        "iconPath": "images/tabBar/my.png",
        "selectedIconPath": "images/tabBar/my@select.png",
        "text": "个人中心"
      }
    ]
  }
```

## 4 全局样式修改 

`app.wxss` 全局样式

```
/**app.wxss**/
page{
  width:100%;
  padding: 0;
  margin: 0;
  height: 100%;
  font-family:  "PingFang SC", -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  background-color: #f2f2f2;
}
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  box-sizing: border-box;
} 
```

## 5 运行效果

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/035.png)

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)