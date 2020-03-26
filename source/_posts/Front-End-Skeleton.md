---
title: 前端骨架屏生成技术揭秘
date: 2020-03-25 08:46:52
tags: ['骨架屏','node']
categories: 前端
keywords: ['骨架屏','前端']
top_img: https://goss1.cfp.cn/creative/vcg/veer/400/new/VCG41N680280554.jpg?x-oss-process=image/format,webp
cover : https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=1912403690,2623160076&fm=26&gp=0.jpg
---


<!-- # 前端骨架屏生成技术揭秘 -->

# 1. 为什么要使用骨架屏

骨架屏就是在页面数据尚未加载前，先给用户展示出页面的大致结构（灰色占位图），直到请求数据返回后再渲染页面，补充进需要显示的数据内容，考拉H5购物车就使用了骨架屏技术：





![markdown](./1.gif)



了解了骨架屏是什么，我们来看看为什么要使用骨架屏。 假如能在加载前把网页的大概轮廓预先显示，接着再逐渐加载真正内容，这样既降低了用户的焦灼情绪，又能使界面加载过程变得自然通畅，不会造成网页长时间白屏或者闪烁。骨架屏能给人一种页面内容“已经渲染出一部分”的感觉，相较于传统的 loading 效果，在一定程度上可**提升用户体验**。尤其在下面场景中，骨架屏技术能极大提高用户体验：

- 网络较慢，需要长时间等待加载处理的情况下。
- 图文信息内容较多的列表/卡片中。
- 首屏加载数据量较大的时候。



# 2. 骨架屏技术总览

目前主流的骨架屏生成技术主要包括以下三种：

1. 通过设计师给出的骨架屏图片。
2. 通过 HTML+CSS 编写骨架屏代码。
3. 非侵入式自动生成骨架屏代码。

前两种情况由于变更成本和续维护成本高，且对业务代码有一定侵入性，不进行讨论。业界对于自动生成骨架屏有多种实践，但是存在一些问题，有些配置较少，生成效果较差；有些操作繁琐，项目集成成本高，且难以定制。

本文主要针对自动生成骨架屏技术进行了深入的探讨，并开发了 awesome-skeleton，支持多种配置，以及骨架屏定制功能，并提供骨架图生成和骨架图模板注入能力。


# 3. 自动生成骨架屏技术揭秘

谷歌浏览器在2017年自行开发了 Chrome Headless 特性，并与之同时推出了 Puppeteer。Puppeteer 是一个 Node库，提供了一组用来操纵 Chrome 的 API，默认 Headless 也就是无界面的chrome，俗称“无头浏览器”。我们在浏览器中完成的大多数操作都可以在 Puppeteer 中完成，比如截图、爬虫、自动化测试、性能分析等。

借助 Puppeteer，我们对自动生成骨架屏方案进行了如下设计：

- 使用 Puppeteer 进行页面渲染，获取页面 DOM 结构；
- 命令行工具生成骨架屏代码，并支持第三方平台接入；
- 支持页面登录态、不同设备页面显示、列表重复渲染、骨架屏灰色块大小配置；
- 支持骨架屏图片压缩和代码自动生成；
- 通过 DOM 属性进行骨架屏细粒度的配置，指定节点配置，包括背景色、是否在骨架屏中显示等。

我们可以通过传入页面地址，使用无头浏览器打开页面，对页面首屏图片和文本等节点进行灰色背景处理，然后对页面首屏进行截图，生成压缩后的 base64 png 图片，并注入 HTML + CSS，从而自动生成页面骨架屏。流程见下图：

![img](./2.jpg)


## 3.1 使用 Puppeteer 渲染页面

使用 Puppeteer 可以指定不同设备模拟器来渲染页面，从而获取页面的 DOM 结构。关键代码：



```javascript
// 初始化无头浏览器
const browser = await puppeteer.launch({
  headless: !options.debug, // 是否打开无头浏览器
  args: [ '--no-sandbox', '--disable-setuid-sandbox' ],
});
// 打开新页面
const page = await browser.newPage();
// 指定设备模拟器
const device = devices[options.device] || desktopDevice;
await page.emulate(device);
// 打开指定页面
await page.goto(options.pageUrl);
```

获取到页面 DOM 结构之后，需要对其进行处理，例如将图片转换为灰色色块，隐藏大小小于一定阈值的节点，这里我们通过向页面中动态插入一端 JavaScript 脚本，对页面节点进行处理，从而生成骨架屏。关键代码：

```javascript
// 获取处理 DOM 节点的脚本代码
const scriptContent = await genScriptContent();
// 将代码插入到页面中
await page.addScriptTag({ content: scriptContent });
```



## 3.2 页面 DOM 处理

下面我们来重点看看如何对页面节点进行处理，主要包括以下内容：

![img](./3.jpg)

### 3.2.1 预处理

首先，对页面所有节点进行下列处理：

- 置空所有大小小于指定阈值的节点
- 所有节点背景图设置为 none，并且将背景色设置为骨架屏主色调（灰色）
- 所有节点阴影、边框颜色设置为主色调
- 置空所有标签中含有 “data-skeleton-empty” 属性的节点
- 若节点设置了“data-skeleton-bgcolo”，则选取其属性值来设置背景色。### 文本处理 本文处理是所有节点中比较复杂的一种，需要考虑文本是一行还是多行、文本是位置是居中还是左对齐等，通过获取文本位置和样式，通过简单的计算来设置灰色色块位置。下面只列出了关键代码，全部代码见：text.js。



```javascript
// 获取行高
if (!/\d/.test(lineHeight)) {
  const fontSizeNum = parseInt(fontSize, 10) || 14;
  lineHeight = `${fontSizeNum * 1.4}px`;
}
// 获取行数
let lineCount = (height - parseFloat(paddingTop, 10) - parseFloat(paddingBottom, 10)) / parseFloat(lineHeight, 10) || 0;
lineCount = lineCount < 1.5 ? 1 : lineCount;
// 设置文本色块样式
ele.classList.add(SKELETON_TEXT_CLASS);
Object.assign(ele.style, {
  backgroundImage: `linear-gradient(
  transparent ${(1 - textHeightRatio) / 2 * 100}%,
  ${MAIN_COLOR} 0%,
  ${MAIN_COLOR} ${((1 - textHeightRatio) / 2 + textHeightRatio) * 100}%,
  transparent 0%
  )`,
  backgroundSize: `100% ${px2rem(parseInt(lineHeight) * 1.1)}`,
  position,
});
// 添加文本Mask
if (lineCount > 1) { // 多行情况特殊处理
  addTextMask(ele, Object.assign(JSON.parse(JSON.stringify(comStyle)), {
    lineHeight,
  }));
} else { // 单行文本处理
  const textWidthPercent = textWidth / (width - parseInt(paddingRight, 10) - parseInt(paddingLeft, 10));
  ele.style.backgroundSize = `${textWidthPercent * 100}% 100%`;
  switch (textAlign) {
    case 'left':
      break;
    case 'right':
      ele.style.backgroundPositionX = '100%';
      break;
    default: // center
      ele.style.backgroundPositionX = '50%';
      break;
  }
}
```



### 3.2.2 列表处理

可以根据配置，对列表进行重复处理，也就是根据列表的第一项重复渲染其他项，关键代码：

```javascript
const listHandler = (node, options) => {
  if (!options.openRepeatList || !node.children.length) return;
  const children = node.children;
  const len = Array.from(children).filter(child => LIST_ITEM_TAG.indexOf(child.tagName) > -1).length;
  if (len === 0) return false;
  const firstChild = children[0];
  // 若元素不是列表节点，则递归处理
  if (LIST_ITEM_TAG.indexOf(firstChild.tagName) === -1) {
    return listHandler(firstChild, options);
  }
  // 只保留第一个项目
  Array.from(children).forEach((li, index) => {
    if (index > 0) {
      removeElement(li);
    }
  });
  // 重复渲染剩余项
  for (let i = 1; i < len; i++) {
    node.appendChild(firstChild.cloneNode(true));
  }
};
```

### 3.2.3 其他节点

其他节点的处理较为简单，有兴趣可以在 Github 上查看代码，这里有几个需要关注的点：

- a 标签需要移除 href 属性
- button 标签需要设置宽高，保证在移除按钮文案后宽高不变
- img 标签也需要设置宽高，并将 src 设置为 1px 的 base64
- input 标签需要移除 placeholder
- 伪元素同样需要处理。

### 3.2.4 文本类型特殊处理

需要注意的是，对于文本节点，由于存在下面的情况，所以需要进行特殊处理：

```html
<span>111<a>222</a></span> 文本中包含超链接
<span>111<img src="xx" /></span> 文本中包含图片
111 文本没有使用标签包裹
```



文本类型进行处理的关键代码：

```javascript
handleText(node) {
  const tagName = node.tagName && node.tagName.toUpperCase();
  // 处理 <div>xxx</div> or <a>xxx</a>
  if (node.childNodes && node.childNodes.length === 1 && node.childNodes[0].nodeType === 3) {
    handler.text(node, this.options);
    return true;
  }
  // 处理 xxx，转换为 <i>xxx</i>
  if (node && node.nodeType === 3 && node.textContent) {
    const parent = node.parentNode;
    // Determine if it has been processed
    if (!parent.classList.contains(SKELETON_TEXT_CLASS)) {
      // It is plain text itself and needs to be replaced with a node
      const textContent = node.textContent.replace(/[\r\n]/g, '').trim();
      if (textContent) {
        const tmpNode = document.createElement('i');
        tmpNode.classList.add(SKELETON_TEXT_CLASS);
        tmpNode.innerText = textContent;
        node.parentNode.replaceChild(tmpNode, node);
        handler.text(tmpNode, this.options);
        return true;
      }
    }
  }
  // 处理 <span>111<a>222</a></span> <span>111<img src="xx" /></span>
  if (tagName === 'SPAN' && node.innerHTML) {
    // Process image and background image first
    this.handleImages(node.childNodes);
    handler.text(node, this.options);
    return true;
  }
  return false;
}
```



### 3.2.5 处理页面节点入口函数

有些上述单个节点的处理函数，我们可以遍历页面所有节点进行处理。关键代码：

```javascript
handleNode(node) {
  if (!node) return;
  // Delete elements that are not in first screen, or marked for deletion
  if (!inViewPort(node) || hasAttr(node, 'data-skeleton-remove')) {
    return removeElement(node);
  }
  // Handling elements that are ignored by user tags -> End
  const ignore = hasAttr(node, 'data-skeleton-ignore') || node.tagName === 'STYLE';
  if (ignore) return;
  // Preprocessing some styles
  handler.before(node, this.options);
  // Preprocessing pseudo-class style
  handler.pseudo(node, this.options);
  const tagName = node.tagName && node.tagName.toUpperCase();
  const isBtn = tagName && (tagName === 'BUTTON' || /(btn)|(button)/g.test(node.getAttribute('class')));
  let isCompleted = false;
  switch (tagName) {
    case 'SCRIPT':
      handler.script(node);
      break;
    case 'IMG':
      handler.img(node);
      break;
    case 'SVG':
      handler.svg(node);
      break;
    case 'INPUT':
      handler.input(node);
      break;
    case 'BUTTON': // Button processing ends once
      handler.button(node);
      break;
    case 'UL':
    case 'OL':
    case 'DL':
      handler.list(node, this.options);
      break;
    case 'A': // A label processing is placed behind to prevent IMG from displaying an exception
      handler.a(node);
      break;
    default:
      break;
  }
  if (isBtn) {
    handler.button(node);
  } else {
    isCompleted = this.handleText(node);
  }
  // If it is a button and has not been processed by handleText, then the child node is processed
  if (!isBtn && !isCompleted) {
    this.handleNodes(node.childNodes); // 递归处理
  }
}
```



### 3.2.6 使用 rollup 打包脚本

由于对页面节点处理的脚本使用 es6 语法编写， 我们需要在插入页面之前，进行编译：



```javascript
// rollup.config.js
export default {
  input: 'src/script/main.js',
  output: {
    file: 'src/script/dist/index.js',
    format: 'iife',
    name: 'AwesomeSkeleton',
  },
};

```

**3.3 生成骨架屏代码**

生成处理页面节点脚本之后，插入到页面中，这个时候需要一些钩子来执行页面DOM的处理。我们定义一个全局对象 AwesomeSkeleton，其中包含 genSkeleton 方法，调用后会处理页面节点。

```javascript
window.AwesomeSkeleton = {
  // Entry function
  async genSkeleton(options) {
    this.options = options;
    if (options.debug) {
      await this.debugGenSkeleton(options);
    } else {
      await this.startGenSkeleton(); // 生成骨架屏
    }
  },
  // Start generating the skeleton
  async startGenSkeleton() {
    this.init();
    try {
      this.handleNode(document.body);
    } catch (e) {
      console.log('==genSkeleton Error==\n', e.message, e.stack);
    }
  },
  ...
}
```

 

在使用 puppeteer 打开页面之后，我们注入了 rollup 打包后的脚本，这时候我们需要执行脚本才能生成骨架屏：



```javascript
await page.evaluate(async options => {
  await window.AwesomeSkeleton.genSkeleton(options);
}, options);
```

生成骨架屏后，我们通过调用 puppeteer 的截图接口，生成骨架屏图片，并使用 images 包进行图片压缩，生成 base64，从而生成骨架屏代码。



```javascript
// First screen skeleton screenshot
await page.screenshot({
  path: screenshotPath,
});
const imgWidth = options.device ? 375 : 1920;
// Use images for image compression
await images(screenshotPath).size(imgWidth).save(screenshotPath);
const skeletonImageBase64 = base64Img.base64Sync(screenshotPath);
// Inject the skeleton into the desired page
const result = insertSkeleton(skeletonImageBase64, options);
```



**4. 使用 awesome-skeleton**

通过上述讨论的技术方案，我们实现了 awesome-skeleton 骨架屏生成工具。支持命令行生成骨架屏代码，同时也可以非常方便的在第三方平台接入。

## 4.1 参数配置

| 参数名称           | 必填 | 默认值          | 说明                                                         |
| ------------------ | ---- | --------------- | ------------------------------------------------------------ |
| pageUrl            | 是   | -               | 页面地址（此地址必须可访问）                                 |
| pageName           | 否   | output          | 页面名称（仅限英文）                                         |
| cookies            | 否   |                 | 页面 Cookies，用来解决登录态问题                             |
| outputPath         | 否   | skeleton-output | 骨架图文件输出文件夹路径，默认到项目 skeleton-output 中      |
| openRepeatList     | 否   | true            | 默认会将每个列表的第一项进行复制                             |
| device             | 否   | PC              | 参考 puppeteer/DeviceDescriptors.js，可以设置为 'iPhone 6 Plus' |
| debug              | 否   | false           | 是否开启调试开关                                             |
| debugTime          | 否   | 0               | 调试模式下，页面停留在骨架图的时间                           |
| minGrayBlockWidth  | 否   | 0               | 最小处理灰色块的宽度                                         |
| minGrayPseudoWidth | 否   | 0               | 最小处理伪类宽                                               |

例如添加 skeleton.config.json，生成考拉首页在 iphone X 下的骨架屏。



```json
{
  "pageName": "baidu",
  "pageUrl": "https://www.kaola.com",
  "openRepeatList": false,
  "device": "iPhone X",
  "minGrayBlockWidth": 80,
  "minGrayPseudoWidth": 10,
  "debug": true,
  "debugTime": 3000
}

```

## 4.2 一键生成骨架屏

```javascript
skeleton -c ./skeleton.config.json
```

页面 DomReady 之后，会在页面顶部出现红色按钮：开始生成骨架屏。

生成完成后，会在运行目录生成 skeleton-output 文件件，里面包括骨架屏 png 图片、base64 文本、html 文件：

- base64-baidu.png # 骨架图图片
- base64-baidu.txt # 骨架图 Base64 编码
- base64-baidu.html # 最终生成 HTML

其中 html 文件可以直接拿来用，复制下面位置：

```html
<html>
  <head>
    <!--- 骨架屏代码 -->
  </head>
</html>
```

注意：

- 骨架图默认在 onload 事件后销毁。
- 手动销毁方式：`javascript window.SKELETON && SKELETON.destroy(); ` 当然，你也可以在项目中直接使用生成的 Base64 图片。


## 4.3 解决登录问题

如果页面需要登录，则需要下载 Chrome 插件 EditThisCookie，将 Cookie 复制到配置参数中。Puppeteer 通过在打开页面的时候注入 Cookie 从而模拟登录态：



```javascript
// Write cookies to solve the login status problem
if (options.cookies && options.cookies.length) {
  await page.setCookie(...options.cookies);
  await page.cookies(options.pageUrl);
  await sleep(1000);
}
```



## 4.4 DOM 节点配置

这是获取优质骨架图的要点，通过设置以下几个 dom 节点属性，在骨架图中对某些节点进行移除、忽略和指定背景色的操作，去除冗余节点的干扰，从而使得骨架图效果达到最佳。

| 参数名称              | 说明                                  |
| --------------------- | ------------------------------------- |
| data-skeleton-remove  | 指定进行移除的 dom 节点属性           |
| data-skeleton-bgcolor | 指定在某 dom 节点中添加的背景色       |
| data-skeleton-ignore  | 指定忽略不进行任何处理的 dom 节点属性 |
| data-skeleton-empty   | 将某dom的innerHTML置为空字符串        |

示例：



```
<div data-skeleton-remove>
    <span>abc</span>
</div>
<div data-skeleton-bgcolor="#EE00EE">
    <span>abc</span>
</div>
<div data-skeleton-ignore>
    <span>abc</span>
</div>
<div data-skeleton-empty>
    <span>abc</span>
</div>
```



# 5. 开发骨架屏生成平台

有了骨架屏生成工具，我们可以非常方便的接入第三方平台，例如我们使用 egg.js 开发骨架屏生成平台，用户输入页面链接，自动生成对应骨架屏。关键代码：



```javascript
const getSkeleton = require('awesome-skeleton');
class SkeletonService extends Service {
  async generator(params) {
    try {
      const result = await getSkeleton(params);
      return {
        success: true,
        ...result,
      };
    } catch (e) {
            ...
    }
  }
}


```



页面效果：

![img](./4.jpg)

骨架屏配置：

![img](./5.jpg)

# 6. 参与贡献

Github：https://github.com/kaola-fed/awesome-skeleton