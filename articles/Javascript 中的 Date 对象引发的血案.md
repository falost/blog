
> 作者：falost
> 原文地址：https://falost.cc/article/5d62760b8d183230607f417a
> 如需转载请注明来源！

本文无任何技术含量，只能算是科普文吧！

### 缘由

在工作中，因和后端配合的时候，后端返回的时间对象不是前端所需要的标准时间，而产生的一系列问题反思！

### Date 对象

Date对象大家众所周知他是干什么用的？怎么用？这些都不是问题，但是在每个语言种类和宿主环境不同的情况下，就会产生些奇奇怪怪的问题，比如：前端通过接口拿到了一个Java后端返回的时间对象（2019-08-25 18:00:00），然后经过 js 处理展示不同的格式（如：2019/08/25、现在、08/25 18:00）前端开发的小伙子拿到这个值之后就是一顿操作：


```JavaScript
const formatTime = function (date, format = '', conn = '-') {
  date = new Date(date)
  
  let year = date.getFullYear()
  let month = date.getMonth() + 1
  let day = date.getDate()

  let hour = date.getHours()
  let minute = date.getMinutes()
  let second = date.getSeconds()
  
  switch (format) {
    case 'YY-MM-DD':
      return [year, month, day].map(formatNumber).join(conn)
    case 'YY-MM-DD HH:mm':
      return [year, month, day].map(formatNumber).join(conn) + ' ' + [hour, minute].map(formatNumber).join(':')
    case 'YY-MM-DD HH:mm:ss':
      return [year, month, day].map(formatNumber).join(conn) + ' ' + [hour, minute, second].map(formatNumber).join(':')
    case 'MM-DD HH:mm':
      return [month, day].map(formatNumber).join(conn) + ' ' + [hour, minute].map(formatNumber).join(':')
    case 'MM-DD':
      return [month, day].map(formatNumber).join(conn)
    case 'YYMMDD':
      return year + '年' + formatNumber(month) + '月' + formatNumber(day) + '日'
    case 'MMDD':
      return formatNumber(month) + '月' + formatNumber(day) + '日'
    default:
      return [year, month, day].map(formatNumber).join(conn) + ' ' + [hour, minute, second].map(formatNumber).join(':')
  }
}
const formatNumber = function (n) {
  n = n.toString()
  return n[1] ? n : '0' + n
}
console.log(formatTime('2019-08-25 18:00:00', 'MM-DD HH:mm', '/')) // 08/25 18:00
```

![image](https://www.fedte.cc/wp-content/uploads/2019/08/Google.jpg)

执行之后没有毛病，我们如愿的拿到了想要的效果 `08/25 18:00`，有同学肯定会说，这么简单，有啥问题呢？

![image](https://www.fedte.cc/wp-content/uploads/2019/08/1566727062087.jpg)

在谷歌这些通用浏览器中，好像的确没啥问题，都是正常情况，但是拿到了 Safari 浏览器之后，傻眼了，什么情况？怎么得到的是一个（`"NaN/NaN NaN:NaN"`）字符串出来了？我的时间呢？

### 时区

**我们先来了解下时区**

> 由于世界各国家与地区经度不同，地方时也有所不同，因此会划分为不同的时区。正式的时区划分，其中包括24个时区，每一时区由一个英文字母表示。每隔经度15°划分一个时区，有一个例外，每个时区有一条中央子午线；例如，GMT属于“z”区，因此其时间后通常添加后缀“Z”（口语中用后缀“Zulu”）。

这是来自百度百科关于时区的一段说明，那么最基本的解释就是：是地球上的区域使用同一个时间。

**然后我们再来看看 JavaScript 中对于时间的描述**

> 创建一个 JavaScript Date 实例，该实例呈现时间中的某个时刻。Date 对象则基于 Unix Time Stamp，即自1970年1月1日（UTC）起经过的毫秒数。

也就是说，在 JavaScript 中Date对象是基于 UTC 时间标准来衡量时间的。

**那么我们再来看看什么是 UTC？**

> 协调世界时，又称世界统一时间、世界标准时间、国际协调时间。由于英文（CUT）和法文（TUC）的缩写不同，作为妥协，简称UTC。

还是来百度百科的说明，看到 UTC 那么肯定会想到 GMT，其实UTC与格林威治平均时(GMT, Greenwich Mean Time)一样，都是英国伦敦的本地时相同，所以在 JavaScript 中 UTC == GMT。

### 发现问题

其实说了这么多文献，就是说明一件事---时区，那为什么要说明时区呢？因为我们这里就是出现时区的问题上，导致 Safari 的 JS 引擎没有识别到当前时间值时区的划分，导致在 new Date() 时候出现了错误,如：

```
new Date('2019-08-25 18:00:00') // Invalid Date 
```

![image](https://www.fedte.cc/wp-content/uploads/2019/08/1566731763128.jpg)

OK，到这里这里之后，我们在回归刚才的问题上：

1、发现了在 Safari 浏览器中不能识别类似时间（`'2019-08-25 18:00:00'`）的，导致出现了 NaN 的情况？

2、发现了问题根本原因是因为时间不是 UTC 标准格式（没有时区信息）导致的？

3、现在给出问题的解决方案。

### 解决方案

既然我们已经知道了出现问题的根本原因，那么就可以对症下药了：

1、后端对时间进行重新转义，将时间转换为 UTC 标准时间格式，如：`'2019-08-25T18:00:00+08:00'`

![image](https://www.fedte.cc/wp-content/uploads/2019/08/1566731789127.jpg)

2、后端将时间转换成时间戳形式，如：`1566727200000`

![image](https://www.fedte.cc/wp-content/uploads/2019/08/1566731802476.jpg)

这两点是后端处理的方法，前端处理方法，比较复杂，需要将时间字符串通过正则替换为标准时间，这里就不多说了，因为这种方法治标不治本。

当然，如果你有什么好的处理方法，欢迎留言给我！

### 总结

另外，这里也提到了一个能高效解决问题的方案：

1、发现出现了什么问题？

2、发生这个问题的根本原因是什么？

3、在提出解决方案。

我们解决问题，不能只解决表象问题，要找问题的根本原因在哪里，然后再着手解决处理，这样我们才能真正的解决问题。不要一发现问题，就想着怎么去处理，那么有可能就会出现很多未知问题。找代码的 bug 如果不按照这种思维去处理的话，我相信你的代码就像下面一样：

![image](https://www.fedte.cc/wp-content/uploads/2019/08/80bc757adab44aed3af42c26bf1c8701a38bfbda.gif)

好了，本文就啰嗦到这里了，文中如有不对之处，望多多指教！

祝大家新的一周愉快！

### 参考文献

> 1、时区：https://baike.baidu.com/item/%E6%97%B6%E5%8C%BA/491122
> 
> 2、协调世界时：https://baike.baidu.com/item/%E5%8D%8F%E8%B0%83%E4%B8%96%E7%95%8C%E6%97%B6/787659
> 
> 3、北京时间：http://www.beijing-time.org/time15.asp
> 
> 4、JavaScript 内置 Date 对象：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date
