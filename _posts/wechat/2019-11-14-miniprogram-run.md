---
layout: post
title: 【坚果商城实战系列学习】第 1-3 课：官方案例先运行
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 1-3 课：官方案例先运行
---

# 第 1-3 课：官方案例先运行

## 1 前期准备

下载微信开发者工具，方便后期调试[官方下载地址]( https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html) 。

小程序云开发本地调式需要 `nodejs` 环境，大家如没有安装 `nodejs` 环境，需要安装下。网上安装的教程也比较多。

## 2 项目初始化

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/023.png)

点击微信开发者工具的`云开发`，根据提示开通云开发功能，完成后会跳转到对应的云开发控制台：

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/024.png)

云开发官方文档地址[点击链接查看](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/getting-started.html )。

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/025.png)

cloudfunctions 旁边是云开发的环境，右击 cloudfunctions 可以切换环境，并把所有的函数`创建并部署：云端安装依赖..`。

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/026.png)

每一个文件带一个云开发的标志上传成功。

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/027.png)

上传成功我们就可以操作界面的功能，在测试官方的功能注意官方的提示。

这里我们在云开发中删除 `login` 函数

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/028.png)

再次 `点击获取openid` 控制台报错，找不到 `login` 函数

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/029.png)

但我把下面的 `login` 函数直接上传并报错

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/030.png)

在我后台云开发删除了 `login` 函数，但是前台还是云开发的标志，我们需要 `cloudfunctions` 同步云函数列表

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/031.png)

云开发中可能经常会报错，函数找不到这种问题就先同步下函数在操作。

官方的功能演示跟着官方说的一步一步来，简单的先熟悉下云开发。

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)