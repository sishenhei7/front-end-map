## 概述

Kibana是一个针对Elasticsearch的开源分析及可视化平台，用来搜索、查看交互存储在Elasticsearch索引中的数据。它操作简单，基于浏览器的用户界面可以快速创建仪表板（dashboard）实时显示Elasticsearch查询动态。

### 安装部署kibana

kibana需要64位操作系统，并且需要先安装[Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html)，运行Elasticsearch又需要先安装[java](https://www.oracle.com/technetwork/java/javase/downloads/index.html)。

参考资料：

[Elasticsearch guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html)
[Kibana guide](https://www.elastic.co/guide/en/kibana/current/index.html)

下面以mac上安装部署kibana为例：

1.安装java。到[java](https://www.oracle.com/technetwork/java/javase/downloads/index.html)里面下载对应的最新java版本，并安装。

2.安装并运行elasticsearch。到[download elasticsearch](https://www.elastic.co/downloads/elasticsearch)里面下载对应的最新elasticsearch版本，解压之后进入elasticsearch文件夹运行命令：./bin/elasticsearch。

3.查看elasticsearch是否运行成功。浏览器打开elasticsearch默认地址：`http://localhost:9200/`，如果显示如下信息则表示运行成功：

```
{
  "name" : "Cv0Qzv6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "LFNZ-yjkRRW4dcVyuWlbug",
  "version" : {
    "number" : "6.5.3",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "159a78a",
    "build_date" : "2018-12-06T20:11:28.826501Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

4.安装并运行kibana。到[download kibana](https://www.elastic.co/downloads/kibana)里面下载对应的最新kibana版本，解压之后进入kibana文件夹运行命令：./bin/kibana。

5.查看kibana是否运行成功。在浏览器打开kibana默认地址：`http://localhost:5601`，如果能够打开kibana则表示kibana运行成功。

6.添加数据。首次打开kibana会提示导入数据，直接倒入demo data即可。然后就可以愉快的调教kibana了。

### kibana的大致结构

kibana是开源的，但是只能用于elastic公司的项目。从这里可以看到kibana的源码：[elastic/kibana](https://github.com/elastic/kibana)。

kibana的大致结构如下：

1. 整体框架：React 和 Angular 结合使用(利用ngReact库在Angular里面内嵌React)。

2. ui框架：EUI。Elastic公司自己开发的一套UI，开源，但是不是MIT协议，只能用于Elastic公司开发的产品。https://github.com/elastic/eui。

3. 可视化框架：D3和Vega。[D3官网](https://d3js.org/) 和 [vega官网](https://vega.github.io/vega/)。

4. 使用的拖拽库：[react-grid-layout](https://github.com/STRML/react-grid-layout)。

5. word cloud用的[d3-cloud](https://github.com/jasondavies/d3-cloud)。优点是能够自动把文本直接转化为word cloud。

6. 实现angular里面使用react的库：[ngreact](https://github.com/ngReact/ngReact)。

### kibana的主要功能

1. 主要研究kibana的visualize和dashboard两个板块。

2. visualize板块的功能：
   + 可以增加或删除自定义的展示图表。
   + 可以给这些展示图表自定义配色方案，标题等。
   + 可以把这个图表分享出去或者内嵌出去。

3. dashboard板块的功能：
   + 自定义添加visualize里面的图表，或者添加saved searches数据。
   + 每个图表或者数据是一个panel，支持panel的自定义拖拽排序和改变大小。
   + 拥有fileters，queries，time picker三种功能，通过搭配这三种可以实时改变panel中的展示。
   + 可以在dashboard里面针对具体visualization跳转到visualize板块修改对应的visualization。
   + 通过点击panel中的不同位置，可以快捷的实现dashboard的filters操作。
   + 可以通过右飘窗实时查看panel里面的数据。
   + 可以把这个dashboard分享出去或者内嵌出去。

*说明：*visualization指的是visualize板块里面那样的展示图表，saved searches指的是储存的数据，它在dashboard里面以表格的形式展示出来。

### kibana的数据结构

为了实现上面的主要功能，kibana有如下的数据结构：

#### ES

通过一个index储存这三类数据：saved searches, visualizations and dashboards。

1. saved searches：是用户自定义的查询数据，可以通过visualize板块转化为图表，也可以在dashboards里面查看纯数据。
2. visualizations：是用户在visualize板块定义的图表数据，包括使用哪个searches，以及自定义配色等。
3. dashboards：是用户在dashboards板块定义的dashboards数据，包括dashboard标题，里面有哪些visualizations或者saved searches。

#### embeddables

kibana把所有能放在dashboard上的数据都叫做embeddable，按我的理解就是visualization 和 saved searches的数据。

每个embeddable数据包含2类数据：

1. 一个是embeddable metadata，它是不变的，包括所有embeddable的配置部分，供用户配置使用；
2. 另一个是embeddable state，它是可变的，是用户对于某个visualization或者saved data的配置信息，比如saved object id，visualization的配色方案等。

注意：embeddable state里面有没有具体查询出来的数据？我觉得应该有，还应该有相应的查询条件。

通过embeddable state，kibana实现了2个功能：
1. 可以通过在url里面添加这些state的参数的形式把visualization分享或者内嵌出去。
2. 通过把这些state和dashboard的state进行交互：实现改变dashboard能够实时改变并且定制panel，比如说定制panel的标题。并且通过panel也可以改变dashboard的filters。

#### dashboards

每个dashboard都有2类数据：

1. 一个是dashboard storage data，这是储存在ES中的data，用来提供给panel进行更新，并不放在redux里面。
2. 另一个是dashboard redux tree，这是dashboard的redux的data，它包含如下几个方面：
  1. metadata。这里修改dashboard的标题和描述。
  2. embeddables。这里放置embeddable state里面redux感兴趣的部分数据，主要把embeddable里面的数据传给dashboard。
  3. panels。这里用来删除panel，增加panel，更新panel，修改panel标题，重置panel标题。
  4. view。这里用来实现fileters，queries，time picker三种功能，还有开启kibana的编辑模式，panel全屏观看等。

#### dashboards和panel的数据交互

1. 在panel方面，通过dashboard的redux提供的redux来修改dashboard的数据。
2. 在dashboard方面，把dashboard storage data通过lodash的_cloneDeep分发给各个panel，每个panel内部再利用lodash的_isequal来比较新旧data来确定更不更新展示，或者利用redux tree的一部分数据来修改panel的展示。（比如自定义panel的标题等。）
3. 对于dashboard storage data，每个panel都有一个id，然后更新的时候dashboard会把所有panel的id聚合在一起，再加上查询条件，上传给服务器，服务器就返回新的数据，然后通过2来更新panel。当点击save的时候，这些数据就保存为这个dashboard的dashboard storage data。（实际上，kibana实现了如下的包装：savedObjects，savedDashboard，savedSearch等等）

### kibana的其它功能

#### inspector

dashboard板块有这么一个功能：可以通过右飘窗实时查看panel里面的数据。

而从panel到右飘窗之间，kibana封装了一层inspecter，用来处理panel数据并显示到右飘窗上面。这之间的东西又叫做adapter。

目前kibana有2个adapter，一个用来处理visualization里面的数据，一个用来发送http请求（saved searches需要这种处理）。

#### courier

由于从dashboard到es之间的数据获取是异步的，并且有等待时间。所以kibana封装了一层courier来处理requests，比如说设置ß时间间距啊，中断requests，分发fetch事件来更新panels等。

其中把requests queued up的必要性没有看懂～

### D3和Vega

kibana同时使用D3和Vega图表库，其中:

1. D3主要用于常规图表配置，用户在界面UI的配置。
2. Vega主要用于让用户使用json数据的形式配置图表，另外Vega也支持多种数据源，用户可以利用URL的形式导入非ES里面的数据源。

### kibana的基本架构

#### 目录说明

kibana的主要目录如下（其他目录有的没有看懂，有的是服务目录，有的是处理特定业务逻辑的目录，就不研究了）：

```
├── src
│   ├── core_plugins            #核心插件
│   │   ├── kibana              #kibana插件
│   │   ├── elasticsearch       #处理elasticsearch的插件
│   │   ├── table_vis           #处理table图表的插件
│   │   └── ...                 #其他插件
│   ├── ui                      #ui
│   │   ├── public              #主要ui模块
│   │   │   ├── chrome          #chrome浏览器模块
│   │   │   ├── draggable       #拖拽模块
│   │   │   ├── embeddable      #embeddable模块
│   │   │   ├── i18n            #国际化模块
│   │   │   ├── inspector       #inspector模块
│   │   │   ├── private         #ag模块
│   │   │   ├── register        #ag注册模块
│   │   │   ├── vis             #可视化模块
│   │   │   └── ...             #其他模块
│   │   ├── ui_render           #ui的render
│   │   ├── ui_setting          #ui的setting
│   │   └── ...                 #其它ui模块
│   └── test                    #测试
│       ├── functional          #功能测试
│       ├── scripts             #封装的js
│       ├── dev_certs           #封装的授权js
│       └── ...                 #其他
```

从上面可以看到：
1. kibana主要分为core_plugins，ui和test三个模块。
2. kibana自身的视图只是作为一个插件被放在core_plugins里面，kibana就相当于一般小型项目的src或者view文件夹。
3. 这么分的目的是因为，kibana项目太大，对于很多模块都会一层层封装，每一层会作为一个独立的模块，这样模块就太多了，然后就考虑把部分模块开发为插件的形式。

下面对于core_plugins，ui和test三个模块分别进行深入研究。

#### core_plugins

core_plugins中的每一个插件都至少有public文件夹，index.js和package.json。其中public文件夹进行业务处理，index.js进行导出，package.json标注这个插件的名字和版本等信息。

其中index.js的结构是这样的：

```
// 以类似npm module的形式挂载模块
Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.default = function (kibana) {
  return new kibana.Plugin({
    id:
    require:
    config() {}
    init(){}
    uiExports: {}
  })
}
```

#### ui

ui主要包括渲染相关的模块。其中最重要的是modules.js，它里面导出一个uiModules来进行ag模块加载，提取和删除等。

其它一些模块会根据provider和factory来进行封装，其中利用provider来导出，利用factory以工厂模式的形式来创建模块。部分代码示例如下：

```
// 提供导出内容
const noneRequestHandlerProvider = function () {
  return {
    name: 'none',
    handler: function () {
      return new Promise((resolve) => {
        resolve();
      });
    }
  };
};

// 注册这个模块
VisRequestHandlersRegistryProvider.register(noneRequestHandlerProvider);

// 导出
export { noneRequestHandlerProvider };
```

```
// 获取kibana模块
const module = uiModules.get('kibana');

// 以工厂模式的形式封装PersistedState
module.factory('PersistedState', ($injector) => {
  const Private = $injector.get('Private');
  const Events = Private(EventsProvider);

  // Extend PersistedState to override the EmitterClass class with
  // our Angular friendly version.
  return class AngularPersistedState extends PersistedState {
    constructor(value, path) {
      super(value, path, Events);
    }
  };
});
```

#### test

像这种大项目又是怎么进行测试的呢？由于测试的每个模块都与很多模块关联，所以kibana做了如下工作：
1. 自己用node开发了一套测试方法。
2. 把测试过程中用到的一些方法都封装在一起，然后测试的时候就只调用这些方法即可。
3. 所有的测试组件都导出成一个函数，然后先加载测试环境，再加载所有的测试进行测试。(所以这不是单元测试！)
4. 对于其他一些能进行单元测试的模块就用jest进行单元测试。

```
import expect from 'expect.js';

// 接受测试环境的getService和getPageObjects作为参数导入
export default function ({ getService, getPageObjects }) {
  const log = getService('log');
  const PageObjects = getPageObjects(['common', 'visualize']);

  describe('chart types', function () {
    before(function () {
      log.debug('navigateToApp visualize');
      return PageObjects.common.navigateToUrl('visualize', 'new');
    });

    it('should show the correct chart types', async function () {
      const expectedChartTypes = [
        'Area',
        'Controls',
        'Coordinate Map',
        'Data Table',
        'Gauge',
        'Goal',
        'Heat Map',
        'Horizontal Bar',
        'Line',
        'Markdown',
        'Metric',
        'Pie',
        'Region Map',
        'Tag Cloud',
        'Timelion',
        'Vega',
        'Vertical Bar',
        'Visual Builder',
      ];

      // find all the chart types and make sure there all there
      const chartTypes = await PageObjects.visualize.getChartTypes();
      log.debug('returned chart types = ' + chartTypes);
      log.debug('expected chart types = ' + expectedChartTypes);
      expect(chartTypes).to.eql(expectedChartTypes);
    });
  });
}
```

#### 和vue项目对比

一般vue项目的目录结构是这样的：

```
├── src
│   ├── api              #api
│   ├── assets           #图片等资源
│   ├── components       #组件
│   ├── i18n             #国际化
│   ├── plugins          #插件
│   ├── router           #路由
│   ├── store            #store
│   ├── styles           #样式
│   ├── utils            #工具
│   ├── views            #视图
│   │   ├── admin        #管理界面
│   │   ├── auth         #权限界面
│   │   ├── layouts      #视图布局
│   │   └── ...          #其他
```

对于小项目来说，上面的目录结构已经足够了，并且比kibana的结构更加清晰(个人认为，kibana的目录结构有点混乱，可能是因为kibana迭代过很多次的原因)。但是随着项目的增大，可以考虑参考kibana的目录结构。

另外，对于vue项目，组件一般都是.vue后缀的单文件组件，所以如果想要在这之上对组件进行封装的话，怎么办？mixin和vue.extend了解一下。

### 声明

以上纯属个人理解，由于我自己水平和时间有限，一些理解错误在所难免，欢迎提出并一起讨论。

感受：
1. 有些项目就算把源码给你看，你也不一定看得懂。(挥泪~)
2. 能架构kibana这种项目的才能叫架构师嘛~

### 参考资料

[Dashboard State Walkthrough](https://github.com/elastic/kibana/tree/108d77d6cd89b52c7b5bfe240e7ad3d4274dd5fb/src/legacy/core_plugins/kibana/public/dashboard)

[Discover Context App Implementation Notes](https://github.com/elastic/kibana/blob/b2576f5a30f8cc4de87f5f77ed3b7e99e311128b/src/legacy/core_plugins/kibana/public/context/NOTES.md)

[Discover Context App Implementation Notes](https://github.com/elastic/kibana/blob/master/src/legacy/core_plugins/kibana/public/context/NOTES.md)

[Inspector](https://github.com/elastic/kibana/tree/a72bd039e639bda58a34833ad49fc1721add07d4/src/ui/public/inspector)

[unify the way global context is passed down to visualize](https://github.com/elastic/kibana/issues/16641)

[[Meta] Improve support for custom embeddable configurations at the dashboard panel level](https://github.com/elastic/kibana/issues/19601)

[Embeddables API](https://github.com/elastic/kibana/issues/19875)

[vega-vs-vegalite](https://www.elastic.co/guide/en/kibana/master/vega-vs-vegalite.html)

[Vislib general overview](https://github.com/elastic/kibana/blob/master/src/ui/public/vislib/VISLIB.md)





