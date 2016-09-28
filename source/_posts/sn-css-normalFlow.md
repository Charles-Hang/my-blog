---
title: '[学习笔记]深入理解CSS定位机制之常规流'
date: 2016-09-10 15:50:26
tags: 
- 学习笔记
- css定位
categories:
- css
comments: true
---
# 概述
CSS有三种基本的定位机制：**常规流**（Normal flow）、**浮动**（Floats）、**绝对定位**（Absolute positioning）。分两篇讲述：常规流篇，浮动和绝对定位篇。
# 常规流
常规流看似就是块级元素、行内元素依据他们的显示属性按照在文档中的先后次序依次显示，好像没啥说的。其实不然，常规流作为默认的定位模式，其他的定位模式都以其为基础。在 CSS2.1中，常规流包括块框( block boxes )的块格式化( block formatting )， 行内框( inline boxes )的行内格式化( inline formatting )，块框或行内框的相对定位，以及插入框的定位。
## 块级格式化上下文（BFC）
### 触发方式
一个新的BFC可以通过给一个容器添加如下属性来创建：
浮动（float值不为none）
绝对定位（position值不为relative和static）
display值为table-cell，table-caption，inline-block，flex，inline-flex中任何一个
overflow值不为visible
### 表现特性及应用
BFC说到底就是一个**独立的渲染区域**，**BFC里面的元素不论怎么翻江倒海**，**使劲折腾都不会影响到外面的元素**。在BFC中，每一个元素左外边与包含块的左边相接触（对于从右到左的格式化，右外边接触右边）， 即使存在浮动也是如此。
![BFC BOX](http://charles-hang.github.io/images/16_9_10/1.jpg)
BFC与普通的块盒子类似，不同之处在于：**可以阻止外边距折叠**，**可以包含浮动元素**，**可以防止元素被浮动元素覆盖**。
#### 用BFC阻止外边距折叠
代码：
```css
p {
    color: #fff;
    width: 200px;
    background: #827171;
    line-height: 100px;
    text-align: center;
    margin: 50px;
}
```
```html
<p>margin折叠了</p>
<p>是的，折叠了</p>
```
页面：
![margin fold](http://charles-hang.github.io/images/16_9_10/2.png)
修改代码：
```css
div {
    overflow: hidden;
}
p {
    color: #fff;
    width: 200px;
    background: #827171;
    line-height: 100px;
    text-align: center;
    margin: 50px;
}
```
```html
<p>成功阻止margin折叠了</p>
<div>
    <p>好尴尬啊</p>
</div>
```
页面：
![margin unfold](http://charles-hang.github.io/images/16_9_10/3.png)
使用overflow：hidden创建了一个BFC。
#### 使用BFC来包含浮动元素
我们都知道float会使父元素高度坍塌，通常我们用清除浮动来解决这个问题，最常用的就是使用clearfix伪类元素，但我们同样可以通过定义一个BFC来达到这个目的，这就是我们平常为父元素添加`overflow: hidden;`属性的原理，这里大家都懂，就不演示了。具体的float特性及清除浮动到下篇详细讲解。
#### 使用BFC防止文字环绕
这里利用的就是BFC的区域不会与float box重叠。
代码：
```css
body {
    width: 300px;
}
.float {
    float: left;
    width: 100px;
    height: 150px;
    background: #FB6E7C;
    opacity: 0.4;
    margin-left: -10px;
    margin-top: -10px;
}
p {
    color: #fff;
    background: #827171;
}
```
```html
<body>
    <div class='float'></div>
    <p>文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字 环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果文字环绕效果</p>
</body>
```
页面：
![text surround](http://charles-hang.github.io/images/16_9_10/4.png)
从这个效果图中我们也可以理解为什么文字会环绕。
修改代码：为p元素增加样式`overflow: hidden;`
页面：
![eliminate surround](http://charles-hang.github.io/images/16_9_10/5.png)
p元素创建了个BFC。这样就不会与左侧的float元素重叠，由此还产生了一些布局应用，下篇再说。
## 行内格式化上下文（IFC）
IFC控制着内联元素的布局，这里就不得不介绍下行内框盒子模型了，所有的内联元素的样式表现都与行内框盒子模型有关。行内框盒子模型里包含着四种盒子。以下面p标签内的一行文字说明。
![line box e.g.](http://charles-hang.github.io/images/16_9_10/6.png)
1. **内容区域**（content area），是一种围绕文字看不见的盒子，它的大小与font-size大小相关。可近似将鼠标选中文字的区域看做内容区域。
![content area](http://charles-hang.github.io/images/16_9_10/7.jpg)
2. **行内框**（inline boxes），行内框不会让内容成块显示，而是排成行。如果文字外含inline水平的标签（span，a，em等），则属于行内框，如果是光秃秃的文字，则属于匿名行内框。如下图中，蓝框表示的是一个行内框，两个红框表示的是两个匿名行内框。
![inline boxes](http://charles-hang.github.io/images/16_9_10/8.jpg)
3. **行框**（line boxes），每一行就是一个行框，每个行框又是由一个一个行内框组成的。如下图。
![line boxes](http://charles-hang.github.io/images/16_9_10/9.jpg)
4. **包含盒子**（containing box），p标签所在的包含盒子，由一行一行的行框组成。注意与p元素的块不是一回事。
![containing box](http://charles-hang.github.io/images/16_9_10/10.jpg)
行内框可能被分割
代码：
```html
<p style="background-color:silver; width:100px;">
    <span style="border:1px solid blue; font-size:50px;">text in span</span>
    <em style="border:1px solid yellow; font-size:30px; vertical-align:top;">great1</em>
</p>
```
页面：
![inline break](http://charles-hang.github.io/images/16_9_10/11.png)
如果一个行内框超出包含它的行框的宽度，它会被分割成几个框，并且这些框会被分布到几个行框内。如果一个行内框被分割，**margin**、**padding 和 border 在所有分割处没有视觉效果**。
IFC布局规则，行内框会从包含块的顶部开始，一个接一个水平摆放。摆放这些框的时候，它们水平的margin、padding、border有效，垂直无效。不能指定宽高。行框的宽度是由包含块和存在的浮动来决定。行框的高度由行高计算规则决定。
![line box width](http://charles-hang.github.io/images/16_9_10/12.png)
这里我想有必要拓展一下，对line-height进行深入理解，以更好地体会IFC。
## line-height深入
### line-height的高度机理
**行高**，**两行文字基线之间的距离**。接着上一小节，**p元素的高度不是由里面的文字撑开的**，**是由line-height决定的**，如下证明：
`font-size: 36px;` `line-height: 0;`下的表现：
![line-height test1](http://charles-hang.github.io/images/16_9_10/13.png)
`font-size: 0;` `line-height: 36px;`下的表现：
![line-height test2](http://charles-hang.github.io/images/16_9_10/14.png)
有这样一个公式：**内容区域高度** + **行间距** = **行高**。内容区域高度只与字号以及字体有关，与行高无关。宋体下，公式可写为：font-size + 行间距 = 行高。上面说了行高决定了高度，但是呢高度的表现是内容区域高度加上下两个半行间距决定的，这里好好结合行高的定义琢磨下。所以做个小总结，**行高决定行内框高度**，**行间距自动调整大小**（甚至负值，上面测试例子里，line-height= 0时，行间距为-36px）**以保证高度正好等于行高**。**高度在页面上的表现是由内容区域高度和行间距决定的**。
### line-height的一些实际应用
#### line-height的三个属性值差别
line-height分别为1.5、150%、1.5em有何区别？
line-height: 1.5 下面的所有可继承元素根据font-size重计算行高。
line-height: 150%/1.5em 当前元素根据font-size计算行高，继承给下面的元素。
#### line-height与图片的表现
我们都应该遇到过下面的情况，图片底部有间隙。
![img gap1](http://charles-hang.github.io/images/16_9_10/15.png)
其实这是图片后的**隐匿文本**起的作用，给图片后加一个字便明了了
![img gap2](http://charles-hang.github.io/images/16_9_10/16.png)
这里它们默认是以基线对齐的，所以下边多出了一块。知道了原因，那么消除这个底部间隙的方法就知道了：
1. 图片块状化（无基线对齐了）
```css
img {display: block;}
```
2. 图片底线对齐
```css
img {vertical-align: bottom;}
```
3. 行高足够小（导致基线上移）
```css
.box {line-height: 0;}
```
#### 多行文本水平垂直居中
代码：
```css
p {
    width: 300px;
    height: 300px;
    border: 1px solid #F44343;
    line-height: 300px;
    text-align: center;
}
span {
    width: 150px;
    display: inline-block;
    line-height: normal;
    text-align: left;
    vertical-align: middle;
}
```
```html
<p>
    <span>多行文本水平垂直居中多行文本水平垂直居中多行文本水平垂直居中多行文本水平垂直居中多行文本水平垂直居中</span>
</p>
```
页面：
![multiline center](http://charles-hang.github.io/images/16_9_10/17.png)
注意：关键在于把多行文本所在容器的display水平设为inline-block，以及重置外部继承的text-align和line-height属性值。这里是**近似垂直居中**，因为字体的中线的位置与实际的居中位置是不同的，不同的字体的中线位置也有区别。IE8+兼容。
## 相对定位（relative positioning）
### 常规流中的占位
相对定位元素处于常规流中，相对于元素在常规流中的原位置进行定位，偏移后，在常规流中依然占据原有位置，不会影响其它元素的布局，但是**相对定位后**，**它的层叠上下文级别提高了**，**意味着可能产生框的重叠**，也就是挡住其它的元素。关于层叠上下文，就不深入了，又是另一个深坑了，想了解就自行百度层叠上下文及层叠水平进行了解。
### 溢出包含块
如果相对定位引起 "overflow:auto" 或 "overflow:scroll" 框的溢出，浏览器必须允许用户访问内容，既，创建需要的滚动条，这可能会影响布局。
代码：
```html
<div style="width:100px; height:100px; overflow:auto; border:2px solid blue;">
    <div style="width:40px; height:40px; background-color:red; position:relative; top:100px; left:100px;">X</div>
</div>
```
页面：
![overflow block1](http://charles-hang.github.io/images/16_9_10/18.png)
![overflow block2](http://charles-hang.github.io/images/16_9_10/19.png)
### 定位的计算
top/bottom 同时存在时，top起作用，bottom无效。
left/right 同时存在时，如果包含块的direction属性是ltr， 那么left起作用，right无效；如果包含块的direction属性是rtl，那么right起作用，left无效。
### 相对定位最小化影响原则
上面说到相对定位会影响层叠上下文，这就可能带来许多想不到的遮挡、覆盖问题。所以在**平时布局中要将relative的影响最小化**。那么怎么做呢？
平时，我们想对一个元素绝对定位时，总是在其父级上添加relative定位属性，在通过top、left、right、bottom对其调整定位。其实这是个不好的习惯，这增加了出现问题的潜在风险，对父级来说relative定位属性是毫无用处的。这时就应该使用**无依赖的绝对定位**（不用relative帮助），具体属于下篇绝对定位的内容了，敬请期待！再有，当无依赖绝对定位难以达到要求必须借助relative帮助时，我们应该将绝对定位元素单独提取出来，为其添加一个相对定位的父级来帮助其定位，类似如下：
```html
<div style="position: relative;">
    <img src="***.png" alt="***" style="position: absolute; top: 0; right: 0;">
</div>
```
这样就大大降低了出现问题的可能。

下篇讲浮动和绝对定位这两大头！