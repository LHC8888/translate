原文链接：https://www.quirksmode.org/mobile/viewports2.html

# 两个viewport的故事之二

这个篇文章我们将谈论移动端浏览器，如果你是刚接触移动端，那么我建议你先阅读本系列第一篇文章，先了解一些在PC端中相同的场景。

## 移动端的问题

当我们比较移动端浏览器和PC浏览器时，最明显不同的地方就是屏幕尺寸。为移动端优化的网页明显比PC端的网页要少，有时文本会被缩小到几乎阅读不了，或者页面仅仅展示了一部分在屏幕上。

平板设备的中间层，比如iPad和传闻中的HP web平板，都降缩小PC端和移动端的差距，但是这边并不会解决本质上的问题。我们仍然需要为了网页更好的显示效果而去适配尺寸更小的屏幕。

最重要的问题在于CSS，特别是视口 viewport 的尺寸。如果我们原封不动地将PC端网页的样式复制到移动端中，CSS将不会达到你想要的效果。

还记得上篇文章的 设置了 width: 10% 的 sidebar 吗？如果你在移动端设置了同样的样式，而且你的屏幕设备为 400px，那么它的宽度将为 40px，这是非常窄的，这将导致你的流式布局变得及其拥挤。

一个解决问题的方法是开发一个专门用于在移动端显示的网页，然而即时你需要解决这个问题，但是很少的开发这会这样做。

## 两个 viewport

移动端浏览器视口太窄而无法很好地进行css布局。一个显而易见的解决方案就是将视口变宽，因此就产生了两个视口：可视视口（visual viewport）和布局视口（layout viewport）。

下面是 George Cummins 在 stack overflow 上的解释：  
布局视口可以想象为一张不会改变尺寸和形状的大图像。然后想象你拿这一个小窗口，并且透过它去看这张大图像。这个小窗口四周都是不透明的像素，并且挡住了这个大图像的内容，但是你可以透过这个小窗口去查看大图像部分的内容。这部分你可以看到的内容就是可视视口。你可以将窗口远离大型图像（缩小），来查看更多的内容，也可以靠近大型图像（放大），来查看部分内容。你也可以改变窗口的方向（切换横竖屏），但是布局视口的尺寸和形状都不会发生改变。

Chris 的解释如下：  
可视视口是用户当前在屏幕上看到的内容，用户可以通过滚动来改变可见内容，也可以通过缩放来改变可视视口的大小。  
![mobile_visualviewport](./mobile_visualviewport.jpeg)

然而，CSS布局，特别是百分比单位的宽度，是相对于布局视口来计算的，而布局适口要比可视视口大很多。

因此，html元素的宽度使用布局视口的来初始化，这就好像你的屏幕比手机屏幕要宽很多，这样可以确保网页在移动端表现得跟在PC端上一致。

那么布局视口有多宽呢？每个浏览器都不一样，Safari iPhone - 980px, Opera - 850px, Android WebKit - 800px, IE - 974px

## 缩放
Both viewports are measured in CSS pixels, obviously. But while the visual viewport dimensions change with zooming (if you zoom in, less CSS pixels fit on the screen), the layout viewport dimensions remain the same. (If they didn’t your page would constantly reflow as percentual widths are recalculated.)

可视视口和布局视口都是使用 css像素进行测量的。可视视口的尺寸会因为缩放而发生变化（如果放大，则屏幕上的css像素尺寸变大，css像素数量就会减少，可视视口的尺寸就变小了，css像素数量决定了可视视口的尺寸？），而布局适口的尺寸会一直保持不变。如果不是这样，那么你的页面将会不断回流，因为百分比宽度一直在重新计算。

## 布局视口的理解
为了有助于理解布局视口，我们需要看看网页在完全缩小的情况下是怎样的。许多移动端浏览器都会以完全缩小的模式来初始化页面。

重点是浏览器会选择他们各自的布局视口尺寸，这使得它会以完全缩小的模式覆盖屏幕，此时，可视视口是跟布局适口一样的。如下图：  
![mobile_viewportzoomedout](./mobile_viewportzoomedout.jpeg)

因此，布局视口的宽度和高度等于在最大缩放模式下在屏幕上显示的任何内容？？ 当用户在这些尺寸中缩放时保持相同？？ 如下图：
![mobile_layoutviewport](./mobile_layoutviewport.jpeg)

如果您旋转手机，则可视视口会发生变化，但浏览器会通过稍微缩放以适应新的方向，这样便使布局视口再次与可视视口相同了。
![mobile_viewportzoomedout_la](./mobile_viewportzoomedout_la.jpeg)
![mobile_layoutviewport_la](./mobile_layoutviewport_la.jpeg)

## 测量布局视口
document.documentElement.clientWidth / document.documentElement.clientHeight 可用于获取布局视口尺寸，采用的是css像素进行测量的  
![mobile_client](./mobile_client.jpeg)
![mobile_client_la](./mobile_client_la.jpeg)

## 测量可视视口
window.innerWidth / window.innerHeight 用于获取可视视口的尺寸，使用css像素进行测量
当用户放大网页，可视视口中css像素的尺寸变大，数量变少，可视适口的尺寸变小，
当用户缩小网页，可视视口中css像素的尺寸变小，数量变多，可视适口的尺寸变大  
![mobile_inner](./mobile_inner.jpeg)

## 屏幕
window.screen.width / window.screen.height 获取屏幕尺寸，使用设备像素来测量的
![mobile_screen](./mobile_screen.jpeg)

无法直接通过js提供的属性获取屏幕的缩放界别，但是可以通过 window.screen.width / window.innerWidth 来计算。一般情况下用不到缩放级别。一般我需要知道的是当前屏幕有多少css像素，这个可以使用window.innerWidth来获取。

## 滚动偏移量
window.pageXOffset / window.pageYOffset 用来获取当前可视视口相对于布局视口的位置，也就是滚动的偏移量，使用css像素进行测量  
![mobile_page](./mobile_page.jpeg)

## html元素
document.documentElement.offsetWidth / document.documentElement.offsetHeight 用于获取html元素的尺寸，使用css像素进行测量  
![mobile_offset](./mobile_offset.jpeg)

## 媒体查询
移动端的媒体查询与PC端的一样
![mobile_mediaqueries](./mobile_mediaqueries.jpeg)

## 事件坐标
移动端的事件坐标在测试中或多或少的都会出现问题。  
event.pageX / event.pageY 相对于页面的位置，使用css像素测量  
event.clientX / event.clientY 相对于可视视口，使用css像素测量  
event.screenX / event.screenY 相对于屏幕，使用设备像素  
![mobile_pageXY](./mobile_pageXY.jpeg)
![mobile_clientXY](./mobile_clientXY.jpeg)

## Meta标签的 viewport
最后我们讨论一下，下面这个标签，最初其只是一个Apple的拓展，后面被其他浏览器也实现了。它会重新定义布局视口的尺寸
```
<meta name="viewport" content="width=320">
```

假设你开发了一个简单的页面，并且没有给元素设置宽度，那么它们的宽度将会等于布局宽度。大多数浏览器在初始化页面的时候都会把整个布局视口展示在屏幕，图下图：  
![mq_none](./mq_none.jpeg)

这时，用户都会通过放大来浏览页面，但大多数浏览器会保持元素的宽度不变，这使得用户难以阅读文本。
![mq_none_zoomed](./mq_none_zoomed.jpeg)

现在可以尝试给html设置 width: 320px ，然后html元素就缩小了，同时其里面的元素也跟着缩小了。This works when the user zooms in, but not initially, when the user is confronted with a zoomed-out page that mostly contains nothing.
![mq_html300](./mq_html300.jpeg)

为了解决上面的问题，Apple就发明了meta viewport标签，当你设置 <meta name="viewport" content="width=320"> 时，就是给布局视口设置320px的宽度，下图给viewport添加了 width=screen.width，如图所示：
![mq_yes](./mq_yes.jpeg)

你可以给布局视口设置任何的尺寸，包括 device-width 。有时候使用screen.width 没啥作用，因为设备像素量太高了。