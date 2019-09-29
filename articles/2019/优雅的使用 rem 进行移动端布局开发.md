
> 作者：falost
> 原文地址：https://falost.cc/article/5d593827f3ad2c3fe3bc9ad9

### 缘起

在工作中，移动APP端的同学问我，在移动web端中是如何去做不同的屏幕大小所展示的效果和设计图都是一致的？

我们在遇到这种问题的时候，脑海中第一反应，应该是使用哪种布局方案去实现这种效果吧？

### 布局方案

先说说布局方案，在前端历史计划中，诞生了几种布局概念，分别是：
1. 静态布局
2. 流式布局
3. 自适应布局
4. 响应式布局
5. 弹性布局

### 题外话

这其中，因 bootstrap 的原因，我们对响应式布局绝对是很熟悉的，它的栅格系统就是使用了响应式布局方案。
那么静态布局，也是我们经常使用在 PC 当中，但是随着科技的发展，越来越多的屏幕大小，以至于静态布局再也满足不了我们需求了，这时候自适应布局就出来了，根据不同尺寸和设备切换不同的样式。关于布局概念，本文就不做过多的介绍，切入正题。

### rem 布局

同样相对于 PC 端，有不少的屏幕尺寸需要做兼容处理，那么移动端也是不例外的，而且移动端的机型是比较复杂的，那么就需要一套比较完善的布局方案来支持了。

因淘宝的 Flexible 让rem布局火了起来，导致前端圈中出现了很多的 rem 布局方案，我就是 rem 布局方案的崇拜者，那么就来说说 rem 布局。

### rem是什么

rem（font size of the root element）是指相对于根元素的字体大小的单位，简单的说它就是一个相对单位。

在面试中，我经常会问答 rem 和 em 的区别，得到的答案也是各有千秋，那么，到底该怎么去理解两个单位？

其实 em（font size of the element）也是一个相对单位。只是 em 相对于父元素的字体大小的单位。rem 是相对于根元素的字体大小的单位。

### 案例

刚才我们说到了淘宝引发的潮流，那么我们在说几个：亚马逊、携程、京东、当当这些网站都有使用了的 rem 的布局方案。

### rem + flex 布局方案

rem 的布局，大家都知道就是操作 html 根元素的字体大小来实现的不同尺寸单位的换算。这里重点的就是根元素字体大小的设置，之前我所使用的是通过媒体查询来设置不同的大小，比如：

```css
@media screen and (max-width: 414px) {
  html {
    font-size: 18px
  }
}

@media screen and (max-width: 375px) {
  html {
    font-size: 16px
  }
}

@media screen and (max-width: 320px) {
  html {
    font-size: 12px
  }
}
```
因视窗单位的设备支持度越来越好，所以就有了

```css
html{
    font-size: calc(100vw/7.5);
}
```

再配合 js 处理不支持视窗单位的设备

```JavaScript
document.documentElement.style.fontSize = window.innerWidth/7.5 + 'px'
```

**可能这里，就会有朋友问我，为什么这里要除于 7.5 呢?**

**答：**这里是因为很多设计稿都是基于 iPhone6 来设计的，一般都是 750px（2 倍图，iPhone6 的设备宽度为 375px）所以除于 7.5 是为了在 iPhone6 设备下让 1rem 等于 100px，当然这个可以根据你设计稿来定。

### 如何在 css 中根据设计图的 px 尺寸来缓存 rem 尺寸呢？

这里，我将使用 scss 的便捷来处理单位的换算：


```scss
@function px2rem($px, $base-font-size: 50px) {
  @if (unitless($px)) {
    @warn "Assuming #{$px} to be in pixels, attempting to convert it into pixels for you";
    @return px2rem($px + 0px); // That may fail.
  } @else if (unit($px) == rem) {
    @return $px;
  }
  @return ($px / $base-font-size) * 1rem;
}
```

因为这里的设计图使用的是 1 倍图，所以`$base-font-size`为 100px / 2 = 50px

```css
div {
    width: px2rem(100px);
}
```
换算结果
```css
div {
    width: 2rem;
}
```

到这里，我们的 rem 布局使用就已经完成了，那么在实际工作当中，肯定一个 rem 是解决不了所有的布局需求的，那么这里我们在配合着 flex 布局，就能解决一大半的布局问题了，关于 flex 布局，我将会在下文中分享！

最后，感谢各位的耐心阅读，如果文中有不对或不全之处，欢迎留言指教！

在最后，如果感觉本文不错或对你有帮助，不妨点个赞呗！
