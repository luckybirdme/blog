---
title: CSS 响应式
date: 2019-10-20 22:34:50
tags: CSS
---

> 考虑到不同设备(PC,Mobile)的宽高度不一样，前端 Web 要不做两套页面，要不就兼容不同宽高度的展示。
> 最早的兼容方式是响应式，在前端利用 CSS 的 meida query 来获取设备宽高度，然后使用不同的样式。
> 后来出现了自适应，在前端区分不同设备，并告诉后端，后端再返回不同的页面。

<!-- more -->

### 一，兼容不同设备的方式
##### 1. 响应式(responsvie)
> Ethan M. 的文章最早提出 RWD 的概念，现在公认为RWD的起源。他提出的 RWD 是采用 CSS 的 media query 技术，配合流体布局（ fluid grids ）和同样可以自适应的图片/视频等资源素材。说人话的话，就是把一行分为12个格子，PC上显示12个一行，而手机上显示2个或者1个一行。所以一般响应式设计都是先画格子，再放内容。和直接上来就绝对定位的不一样。以上所说的各种，在技术上都是通过 HTML 和 CSS 就能完成的，当然偶尔也会用到一点点JS。但是只要用的不多就好。一般来说，RWD 倾向于只改变元素的外观布局，而不大幅度改变内容。 Jeffrey Z. 总结非常好，把 RWD 定义为一切能用来为各种分辨率和设备性能优化视觉体验的技术吧。（其实等于没总结，人家本来就是这个意思么）简单的概括，服务器不知道来访的设备是什么，所有的设备都是用的同一套逻辑。纯正的RWD非常适合CDN，这个可不是盖的。


##### 2. 自适应(adaptive)
> Adaptive Design 是 Aaron G. 的一本技术书的标题，时间要比RWD晚一些。他认为 AWD 在包括 RWD 的 CSS media query 技术以外， AWD 有可能会针对移动端用户减去内容，减去功能。AWD 可以在服务器端就进行优化，把优化过的内容送到终端上。意思就是，服务器知道用户是从手机上访问的，所以就发送手机上专用的资源给手机，这样打开会更快些。AWD其实是CDN不友好的，因为CDN不会识别访问设备哦，至少目前还没有一个支持。但是 AWD 的好处是后端根据不同设备返回优化后的页面给前端，比如手机页面，无需携带电脑端的样式代码，节省带宽，也方便分开调试。


#### 总结：
响应式纯正的前端解决方式，只需要开发一套网页，但是需要考虑更多兼容性问题；自适应从后端返回不同页面，维护多套网页，考虑兼容性少，但是维护成本也高。





### 二，响应式的实现
##### 1. 首先获取设备宽高度
- 页面设置

```html
<meta name="viewport" content="width=device-width; initial-scale=1.0; maximum-scale=1; minimum-scale=1; user-scalable=no;">
```

- 参数解释
|属性名|取值|描述|
|-|-|-|
|width|正整数或device-width|定义视口的宽度，单位为像素，device-width是指自动获取不同设备宽度
|height|正整数或device-height|定义视口的高度，单位为像素，一般不用
|initial-scale|[0.0-10.0]|定义初始缩放值，等于 1 代表首次打开的页面时不缩放
|minimum-scale|[0.0-10.0]|定义缩小最小比例，等于 1 代表不能小于初始宽高
|maximum-scale|[0.0-10.0]|定义放大最大比例，它必须大于或等于minimum-scale设置，等于 1 代表不大于初始宽高
|user-scalable|yes/no|定义是否允许用户手动缩放页面，默认值yes

根据以上解释，刚才页面设置意思是：自动获取设备宽度(width=device-width)，首次打开页面不缩放(initial-scale=1.0)，后续也不缩小和放大(maximum-scale=1; minimum-scale=1)，也不允许用户手动缩放页面(user-scalable=no)



##### 2. 根据不同宽高度使用不同样式

##### 主要两种使用方式
- 通过 link 使用不同的样式，当页面最大宽度小于 800px，那么就使用 example.css 样式。如果大于 800px，就不使用，但依然会下载。


```html
<link rel="stylesheet" media="(max-width: 800px)" href="example.css" />

```

- 直接在 style 编写不同样式，当页面最大宽度小于 800px，那么就不显示 sidebar



```css
<style>
@media (max-width: 800px) {
  .sidebar {
    display: none;
  }
}
</style>
```

##### media query 根据判断条件，确定使用哪种样式，语法
```
@media [设备] [逻辑] (参数)
```
- 常用设备
|属性值|描述|
|-|-|
|all|所有设备，默认值|
|screen|电脑或者手机屏幕|
|print|打印机|
|tv|电视机|

- 常用逻辑
|属性值|描述|
|-|-|
|and|同时满足|
|only|限定条件|
|or|或者，也可用逗号(,)|

- 常用参数
|属性值|描述|
|-|-|
|width: 320px|宽度等于 320px|
|max-width: 800px|最大宽度小于 800px|
|orientation: landscape|设备处于横屏状态|
|orientation: portrait|设备处于竖屏状态|

##### media query 具体使用
```css
/* 只在电脑或者手机屏幕上，如果最大宽度小于 320px，则不显示 sidebar */
@media only screen and (max-width: 320px) {
	.sidebar {
    	display: none;
  	}
}
/* 在电脑或者手机屏幕上，如果最大宽度小于 800px ，或者在所有设备上，最大宽度小于 500px，则不显示 sidebar */
@media screen and (max-width: 800px), all and (max-height: 500px) {
	.sidebar {
    	display: none;
  	}
}

```


##### 3. 通过百分比设置元素的宽高度

```css

/* 宽度和高度按照比例自由缩放，最大宽度不能超过全屏 */
img {
	max-width: 100%;
	height: auto;
}
.content {
	/* 全屏宽度再减去 20px */
	width: calc(100% - 20px);
	/* 避免浏览器不兼容 calc 特性*/
	width: 90%;
}

```


##### 4. 通过相对单位设置字体的大小

```css

/* 文字使用 em(相对于父节点) 和 rem(相对root) 设置相对大小 */
/* em 会将多层父节点的字体相对大小相乘 */
/* rem 只根据 root 节点，一般是 body 节点的字体相对大小 */
/* 浏览器默认字体大小 16px，字体整体缩小 16px * 62.5% = 10px */
/* 根据字体单位 rem 相对大小定义，此时，1rem = 10px */
body{
	font-size:62.5%;
}
h2{
	/* 相当于 3 * 10px = 30px */
	font-size:3rem;
	/* 避免浏览器不兼容 rem 特性*/
	font-size:30px;
}
h4{
	/* 相当于 1.2 * 10px = 12px */
	font-size:1.2rem;
	/* 避免浏览器不兼容 rem 特性*/
	font-size:12px;
}
```



### 三，Bootstrap3 响应式原理
##### 1. 布局容器，确定网页宽度
网页展示首先确定宽度，即 media query 读取的 device-width，为了适应不同视图窗口，网页宽度通过两种方式确定：
- 根据不同视图窗口设定固定宽度，比如视图窗口大于 992px 的电脑屏幕，网页固定宽度是 970px，样式名 container。
- 对于小屏幕展开全屏幕宽度，比如视图窗口小于 768px 的手机和平板设备，网页宽度自动满屏 100%，样式名 container-fluid。

|窗口大小|网页宽度|
|-|-|
|大于 1200px| 1170px|
|大于992px，小于 1200px| 970px |
|大于 768px，小于 992px| 750px |
|小于 768px|满屏 100%|

![](/example/img/bootstrap.jpg)



##### 2. 栅格系统，响应式核心实现
将网页分成不同栅格，根据不同网页宽度，自动调节栅格大小，数量以及排列方式，以便在不同设备的达到最合适的展示效果。
通过动态图来直观感受下栅格的响应式效果
![](/example/img/bootstrap.gif)

- bootstrap 使用 12 列的栅格模式，在不同网页宽度下，可自动调节为 1 列、 2 列、 3 列、 4 列、 6 列、 12 列，超出的列数会自动换行展示，样式名 col-xs-2 分 6 列，col-xs-4 分 3 列，以此类推。
![](/example/img/bootstrap-col-all.jpg)


- 栅格宽度可以是等宽，也可以不对等的宽度栅格，如下图所示。
![](/example/img/bootstrap-col-different.jpg)

- 栅格内部还可以嵌套栅格，依然按照栅格模式进行自由组合。
![](/example/img/bootstrap-col-more.jpg)



##### 3. 元素切换，显示和隐藏内容
对于不同网页大小，某些元素可以选择性显示或者隐藏，以达到适应屏幕展示。

|样式|xs 超小屏幕手机 (<768px)|sm 小屏幕平板 (≥768px)|md 中等屏幕桌面 (≥992px)|lg 大屏幕桌面 (≥1200px)|
|-|-|-|-|-|
|.visible-xs-* |可见|隐藏|隐藏|隐藏|
|.visible-sm-* |隐藏|可见|隐藏|隐藏|
|.visible-md-* |隐藏|隐藏|可见|隐藏|
|.visible-lg-* |隐藏|隐藏|隐藏|可见|
|.hidden-xs|隐藏	|可见|可见|可见|
|.hidden-sm|可见|隐藏|可见|可见|
|.hidden-md|可见|可见|隐藏|可见|
|.hidden-lg|可见|可见|可见|隐藏|




### 四，Bootstrap4 响应式原理
- Bootstrap3 响应式是使用 float, position 样式实现。
- Bootstrap4 是通过 CSS3 的新特性 flex box 来实现的，具体细节请参考文章 [CSS 弹性盒子](./2019/09/26/CSS-Flex)


> 部分图片来源网络，侵删