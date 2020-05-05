---
layout: post
title: 【坚果商城实战系列学习】第 5-1 课：课程扩充  
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 5-1 课：课程扩充  
---

# 第 5-1 课：课程扩充  

## 1 定时触发器

如果云函数需要定时 / 定期执行，也就是定时触发，我们可以使用云函数定时触发器。配置了定时触发器的云函数，会在相应时间点被自动触发，函数的返回结果不会返回给调用方，详情进入[官方网址](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/functions/triggers.html )，比如：两小时后取消订单、定点定时推送商品信息等。

右击 cloud 选择 `新建 Node.js 云函数`  命名为 triggers

![1563451498460](https://nux.oss-cn-beijing.aliyuncs.com/doc/059.png)

云函数创建触发器，必须建一个 `config.json` 文件，因为只有 `config.json` 才会有上传触发器选项。

- 一般 json 文件

![1563456965912](https://nux.oss-cn-beijing.aliyuncs.com/doc/060.png)

- config.json 

![1563457028572](https://nux.oss-cn-beijing.aliyuncs.com/doc/061.png)

> config.json 相比一般的 json 多出上传触发器和删除触发器 触发器也只有通过这样上传和删除后台才会执行。

config.json 解读

```
{
  // triggers 字段是触发器数组，目前仅支持一个触发器，即数组只能填写一个，不可添加多个
  "triggers": [
    {
      // name: 触发器的名字，可以随意更改，规则见给出的官方链接
      "name": "myTrigger",
      // type: 触发器类型，目前仅支持 timer (即 定时触发器)
      "type": "timer",
      // config: 触发器配置，在定时触发器下，config 格式为 cron 表达式，规则见官方链接
      "config": "0 0 2 1 * * *"
    }
  ]
}
```

**示例**

下面展示了一些 Cron 表达式和相关含义的示例：

- `*/5 * * * * * *` 表示每5秒触发一次
- `0 0 2 1 * * *` 表示在每月的1日的凌晨2点触发
- `0 15 10 * * MON-FRI *` 表示在周一到周五每天上午10:15触发
- `0 0 10,14,16 * * * *` 表示在每天上午10点，下午2点，4点触发
- `0 */30 9-17 * * * *` 表示在每天上午9点到下午5点内每半小时触发
- `0 0 12 * * WED *` 表示在每个星期三中午12点触发

接下来我们编写每十秒的触发器，进入 config.json

```
{
  "triggers": [
    {
      "name": "trigger",
      "type": "timer",
      "config": "*/10 * * * * * *"
    }
  ]
}
```

> 注意：在创建触发器去掉触发器的注释 有注释上传在解析 json 会报错

编写 index.json

```
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init()

// 云函数入口函数
exports.main = async (event, context) => {
  const wxContext = cloud.getWXContext()
  console.log("每十秒触发器执行一次测试。。。")
  
}
```

> 目前作为测试所以我们输出一句话测试下是否成功   因为云函数不会返回 所以 return 没有必要返回数据 。

编写云函数 triggers 下的 index.json 和 config.json ，点击 triggers 上传并部署:云端安装依赖，上传成功后，代有击 config.json 上传触发器，成功后我就在云开发后台去看下，选择`云函数`函数名为 triggers  我们在日志控制面看可以看出输出时间是不是我们的触发时间，云函数的输出内容，如下图：

![1563457954585](https://nux.oss-cn-beijing.aliyuncs.com/doc/062.png) 

## 2 云函数间的调用 

云函数之间不能直接通信需要相互调用传递数据，服务端云函数跟我前台调用云函数形式一样。这是服务端没有 `wx.` 前台调用则需要加上，后台云函数如下：

```
// 云函数入口文件
const cloud = require('wx-server-sdk')
cloud.init()

// 云函数入口函数
exports.main = async (event, context) => {
  const res = await cloud.callFunction({
    // 如果是 tcb-router 则需要 
    $url:'url',
     // 要调用的云函数名称
    name: 'add',
    // 传递给云函数的参数
    data: {
    }
  })
  return res
}
```

在云函数 `index/index.js` 新增一个云函数调用的路由，如下：

```
  app.router('callFunc', async (ctx, next) => {
    // test 可参数类型 是否决定传参
    ctx.data = "云函数之间的调用"
    ctx.body = await returnUtil.success(ctx);
    await next();
  })
```

接下来我们创建一个测试的云函数 `cloud/callFunctionTest`  ，编写 index.js 调用云函数的 index 路由为 callFunc 的路由地址，代码如下：

```
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init()

// 云函数入口函数
exports.main = async (event, context) => {
  const res = await cloud.callFunction({
    $url:'callFunc',
    name: 'index',
    data: {
    }
  })

 console.log(res)

  return res
}
```

打开本地测试

![1563605643154](https://nux.oss-cn-beijing.aliyuncs.com/doc/063.png)

### 3 支付

在这里我并没有涉及到支付问题，支付取消订单和模板消息发送通知用户，一个真正的商业系统可能涉及到太多的东西，如果大家在实践中需要运用到支付问题，我这里也为大家推荐一篇腾讯出的支付的文章和小程序模板消息服务

- 小程序支付参考地址：https://cloud.tencent.com/developer/article/1457922
- 小程序模板消息服务： https://github.com/TencentCloudBase/tcb-demo-message 

## 3 总结

到这里了我们的课程暂时完成了，这个课程比计划晚了半年多，云开发已经有大半年了，在前段时间写代码的时候发现微信开发者工具有的时候还是会出问题，有时候需要重启和反复尝试才能成功。个人能力的提升很多就是解决问题的能力，解决的问题越多提升能力相对也是比较快的。我在课程的时候因为版本的问题也才了一个比较大的坑，涉及到操作数据库本地调式就报错，这个问题困扰了几天然后坚持更新完课程，然后慢慢的排查，结果我升级了版本问题就被成功解决，如下图：

![1563605643154](https://nux.oss-cn-beijing.aliyuncs.com/doc/064.png)

最后，我感谢购买课程的每一个人，希望大家在后面的开发中，学会代码的优化和提高代码的质量。

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)