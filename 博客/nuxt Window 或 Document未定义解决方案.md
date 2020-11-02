## 概述

在用nuxt开发服务端渲染项目并引入第三方库的时候，经常会遇到**window或document未定义**的情况，原因是这个第三方库里面用到了window或者document，然后在服务端打包的时候，node环境并没有window或者document，所以就报了window或document未定义的错误。

而且，我们在引入第三方库的时候，并不希望把第三方库打包进app.js，而是希望这个第三方库只在需要的页面才加载。

下面以tinymce这个第三方库为例，记录我在nuxt.js框架中的实现方法，供以后开发时参考，相信对其他人也有用。

### 官网方法

我们不能把tinymce放到plugin里面去引入，因为这样会引入到全局js里面去。

nuxt官网介绍了一种方法：[Window 或 Document 对象未定义？](https://zh.nuxtjs.org/faq/window-document-undefined)，但是写的很简略，我这里详细说明一下。

首先我们在要引入的blog.vue文件中，通过判断是否是客户端来选择性的加载tinymce这个库：

```
let tinymce;
if (process.client) {
  tinymce = require('tinymce/tinymce');

  // A theme is also required
  require('tinymce/themes/silver/theme');

  // Any plugins you want to use has to be imported
  require('tinymce/plugins/advlist');
  require('tinymce/plugins/wordcount');
  require('tinymce/plugins/autolink');
  require('tinymce/plugins/autosave');
  require('tinymce/plugins/charmap');
  require('tinymce/plugins/codesample');
  require('tinymce/plugins/contextmenu');
  require('tinymce/plugins/emoticons');
  require('tinymce/plugins/fullscreen');
  require('tinymce/plugins/hr');
  require('tinymce/plugins/imagetools');
  require('tinymce/plugins/insertdatetime');
  require('tinymce/plugins/link');
  require('tinymce/plugins/media');
  require('tinymce/plugins/noneditable');
  require('tinymce/plugins/paste');
  require('tinymce/plugins/print');
  require('tinymce/plugins/searchreplace');
  require('tinymce/plugins/tabfocus');
  require('tinymce/plugins/template');
  require('tinymce/plugins/textpattern');
  require('tinymce/plugins/visualblocks');
  require('tinymce/plugins/anchor');
  require('tinymce/plugins/autoresize');
  require('tinymce/plugins/bbcode');
  require('tinymce/plugins/code');
  require('tinymce/plugins/colorpicker');
  require('tinymce/plugins/directionality');
  require('tinymce/plugins/fullpage');
  require('tinymce/plugins/help');
  require('tinymce/plugins/image');
  require('tinymce/plugins/importcss');
  require('tinymce/plugins/legacyoutput');
  require('tinymce/plugins/lists');
  require('tinymce/plugins/nonbreaking');
  require('tinymce/plugins/pagebreak');
  require('tinymce/plugins/preview');
  require('tinymce/plugins/save');
  require('tinymce/plugins/spellchecker');
  require('tinymce/plugins/table');
  require('tinymce/plugins/textcolor');
  require('tinymce/plugins/toc');
  require('tinymce/plugins/visualchars');

  require('tinymce/skins/lightgray/skin.min.css';
}
```

这样，在服务端就不会引入这些库，只会在客户端引入。但是服务端没有引入的话，相关js在执行的时候会报不存在的错误，这里就需要再用process.client判断一下环境再执行。示例如下：

```
if (process.client) {
  tinymce.init({
    ...options,
    ...this.otherOptions,
    language: this.language,
  });
)
```

### script方法

有时候我们希望用引入tinymce.js的方法来引入，而不用webpack打包的方式。这个时候我们需要在blog.vue里面加上如下代码即可：

```
export default {
  name: 'Blog',
  layout: 'blank',
  head: {
    script: [
      { src: '/tinymce.5.0.4/tinymce.min.js' },
    ],
  },
}
```

其中上面src的路径是static文件夹的绝对路径。

按照上述的方法会有一个问题，就是执行下面的代码的时候，即使用了process.client，但还是会报tinymce不存在的错误：

```
if (process.client) {
  tinymce.init({
    ...options,
    ...this.otherOptions,
    language: this.language,
  });
)
```

原因是，客户端打包的时候，tinymce确实是没有定义的。所以这里改成如下形式即可：

```
if (process.client) {
  window.tinymce.init({
    ...options,
    ...this.otherOptions,
    language: this.language,
  });
)
```

### 其它

nuxt有一个组件是no-ssr组件，所以上面的html最好用no-ssr包起来，不然会报tinymce组件没有定义的错误：

```
<no-ssr placeholder="Loading...">
  <tinymce
    id="myTinymce"
    v-model="content"
    :height="600"
  />
</no-ssr>
```