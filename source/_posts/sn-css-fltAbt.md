---
title: [学习笔记]深入理解CSS定位机制之浮动与绝对定位
date: 2016-09-28 17:30:57
tags:
- 学习笔记
- css定位
categories:
- css
comments: true
---
# 前记
本篇深入理解浮动与绝对定位，会加入较多的布局技巧与应用。本篇结论与方法绝大多数来自于**张鑫旭**大神，先在此拜谢！
# 浮动(Float)
浮动的**设计初衷只是让文字环绕图片**而已，但是没有一件东西的使用规则是完全按照设计师来的，决定权在使用者，使用者们总是会探寻一切其它的可能。所以，我们目前利用浮动来布局本不是浮动该干的事，那么出现的种种问题也就理所当然了。
## 浮动特性
### 浮动的“包裹性”
浮动属性在某种意义上来说与`display: inline-block`属性的作用是一样的，只是浮动具有方向性而已。所以也可以这样说：浮动就是个带有方位的`display:inline-block`属性.
### 浮动的“破坏性”
浮动脱离了文档流，使父元素高度坍塌,也造成了浮动元素间紧密排列的特性。
### 浮动的定位原则
浮动的定位原则有balabala好多，但可以总结为：**在其左**、**上**、**右外边界不溢出包含块边界的情况下**，**尽量的靠上**、**靠左**（"float:left"）**或靠右**（"float:right"）**放置**，**但是不能高于它前面生成的块框**、**浮动框和行框的顶边**，**并且不能与其他浮动元素重叠**。
## 清除浮动
上篇提到了可以通过创建BFC清除浮动，还可以在父级元素上加伪元素：
```css
.clearfix:after {
    content: '';
    display: block;
    height: 0;
    overflow: hidden;
    clear: both;
}
```
上面是大家常用的，其实还有个简短的，应推广：
```css
.clearfix:after {
    content: '';
    display: table;
    clear: both;
}
```
### clear特性
设置了clear特性值的元素，其 `top border edge` 要放在相关的浮动元素的 `bottom margin edge` 之下。注意这两种元素接触边界的区别。一个是 border，一个是 margin。这与外边距折叠无关，因为浮动元素不与其他元素发生外边距折叠。
代码：
```html
<div style="width:300px; height:100px; background-color:green; float:left; border:5px solid red; margin-bottom:10px;">
    float
</div>
<div style="clear:left; width:350px; height:50px; background-color:green; border:5px solid yellow; margin-top:50px;">
    clearance
</div>
```
页面：
![clear prop](http://charles-hang.github.io/images/16_9_28/1.png)
## 浮动与流体布局
### 文字环绕衍生-单侧固定
以下是左侧固定，右侧自适应的布局示例：
代码：
```css
.box {
    width: 300px;
    background: #D2CCCC;
    padding: 10px;
    overflow: hidden;
}
img {
    width: 80px;
    float: left;
}
.word {
    margin-left: 100px;
    /*或者padding-left: 100px;*/
}
```
```html
<div class="box">
    <img src="xing.jpg" alt="xing">
    <div class="word">
        文字环绕衍生-单侧固定文字环绕衍生-单侧固定文字环绕衍生-单侧固定文字环绕衍生-单侧固定文字环绕衍生-单侧固定
    </div>
</div>
```
页面：
![left fixed](http://charles-hang.github.io/images/16_9_28/2.jpg)
上面的**关键**是，左侧：width+float；右侧：padding-left/margin-left。局限在于固定侧的宽度必须已知。
### DOM与显示位置相匹配的单侧固定
![DOM match](http://charles-hang.github.io/images/16_9_28/3.jpg)
要实现上面右侧固定，左侧自适应的布局效果且DOM流中图片在文本后面，如按前面的方法，则需将图片在DOM流中的位置提前，再改为右侧固定即可，但这样就修改了DOM顺序，那怎么做呢？
代码：
```css
.box {
    width: 300px;
    background: #D2CCCC;
    padding: 10px;
    overflow: hidden;
}
img {
    width: 80px;
    float: left;
    margin-left: -80px;
}
.word-wrap {
    width: 100%;
    float: left;
}
.word {
    margin-right: 100px;
    /*或者padding-right: 100px;*/
}
```
```html
<div class="box">
    <div class="word-wrap">
        <div class="word">
            DOM与显示位置相匹配的单侧固定DOM与显示位置相匹配的单侧固定DOM与显示位置相匹配的单侧固定DOM与显示位置相匹配的单侧固定DOM与显示位置相匹配的单侧固定
        </div>
    </div>
    <img src="xing.jpg" alt="xing">
</div>
```
**关键**是，左侧自适应部分加一包裹容器并设置：width:100%+float:left，左侧自适应部分：padding-right/margin-right，右侧：width+float:left+margin-left负值。局限仍是固定侧的宽度必须已知。这里需要大家好好琢磨琢磨，多动手试试。
### 智能自适应单侧固定
当固定侧宽度不定时，怎么达到自适应的效果呢？还记得上篇BFC消除文字环绕吗？对，这里就引申出了**浮动+BFC**的强大的自适应固定搭配。
代码：
```css
.box {
    width: 300px;
    background: #D2CCCC;
    padding: 10px;
    overflow: hidden;
}
img {
    width: 80px;
    float: left;
    margin-right: 10px;
}
.word {
    overflow: hidden;
}
```
```html
<div class="box">
    <img src="xing.jpg" alt="xing">
    <div class="word">
        智能自适应单侧固定智能自适应单侧固定智能自适应单侧固定智能自适应单侧固定智能自适应单侧固定智能自适应单侧固定智能自适应单侧固定智能自适应单侧固定
    </div>
</div>
```
左侧`float:left`,右侧`overflow:hidden`。当调节img宽度时，右侧能自适应左侧的宽度。
不过，有些场景下我们不能使用`overflow:hidden`,**这时**，**另一个能创建BFC的属性登场了**：
将上面`overflow:hidden;`处代码更换为：`display:table-cell;width:2000px;`,单元格有个非常神奇的特性，就是你宽度值设置地再大，实际宽度也不会超过表格容器的宽度。所以说，只要我们将宽度设置得足够大，那其实就跟block水平元素自动适应容器空间效果一模一样了。兼容性：IE8+，IE6/7下也有办法，但对IE深恶痛绝的我拒绝解决这个问题。
# 绝对定位(Absolute positioning)
绝对定位同样也有浮动的‘包裹性’与‘破坏性’，所以布局时，我们也经常让两者相互替代。
## 无依赖的绝对定位
无依赖的绝对定位的意思是**不受包含块(如设了relative的父级)的限制**，**不使用top/right/bottom/left来帮助精确定位的绝对定位**。使用的是margin-top/margin-left(margin-right和margin-bottom不起作用)。绝对定位后，元素会脱离文档流，且有**位置跟随特性**，位置跟随就是绝对定位之前它在什么位置，绝对定位之后它还在什么位置。这样，利用跟随性和margin来实现的定位就是无依赖的绝对定位，特性非常强大！
## 无依赖的绝对定位应用
### 图标定位
例一代码：
```css
img {
    width: 15px;
    height: 15px;
    position: absolute;
    margin: -12px 0 0 2px;
}
```
```html
<span>课程</span>
<img src="xin.png">
```
页面：
![s icon1](http://charles-hang.github.io/images/16_9_28/4.jpg)
当标签变为‘新课程’后
![s icon2](http://charles-hang.github.io/images/16_9_28/5.jpg)
如果使用的是relative帮助的话，当标签名变长后，可能会出现这样的重叠现象：
![s icon3](http://charles-hang.github.io/images/16_9_28/6.jpg)
例二代码：
```css
div {
    width: 480px;
}
i {
    height: 22px;
    width: 38px;
    position: absolute;
    background: url(-58-43.png) -58px -43px no-repeat;
}
.vip {
    position: absolute;
    margin-left: -21px;
}
```
```html
<div>
    <i></i>
    <img src="wwdc.jpg" alt="wwdc"><!--
 --><img class="vip" src="vip.png" alt="vip">
</div>
```
页面：
![s icon4](http://charles-hang.github.io/images/16_9_28/7.jpg)
免费图标仅用了`position:absolute;`vip图标仅用了`position:absolute;margin-left:-21px`。无依赖的绝对定位对DOM流中的位置有要求，免费图标absolute前大图前面，绝对定位后由于跟随性自然就到位了。vip图标呢，绝对定位之后，宽高相当于零了，还有大家注意到`<!-- -->`了吗，这是消除vip图标与前面元素间隙的方法，这样绝对定位后，宽高相当于零的vip图标由于跟随性就自然跟在大图后，向下面这样，然后margin负拉回来即可。
![s icon5](http://charles-hang.github.io/images/16_9_28/8.jpg)
这样定位的优点显而易见，如果按常规实现，就要在父容器加relative，如果日后维护修改需要去掉父级的relative，那整个定位就崩了。无依赖的绝对定位与父级没有关系，这样更健壮，更易于日后维护修改。
需要注意的是，无依赖的绝对定位对跟随性的理解要求较高，需在工作中积累经验达到能熟练的使用。
### 居中及边缘对齐定位
代码：
```css
body {
    background: #EDEFF0;
}
.container {
    width: 950px;
    margin: 0 auto;
    height: 2000px;
    background: #fff;
}
.box1 {
    height: 800px;
    background: #CE96F6;
}
.box2 {
    height: 0;
    text-align: center;
}
.box2 img {
    width: 50px;
    height: 50px;
    margin-left: -25px;
}
.box3 {
    height: 0;
    text-align: right;
}
.box3-fixed {
    display: inline;
    position: fixed;
    margin-left: 20px;
    bottom: 100px;
}
.box3-fixed a {
    display: block;
    width: 48px;
    height: 48px;
    margin-top: 10px;
    background: url(1.png) -48px 0 no-repeat;
}
```
```html
<div class="container">
    <div class="box1"></div>
    <div class="box2">
        &nbsp;<img src="xing.jpg" alt="xing">
    </div>
    <div class="box3">
        &nbsp;<div class="box3-fixed">
            <a href="#"></a>
            <a href="#"></a>
            <a href="#"></a>
        </div>
    </div>
</div>
```
页面：
![center edge](http://charles-hang.github.io/images/16_9_28/9.jpg)
这里图片居中的原理就是先将空格居中对齐，再对图片absolute+margin负。这个方式并不是最佳实践，只是开拓一下定位思路用。
接下来的边缘固定可有说法了，先将空格右对齐，再利用`display:inline`属性跟随其后，绝对定位后再用margin微调即可。这里可就比常规实现好多了，大多数网站都是绝对定位后，left:50%+margin-left大容器宽度一半的方式来实现，这样需要计算，且如果大容器宽度会变化，那么还要利用js来处理。从这里，无依赖绝对定位的优势就充分体现了。
其实无依赖绝对定位应用还有很多，像什么图标后跟文字，两者经常对不齐，位置需微调时可用。还有因为不依赖relative，就可以突破overflow的限制实现一些小功能等等。无依赖的绝对定位在便捷性，健壮性上都非常优秀，为页面布局与重构提供了许多新思路。
## 依赖的绝对定位
这里也就是行为表现上使用top/right/bottom/left的绝对定位。
常规的绝对定位这里就不说了，还记得上篇讲relative时，top和bottom,left和right同时出现时的情况吗？结果必定是一方得胜。绝对定位就不一样了，top和bottom，left和right同时出现时，结果不是一方得胜，而是拉伸的实现(兼容性IE7+)。也就是说，很多情况下，absolute的拉伸与width/height是可以相互替代的！比如：
`position: absolute; left: 0; top: 0; width: 50%;`
等同于
`position: absolute; left: 0; top: 0; right: 50%;`
在某种程度上，拉伸更强大。比如，要实现一个距离右侧200像素的全屏自适应的容器层。
拉伸实现：`position: absolute; left: 0; right: 200px;`
但是，要用width的话，只能使用CSS3 的calc计算：`position: absolute; left: 0; width: calc(100% - 200px)`
两者具有相互支持性：
1. **容器无需固定width/height值**，**内部元素就可拉伸**。这里自己体会。
2. **容器拉伸**，**内部元素支持百分比width/height值**。
通常情况下，元素百分比高要想起作用，需要父级容器的height不是auto。但在绝对定位拉伸下，即使父级容器的height值是auto，只要容器绝对定位拉伸形成，就支持百分比高度。在移动端的应用较为常见。
代码：
```css
.page {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
}
.list {
    float: left;
    height: 33.3%;
    width: 30%;
    background: #CDADE7;
    margin: 16px;
}
```
```html
<div class="page">
    <div class="list"></div>
    <div class="list"></div>
    <div class="list"></div>
    <div class="list"></div>
    <div class="list"></div>
    <div class="list"></div>
    <div class="list"></div>
    <div class="list"></div>
    <div class="list"></div>
</div>
```
页面：
![stretch](http://charles-hang.github.io/images/16_9_28/10.jpg)
最后，拉伸与width/height同时存在时有相互合作性（引申出另一种居中方法）：
当两者同时存在时，实际宽高以width/height为准。当出现`margin: auto;`时，就实现了绝对居中效果（兼容性IE8+）。
代码：
```css
.page {
    width: 300px;
    height: 300px;
    border: 1px solid #333;
    position: relative;
}
.con {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    width: 50%;
    height: 50%;
    margin: auto;
    background: #B991F4;
}
```
```html
<div class="page">
    <div class="con"></div>
</div>
```
页面：
![abt center](http://charles-hang.github.io/images/16_9_28/11.jpg)
## absolute网页整体布局（适合于移动web的布局）
代码：
```css
html,
body {
    height: 100%;   
}
.page,
.overplay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
}
header,
footer {
    position: absolute;
    left: 0;
    right: 0;
}
header {
    height: 60px;
    top: 0;
    background: #FA5B5B;
}
footer {
    height: 55px;
    bottom: 0;
    background: #F174B8;
}
aside {
    width: 260px;
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    background: #FD996E;
}
.content {
    position: absolute;
    top: 60px;
    bottom: 55px;
    left: 260px;
    overflow: auto;
}
.overplay {
    background: rgba(0,0,0,.4);
    z-index: 3;
}
```
```html
<body>
    <div class="page">
        <header></header>
        <aside></aside>
        <div class="content"></div>
        <footer></footer>
    </div>
    <div class="overplay"></div>
</body>
```
页面：
![layout](http://charles-hang.github.io/images/16_9_28/12.jpg)
布局思路：
1. body降级 子元素升级。page拉伸，需要设置html body高度百分百。
2. 各模块-头尾、侧边框用绝对定位各居其位。
3. 内容区域想象成body，里面小伙伴愉快玩耍。
4. 为实现一些加载提示效果，布局全屏覆盖与page平级。这样两者相互不影响。
这样的布局头尾侧边框等实现了与fixed一样的效果，还避免了移动端`position:fixed`实现的诸多问题，值得一试。
## display、position和float的相互关系
可以把这看做是一个类似**优先级的机制**，`position:absolute` 和 `position:fixed` 优先级最高，有它存在的时候，浮动不起作用，`display`的值会重置（规则见下表）； 其次，元素的`float`特性的值不是`none` 的时候或者它是根元素的时候，`display`的值会重置； 最后，非根元素，并且非浮动元素，并且非绝对定位的元素，`display` 特性值同设置值。这些规则简单但平时却容易忽视。
![display trans](http://charles-hang.github.io/images/16_9_28/13.jpg)
