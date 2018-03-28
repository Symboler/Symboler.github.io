---
title: dva的基本使用以及核心源码分析
date: 2018-03-22 20:02:01
tags:
---
## dva基本思想 ##
dva是对redux、redux-router和redux-saga进行了封装，未引入其他新的概念。在处理redux异步控制上实现了优化。按照作者sorrycc的定义：dva 是 react 和 redux 的最佳实践。
## dva的demo分析 ##
语言基础：es6和react
主要工具：

- 开发工具（vscode，subline）
- 包管理工具（npm）
- es6编译工具（babel https://my.oschina.net/dkvirus/blog/1517941）
- 项目启动打包工具（Roadhog https://github.com/sorrycc/roadhog）
- 模拟后台接口（mockjs）
- 版本控制工具（git）
- 代码检查工具（ESLint https://my.oschina.net/dkvirus/blog/1506899）

项目结构：
![](https://i.imgur.com/PYfnS2T.png)
脚手架做的几件事情：
- 自动创建一个包含package.json的项目
- 自动创建成体系的目录结构
- 自动安装项目需要的基础包
- 集成代码检查工具ESLint
- 集成模拟接口工具mock
- 集成服务启动打包工具Roadhog
- 集成版本控制工具Git

### 各文件目录主要的作用 ###
- package.json 项目的配置文件
- public 文件夹中存放静态资源，比如图片，静态页面html等
- src 文件夹是项目代码存放文件夹，是项目最核心的代码，具体在后面进行详细介绍。
- node_modules 放置所有项目引入的依赖包
- .eslintrc 代码检查的配置文件
- mock 文件夹、.roadhogrc.mock.js文件 是项目中的模拟接口
- .roadhogrc 集成服务启动打包工具Roadhog（主要作用：与其他插件集成、启动服务、打包项目）
- .gitingore 配置git相关设置

### 具体的代码分析 ###
#### 从入口文件index.js开始 ####
初始化：`const app = dva({
    history: browserHistory
});`

    function dva(hooks={}){
		const history = hooks.history || defaultHistory;
		const initialState = hooks.initialState || {};
		delete hooks.history;
		delete hooks.initialState;

		const plugin = new Plugin();
		plugin.use(hooks);

		const app = {
			//properties
			_models:[],
			_router:null,
			_history:null,
			_plugin:plugin,
			_getProvider:null,
			//methods
			use,
			models,
			router,
			start,
		};
		return app;
	}

hooks:传入的配置，如history，这里可以知道dva默认采用的是hashHistory；
plugin:插件，此处暂不分析；
app.router()：指定路由，需要传入一个函数，一般类似于({history})=>(<Router>...</Router>);
app.use():添加插件；
app.model():添加model，也就是对应的添加一个store下的数据，该方法做的就是对传入的model进行检查，对reducers添加命名空间，而后将其push到_models中
app.start():初始化应用，接受参数为选择器或者dom节点






