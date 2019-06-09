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
# 3.Flex容器：`flex`和`inline-flex`display值
>Name: display

>values: flex | inline-flex

flex: 这个值引发元素生成一个`flex container`盒子，当它处于流布局的时候它是一个块级元素。

inline-flex: 这个值引发元素生成一个`flex container`盒子，当它处于流布局的时候它是一个行内级元素。

一个`flex container`为它的内容建立了一个flex格式化上下文，这和建立块级格式化上下文类似的，但是它们之间
也有一些不同，一些适用于`block container`的属性不一定适用于`flex container`。`overflow`属性适用于`flex container`。
- `float`和`clear`并不会为`flex item`创建浮动或者清除浮动，并且也不会让它脱离布局流。
- `vertical-align`对`flex item`没有效果。
- `::first-line`和`::first-letter`伪元素不适用于`flex container`,并且`flex container`元素也不会成为它祖先的`::first-line`和`::first-letter`。

在某些情况下指定为`inline-flex`的display值会被计算为`flex`,请参考：[转换表格](https://www.w3.org/TR/CSS2/visuren.html#dis-pos-flo)

# 4. Flex Items
在布局流中，`flex container`的每一个孩子成为一个`flex item`,并且每一个连续的`text`被包装为一个匿名的`block container flex item`，然而如果一个`text`只包含空格，那么
它就不会被渲染（就好像它的display值是none）。
```html
<div style="display:flex">

    <!-- flex item: block child -->
    <div id="item1">block</div>

    <!-- flex item: floated element; floating is ignored -->
    <div id="item2" style="float: left;">float</div>

    <!-- flex item: anonymous block box around inline content -->
    anonymous item 3

    <!-- flex item: inline child -->
    <span>
        item 4
        <!-- flex items do not split around blocks -->
        <q style="display: block" id=not-an-item>item 4</q>
        item 4
    </span>
</div>

```
![4.1图片]()
>注意内部元素的空格消失了，匿名的`item box`是不能设置style的，因为没有元素去设置，它只能从`flex container`继承styles。

一个`flex item`为它的内容建立了一个独立的块级格式化上下文。

在某些情况下`flex item`的行内display值会被块级化，详情请参考：[转换表格](https://www.w3.org/TR/CSS2/visuren.html#dis-pos-flo)
，[css display](https://www.w3.org/TR/css-display/#transformations)

>注意：一些`display`值会在原始盒子周围创建匿名的盒子，但对于`flex item`来说，它首先被块级化，所以不会创建匿名盒子，例如两个连续的`flex items`的
display值被设置为`table-cell`将会创建两个分开的块级`flex items`,而不是被包装为一个单独的匿名table。

对于display值被指定为`table`的flex item，表格包装盒子成为一个flex item，`order`和`align-self`适用于它，任何标题框的内容都有助于计算表格盒子的最小和最大宽度。
然而对于`width`和`height`采用以下规则计算：flex items的最终尺寸是在执行布局的时候计算的，就好像表格盒子和表格内容的间隔是表格盒子的`border+padding`区域一样。

## 4.1 绝对定位Flex Children
绝对定位的child不参与flex布局，因为它脱离了文档流。

flex container中的绝对定位元素的位置被计算就好像它是flex container中仅有的唯一元素一样，并且假设flex container和flex item都是固定大小的尺寸。所以基于此目的，`auto` margin被视为0。

例如你对flex container中的绝对定位的元素设置`align-self: center;`,自动offset将会让它在`flex container cross axis`居中。然而因为一个绝对定位的flex item是固定尺寸的，所以值`stretch`被视为和
`flex start`相同。

## 4.2 Flex item Margins and Paddings
相邻flex items的margins不会合并。百分比的margins和paddings和`block boxes`一样，相对于它们的`containing block`计算。
`Auto`margins会扩展去吸收相应区域额外的空间，它们被用于对齐或者使相邻的flex items分离。详情参考：[Aligning with auto margins](https://www.w3.org/TR/2018/CR-css-flexbox-1-20181119/#auto-margins)

## 4.3 Flex Item Z-Ordering
flex items绘制和`inline block`类似，除了`order`属性会改变源文档的顺序，`z-index`的值不为`auto`的会创建一个栈上下文，甚至是position被设置`static`的时候也一样（表现的好像是position是relative一样）。

# 5. 顺序和方向
flex container的内容可以沿着任意方向和任意顺序布局，这个功能通过使用`flex-direction`,`flex-wrap`和`order`属性实现。

## 5.1 Flex流方向：flex-direction属性
>Name: flex-direction

>value: row | row-reverse | column | column-reverse

>Initial: row

>Applies to: flex containers

flex-direction决定了flex items在主轴上沿着什么方向布局。

row：在现在的[writing mode](https://www.w3.org/TR/css-writing-modes-4/#writing-mode)中,在主轴的方向和`inline axis`相似，
`main-start`和`main-end`分别和`inline-start`和`inline-end`方向相同。

row-reverse：和row相似，除了main-start和main-end调转过来。

column：在writing mode中，主轴的方向和`block axis`一致，
`main-start`和`main-end`分别和`block-start`和`block-end`方向相同。

column-reverse：和column相似，除了main-start和main-end调转过来。

>注意，reverse值并没有改变盒子的顺序：像`writing mode`和`direction`一样，它们仅仅改变了流的方向，绘制顺序，语音顺序和顺序导航顺序不受影响。

## 5.2 Flex行包装：flex-wrap属性
> Name: flex-wrap

> Value: nowrap | wrap | wrap-reverse

> Inital: nowrap

> Applies to: flex containers

flex wrap属性控制flex containers是单行的还是多行的，以及交叉轴的方向。
交叉轴的方向决定了新行被叠加的方向。

nowrap: flex containers是单行的。
wrap: flex containers是多行的。
wrap-reverse: 和wrap一样，不过方向相反。

## 5.3 Flex方向和包装：flex-flow快捷方式
> Name: flex-flow

> Value：<flex-direction> || <flex-wrap>

> Initial: row nowrap

> Applies: flex containers

flex-flow是flex-directive和flex-wrap的简写形式，定义了flex container的主轴和交叉轴。
```css
div { flex-flow: row; }
/* Initial value. Main-axis is inline, no wrapping.
   (Items will either shrink to fit or overflow.) */
```
![5.3.1图]()
```css
div { flex-flow: column wrap; }
/* Main-axis is block-direction (top to bottom)
   and lines wrap in the inline direction (rightwards). */
```
![5.3.2图]()
```css
div { flex-flow: row-reverse wrap-reverse; }
/* Main-axis is the opposite of inline direction
   (right to left). New lines wrap upwards. */
```
![5.3.3图]()

注意flex-flow是和写模式相关的。
English
```css
div {
    flex-flow: row wrap;        
    writing-mode: horizontal-tb;
}
```
![5.3.4图]()
Japanese
```css
div {
    flex-flow: row wrap;        
    writing-mode: vertical-rl;
}
```
![5.3.5图]()

## 5.4 展示顺序：order属性
flex item默认的展示顺序和源文档一样，order属性可以改变这种顺序。
> Name: order

> Value: <Interger>

> Initial: 0

> Applies to: flex items

order接受一个整整值，从最低的值开始排列，值相同的按源文档的顺序排序，绝对布局的flex items被视为`order: 0`。

例子:
在网页布局常有这样一种布局，有一个头部一个顶部，中间有一个内容区域和两个侧边栏，通常我们希望先获取内容区域，然后才是侧边栏，
以前这种布局有一定的难度，通常使用称为"Holy Grail Layout"的方式实现，现在flex的order属性让它变得容易：
```html
<!DOCTYPE html>
<header>...</header>
<main>
   <article>...</article>
   <nav>...</nav>
   <aside>...</aside>
</main>
<footer>...</footer>
```
```css
main { display: flex; }
main > article { order: 2; min-width: 12em; flex:1; }
main > nav     { order: 1; width: 200px; }
main > aside   { order: 3; width: 200px; }
```
![图5.4.1]()

注意到侧边栏和内容区域默认是等高的，内容区域会按需填充屏幕的区域，另外这可以使用媒体查询来在窄屏幕上显示列布局：
```css
@media all and (max-width: 600px) {
  /* Too narrow to support three columns */
  main { flex-flow: column; }
  main > article, main > nav, main > aside {
    /* Return them to document order */
    order: 0; width: auto;
  }
}
```
# 6. Flex Lines
flex items在flex container中排列在`flex lines`中，一个flex container可以是单行的或者多行的，
这取决于`flex-wrap`属性。

这个例子展现了4个水平排列按钮，但是因为前3个按钮占了240px，第4个按钮放不下所以被包装成了多行。
```css
#flex {
  display: flex;
  flex-flow: row wrap;
  width: 300px;
}
.item {
  width: 80px;
}
```
```html
<div id="flex">
  <div class="item">1</div>
  <div class="item">2</div>
  <div class="item">3</div>
  <div class="item">4</div>
</div>
```
![6.1图片]()

一旦内容被分割为多行，每一行就是独立布局的，弹性长度和`justify-content`,`align-self`属性一次只考虑在单行中的items。

在一个多行的flex container中（甚至只有一行），每行的交叉轴尺寸就是能够包含下该行的items的最小尺寸（通过`align-self`属性排列后），
行在flex container中的排列使用`align-content`属性设置。在单行的flex container中，行的交叉轴尺寸就是flex container的交叉轴尺寸，
`align-content`没有效果。每一行的主轴的尺寸总是和flex container内容盒子的主轴尺寸相同的。

这个例子和前一个一样，除了每个flex item被设置为`flex: autp`。
第一行的尺寸有60px的剩余空间，并且所有的item都是弹性布局，所以第一行的每一个item多接受20px的额外宽度，最终每一项100px，
剩下的第4个按钮单独占据一行，并且扩展到整行的宽度。

![6.2图片]()

# 7. 弹性
flex布局的定义方面是能够使flex items弹性，改变它们的宽度/高度去填充`main dimension`剩余的空间，这可以通过`flex`属性实现。flex container分配剩余空间给它的items（通过设置`flex-grow`因子）来填充填充容器，或者
收缩它们（通过设置`flex-shrink`因子）来阻止溢出。

一个flex item是完全没有弹性的如果它的`flex-grow`和`flex-shrink`都是0，否则是弹性的。

## 7.1 `flex`快捷方式
> Name: flex

> Value: none | [<flex-grow> <flex-shrink>? || <flex-basis>]

> Initial: 0 1 auto

> Applies to: flex items

`flex`属性指定了组件的弹性长度：弹性因子（`grow`,`shrink`）和`flex basis`。当一个盒子是flex item，使用`flex`来计算盒子的`main size`，而不是使用`main size property`。
如果盒子不是flex item，则`flex`没有效果。

<flex-grow>: 数字，指定了flex的增长因子，当有正的剩余空间的时候，增长因子决定了flex item相对于容器中其他的flex items的增长程度，
缺省时设置为1。

> 0到1之间的Flex值有一些特殊的行为：当line上的flex值之和小于1时，它们将占用小于100％的可用空间。

<flex-shrink>: 数字，指定了flex的收缩因子，当有负的剩余空间的时候，收缩因子决定了flex item相对于容器中其他的flex items的收缩程度，
缺省时设置为1。

>在分配负空间的时候，`flex shrink factor`会乘以`flex base size`，这使得负空间与item能够收缩的程度成比例的分布，例如在较大的item明显减少之前，
小的item不会缩小到零。

<flex-basis>: 在按照弹性因子为item分配剩余空间之前，这个属性指定了flex item的初始化`main size`。flex-basis接受和`width`,`height`一样的值（除了`auto`被视为不同的），以及`content`关键字。

"auto": 当在flex item使用该值时，`auto`关键字将item的`main size property`设为它的flex-basis。若那个值是也是`auto`，就使用值就是`content`。

"content": 表示基于flex item内容的自动尺寸。
> 注意：这个值没有定义在初始的flexible box layout中，因此一些老的浏览器不支持它，相同的效果可以使用`auto`和main size(`width`,`height`)设置为`auto`来实现。

<width>: 除此之外所有的其他值，都被视为和width和height一样，当缺省时，设定为0。

"none": 等同于`0 0 auto`。

![7.1图]()

这个例子展示flex-basis设置为0和auto的情况，3个item弹性因子分别被设置为1，1，2。注意到弹性因子是2的增长速率是其他的两倍。

> 注意：在flex快捷方式中，`flex-grow`和`flex-basis`的缺省值和`flex`快捷方式的初始值是不同的，以便flex可以更加好的适应比较常见的情况。

### 7.1.1 常见的flex值
"flex: initial": 等效于`flex: 0 1 auto`，基于它的width/height来设置它的尺寸（若item的`main size property`被设置为`auto`,它的尺寸将会基于它的内容），
在正的剩余空间的时候flex item没有弹性，但是没有充足空间的时候会收缩到它的最小尺寸，`alignment abilities`或者`auto` margin在主轴上排列flex items。

"flex: auto": 等效于`flex: 1 1 auto`,基于它的width/height来设置它的尺寸，并且使它完全弹性的，它会吸收任何多余的空间。

"flex: none": 等效于`flex: 0 0 auto`, 基于它的width/height来设置它的尺寸，并且使它完全不可弹性的。甚至在溢出的情况也不会收缩。

"flex: <positive-number>": 等效于`flex: <positive-number> 1 0`，使flex item弹性的，并且设置flex basis为0。

默认情况下flex items并不会缩减到它的最小尺寸以下（最大的text长度或者固定尺寸的元素），改变这个可以通过设置`min-width`或者`min-height`属性。参考：[ §4.5 Automatic Minimum Size of Flex Items](https://www.w3.org/TR/2018/CR-css-flexbox-1-20181119/#min-size-auto)

## 7.2 弹性组成部分
弹性的组成部分可以通过独立的属性来控制，但是推荐使用`flex`的快捷方式。

### 7.2.1 flex-grow属性
> Name: flex-grow

> Value: <number>

> Initial: 0

> Applies to: flex items

`flex-grow`属性设置弹性增长因子通过提供一个整数值，负值是非法的。

### 7.2.2 flex-shrink属性
> Name: flex-shrink

> Value: <number>

> Initial: 1

> Applies to: flex items

描述参考以上flex

### 7.2.3
> Name: flex-basis

> Value: content | <width>

> Initial: auto

> Applies to: flex items

> Percentages: 相对于flex container的内部main size

描述参考以上flex

# 8. 对齐
在flex container的内容完成它们的弹性，并且所有的flex item的尺寸最终确定后，它们就可以在
flex容器中对齐了。

`margin`可以用来对齐items，这和`block layout`相似，但是更加强大。flex items也遵循[CSS Box Alignment](https://www.w3.org/TR/css-align-3/)
的对齐属性，这允许在主轴和交叉轴轻松进行基于关键字的对齐。这些属性使得css2.1中许多常见的比较难以实现的对齐变得微不足道，例如
垂直和水平居中。

> 注意这里flex模块重新定义了[CSS Box Alignment](https://www.w3.org/TR/css-align-3/)的属性，以便不会创建可能减慢规范进度的规范依赖。
换句话说，一旦[CSS Box Alignment Level 3](https://www.w3.org/TR/css-align-3/)定义完成，将取代这里的定义。
















