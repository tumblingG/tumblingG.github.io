---
title: 深入Flex布局
categories: 前端
tags: css
---
本篇博客参考w3c权威技术报告编写：[CSS Flexible Box Layout](https://www.w3.org/TR/2018/CR-css-flexbox-1-20181119/#flex-lines)

<!--more-->

注：由于一些专业名称使用英语来表示更明确，所以文章会出现较多的英语名称。

# 1. 摘要
css2.1定义了`block layout`,`inline layout`,`table layout`,`positioned layout`
4种布局来计算`boxes`的尺寸和位置。`flex`是一种新的布局模式，用来实现更复杂的布局，flex布局
可以布局在任意方向上，也可以缩放它的尺寸以避免溢出父元素。

flex布局和block布局很类似，但是它没有block布局的一些更复杂的以文本或文档为中心的属性，例如`floats`和
`columns`，相反它拥有更加强大的分配空间和排列内容的能力。

这里给出一个简单的例子：

实现一个目录，目录的每一项有一个标题，照片，描述，的支付按钮，设计者的意图是每个条目具有相同的整体尺寸，照片在文本上方，并且购买按钮在底部对齐，
而不管项目描述的长度。Flex布局使这种设计的许多方面变得容易：
- 目录使用flex布局来让项目按行排列并且确保它们是等高的，每一项又是一个按列排列的flex容器。
- 在每一项中，在源代码中按标题，描述，照片依次排列，这为文档阅读器和不支持css的浏览器提供了友好的顺序，
但在视觉上使用`order`来把照片拉到首位，并使用`align-self`来让它水平居中。
- 使用['auto' margin]()强制支付按钮放置在底部而不管描述的长度。
```css
#deals {
  display: flex;        /* Flex layout so items have equal height  */
  flex-flow: row wrap;  /* Allow items to wrap into multiple lines */
}
.sale-item {
  display: flex;        /* Lay out each item using flex layout */
  flex-flow: column;    /* Lay out item’s contents vertically  */
}
.sale-item > img {
  order: -1;            /* Shift image before other content (in visual order) */
  align-self: center;   /* Center the image cross-wise (horizontally)         */
}
.sale-item > button {
  margin-top: auto;     /* Auto top margin pushes button to bottom */
}
```
```html
<section id="deals">
  <section class="sale-item">
    <h1>Computer Starter Kit</h1>
    <p>This is the best computer money can buy, if you don’t have much money.
    <ul>
      <li>Computer
      <li>Monitor
      <li>Keyboard
      <li>Mouse
    </ul>
    <img src="images/computer.jpg"
         alt="You get: a white computer with matching peripherals.">
    <button>BUY NOW</button>
  </section>
  <section class="sale-item">
    …
  </section>
  …
</section>
```
![flex实例图片]()

# 2. Flex布局盒模型和术语
一个`flex container`就是一个元素把`display`设置为`flex`或`inline-flex`后产生的盒子，
flex容器中的孩子称为`flex items`，并且使用flex的布局模式。

为了让flex更加容易理解，这节定义了一些flex流的相关术语，`flex-flow`和`writing mode`决定了这些`items`
如何布局。
![flex layout model](https://www.w3.org/TR/2018/CR-css-flexbox-1-20181119/images/flex-direction-terms.svg)
main axis

main dimension

`main axis`就是flex容器的主轴，flex items沿着此轴排列，在`main dimension`中扩展。

main-start

main-end

容器中的`flex item`从main-start边开始排列，方向朝着main-end边。

main size

main size property

`flex container`或`flex item`的width或者height就是盒子的`main size`，它的`main size property`
    就是it的`width`or`height`。类似的min-width/min-height，max-width/max-height。

cross axis

cross dimension

垂直于主轴的轴，称为交叉轴，它在`cross dimension`上扩展。
    
cross-start

cross-end

和main-start，main-end类似。
    
cross size

cross size property

和main size, cross size property类似。

