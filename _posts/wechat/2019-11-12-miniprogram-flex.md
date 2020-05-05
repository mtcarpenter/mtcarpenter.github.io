---
layout: post
title: 【坚果商城实战系列学习】第 1-1 课：微信小程序实操弹性盒子
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 1-1 课：微信小程序实操弹性盒子
---

# 第 1-1 课：微信小程序实操弹性盒子

## 1 弹性盒子

网页布局（ layout ）是  CSS  的一个重点应用。 布局的传统解决方案，基于[盒状模型](https://developer.mozilla.org/en-US/docs/Web/CSS/box_model)，依赖  [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display)  属性 + [`position`](https://developer.mozilla.org/en-US/docs/Web/CSS/position)属性 +  [`float`](https://developer.mozilla.org/en-US/docs/Web/CSS/float) 属性。它对于那些特殊布局非常不方便，比如，[垂直居中](https://css-tricks.com/centering-css-complete-guide/)就不容易实现。 

2009年，W3C 提出了一种新的方案---- Flex 布局，可以简便、完整、响应式地实现各种页面布局。目前，它已经得到了所有浏览器的支持，这意味着，现在就能很安全地使用这项功能。

前两段摘自阮一峰的 flex 布局教程，关于 flex 布局相关知识，大家可以参考阮一峰的  [flex  布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)和 [flex 布局教程：实例篇](http://www.ruanyifeng.com/blog/2015/07/flex-examples.html)。在此篇中不能为大家一一展示弹性盒子的每一个功能，所以大家在给的实例中，练习之余还是多多看上一句我给的两个 flex 阮一峰的链接，flex 在实际开发中能大大提高我们的开发效率，在开篇就介绍 flex ，也是因为微信小程序对弹性盒子支持非常好的，笔者在日常开发中也会经常使用 flex 来提高开发效率。

  block 、inline 与 inline-block 

- 块级元素（ block )，即一个元素独占一行，小程序里的view组件类似于html里的 div 标签，是块级元素；
- 行内元素（ inline ）在一行内依次排列，但是不能设置高宽；
- 行内块元素（ inline-block ）具有行内元素特性，同时可以设置高宽。 任何一个容器都可以指定为 Flex 布局 display :  flex ; 行内元素也可以使用 Flex 布局 。

> display : inline-flex;  注意 设为 Flex 布局以后，子元素的 float 、clear 和 vertical-align 属性将失效。

在官方的给的文档中只是简单的介绍了弹性盒子（[官方地址](https://developers.weixin.qq.com/ebook?action=get_post_info&docid=00080e799303986b0086e605f5680a)），接下来我将同样使用四个小方块给大家做演示，对于开发者来说谷歌浏览器是日常必备，这里就采用谷歌浏览器的配色给大家做展示，发车发车。

例如:

index.xml : 

```html
<view class="container">
  <view class='flex-item bg1' />
  <view class='flex-item bg2' />
  <view class='flex-item bg3' />
</view>
```

index.wxss :

```css
.flex-item{
	width:100px;
	height:100px;
}
.bg1{
	background-color:#ff3f34;
}
.bg2{
	background-color:#fbbf00;
}
.bg3{
	background-color:#44ac45;
}
```

 运行效果：

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/005.png)

view 相等于 div，此段代码每一个 view 块级元素，每一个 view 独占一行，这里我们使用 display 来改变块级元素与行内元素相互转换。

任何一个容器都可以指定为 Flex 布局。 

```
<view class="container">
  <view class='flex-item bg1' />
  <view class='flex-item bg2' />
  <view class='flex-item bg3' />
</view>
```

```css
.container{
  display: flex;
}
.flex-item{
	width:100px;
	height:100px;
}
.bg1{
	background-color:#ff3f34;
}
.bg2{
	background-color:#fbbf00;
}
.bg3{
	background-color:#44ac45;
}

```

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/006.png)

现在我们将 flex-item 块级元素设置成行内块元素。

```css
.flex-item{
     display: inline-block; /**使得变为行业元素**/	
	width:100px;
	height:100px;
}
```

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/006.png)

## 2 容器的属性

常用为以下6个属性设置在容器上。 

```
flex-direction
flex-wrap
flex-flow
justify-content
align-items
align-content
```

### 2.1 flex-direction 属性

`flex-direction` 属性决定主轴的方向（即项目的排列方向）。 

```
.container {
  flex-direction: row | row-reverse | column | column-reverse;
}

```

> row（默认值）：主轴为水平方向，起点在左端。
> row-reverse：主轴为水平方向，起点在右端。
> column：主轴为垂直方向，起点在上沿。
> column-reverse：主轴为垂直方向，起点在下沿。

在这里我就抽出 row 和 column 这两个常用的为大家讲解。

#### 2.1.2 row 

```
<!--pages/test/test.wxml-->
<view class="container">
  <view class='flex-item bg1' ><text>1</text></view>
  <view class='flex-item bg2' ><text>2</text></view>
  <view class='flex-item bg3' ><text>3</text></view>
  <view class='flex-item bg4' ><text>4</text></view>
</view>
```

```
/* pages/test/test.wxss */
.container {
  width: 700rpx;
  margin-left: 25rpx;
  display: flex;
  flex-direction: row;
}

.flex-item {
  width: 130rpx;
  height: 130rpx;
  border-radius: 10rpx 10rpx;
}

.bg1 {
  background-color: #ff3f34;
}

.bg2 {
  background-color: #fbbf00;
}

.bg3 {
  background-color: #44ac45;
}

.bg4 {
  background-color: #2f84d5;
}

```

![1563194198328](https://nux.oss-cn-beijing.aliyuncs.com/doc/051.png)

#### 2.1.1 column

```
.container {
  width: 700rpx;
  margin-left: 25rpx;
  display: flex;
  flex-direction: column;
}
```

![1563194955644](https://nux.oss-cn-beijing.aliyuncs.com/doc/052.png)

### 2.2 flex-wrap 属性

设置是否允许项目多行排列，以及多行排列时换行的方向。  

```
.container{
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

> nowrap（默认）：不换行。
> wrap ：换行，第一行在上方。
> wrap-reverse ：换行，第一行在下方。

wrap 日常使用频率最高。

#### 2.2.1 wrap 

```
<!--pages/test/test.wxml-->
<view class="container">
  <view class='flex-item bg1' ><text>1</text></view>
  <view class='flex-item bg2' ><text>2</text></view>
  <view class='flex-item bg3' ><text>3</text></view>
  <view class='flex-item bg4' ><text>4</text></view>
</view>
```

```
/* pages/test/test.wxss */
.container {
  width: 700rpx;
  margin-left: 25rpx;
  display: flex;
}

.flex-item {
  width: 200rpx;
  height: 200rpx;
  border-radius: 10rpx 10rpx;
  margin: 10rpx;
}
.bg1 {
  background-color: #ff3f34;
}
.bg2 {
  background-color: #fbbf00;
}
.bg3 {
  background-color: #44ac45;
}
.bg4 {
  background-color: #2f84d5;
}
```

在这里我将每一个小方块长度和宽度分别调整为 200 ，很明显方块很明显就变形了。 

![1563195599247](https://nux.oss-cn-beijing.aliyuncs.com/doc/053.png)

当我们设置了 flex-wrap : wrap 属性，方块超出一定的部分就自动换行解决被挤压的情况。

```
.container {
  width: 700rpx;
  margin-left: 25rpx;
  display: flex;
  flex-wrap: wrap; 
}
```

![1563195803753](https://nux.oss-cn-beijing.aliyuncs.com/doc/054.png)

### 2.3 justify-content 属性

`justify-content`  属性定义了项目在主轴上的对齐方式。

```
.container {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

> flex-start（默认值）：左对齐
> flex-end：右对齐
> center： 居中
> space-between：两端对齐，项目之间的间隔都相等。
> space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

#### 2.3.1 flex-end 

为了让方块很容易看清楚这里调整了大小为 100 。为 container 添加属性  justify-content : flex-end

```
.container {
  width: 700rpx;
  margin-left: 25rpx;
  display: flex;
  justify-content:flex-end;
}

.flex-item {
  width: 100rpx;
  height: 100rpx;
  border-radius: 10rpx 10rpx;
  margin: 10rpx;
}
.....
```

![1563196300949](https://nux.oss-cn-beijing.aliyuncs.com/doc/055.png)

#### 2.2.2 space-between

将  container 中的属性  justify-content 中的 flex-end 修改为 space-between 。

```
.container {
  width: 700rpx;
  margin-left: 25rpx;
  display: flex;
  justify-content:space-between;
}
```

![1563196490047](https://nux.oss-cn-beijing.aliyuncs.com/doc/056.png)

### 2.4 align-items 属性

`align-items` 属性定义项目在交叉轴上如何对齐。

 ```
 .container{
   align-items: flex-start | flex-end | center | baseline | stretch;
 }
 ```

> flex-star ：交叉轴的起点对齐。
>
> flex-end ：交叉轴的终点对齐。
>
> center ：交叉轴的中点对齐。
>
> baseline : 项目的第一行文字的基线对齐。
>
> stretch（默认值）：如果项目未设置高度或设为 auto ，将占满整个容器的高度。

在这里为了更好的展现这个属性，我将第二个方块高度设置成 200 rpx ; 在进行  align-items 交叉轴的中点对齐。

#### 2.4.1 center

```
<!--pages/test/test.wxml-->
<view class="container">
  <view class='flex-item bg1' ><text>1</text></view>
  <view class='flex-item bg2' ><text>2</text></view>
  <view class='flex-item bg3' ><text>3</text></view>
  <view class='flex-item bg4' ><text>4</text></view>
</view>
```

```
/* pages/test/test.wxss */

.container {
  width: 700rpx;
  margin-left: 25rpx;
  display: flex;
  align-items: center;
}

.flex-item {
  width: 100rpx;
  height: 100rpx;
  border-radius: 10rpx 10rpx;
  margin: 10rpx;
}

.bg1 {
  background-color: #ff3f34;
}

.bg2 {
  background-color: #fbbf00;
   width: 100rpx;
  height: 160rpx;
}

.bg3 {
  background-color: #44ac45;
}

.bg4 {
  background-color: #2f84d5;
}

```

![1563196944297](https://nux.oss-cn-beijing.aliyuncs.com/doc/057.png)

#### 2.4.1 flex-end 

```
.container {
  width: 700rpx;
  margin-left: 25rpx;
  display: flex;
  align-items: flex-end;
}
```

![1563196991392](https://nux.oss-cn-beijing.aliyuncs.com/doc/058.png)





## 3 容器的组合

#### 3.1 导航栏菜单

```html
<view class="container">
  <view class='flex-item bg1' ><text>1</text></view>
  <view class='flex-item bg2' ><text>2</text></view>
  <view class='flex-item bg3' ><text>3</text></view>
  <view class='flex-item bg4' ><text>4</text></view>
</view>
```

```css
.container{
  width: 700rpx;
  margin-left: 25rpx;
  display: flex;
  /* 两端对齐，项目之间的间隔都相等  */
  justify-content: space-between;
}
.flex-item{
	width:130rpx;
	height:130rpx;
    border-radius: 10rpx 10rpx;
}
.bg1{
	background-color:#ff3f34;
}
.bg2{
	background-color:#fbbf00;
}
.bg3{
	background-color:#44ac45;
}
.bg4{
	background-color:#2f84d5;
}
```

只使用了 display: flex; 和  justify-content: space-between;  让我们的菜单导航栏分布均匀，相隔相等。

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/019.png)





下面的内容在首页中和个人中心特定菜单栏想对比较多

```
<view class="container">
  <view class='flex-item' >
    <view class="tab bg1"></view>
    <text>栏目1</text>
  </view>
  <view class='flex-item' >
    <view class="tab bg2"></view>
    <text>栏目2</text>
  </view>
  <view class='flex-item' >
    <view class="tab bg3"></view>
    <text>栏目3</text>
  </view>
  <view class='flex-item' >
    <view class="tab bg4"></view>
    <text>栏目4</text>
  </view>
</view>
```

```
.container{
  width: 680rpx;
  margin-left: 30rpx;
  display: flex;
  /* 两端对齐，项目之间的间隔都相等  */
  justify-content: space-between;
}
.flex-item{
  display: inline-flex;
  /**flex-direction 属性决定主轴的方向*/
  flex-direction: column;
  /* justify-content: center; */
  align-items:center;
}
.tab{
  width:130rpx;
  height:130rpx;
  border-radius: 10rpx 10rpx;
}

.bg1{
	background-color:#ff3f34;
}
.bg2{
	background-color:#fbbf00;
}
.bg3{
	background-color:#44ac45;
}
.bg4{
	background-color:#2f84d5;
}
```



![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/020.png)



`flex-direction` 改变主轴方向，如下：

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/021.png)

```
<!--index.wxml-->
<view class="container">
  <view class='flex-item' >
    <view class="tab bg1">图片</view>
    <text>栏目1</text>
  </view>
  <view class='flex-item' >
    <view class="tab bg2">描述</view>
    <text>栏目2</text>
  </view> 
</view>
<view class="container reverse">
  <view class='flex-item' >
    <view class="tab bg3">描述</view>
    <text>栏目3</text>
  </view>
  <view class='flex-item' >
    <view class="tab bg4">图片</view>
    <text>栏目4</text>
  </view>
</view>
```

```
/**index.wxss**/
.container{
  width: 680rpx;
  margin-left: 30rpx;
  display: flex;
  flex-wrap: wrap;
  
}
.flex-item{
  display: inline-flex;
  flex-direction: column;
  align-items:center;
}
.tab{
  width:300rpx;
	height:300rpx;
  border-radius: 10rpx 10rpx;
  margin: 20rpx;
}

.bg1{
	background-color:#ff3f34;
}
.bg2{
	background-color:#fbbf00;
}
.bg3{
	background-color:#44ac45;
}
.bg4{
	background-color:#2f84d5;
}
.reverse{
  display: flex;
  flex-direction: row-reverse;
}
```

在弹性盒子 flex-direction: row-reverse，轻松改变第二个块元素改变方向。

## 4 小结

flex 还有很多在日常开发中，让我们布局很简便就能实现。开发者多多使用，便能熟能生巧。不用不会，用了便会，此篇的 flex 例子，只是演示了部分其他大家也练习，方便有需之日。

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)