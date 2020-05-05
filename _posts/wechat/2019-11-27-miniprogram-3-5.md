---
layout: post
title: 【坚果商城实战系列学习】第 3-5 课：云开发之数据库操作
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 3-5 课：云开发之数据库操作
---

# 第 3-5 课：云开发之数据库操作

## 1 数据库操作公共类

开发所需[官方文档](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-server-api/init.html),大家在实际开发中一定多看文档，我们走在时代的前沿，很多东西出错，无法百度出想样的结果，所以希望大家作时代的先锋，时代的楷模，分析自己的踩坑爬坑的艰难过程。
对于数据库常见的操作增删查改，如果每一个集合都去写一个数据库操作的代码，后期我们的版本升级和产品的不断变化，我们需要大量的时间去排查和修改。提取我们认为所有的重合的代码，重合的代码是不利于后期的维护，在代码的编写的时候重合的代码我们得及时提取出来和接下来我们未来极大可能重合的。
在 `index` 云函数新建一个文件 `models/BaseModel.js` ,相同的方法不同的业务层调用，很多返回条件不是必须传的，这里我们就默认给出一个初始值，使得代码更容易会扩展，实现代码如下:

```
// 公共BaseModel
const cloud = require('wx-server-sdk');
cloud.init({
  // 自身环境
  env: 'release-prod',
  traceUser: true,
});
const db = cloud.database();


/**
 * 查询处理 
 * @param  {object} model       集合名称
 * @param  {String} id          查询id
 * @return  {object|null}       查找结果
 */
const findById = (model, fields = {} , id ) => {
  try {
    return db.collection(model)
      .doc(id)
      .field(fields) 
      .get()
  } catch (e) {
    console.error(e)
  }
}


/**
 * 查询处理 带多条件的
 * @param  {object} model         集合名称
 * @param  {Object} [options={}]    查询条件
 * @param  {Number} [page]        开始记录数
 * @param  {Number} [size]        每页显示的记录数
 * @return  {object|null}         查找结果
 */
const query = (model, fields = {}, options = {}, page = 0, size = 10, order = { name: '_id', orderBy:'asc'} ) => {
  try {
    return db.collection(model)
    .where(options)
    .field(fields) 
    .skip(page)
    .limit(size)
    .orderBy(order.name, order.orderBy)
    .get()

  } catch (e) {
    console.error(e)
  }
}

/**
 * 新增处理
 * @param  {object} model  集合名称
 * @param  {object} params 参数
 * @return {object| null}  操作结果
 */
const add = (model, params) => {
  try {
    return db.collection(model).add({
      data: params
    });
  } catch (e) {
    console.error(e);
  }
}

/**
 * 编辑处理
 * @param  {object} model      集合名称
 * @param  {object} params     参数
 * @return {object|null}       操作结果
 */
const update = (model, params) => {
  try {
    return db.collection(model).doc(params._id)
    .update({
      data: params
    })

  } catch (e) {
    console.error(e);
  }
}

/**
 * 删除结果
 * @param  {object} model      集合名称
 * @param  {String} id         参数
 * @return {object|null}       操作结果
 */
const remove = (model, id) => {
  try {
    return  db.collection(model).doc(id).remove()
  } catch (e) {
    console.error(e)
  }
}



module.exports = {
  query,
  findById,
  add,
  update,
  remove

}

```
```
env : 默认环境配置，传入字符串形式的环境 ID 可以指定所有服务的默认环境，传入对象可以分别指定各个服务的默认环境
traceUser : 是否在将用户访问记录到用户管理中，在控制台中可见
上面的对数据库的操作封装，大家对于部分不了解的操作指令记得查看下官方文档
```
接下来验证下我们的代码是否有错
在云函数 `index` 中新建一个文件 `test/models/BaseModelsTest.js` ， `test` 大家在测试的主要每一个测试文件的关系不要直接全部新建在一个文件夹，命令的时候也需要注意下，不然后面排除需要寻找半天。
`BaseModelsTest.js`
```
const model = require('models/BaseModel.js')
// 测试BaseModel

/**
 * 编辑处理
 * @param  {object} document      集合名称
 * @param  {object} type     参数
 * @return {object|null}       操作结果
 */
const baseTest = (type = 1, field = {},doc='test') => {
  let result 
  let id = 'f896855d5cfb5f6901c18d424a19b0bd'
  switch (type) {
    case 1:
      // 1、查询所有
      result = model.query(doc,field)
      break;
    case 2:
      //2、 根据id查询
      result = model.findById(doc,id)
      break;
    case 3:
      //3、新增
      let params_add = {name:'test01',age:12,sex:0,show:true}
      result = model.add(doc, params_add)
      break;  
    case 4:
       //4、修改
      let params_update = {id:id, name:'test02', age:20,sex:1, show: true }
      result = model.update(doc, params_update)
      break; 
    case 5:
     // 5、删除
      result = model.remove(doc,id)
      break; 
  } 

  return result
}

module.exports = {
  baseTest
}
```
可能测试的有点麻烦 ，我们需要在入口文件新建一个路由本地调式下，可根据 `BaseModelsTest.js` 参数改变操作 
`index.js`

```
 /***************************    订单   *****************************************/
  app.router('tests', async (ctx, next) => {
    // test 可参数类型 是否决定传参
    ctx.data = await baseTest.test()
    ctx.body = await returnUtil.success(ctx);
    await next();
  })

```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)


