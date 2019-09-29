> 作者：falost
> 
> 地址：https://falost.cc/article/5d74e54f8457894be1bc3b45
> 
> 如需转载请注明来源！

### 缘起

在朋友圈，你可能会见过有很多带着个人信息或者二维码的绚丽的海报图片，看起来很高大上的样子。在很早之前，我有过了解的是，这些海报图片都是由 UI 设计师，进行人肉设计出来的，非常考验设计师的忍耐力。再到后来，随着 web 技术的发展，出现了由前端来生成一张又一张的海报图片，这样 UI 设计师只需要设计一张模版图片，剩余的交给前端来完成即可，大大的减轻了 UI 小姐姐们苦力活(滑稽笑)。

虽然，UI 小姐姐的活减轻了，但是前端（包含移动端）小哥哥的活就加重了，还面临着很多头疼的问题，比如：生成的图片清晰度不够、手机兼容问题导致生成失败、资源跨域问题导致生成失败、web 端和移动端生成的图片不一样等等问题。

### 我们了解下前端是如何生成海报图片的呢？

其实前端是通过 HTML5 新增加的 canvas API 来绘制的，但是 canvas 绘制会有很多的痛点：上手门槛比较高，需要掌握 canvas API；代码可读性比较差、调试复杂；代码可复用低，每个端需要重新编码；无缓存、同一张相同图片会多次绘制，用户体验差；如果有远程图片，可能会引起跨域问题，导致绘制失败等等问题。

针对这些问题，社区出现了一个开源库（html2canvas），通过编写 HTML 页面来生成图片，有效解决了大部分问题，但是图片跨域、缓存、代码复用问题，还是无法解决。针对这些问题，社区提出了使用服务端来完成海报图片渲染，就有可能彻底解决这些问题。

### 我们该如何通过服务端来渲染这个海报呢？

在小程序社区，有赞商城前端团队成员提出基于 Puppeteer 来实现公共海报渲染服务，使用方只需传入海报图片的html，海报渲染服务绘制一张对应的图片作为返回结果，解决了canvas绘制的各种痛点问题。那么，我今天就来体验下快感吧。在这之前，我们先来了解下！

### Puppeteer 是什么

puppeteer 是一个Chrome官方出品的headless Chrome node库。它提供了一系列的API, 可以在无UI的情况下调用Chrome的功能, 适用于爬虫、自动化处理等各种场景等。

### Puppeteer 能做什么

根据官网上描述，Puppeteer几乎能实现你能在浏览器上做的任何事情，比如:

1、生成页面截图和 PDF；

2、自动化表单提交、UI 测试、键盘输入等；

3、创建一个最新的自动化测试环境。使用最新的 JavaScript ；

4、和浏览器功能，可以直接在最新版本的 Chrome 中运行测试；

5、捕获站点的时间线跟踪，以帮助诊断性能问题；

6、爬取 SPA 页面并进行预渲染(即'SSR')。

本文的渲染服务将会使用它的截图功能，来实现图片生成。

### 代码实现

接下来，就让我们简单的看看如何代码实现吧！

首先，需要先初始化一个 npm 项目，并且安装响应的模块，这里就不一一说明了。


```
npm init
npm install puppeteer koa crypto --save
```

这里，我们安装了三个模块：puppeteer 我们今天的关键性模块、koa 快速搭建一个 web 服务、crypto 通过加密内容字符串来生成唯一标识符。

另外，我的 node 版本是 10.11 的，所以使用了很多新语法。
app.js

```JavaScript
/*
 * @Descripttion: 入口文件
 * @version: 1.0.0
 * @Author: falost
 * @Date: 2019-08-27 10:54:32
 * @LastEditors: falost
 * @LastEditTime: 2019-09-08 18:20:41
 */
const SnapshotController = require('./libs/SnapshotController')

const Koa = require('koa')

const controller = new SnapshotController()

const app = new Koa()

app.use(async ctx => {
  return await controller.postSnapshotJson(ctx)
})
app.listen(3000)
```

/libs/SnapshotController.js

```JavaScript
/*
 * @Descripttion: 调取 puppenter 来生成接收到的html 数据生成图片
 * @version: 1.0.0
 * @Author: falost
 * @Date: 2019-08-27 09:55:52
 * @LastEditors: falost
 * @LastEditTime: 2019-09-08 18:20:56
 */
const crypto = require('crypto');
const PuppenteerHelper = require('./PuppenteerHelper');

const oneDay = 24 * 60 * 60;

class SnapshotController {
  /**
   * 截图接口
   * @param {Object} ctx 上下文
   */
  async postSnapshotJson(ctx) {
    const result = await this.handleSnapshot()

    ctx.body = {code: 10000, message: 'ok', result}
  }

  async handleSnapshot() {
    const { ctx } = this
    const { html } = ctx.request.body  // html 是我们将要生成的海报图片的 HTML 实现代码字符串
    // 根据 html 做 sha256 的哈希作为 Redis Key
    const htmlRedisKey = crypto.createHash('sha256').update(html).digest('hex');

    try {
      // 首先看海报是否有绘制过的
      let result = await this.findImageFromCache(htmlRedisKey);

      // 获取缓存失败
      if (!result) {
        result = await this.generateSnapshot(htmlRedisKey);
      }

      return result;
    } catch (error) {
      ctx.status = 500;
      return ctx.throw(500, error.message);
    }

  }

  /**
   * 判断kv中是否有缓存
   * @param {String} htmlRedisKey kv存储的key
   */
  async findImageFromCache(htmlRedisKey) {
    return false
  }

  /**
   * 生成截图
   * @param {String} htmlRedisKey kv存储的key
   */
  async generateSnapshot(htmlRedisKey) {
    const { ctx } = this
    const {
      html,
      width = 375,
      height = 667,
      quality = 80,
      ratio = 2,
      type: imageType = 'jpeg',
    } = ctx.request.body;

    if (!html) {
        return 'html 不能为空'
    }

    let imgBuffer;
    try {
      imgBuffer = await PuppenteerHelper.createImg({
        html,
        width,
        height,
        quality,
        ratio,
        imageType,
        fileType: 'path',
        htmlRedisKey
      });
    } catch (err) {
      // logger
      console.log(err)
    }
    let imgUrl;

    try {
      imgUrl = await this.uploadImage(imgBuffer);
      // 将海报图片路径存在 Redis 里
      await ctx.kvdsClient.setex(htmlRedisKey, oneDay, imgUrl);
    } catch (err) {
    }

    return {
      img: imgUrl || ''
    }
  }

  /**
   * 上传图片到 CDN 服务
   * @param {Buffer} imgBuffer 图片buffer
   */
  async uploadImage(imgBuffer) {
    // upload image to cdn and return cdn url
  }
}

module.exports = SnapshotController  
```

./libs/PuppenteerHelper.js


```JavaScript
/*
 * @Descripttion: 创建生成图片类
 * @version: 1.0.0
 * @Author: falost
 * @Date: 2019-08-27 11:43:41
 * @LastEditors: falost
 * @LastEditTime: 2019-09-08 18:20:51
 */
const puppeteer = require('puppeteer')
const { mkdirsSync, formatNumber } = require('../utils/utils')

class PuppenteerHelper {
  async createImg(params) {
    const browser = await puppeteer.launch({
      headless: false, // 默认为 true 打开浏览器，设置 false 不打开
    })
    const date = new Date()
    const path = `static/upload/${date.getFullYear()}/${formatNumber(date.getMonth() + 1)}`
    mkdirsSync(path)
    // 通过创建浏览器标签来打开
    const page = await browser.newPage()
    // 设置视窗大小
    await page.setViewport({
      width: params.width,
      height: params.height,
      deviceScaleFactor: params.ratio
    })
    // 设置需要截图的html内容
    await page.setContent(params.html)
    await this.waitForNetworkIdle(page, 50)
    let filePath
    // 根据 type 返回不同的类型 一种图片路径、一种 base64
    if (params.fileType === 'path') {
      filePath = `${path}/${params.htmlRedisKey}.${params.imageType}`
      await page.screenshot({
        path: filePath,
        fullPage: false,
        omitBackground: true
      })
    } else {
      filePath = await page.screenshot({
        fullPage: false,
        omitBackground: true,
        encoding: 'base64'
      })
    }
    browser.close()
    return filePath
  }
  // 等待HTML 页面资源加载完成
  waitForNetworkIdle(page, timeout, maxInflightRequests = 0) {  
    page.on('request', onRequestStarted);
    page.on('requestfinished', onRequestFinished);
    page.on('requestfailed', onRequestFinished);
  
    let inflight = 0;
    let fulfill;
    let promise = new Promise(x => fulfill = x);
    let timeoutId = setTimeout(onTimeoutDone, timeout);
    return promise;
  
    function onTimeoutDone() {
      page.removeListener('request', onRequestStarted);
      page.removeListener('requestfinished', onRequestFinished);
      page.removeListener('requestfailed', onRequestFinished);
      fulfill();
    }
  
    function onRequestStarted() {
      ++inflight;
      if (inflight > maxInflightRequests)
        clearTimeout(timeoutId);
    }
  
    function onRequestFinished() {
      if (inflight === 0)
        return;
      --inflight;
      if (inflight === maxInflightRequests)
        timeoutId = setTimeout(onTimeoutDone, timeout);
    }
  }
}
module.exports = new PuppenteerHelper()
```

../utils/utils.js

```JavaScript
/*
 * @Descripttion: 工具类库
 * @version: 
 * @Author: falost
 * @Date: 2019-08-27 14:10:16
 * @LastEditors: falost
 * @LastEditTime: 2019-08-27 14:15:52
 */
const fs = require('fs')
const path = require('path')

const mkdirsSync = (dirname) => {
  if (fs.existsSync(dirname)) {
    return true;
  } else {
    if (mkdirsSync(path.dirname(dirname))) {
      fs.mkdirSync(dirname);
      return true;
    }
  }
}
const formatNumber = function (n) {
  n = n.toString()
  return n[1] ? n : '0' + n
}

module.exports = {
  mkdirsSync,
  formatNumber
}

```
### 效果

到这里，简单的代码实现步骤，现已完成，接下来我们看看最后生成的效果图吧！

![image](https://www.fedte.cc/wp-content/uploads/2019/09/9d8d94e3dd85c9fff1b8273f2b377d3853c16ba275dba1daa16af8044ccd1ab5.jpeg)

### 结语

如果你想要用于生产环境，那么你还需要做一些其他个工作，这样才能保证他能更好的产出。当然这里面也有些坑，还需要我们去完成填坑的。

git 仓库地址：https://github.com/falost/node-puppeteer-poster

如果你想了解更多关于 puppeteer 的使用， 可以查询官方仓库说明文档：https://github.com/GoogleChrome/puppeteer

非常感谢您的耐心阅读！

文中如有不对之处，欢迎留言指教！
