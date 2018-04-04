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
需要注意的是： 
- reducers和effects的key不需要用namespace/action的形式了，因为dva会自动将其加上，dispatch的时候，saga需要加上namespace，而saga中的put不需要加入namespace，原因是dva对put进行了重载


### start函数 ###
    function start(container) {
    // 允许 container 是字符串，然后用 querySelector 找元素
    	if (isString(container)) {
      		container = document.querySelector(container);
      		invariant(
        	  container,
        	`[app.start] container ${container} not found`,
      		);
    	}

    // 并且是 HTMLElement
    	invariant(
      		!container || isHTMLElement(container),
      		`[app.start] container should be HTMLElement`,
    	);

    // 路由必须提前注册
    	invariant(
      		app._router,
      		`[app.start] router must be registered before app.start()`,
    	);

    	if (!app._store) {
      		oldAppStart.call(app);
    	}
    	const store = app._store;

    // export _getProvider for HMR
    // ref: https://github.com/dvajs/dva/issues/469
    	app._getProvider = getProvider.bind(null, store, app);

    // If has container, render; else, return react component
    	**if (container) {
      		render(container, store, app, app._router);
      		app._plugin.apply('onHmr')(render.bind(null, container, store, app));
    	} else {
      	return getProvider(store, this, this._router);
    	}**
  	}

比较关心的页面初始渲染的函数
    
	function render(container, store, app, router) {
  		const ReactDOM = require('react-dom');  // eslint-disable-line
  		ReactDOM.render(React.createElement(getProvider(store, app, router)), container);
	}

比较重要的路由模块
  `function getProvider(store, app, router) {
  		return extraProps => (
    	<Provider store={store}>
      	{ router({ app, history: app._history, ...extraProps }) }
    	</Provider>
  	)；
	}`

此时建议看一下一般项目中的入口文件index.js和路由文件router.js

index.js
   	
	`// 1. Initialize
		const app = dva({
  			history: browserHistory(),
		});
	// 2. Plugins
	// app.use({});

	// 3. Register global model
	app.model(require('./models/global'));

	// 4. Router
	app.router(router);

	// 5. Start
	app.start('#root');

	export const store = app._store;`

router.js

    `function RouterConfig({ history, app }) {
  		const navData = getNavData(app);
  		const BasicLayout = getLayout(navData, 'BasicLayout').component;

  		const passProps = {
    		app,
    		navData,
    		getRouteData: (path) => {
      		return getRouteData(navData, path);
    		},
  		};

  		return (
    		<LocaleProvider locale={zhCN}>
      			<Router history={history}>
        			<Switch>
          				<Route path="/" render={props => <BasicLayout {...props} {...passProps} />} />
        			</Switch>
      			</Router>
    		</LocaleProvider>
  			);
	}

	export default RouterConfig;`

以上可以基本明白一个dva的初始化页面是如何实现的，需要注意的是：
1、LocaleProvider是antd提供的国际化组件
2、Provider是react-redux提供的组件，其本质是一个react组件，具体参看源码，其核心是getChildContext方法
 
	`import { Component, Children } from 'react'
	 import PropTypes from 'prop-types'
	 import storeShape from '../utils/storeShape'
	 import warning from '../utils/warning'

	export default class Provider extends Component {
  		getChildContext() {
    	return { store: this.store }
  	}

  	constructor(props, context) {
    	super(props, context)
    	this.store = props.store
  	}

  	render() {
    	return Children.only(this.props.children)
  		}
	}`
接下来是dva的核心部分，就是如何完成状态管理，必须看明白model的作用

	` /**
   	* Register model before app is started.
   	*
   	* @param m {Object} model to register
   	*/
  	function model(m) {
		//开发模式下，需要对model的格式进行检查
    	if (process.env.NODE_ENV !== 'production') {
      	checkModel(m, app._models);
    	}
		//通过model方法将model注入_models属性，把 reducer, initialState, action, saga 封装到一起
   	 app._models.push(prefixNamespace(m));
  	}`

    `function prefix(obj, namespace, type) {
  		return Object.keys(obj).reduce((memo, key) => {
    		warning(
      		key.indexOf(`${namespace}${NAMESPACE_SEP}`) !== 0,
      		`[prefixNamespace]: ${type} ${key} should not be prefixed with namespace ${namespace}`,
    		);
    		const newKey = `${namespace}${NAMESPACE_SEP}${key}`;
    		memo[newKey] = obj[key];
    		return memo;
  		}, {});
	}

	export default function prefixNamespace(model) {
  		const {
    		namespace,
    		reducers,
    		effects,
  		} = model;

  	if (reducers) {
    	if (isArray(reducers)) {
      		model.reducers[0] = prefix(reducers[0], namespace, 'reducer');
    	} else {
      		model.reducers = prefix(reducers, namespace, 'reducer');
    	}
  	}
  	if (effects) {
    	model.effects = prefix(effects, namespace, 'effect');
  	}
  	return model;
	}
	import warning from 'warning';
	import { isArray } from './utils';//const isArray = Array.isArray.bind(Array);
	import { NAMESPACE_SEP } from './constants';//const NAMESPACE_SEP = '/';

	function prefix(obj, namespace, type) {
  		return Object.keys(obj).reduce((memo, key) => {
    		warning(
      		key.indexOf(`${namespace}${NAMESPACE_SEP}`) !== 0,
      		`[prefixNamespace]: ${type} ${key} should not be prefixed with namespace ${namespace}`,
    	);
    	const newKey = `${namespace}${NAMESPACE_SEP}${key}`;
    	memo[newKey] = obj[key];
    	return memo;
  	}, {});
	}

	export default function prefixNamespace(model) {
  		const {
    		namespace,
    		reducers,
    		effects,
  		} = model;

  		if (reducers) {
		//此处是将所有的reducers和effects添加为完整路径，前面加上"namespace/**"
    		if (isArray(reducers)) {
      		model.reducers[0] = prefix(reducers[0], namespace, 'reducer');
    		} else {
      		model.reducers = prefix(reducers, namespace, 'reducer');
    		}
  		}
  		if (effects) {
    		model.effects = prefix(effects, namespace, 'effect');
  		}
  		return model;
	}`

此处可结合model的实例进行分析：

	`import { queryNotices } from 'Services/notices';
	 import { getWebCig } from 'Services/global';
	 import { webCig, headerMenu } from '../common/constants/initState.js';

	 export default {
  		namespace: 'global',

  		state: {
    		notices: [],
    		fetchingNotices: false,
    		webCig,
    		headerMenu: headerMenu,
  		},

  		effects: {
    		*fetchNotices({ query }, { call, put }) {
      			yield put({
        			type: 'changeNoticeLoading',
        			payload: true,
      			});
      			const res = yield call(queryNotices, query);
      			yield put({
        			type: 'saveNotices',
        			payload: res.data.list,
      			});
    		},
    		*clearNotices({ payload }, { put, select }) {
      			const count = yield select(state => state.global.notices.length);
      			yield put({
        			type: 'user/changeNotifyCount',
        			payload: count,
      			});

      			yield put({
        			type: 'saveClearedNotices',
        			payload,
      			});
    		},
    		*fetchWebCig(_, { call, put }) {
      			try {
        			const res = yield call(getWebCig);
        			if (res.status === 'success') {
          				yield put({
            				type: 'saveWebCig',
            				payload: res.data,
          				});
        			}
      			} catch(e) {

      			}
    		},
  	},

  	reducers: {
    	saveNotices(state, { payload }) {
      	return {
        	...state,
        	notices: payload,
        	fetchingNotices: false,
      	};
    	},
    	saveClearedNotices(state, { payload }) {
      	return {
        	...state,
        	notices: state.notices.filter(item => item.type !== payload),
      	};
    	},
    	changeNoticeLoading(state, { payload }) {
      	return {
        	...state,
        	fetchingNotices: payload,
      	};
    	},
    	cleanNotices(state, _) {
      	return {
        	...state,
        	notices: [],
      	};
    	},
    	saveWebCig(state, { payload }) {
      	return {
        	...state,
        	webCig: payload,
      	};
    	},
  	},

  	subscriptions: {
    	setup({ history }) {
      	// Subscribe history(url) change, trigger `load` action if pathname is `/`
      	return history.listen(({ pathname, search }) => {
        	if (typeof window.ga !== 'undefined') {
          		window.ga('send', 'pageview', pathname + search);
        	}
      	});
    	},
  	},
	};`

关注一下dva里面的store来源

	`const store = app._store = createStore({ // eslint-disable-line
      		reducers: createReducer(),
      		initialState: hooksAndOpts.initialState || {},
      		plugin,
      		createOpts,
      		sagaMiddleware,
      		promiseMiddleware,
    	});
	这里的createStore是对redux的createStore方法进行了扩展封装
	import { createStore, applyMiddleware, compose } from 'redux';
	import flatten from 'flatten';
	import invariant from 'invariant';
	import window from 'global/window';
	import { returnSelf, isArray } from './utils';

	export default function ({
  	reducers,
  	initialState,
  	plugin,
  	sagaMiddleware,
  	promiseMiddleware,
  	createOpts: {
    	setupMiddlewares = returnSelf,
  	},
	}) {
  		// extra enhancers
  		const extraEnhancers = plugin.get('extraEnhancers');
  		invariant(
    		isArray(extraEnhancers),
    		`[app.start] extraEnhancers should be array, but got ${typeof extraEnhancers}`,
  		);

  	const extraMiddlewares = plugin.get('onAction');
  	const middlewares = setupMiddlewares([
    	sagaMiddleware,
    	promiseMiddleware,
    	...flatten(extraMiddlewares),
  	]);

  	let devtools = () => noop => noop;
  	if (process.env.NODE_ENV !== 'production' && 	window.__REDUX_DEVTOOLS_EXTENSION__) {
    	devtools = window.__REDUX_DEVTOOLS_EXTENSION__;
  	}

  	const enhancers = [
    	applyMiddleware(...middlewares),
    	...extraEnhancers,
    	devtools(window.__REDUX_DEVTOOLS_EXTENSION__OPTIONS),
  	];

  	return createStore(reducers, initialState, compose(...enhancers));
	}`

关于redux的部分可以深入了解https://zhuanlan.zhihu.com/p/22809799，http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html 我们这里不再展开

flux———redux————react-redux————sage————dva




