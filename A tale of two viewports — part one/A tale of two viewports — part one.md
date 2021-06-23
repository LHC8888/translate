原文：https://www.quirksmode.org/mobile/viewports.html

# 两个viewport的故事之一

————在这个迷你系列我将会解释视口和几个不同的且重要的元素宽度是如何生效的，比如html元素、window和屏幕。

这篇文章是讨论桌面浏览器的，不过还有个目的是讨论在移动端浏览器的相同业务场景。大多数开发者已经直观地了解了大多数桌面概念。在移动端我们可以发现同样的概念，但移动端较之更为复杂，不过拥有有以前的基础理论将让你更容易理解移动端浏览器。

## 设备像素和css像素

第一个需要掌握的概念是css像素，以及它与设备像素之间的对比。

设备像素是像素的一类，我们直观上认为其为对的？设备像素定义了你所使用的设备的分辨率，并且可以使用 window.screen.width / window.screen.height 来访问。

如果你添加了一个元素并将其width设置为128px并且最大化浏览器视口，同时你的显示器是1024px，那么这个元素的宽度将是你显示器的 1/8（1024/128=8）。。。
这时，如果用户缩放了，那么这个算法将会改变。如果用户放大为原来的200%，原来的128px宽的元素变为原来的1024px分辨率的屏幕为 1/4。（因为元素放大了，而分辨率没变）

现代浏览器实现的缩放功能无非是包含了stretching up像素。也就是说上面那个元素的宽度并不会从128px变成256px，相反的是像素的大小会变为之前的两倍。实际上，元素的宽度仍然只有128个的css像素，即时它占用了256个设备像素。

换句话说，浏览器放大到200%导致 1 css pixel = 4 device pixel （width放大2倍，height放大2倍，最终导致 1个css像素使用4个设备像素表示）。下面将使用一些图来说明这些概念。

css像素与设备像素等大（蓝色表示css像素、黑色网格表示设备像素），如下图所示：
![csspixels_100](./csspixels_100.gif)

放大浏览器后，css像素的尺寸变大（蓝色块变大），设备像素的尺寸不变，意味着1个css像素需要使用多个设备像素来表示
![csspixels_100](./csspixels_in.gif)

缩小浏览器后，css像素的尺寸变小（蓝色块变小），设备像素的尺寸不变，意味着1个设备像素与多个css像素发生了重叠
![csspixels_100](./csspixels_out.gif)

你感兴趣的只有css像素，因为它决定了你的样式如何渲染。

我通过假设当前的缩放级别为100%来说明这个例子，是时候将条件限制得更严格点了：
在缩放级别为100%时，1 css像素 = 1 设备像素

上面的缩放级别在接下来的解释中非常有用，不过你在日常开发中不必过于关心这个。你一般在开发测试PC端页面的时候都处于100%的缩放级别，即时用户缩放了页面，浏览器也会确保你的布局比例保持一致。

## Screen size 屏幕分辨率
window.screen.width 和 window.screen.height 表示用户屏幕的宽度和高度。这两个尺寸测量的是设备像素，因为它屏幕的设备像素不会发生改变，他们是屏幕设备的特性而不是浏览器的特性。

![desktop_screen](./desktop_screen.jpeg)

⚠️IE8中的IE7和IE8模式测量的是css像素

我们可以通过屏幕分辨率来做什么呢？
一般用不到这两个变量，除非你想统计web用户屏幕分辨率数据。

## Window size 窗口尺寸
相反的是，你想知道的是在浏览器窗口内部的尺寸，它让你能够准确知道用户当前可用于css布局的空间有多少。你可以通过 window.innerWidth 和 window.innerHeight 来获取这个测量信息。window.innerWidth 和 window.innerHeight 表示浏览器窗口的宽度和高度，其包括了滚动条的宽度和高度，滚动条也是窗口的一部分。

![desktop_inner](./desktop_inner.jpeg)

window.innerWidth 和 window.innerHeight 使用的是css像素进行测量的。你知道你能够放进浏览器窗口的布局内容有多少，并且随着用户放大页面内容慢慢会减少。所以当用户放大了页面，你在窗口中的可用空间也会慢慢减少（这里的可用空间应该是指能够展示的内容数量），window.innerWidth 和 window.innerHeight 的数值也会慢慢减小。

## Scrolling offset 滚动偏移量
window.pageXOffset 和 window.pageYOffset 表示文档水平和垂直的滚动偏移量。

![desktop_page](./desktop_page.jpeg)

这两个也是使用css像素进行测量的。你能够通过他们来获取文档滚动的偏移量，无论用户是否有缩放页面。理论上，如果用户滚动了页面，然后进行缩放的操作，window.pageXOffset 和 window.pageYOffset 可能会发生改变。然而，当你进行缩放的时候浏览器会保持当前页面的位置不变。（但是这个并不总是能够完美地运行，这意味着 window.pageXOffset 和 window.pageYOffset 没有发生变化，滚动到窗口外的css像素数量大体上会保持不变。这个地方感觉描述有点奇怪）

![desktop_page_zoomed](./desktop_page_zoomed.jpeg)

## 视口

视口的功能是去约束html标签的，html标签是页面的顶层包含块。

这听起来有点模糊，所以看看接下来的这个例子。假设你有一个流失布局，其中又一个sidebar，它的宽度是10%，当你放大和缩小窗口的时候这个sidebar也会平滑地变动。这是如何工作的呢？

技术角度上看，sidebar是被设置为了其父元素宽度的10%，其父元素为body标签，并且body标签并没有设置width属性。

那么body标签的width是从哪里来的呢？一般来说，块级元素的宽度等于其父元素的宽度，所以body标签的宽度与html标签的宽度相同。

那么html标签的宽度是多少？html标签的宽度是跟浏览器窗口一样大的。因此也就解释了为什么sidebar的宽度是窗口宽度的10%。

理论上来说，html标签的宽度是受viewport宽度的限制。html标签的宽度占用当前视口的宽度。viewport就是浏览器的窗口，viewport并不是 HTML construct，所以你无法使用css来控制它，在PC端上，viewport拥有浏览器窗口的宽度和高度，在移动端上它会复杂一点。

下图中水平的蓝色条状的 width 被设置为 100%，放大页面后，右上角的图片就溢出视口了（当然，你可以向右滚动查看）。
重点是我们放大页面后，视口变得比页面的宽度小了。虽然内容溢出了视口但是元素是 overflow: visible 的，这也意味着溢出的内容可以被显示出来。而蓝色条状没有溢出是因为其被设置了 width: 100%，所以其宽度与适口的宽度一样大。

![desktop_htmlbehaviour](./desktop_htmlbehaviour.jpeg)

我需要知道的是包含完整内容的页面的宽度是多少（including the bits that “stick out.” ）。据我所致，不可能获取那个数值。（当然，你可以通过获取会影响到页面宽度的元素的宽度，并求和，但这样是容易出错的）。

![desktop_documentwidth](./desktop_documentwidth.jpeg)

## 测量视口
document.documentElement.clientWidth 和 document.documentElement.clientHeight 就是视口的尺寸，它是使用css像素进行测量的。
document.documentElement 事实上就是HTML元素，是HTML文档的根元素。然而视口更高一级，可以这么说，视口包含了HTML元素。给HTML元素设置宽度很容易造成影响，这个是不建议的。当然如果你给html元素设置了宽度，document.documentElement.clientWidth 和 document.documentElement.clientHeight 都会坚定不移地返回 ** 视口的尺寸 ** ，而不是HTML元素的尺寸（这个特殊情况仅针对这个对象的这两个属性）。

![desktop_client](./desktop_client.jpeg)

## 测量html元素
那么我们如何测量 html 元素的尺寸呢？document.documentElement.offsetWidth 和 document.documentElement.offsetHeight
如果你给html元素设置了width属性，那就可以通过 document.documentElement.offsetWidth来获取到。

![desktop_offset](./desktop_offset.jpeg)
![desktop_offset_smallpage](./desktop_offset_smallpage.jpeg)

## 事件坐标
当发生鼠标事件的时候，浏览器暴露来不少于5个属性对给你用于获取发生事件的具体位置。在本文中，以下三个比较重要，
1. pageX、pageY 表示相对于html标签左上角的位置，使用css像素测量，使用频率大概为90%

![desktop_pageXY](./desktop_pageXY.jpeg)

2. clientX、clientY 表示相对于视口左上角的位置，使用css像素测量，使用频率大概为10%

![desktop_clientXY](./desktop_clientXY.jpeg)

3. screenX、screenY 表示屏幕左上角的位置，使用设备像素测量，几乎不使用

![desktop_screenXY](./desktop_screenXY.jpeg)

## 媒体查询
媒体查询可以让你定义特殊的css样式应用于给定的页面尺寸。

对于下面的规则，max-width 指的是哪种width？
```
@media all and (max-width: 400px) {
	// styles assigned when width is smaller than 400px;
	div.sidebar {
		width: 100px;
	}

}
```

在媒体查询中有两种尺寸：width/height 以及 device-width/device-height：
1. width/height 与document.documentElement.clientWidth/document.documentElement.clientHeight相同，也就是视口尺寸
2. device-width/device-height 与window.screen.width/window.screen.height相同，也就是屏幕尺寸

![desktop_mediaqueries](./desktop_mediaqueries.jpeg)

一般我们都使用 width 而不是 device-width。

