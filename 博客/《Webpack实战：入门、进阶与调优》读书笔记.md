## 读书笔记

1.CommonJS 与 ES6 Module 最本质的区别在于前者对模块依赖的解决是“动态”的，而后者是“静态”的。动态的含义是：模块依赖关系的建议发生在代码运行阶段；而静态则是模块依赖关系的建立发生在代码编译阶段。所以 ES6 Module 具有以下几点优势：1.死代码检测和排除。2.模块变量类型检查。3.编译器优化。

2.在导入一个模块时，对于 CommonJS 来说获取的是一份导出值得拷贝，而在 ES6 Module 中则是值的动态映射，并且这个映射是只读的。

3.在 CommonJS 中，若遇到循环依赖我们没有办法得到预想中的结果；但是在 ES6 Module 中，我们只需要保证当导入的值被使用时已经设置好正确的导出值，就可以支持循环依赖。

4.每个 npm 模块都有一个入口。当我们加载一个模块时，实际上就是加载该模块的入口文件。这个入口被维护在模块内部 package.json 文件的 main 字段中。

5.webpack 的 context 字段表示的是基本目录，可以理解为资源入口的路径前缀，在配置时要求必须使用绝对路径的形式。默认使用当前目录。

6.为了解决打包体积过大的问题，我们可以使用提取 vendor 的方法。vendor 的意思是“供应商”，在 webpack 中 vendor 一般指的是工程所使用的的库、框架等第三方模块集中打包而产生的 bundle。

7.在多入口的场景中，我们需要为对应产生的每个 bundle 指定不同的名字，webpack 支持使用一种类似模板语言的形式动态地生成文件名。除了```[name]```可以指代 chunk name 以外，还有其它几种模板变量可以用于 filename 的配置中。

8.在 webpack4 以前的版本中，打包资源默认会生成在工程根目录，因此我们需要上述配置；而在 webpack4 之后，output.path 已经默认为 dist 目录，除非我们需要更改它，否则不必单独配置。

9.loader 里面的 test、exclude、include 本质上属于对 resource 也就是被加载者的配置，如果想要对 issuer 加载者也增加条件限制，则要额外写一些配置。

10.webpack 中的 loader 按照执行顺序可分为 pre、inline、normal、post 四种类型，上面我们直接定义的 loader 都属于 normal 类型，inline 形式官方已经不推荐使用，而 pre 和 post 则需要使用 enforce 来指定。enforce 的值为 pre，代表它将在所有正常 loader 之前执行，这样可以保证其检测的代码不是被其它 loader 更改过的。而 post 表示在所有 loader 之后执行。

11.对于 babel-loader 本身我们添加了 cacheDirectory 配置项，它会启用缓存机制，在重复打包未改变过的模块时防止二次编译，同样也会加快打包的速度。cacheDirectory 可以接收一个字符串类型的路径来作为缓存路径，这个值也可以为 true，此时缓存目录会指向 node_modules/.cache/babel-loader。

12.由于 @babel/preset-env 会将 es6 module 转化为 commonjs 的形式，这会导致 webpack 中的 tree-shaking 特性失效。将 @babel/preset-env 的 modules 配置项设置为 false 会禁用模块语句的转化，而将 es6 module 的语法交给 webpack 本身处理。

13.url-loader 与 file-loader 作用类似，唯一的不同在于用户可以设置一个文件大小的阈值，当大于该阈值时与 file-loader 一样返回 publicPath，而小于该阈值时返回文件 base64 形式编码。

14.在写自定义 loader 的时候，可以使用 this.sync 获取 callback 函数。callback 函数的3个参数分别是抛出的错误、处理后的源码，以及 source-map。

15.css-loader 在把 css 打包之后，在运行的时候会将其插入到 style 标签中，但是在生产环境下，我们希望样式存在于 css 文件中，这样更有利于客户端缓存。而 mini-css-extract-plugin 就是专门用于提取样式到 css 文件的。而说到 mini-css-extract-plugin 的特性，最重要的就是它支持按需加载 css。

16.Sass 本身是对 css 的语法增强，它有两种语法，现在使用更多的是 scss。所以你会发现，在安装和配置 loader 时都是 sass-loader，而实际的文件后缀是 .scss。

17.loader 本身只是编译核心库与 webpack 的连接器，因此这里我们除了 sass-loader 以外还要安装 node-sass，node-sass是真正用来编译 scss 的，而 sass-loader 只是起到黏合的作用。值得一提的是，加入我们想要在浏览器的调试工具里查看源码，需要分别为 sass-loader 和 css-loader 单独添加 source map 的配置项。

18.解决 node-sass 下载较慢的方法是为其单独设置一个 cnpm 的镜像地址，可使用如下命令：npm config set sass_binary_site=https://npm.taobao.org/mirrors/node-sass/

19.严格来说，postcss-loader 不能算是一个 css 预编译器，它只是一个编译插件的容器，它的工作模式是接收样式源代码并交由编译插件处理，最后输出 css。

20.我们可以在 autoprefixer 中添加需要支持的特性（如grid）以及兼容哪些浏览器（browsers）。

21.postcss 可以与 cssnext 结合使用，让我们在应用中使用最新的css语法特性。

22.使用 css modules 时 css 文件会导出一个对象，我们需要把这个对象中的属性添加到 html 标签上。

23.代码分片可以有效降低首屏加载资源的大小，但同时也会带来新的问题，比如我们应该对哪些模块进行分片、分片后的资源如何管理等，这些也是需要关注的。

24.有些时候我们不希望所有的公共模块都被提取出来，比如项目中一些组件或工具模块，虽然被多次引用，但是可能经常修改，如果将其和 react 这种库放在一些反而不利于客户端缓存。这个时候可以把 minChunks 配置项的值调大一点来解决。

25.minChunks 设置为 Infinity 的意义有两个，第一个是我们只想让 webpack 提取 vendor 字段写的几个模块，让提取更加可控；另一个是能够生成一个仅仅包含 webpack 初始化环境的文件，称为 manifest 文件。manifest 文件其实就是各个 chunk 的打包地图，以前可以用它来有效解决打包的时候 verdor 包的 chunId 变化的问题。

26.webpack runtime 指的是初始化环境的代码，如创建模块缓存对象、声明模块加载函数等。

27.CommonChunkPlugin 的不足：1.一个 CommonChunkPlugin 只能提取一个 verdor；2.会多加载一个 manifest 文件；3.会破坏掉原有 chunk 中模块的依赖关系，导致难以进行更多的优化。而 webpack4 里面的 optimization.SplitChunks 就是为了改进 CommonChunkPlugin 而重新设计和实现的代码分片特性，是 webpack 的内置模块，可以直接使用。

28.资源异步加载主要解决的问题是，当模块数量过多、资源体积过大时，可以把一些暂时使用不到的模块延迟加载。这样使页面初次渲染的时候用户下载的资源尽可能小，后续的模块等到恰当的时机再去触发加载，因此一般也把这种方法叫做按需加载。

29.与正常 es6 的 import 语法不通，通过 import 函数加载的模块及其依赖会被异步地进行加载，并返回一个 Promise 对象。

30.首屏加载的 js 地址是通过页面中的 script 标签来指定的，而间接资源（通过首屏 JS 再进一步加载的 JS）的位置则要通过 output.publicPath 来指定。

31.动态加载的 js 会插入到页面的 head 标签，并有一个 async 属性。

32.在生产环境中我们关注的是如何让用户更快地加载资源，涉及如何压缩资源、如何添加环境变量优化打包、如何最大限度地利用缓存等。

33.在 DefiinePlugin 中我们需要在值得外面加上 JSON.stringify。

34.如果对打包速度需求比较高的话，建议使用一个简化版的 source map。比如，在开发环境中，cheap-module-eval-source-map 通常是一个不错的选择，属于打包速度和源码信息还原程度的一个良好折中。

35.webpack 提供了 hidden-source-map 及 nosources-source-map 两种策略来提升 source map 的安全性。前者会产出完整的 map 文件，只不过不会在 bundle 文件中添加对于 map 文件的引用。如果我们要追溯源码，则要利用一些第三方服务，将 map 文件上传到那上面。目前最流行的解决方案是 sentry。后者的安全性没那么强，打包之后能在 source 选项卡中看到源码的目录结构，但是文件的具体内容会被隐藏起来。

36.对于生产环境的 source map 我们还有另外一种选择，就是使用 nginx 将 .map 文件只对固定的白名单（比如公司内网）开放。

37.压缩 js 的工具是 terser，它在 webpack4 里面是内置模块，可以直接使用。它相比 uglifyjs 的有点事支持 es6+ 代码的压缩。

38.压缩 css 文件的前提是使用 exract-text-plugin 或 mini-css-extract-plugin 将样式提取出来，接着使用 optimize-css-assets-webpack-plugin 来进行压缩，这个插件本质上使用的是压缩器 cssnano。

39.资源名的改变也就意味着 html 中的引用路劲的改变。每次更改后都要手动去维护它是很困难的，理想的情况是在打包结束后自动把最新的资源名同步过去。使用 html-webpack-plugin 可以帮我们做到这一点。

40.bundlesize 这个工具包可以帮助我们自动化地对资源体积进行监控。

41.开发环境中我们可能关注的是打包速度，而在生产环境中我们关注的则是输出的资源体积以及如何优化客户端缓存来缩短页面渲染时间。

42.webpack 是单线程的，假设一个模块依赖于几个其它模块，webpack 必须对这些模块逐个进行转译。虽然这些转译任务彼此之间没有任何依赖关系，却必须串行地执行。HappyPack 恰恰以此为切入点，它的核心特性是可以开启多个线程，并行地对不同模块进行转译，这样就可以充分利用本地的计算资源来提升打包速度。

43.从宏观角度来看，提升性能的方法无非两种：增加资源或者缩小范围。增加资源指的是使用更多CPU和内存，用更多的计算能力来缩短执行任务的时间；缩小范围则是针对任务本身，比如去掉冗余的流程，尽量不做重复性的工作等。

44.有些库我们是希望 webpack 完全不要去进行解析的，即不希望应用任何 loader 规则，库的内部也不会有对其他模块的依赖，那么这时可以使用 noParse 对其进行忽略。

45.exclude 和 include 是确定 loader 的规则范围，noParse 是不去解析但仍会打包到 bundle 中。最后让我们再看一个插件 IgnorePlugin，它可以完全排除一些模块，被排除的模块即使被引用了也不会被打包进资源文件中。

46.webpack 的 tree shaking 功能可以在打包过程中帮助我们检测工程中没有被引用过的模块，这部分代码将永远无法被执行到，因此也被称为“死代码”。webpack 会对这部分代码进行标记，并在资源压缩时将它们从最终的 bundle 中去掉。tree shaking只是为死代码添加上标记，真正去除死代码是通过压缩工具来进行的。比如说 terser-webpack-plugin（它已经内置在 webpack4 里面了）

47.即使我们的项目本身仅仅有一行代码，webpack 也需要将自身代码注入进去（大概50行左右）。显然 rollup 的产出更符合我们的预期，不包含无关代码，资源体积更小。

48.在进行技术选型的时候，我们不仅要结合目前工具的一些特性，也要看其未来的发展路线图。如果其能在后续保持良好的社区生态及维护状况，对于项目今后的发展也是非常有利的。

