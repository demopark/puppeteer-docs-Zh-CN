# Puppeteer [![Linux Build Status](https://img.shields.io/travis/GoogleChrome/puppeteer/master.svg)](https://travis-ci.org/GoogleChrome/puppeteer) [![Windows Build Status](https://img.shields.io/appveyor/ci/aslushnikov/puppeteer/master.svg?logo=appveyor)](https://ci.appveyor.com/project/aslushnikov/puppeteer/branch/master) [![NPM puppeteer package](https://img.shields.io/npm/v/puppeteer.svg)](https://npmjs.org/package/puppeteer)

<img src="https://user-images.githubusercontent.com/10379601/29446482-04f7036a-841f-11e7-9872-91d1fc2ea683.png" height="200" align="right">

> 此项目同步自 [GoogleChrome](https://github.com/GoogleChrome) / [puppeteer](https://github.com/GoogleChrome/puppeteer) 项目中的  docs. 除特殊情况, 将保持每月一次的同步频率.

###### [API](#api-文档) | [FAQ](#faq) | [Contributing](#贡献-puppeteer)

> Puppeteer 是一个 Node 库，它提供了一系列高级 API 来通过 [DevTools 协议](https://chromedevtools.github.io/devtools-protocol/) 控制 [headless](https://developers.google.com/web/updates/2017/04/headless-chrome) Chrome 或 Chromium。 它也可以配置为使用完整的（non-headless）Chrome 或 Chromium。


###### Puppeteer 能做什么?

可以在浏览器中手动完成的大部分事情都可以使用 Puppeteer 完成！ 你可以通过这里的几个例子来起步：

* 生成页面的截图和PDF。
* 抓取 SPA 并生成预渲染的内容（即“SSR”）。
* 从网站抓取内容。
* 自动表单提交，UI测试，键盘输入等。
* 创建一个最新的自动化测试环境。 使用最新的 JavaScript 和浏览器功能，直接在最新版本的 Chrome 中运行测试。
* 捕获你网站的 [timeline trace](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference)，来帮助诊断性能问题。

在线尝试: https://try-puppeteer.appspot.com/

## 起步

### 安装

> **注意**：Puppeteer 至少需要 Node v6.4.0，而下面的例子中使用的 async/await，只有 Node v7.6.0 或更高版本支持

要在你的项目中使用 Puppeteer, 运行:

```
yarn add puppeteer
# 或 "npm i puppeteer"
```

> **注意**：当你安装 Puppeteer，它会下载最新版本的 Chromium  (~71Mb Mac, ~90Mb Linux, ~110Mb Win) 来保证与 API 协同工作。要跳过下载，参考 [Environment variables](docs/api.md#environment-variables)。

### 使用

Puppeteer 对于使用过其他浏览器测试框架的人来说会很熟悉。 你创建一个 `Browser` 实例，打开页面，然后使用 [Puppeteer 的 API](docs/api.md#) 来操作它们。

**示例** - 导航到 https://example.com 然后保存屏幕截图为 *example.png*:

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');
  await page.screenshot({path: 'example.png'});

  await browser.close();
})();
```

Puppeteer 设定一个初始页面大小为 800px x 600px, 这决定了截图的大小. 页面大小可以用 [`Page.setViewport()`](docs/api.md#pagesetviewportviewport) 来自定义。

**示例** - 创建一个 PDF.

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://news.ycombinator.com', {waitUntil: 'networkidle2'});
  await page.pdf({path: 'hn.pdf', format: 'A4'});

  await browser.close();
})();
```

参考 [`Page.pdf()`](docs/api.md#pagepdfoptions) 来获取关于 pdf 的更多内容。

**示例** - 评估页面上下文中的脚本

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');

  // Get the "viewport" of the page, as reported by the page.
  const dimensions = await page.evaluate(() => {
    return {
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight,
      deviceScaleFactor: window.devicePixelRatio
    };
  });

  console.log('Dimensions:', dimensions);

  await browser.close();
})();
```

请参阅 [`Page.evaluate()`](docs/api.md#pageevaluatepagefunction-args) 以获取更多关于 `evaluate` 和相关方法的信息，如`evaluateOnNewDocument` 和 `exposeFunction`。

## 默认运行时设置

**1. 使用 Headless 模式**

Puppeteer 在 [headless 模式](https://developers.google.com/web/updates/2017/04/headless-chrome) 中运行 Chromium。要运行完整版本的 Chromium, 需在浏览器加载时设置 ['headless' 参数](docs/api.md#puppeteerlaunchoptions):

```js
const browser = await puppeteer.launch({headless: false}); // 默认是 true
```

**2. 运行 Chromium 的捆绑版本**

默认情况下，Puppeteer 下载并使用特定版本的 Chromium，以便使它的 API 保证能开箱即用。 要让 Puppeteer 使用不同版本的 Chrome 或 Chromium，则当创建一个 `Browser` 实例时传入可执行文件的路径：

```js
const browser = await puppeteer.launch({executablePath: '/path/to/Chrome'});
```

参考 [`Puppeteer.launch()`](docs/api.md#puppeteerlaunchoptions) 获取更多信息。

参考 [`本文`](https://www.howtogeek.com/202825/what%E2%80%99s-the-difference-between-chromium-and-chrome/) 来了解 Chromium 和 Chrome 之间的区别。 [`本文`](https://chromium.googlesource.com/chromium/src/+/lkcr/docs/chromium_browser_vs_google_chrome.md) 将介绍对于 Linux 用户的一些差异.

**3. 创建一个新的用户配置文件**

Puppeteer 创建它自己的 Chromium 用户配置文件，并在**每次运行时清理它**。

## API 文档

浏览 [API 文档](docs/api.md) 和 [示例](https://github.com/GoogleChrome/puppeteer/tree/master/examples/) 来学习更多内容。

## 调试提示

1. 关闭 headless 模式 - 有时查看浏览器显示的内容是非常有用的，而不是在 headless 模式下启动。使用 `headless:false` 启动完整版本的浏览器：

    ```js
    const browser = await puppeteer.launch({headless: false});
    ```

2. 慢下来 - `slowMo` 参数可是使 Puppeteer 操作速度减少指定的毫秒数。 这是另一种帮助查看发生了什么的方法。

    ```js
    const browser = await puppeteer.launch({
      headless: false,
      slowMo: 250 // 放慢了 250ms
    });
    ```

3. 捕获控制台输出 - 您可以侦听 `console` 事件。在 `page.evaluate()` 中调试代码时，这也很方便：

    ```js
    page.on('console', msg => console.log('PAGE LOG:', ...msg.args));

    await page.evaluate(() => console.log(`url is ${location.href}`));
    ```

4. 启用详细日志记录 - 所有公共 API 调用和内部协议流将通过 `puppeteer` 命名空间下的 [`debug`](https://github.com/visionmedia/debug) 模块进行记录。

   ```sh
   # 基本的详细记录
   env DEBUG="puppeteer:*" node script.js

   # DEBUG 输出可以通过命名空间来启用/禁用
   env DEBUG="puppeteer:*,-puppeteer:protocol" node script.js # 所有协议消息
   env DEBUG="puppeteer:session" node script.js # 协议会话消息（协议消息到目标）
   env DEBUG="puppeteer:mouse,puppeteer:keyboard" node script.js # 只有鼠标和键盘的 API 调用

   # 协议流可能相当嘈杂。 这个例子过滤掉所有网络域的消息
   env DEBUG="puppeteer:*" env DEBUG_COLORS=true node script.js 2>&1 | grep -v '"Network'
   ```

## 贡献 Puppeteer

查看 [贡献指南](https://github.com/GoogleChrome/puppeteer/blob/master/CONTRIBUTING.md) 来获得有关 Puppeteer 开发的概述。

# FAQ

#### Q: Puppeteer 使用哪个 Chromium 版本？

在 [package.json](https://github.com/GoogleChrome/puppeteer/blob/master/package.json) 中查看 `chromium_revision`.

Puppeteer 捆绑 Chromium 来确保它使用的最新功能可用。 随着 DevTools 协议和浏览器的不断改进，Puppeteer 将被更新为依赖于更新版本的 Chromium。

#### Q: Puppeteer，Selenium / WebDriver 和 PhantomJS 有什么区别？

Selenium / WebDriver是一个完善的跨浏览器API，可用于测试跨浏览器支持。

Puppeteer 仅适用于 Chromium 或 Chrome。 但是，许多团队只使用一个浏览器（例如PhantomJS）进行单元测试。 在非测试用例中，Puppeteer 提供了一个功能强大但简单的API，因为它只针对一个浏览器，使您能够快速开发自动化脚本。

Puppeteer 捆绑了最新版本的 Chromium。

#### Q: 谁维护 Puppeteer?

Chrome DevTools 团队负责维护该库，然而我们非常乐意在项目中提供帮助和专业知识！
参考 [Contributing](https://github.com/GoogleChrome/puppeteer/blob/master/CONTRIBUTING.md).

#### Q: Chrome 团队为什么要打造 Puppeteer?

该项目的目标很简单：

- 提供一个精简的，规范的库，强调 [DevTools 协议](https://chromedevtools.github.io/devtools-protocol/) 的功能。
- 为类似的测试库的实现提供参考。 最终，这些其他框架可以采用 Puppeteer 作为其基础层。
- 越来越多的采用 headless/automated 浏览器测试。
- 帮助养成新的 DevTools 协议功能...并捕捉错误！
- 详细了解自动浏览器测试的难点，并帮助填补这些空白。

#### Q: Puppeteer 与其他 headless  Chrome 项目相比如何？

过去几个月，[为自动化 headless Chrome 带来了一些新的库](https://medium.com/@kensoh/chromeless-chrominator-chromy-navalia-lambdium-ghostjs-autogcd-ef34bcd26907)。 作为开发 DevTools 协议的团队，我们很高兴看到和支持这个蓬勃发展的生态系统。

我们已经联系了一些这样的项目，看看是否有合作的机会，我们很乐意尽我们所能帮助。
