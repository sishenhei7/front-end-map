## 使用 Chrome 插件拦截广告的原理

项目地址：[chrome_plugin_zhihu_adblock](https://github.com/sishenhei7/chrome_plugin_zhihu_adblock)

本文阅读起来可能需要先了解一些 Chrome 插件基础知识，通过本文您可以学到什么？

1. chrome 插件拦截广告```思考方法和一般原理```
2. 浏览器```拦截 fetch 和 xhr 请求```的方法

### 思路

网页上的广告可以分以下三种情况：

1. 网页上的广告都是由 html 构成的，所以只要```用 chrome 插件删除这些 html```即可。
2. 有些广告是在正常加载后进行动态加载的，它们会和一般的代码块混在一起，这里我们可以拦截代码块的 http 请求，然后```在请求成功之后，把广告从 html 上面删去```。
3. 有些广告是通过专门的 http 请求加载的，这里我们拦截这些 http 请求，让他们```发不出去```即可。

### 删除 html

我们只需要知道广告 html 的 selector，然后直接删除即可，这里我们写一个比较通用的函数：

```js
// fuckAd
const createFuckAd = (adSelector, textSelector) => () => {
    // 广告
    const ads = document.querySelectorAll(adSelector);

    if (ads.length > 0) {
        const cardBrand = document.querySelector(textSelector);

        if (cardBrand) {
            console.log(`已屏蔽广告:${cardBrand.innerText}`);
        }

        // 删掉广告
        [...ads].forEach(item => item.parentNode.removeChild(item));
    }
}
```

然后我们```在 onload 事件里面删除广告```即可。我们以知乎为例，代码如下：

```js
window.onload = () => {
    createFuckAd('.Pc-card')();
    createFuckAd('.Pc-feedAd-container', '.Pc-feedAd-card-brand--bold')();
    createFuckAd('.Pc-word-card', '.Pc-word-card-brand-wrapper > span')();
}
```

### 拦截请求

对于动态加载的广告，我们可以拦截请求，判断是否有加载广告的请求进来了，然后对于混有正常请求的代码，我们```在请求成功之后删掉广告```；对于只有广告的请求，我们```直接拦截这些请求```。这里我们以 fetch 为例（对于 xhr 请求，可以通过这个库[Ajax-hook](https://github.com/wendux/Ajax-hook)），代码如下：

```js
// hook fetch
const fetch_helper = {
    originalFetch: window.fetch.bind(window),
    myFetch: function (...args) {
        // 拦截只有广告的 http 请求
        if (args[0].includes('https://www.zhihu.com/commercial_api/')) {
            return Promise.reject(1);
        }

        return fetch_helper.originalFetch(...args).then((response) => {
            // 对于有正常代码的 http 请求，在请求完成之后，删除广告
            if (response.url.startsWith('https://www.zhihu.com/api/v3/feed/topstory/recommend?')) {
                setTimeout(createFuckAd('.Pc-feedAd-container', '.Pc-feedAd-card-brand--bold'), 188);
            }
            return response;
        });
    },
}

window.fetch = fetch_helper.myFetch;
```

### 其它

核心代码我们已经写完了，然后只需要在```document_start```的时候引入代码即可。这里需要注意的是，我们不能直接在 contentScript 里面运行上面的代码，因为虽然 contentScript 能够操作页面上的 dom 元素，但是它运行在一个```独立的环境```里面，这里我们需要使用[chrome.runtime.getURL](https://developer.chrome.com/extensions/runtime#method-getURL)获取 url，然后动态加载：

```js
const s = document.createElement("script");
s.src = chrome.runtime.getURL("main.js");
s.onload = function () {
    s.parentNode.removeChild(s);
};
(document.head || document.documentElement).appendChild(s);
```

### 思考

其实上面只是拦截广告的方法，但是对于一个好的广告拦截插件，**怎么判断 html 和 js 是一个广告**，才真正是一个非常大的挑战，我们通过查看[adblock plus的源码](https://github.com/adblockplus/adblockpluschrome/tree/20d522a45dcce3328d2f6a1a507b81d19bd1a820/lib)可以发现它是这么判断的：

```js
// popupBlocker.js
function checkPotentialPopup(tabId, popup)
{
  let url = popup.url || "about:blank";
  let documentHost = extractHostFromFrame(popup.sourceFrame);

  let specificOnly = !!checkWhitelisted(
    popup.sourcePage, popup.sourceFrame, null,
    contentTypes.GENERICBLOCK
  );

  let filter = defaultMatcher.matchesAny(
    parseURL(url), contentTypes.POPUP,
    documentHost, null, specificOnly
  );

  if (filter instanceof BlockingFilter)
    browser.tabs.remove(tabId);

  logRequest(
    [popup.sourcePage.id],
    {url, type: "POPUP", docDomain: documentHost, specificOnly},
    filter
  );
}
```

它首先获取可能的跳转弹窗的 url，然后**判断 url 的 host 和当前页面的 host是否一样，再判断白名单里面有没有**。当然还有使用[tensorflow](https://github.com/tensorflow/tensorflow)通过机器学习收集可能的广告 url:

```js
// ml.js
const tfCore = require("@tensorflow/tfjs-core");
const tfConverter = require("@tensorflow/tfjs-converter");

for (let object of [tfCore, tfConverter])
{
  for (let property in object)
  {
    if (!Object.prototype.hasOwnProperty.call(tf, property))
      tf[property] = object[property];
  }
}
```
