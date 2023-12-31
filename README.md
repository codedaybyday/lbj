# 刘北京

- 男/1994
- 擅长：工程效率/富文本
- 本科/湖南文理学院/网络工程
- 工作年限：5年
- Email: 1573018715@qq.com
- Github: https://github.com/codedaybyday

- 期望城市：北京

------

# 工作经历

## 美团（办公效率）

### 虚拟滚动重构

#### 背景

1. 大象(内部通讯工具)中每个会话消息过多导致页面dom节点过多，影响到渲染速度。尤其是当到2000+dom节点的时候卡顿明显
2. 虚拟滚动采用react-virtualized，存在一些隐藏的坑，且目前没有在持续维护，属于历史问题
3. 大象本身架构上的问题：消息同步到视图链路太长/组件性能问题/影响面广
4. 项目启动时有外部依赖：im-sdk正在接入大象
5. 虚拟滚动支持搜索消息定位到气泡页，基于本次的需求下对虚拟滚动进行重构

#### 难点

1. 改动大：修改了大象消息发送/在线收取/拉取历史核心链路，并且需要对消息的存储层有改动
2. 消息断层：消息跳转到任意位置时,由于历史消息是基于接口拉取（每次20条），可能出现断层问题，导致消息丢失
3. 并发问题处理：大象请求消息太快（频繁切换）没有中断请求的方法，会出现浪费请求并且有并发问题，导致不同群消息串群
4. 虚拟滚动不定高：易导致消息重叠，需要及时更新高度缓存，以及考虑什么时机更新缓存，并重新渲染的问题
5. 性能：消息存在较多异步更新情况（异步请求/socket长连接推送）,导致会频繁更新
6. 滚动位置准确性：滚动位置可能不准确，尤其是滚动到底部的时候（比如切换会话时两个会话的消息长度/scrollToRow/height不发生变化时并不会触发scrollTop的更新）
7. react-virtualized黑盒：对react-virtualized使用很多时候停留在表面api,对源码底层不清楚，不利于后面的怪异问题的定位和性能优化
8. 滑动过快：未读数数统计不准确/intersection-observer触发加载更多"失灵"

#### 措施

1. 改动大问题：前期经过多轮的技术评审和讨论，控制修改后后期经过灰度机制避免出现大范围问题
2. 断层处理：
    1. 与im-sdk达成存储规范：首屏数据存入im-sdk，其他情况im-sdk只负责拉取数据不存储（redux负责存储）
    2. 合并排序：后续如跳转到最新位置会先和在线收到的消息合并并排序，避免在线收到的消息丢失
3. 并发问题：并发问题请求消息由于依赖im-sdk，暂时没有提供中断的方法，通过实现一个promiseAbortController实现promise执行链的中断，可避免并发时导致的串群
4. 不定高问题：需要及时清除列表的高度缓存,收敛几种高度变化的时机：切换会话/发生跳转/socket长连推送/加载更多等
5. 更新频繁问题：
    1. 新加第三道缓存：模拟useMemo的思路，可以在类组件使用Memorizer,重复渲染的部分将会命中缓存
    2. 优化：Memorizer中使用LRU缓存，使得高频出现的内容能够最大限度的进行缓存并且不会无限占用内存
6. 滚动位置准确性：通过patch-package修改react-virtualized源码，新增onScrollToRowDidUpdate和onScrollToRowWillUpdate的生命钩子,目的是能在切换会话后准确将scrollToRow重置，清除副作用（通过scrollToRow属性实现的消息跳转在多会话的场景下有副作用）
7. react-virtualized黑盒：阅读react-virtualized源码，了解列表渲染，滚动的相关逻辑，梳理相关的流程图
8. 滑动过快：
   1. 计算滑动期间有多少消息被已读：不再采用直接暴力的方式统计当前可视区域有多少条消息，而是采用本次滑动可视区域第一条消息的索引-上一次可视区域第一条索引
   2. intersection-observer采用单例模式，每条消息订阅observer时往intersection-observer的"回调池"中push回调，销毁时删除对对应回调，避免了每条消息都会observe一次，造成的资源浪费


#### 收益

1. 性能提升: 通过实现useMemo,减少50%的render次数，经过测试cpu峰值占用减少50%
2. 业务赋能: 开放了公共的api，后续支持了搜索消息跳转到任意位置的能力
3. 流畅度：解决会话中2000+消息时，页面卡顿问题。丢帧率由23%=>10%
4. 组内分享: 从源码实现上以及业务自身的角度进行了一次分享，介绍了虚拟滚动的实现以及遇到的一些问题

### 卡片消息重构

#### 背景

1. 开放：大象卡片消息计划对业务方开放
2. 调试：需要卡片消息配置后台的支持，方便接入方快速调试卡片的样式

#### 难点

1. 低代码协议需要升级，并且需要考虑向下兼容以及三端（pc/andriod/ios）一致性问题
2. 卡片消息组件后续需要在卡片消息配置后台中接入，所以需要对大象中的卡片进行重构，并从大象现有的代码中抽离出来
3. 卡片消息配置后台需要从0搭建，跟后端接口通过thrift通讯，对前端人员来说是个巨大的挑战

#### 措施

1. 参与低代码协议的设计，考虑到重构时需新增新功能，对协议版本进行升级
2. 从0-1基于rollup+ts+hooks搭建大象卡片消息sdk，具体实现分为协议层，代理层，渲染层启动入口，通过依赖注入的方式实现大象pc端宿主环境和sdk的解耦,并支持实现自定义视图层
3. 从0-1基于node+koa+thrift搭建的大象卡片消息配置后台（低代码）

#### 收益

1. 沉淀一套基于卡片消息可扩展可迭代的sdk,目前在两个平台接入
2. 大象卡片消息后台可给业务方接入时提供快速调试的平台，提供接入的效率，预计可减少3-4pd的人力

## 百度 (2017.7-2021.8,带8人)

### 百家号灰度平台

#### 背景

上线重⼤项目时缺乏灰度机制，加上前端模块上线排队现象严重，⼀旦出现问题不能快速⽌损

#### 措施

1. 搭建百家号的灰度平台，后台通过白名单和百分⽐的方式进行放量，
2. 部署层⾯通过脚本部署两套代码，正常分⽀每次上线会⾃动带上灰度产出，避免每次需要⼿动上线灰度
3. 结合百度内部agile平台，新建graypipeline流水线，灰度发布需要经过权限验证，避免开发⼈员随意发布，影响灰度产出

#### 收益

1. 平均缩短上线时⻓40分钟
1. 灰度线上问题可达到分钟级别止损

### **global-less-loader**

#### 背景

1. 百家号主题UI需要进行改版和主题⾊升级
2. 通过less变量收敛主题配置，需要将所有的主题⾊替换成新的主题⾊变量。由于历史遗留原因，多数less⽂件都是直接⽤的颜⾊值，维护难度大。

#### 难点

1. 在于批量替换成变量后，需要手动引⼊var.less等相关的依赖库文件，⼯作量巨⼤
2. 手动引入依赖文件后如何避免团队中less导入代码风格不一致问题
3. Var.less文件自动加载后避免重复打包问题

#### 措施

1. 通过⾃定义loader的⽅式实现了了⾃动加载less依赖文件
2. 通过实现⼿写stylelint 代码⻛格检测插件保持团队代码⻛格的统⼀

#### 收益

1. 通过自动加载less依赖文件，优化了之前less依赖⽂件会重复打包的问题
1. 通过stylint以及制定规范，限制了组内其他人员的代码风格，防止代码劣化

### **bc**端富⽂本样式打通

#### 背景

该项目是对百家号和⼿机百度跨部⻔数据流的升级，升级后⽀持了百家号作者富⽂本的发⽂能力

#### 难点

1. 跨部分沟通协调：百家号到手机百度之间需要经过审核,图片提取，物料库等多个环节，富文本格式一旦有修改，需要下游各个环节的配合联调
2. 需要对编辑器的底层进行修改，如粘贴板功能
3. 历史遗留原因以及之前开发人员使用不当，导致后期ueditor更新富文本存在大量的dom操作，存在性能问题
4. 由于ueditor架构原因，加上修改了编辑器底层，届时会存在两份编辑器实例，需要对编辑器实例上下文进行隔离

#### 措施

1. 协调：前期立项/调研，提前跟下游沟通，通知改动点/影响面，确定各个环节需要配合的时间节点和负责人
2. 升级：对百家号编辑器的升级，对ueditor做二次开发，封装常⽤用插件，防⽌止 XSS/scrf等安全问题，
3. 优化：采⽤用虚拟dom的方式提升编辑器的流畅度，并沉淀出⼀套基于ueditor虚拟dom操作的引擎ueditor-quey，模拟webpack中loader的思想将编辑器预处理器抽象成各种loader
4. 构建插件实现：实现一个babel插件，在编译时将产出两份ueditor实例，确保线上环境能存在两个编辑器，并且互不冲突

### 前端模块打包优化

#### 背景

项目庞大，前端开发时项目启动较慢，影响开发效率

#### 措施

1. 编译时打包优化，如dll处理，多进程happypack处理,减少webpack寻址范围
2. 运行时优化，如懒加载处理
3. 流水线缓存处理:结合内部平台icode对node_module包的缓存，减少上线前代码在平台上的编译时间

#### 收益

1. 打包速度提升40%
2. 平台编译速度提高50%
3. 首屏加载时间减少2s左右

### 活动平台（低代码）

#### 背景

由于百家号运营活动较多，页⾯格式⽐较固定，经常需要前端⼈力切⻚面，比较耗费人力。因此设计了了一个活动平台， 旨在减少前端人员的重复⼯作

#### 措施

1. 参与toc，toB的整体设计，实现一个运营人员可以通过拖拽就能配置的活动页面的低代码平台
2. 负责toB的搭建以及常用的组件的封装如分享，轮播，tab组件，手百sdk的接入，度口令

#### 收益

1. 每配置一个活动页面可节省2pd的人力
2. 可以支持活动页面的快速上线下线

### 百家号脚手架

#### 背景

1. 前端新项目较多，大部分项目的配置环境有重复，每次新建项目存在大量复制操作
2. 前端生产环境缺乏一套统一的持续维护的机制（未收敛）

#### 措施

1. 搭建一套脚手架，提供基本的命令行交互
2. 收敛了webpack优化后的配置/百家号自有的loader和插件/scp、csrf安全问题处理

#### 收益

1. 新项目可避免重复复制粘贴工作，可节省3pd左右的人力

# 开源项目和作品

+ [simple-react](https://github.com/codedaybyday/simple-react):个简单的react轮⼦
+ [react-redux/redux](https://github.com/codedaybyday/react-redux-demo):redux和react-redux实现

# 技能清单

+ 熟悉react+redux+react-router,并写过react轮⼦
+ 掌握webpack，了解前端编译打包原理，⼿写过webpack插件和loader
+ 熟悉富⽂本编辑器开发
+ 掌握原⽣JavaScript，熟悉ajax原理和机制, 熟悉ES6
+ 熟悉 HTML5 +CSS3.0，熟练掌握响应式布局方式，熟悉各⼤主流浏览器的兼容性差异
+ 熟悉less动态样式语言, 熟悉moduleCss用来提升开发体验
+ 熟悉Git版本控制⼯具