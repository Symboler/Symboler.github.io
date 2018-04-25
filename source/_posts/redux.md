---
title: flux->redux->redux-saga->dva说明
date: 2018-04-25 16:58:48
tags:
---
## flux
简单说，Flux 是一种架构思想，专门解决软件的结构问题。它跟MVC 架构是同一类东西，但是更加简单和清晰
首先，Flux将一个应用分成四个部分。
View： 视图层
Action（动作）：视图层发出的消息（比如mouseClick）
Dispatcher（派发器）：用来接收Actions、执行回调函数
Store（数据层）：用来存放应用的状态，一旦发生变动，就提醒Views要更新页面
Flux 的最大特点，就是数据的"单向流动"。
用户访问 View
View 发出用户的 Action
Dispatcher 收到 Action，要求 Store 进行相应的更新
Store 更新后，发出一个"change"事件
View 收到"change"事件后，更新页面
## redux
### 设计思想
（1）Web 应用是一个状态机，视图与状态是一一对应的。

（2）所有的状态，保存在一个对象里面。

### API
1、Store

Store 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 Store。

Redux 提供createStore这个函数，用来生成 Store。
	`

	import { createStore } from 'redux';
	const store = createStore(fn,initial_state,applyMiddleware());
	//后面会知道这个fn就是reducer,initial_state是初始状态，applyMiddleware是Redux的原生方法，作用是将所有中间件组成一个数组，依次执行`

2、state
Store对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。

当前时刻的 State，可以通过store.getState()拿到。

    `
	import { createStore } from 'redux';
	const store = createStore(fn);
	const state = store.getState();`

3、Action
state 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。

Action 是一个对象。其中的type属性是必须的，表示 Action 的名称。

    `
	const action = {
  		type: 'ADD_TODO',
  		payload: 'Learn Redux'
	};`

4、store.dispatch()
store.dispatch()是 View 发出 Action 的唯一方法。
    `

	import { createStore } from 'redux';
	const store = createStore(fn);

	store.dispatch({
  		type: 'ADD_TODO',
  		payload: 'Learn Redux'
	});`

5、Reducer

Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。

Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。

    `
	const reducer = function (state, action) {
  	// ...
  	return new_state;
	};

	const defaultState = 0;
	const reducer = (state = defaultState, action) => {
  		switch (action.type) {
    		case 'ADD':
      		return state + action.payload;
    		default:
      		return state;
  		}
	};

	const state = reducer(1, {
  		type: 'ADD',
  		payload: 2
	});`

实际应用中，Reducer 函数不用像上面这样手动调用，store.dispatch方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入createStore方法。
    `

	import { createStore } from 'redux';
	const store = createStore(reducer);`

6、 store.subscribe()
	Store 允许使用store.subscribe方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。
    `

	import { createStore } from 'redux';
	const store = createStore(reducer);

	store.subscribe(listener);//结合react，可以对render进行监听`

7、Store的实现

store提供了三个方法
store.getState()
store.dispatch()
store.subscribe()

简单实现原理
    `

	const createStore = (reducer) => {
  	let state;
  	let listeners = [];

  	const getState = () => state;

  	const dispatch = (action) => {
    	state = reducer(state, action);
    	listeners.forEach(listener => listener());
  	};

  	const subscribe = (listener) => {
    	listeners.push(listener);
    	return () => {
      		listeners = listeners.filter(l => l !== listener);
    	}
  	};

  	dispatch({});

  	return { getState, dispatch, subscribe };
	};`

以上简单说明了redux的工作原理，但是会发现一个问题，现在的action都是同步操作，那么redux如何处理异步的action，于是产生了中间件的概念，也就是对dispatch过程进行了处理，实现异步操作

异步操作的基本思路
同步操作只要发出一种 Action 即可，异步操作的差别是它要发出三种 Action
操作发起时的 Action
操作成功时的 Action
操作失败时的 Action

操作开始时，送出一个 Action，触发 State 更新为"正在操作"状态，View 重新渲染
操作结束后，再送出一个 Action，触发 State 更新为"操作结束"状态，View 再一次重新渲染

异步操作至少要送出两个 Action：用户触发第一个 Action，这个跟同步操作一样，没有问题；如何才能在操作结束时，系统自动送出第二个 Action 呢？

    `
	const fetchPosts = postTitle => (dispatch, getState) => {
  		dispatch(requestPosts(postTitle));
  		return fetch(`/some/API/${postTitle}.json`)
    			.then(response => response.json())
    			.then(json => dispatch(receivePosts(postTitle, json)));
  		};
	};
	store.dispatch(fetchPosts('reactjs'));
	上面代码中，fetchPosts是一个Action Creator（动作生成器），返回一个函数。这个函数执行后，先发出一个Action（requestPosts(postTitle)），然后进行异步操作。拿到结果后，先将结果转成 JSON 格式，然后再发出一个 Action（ receivePosts(postTitle, json)）。
（1）fetchPosts返回了一个函数，而普通的 Action Creator 默认返回一个对象。

（2）返回的函数的参数是dispatch和getState这两个 Redux 方法，普通的 Action Creator 的参数是 Action 的内容。

（3）在返回的函数之中，先发出一个 Action（requestPosts(postTitle)），表示操作开始。

（4）异步操作结束之后，再发出一个 Action（receivePosts(postTitle, json)），表示操作结束。

这样的处理，就解决了自动发送第二个 Action 的问题。但是，又带来了一个新的问题，Action 是由store.dispatch方法发送的。而store.dispatch方法正常情况下，参数只能是对象，不能是函数。
所以就需要使用中间件（redux-thunk），改造store.dispatch，使得后者可以接受函数作为参数。

因此，异步操作的第一种解决方案就是，写出一个返回函数的 Action Creator，然后使用redux-thunk中间件改造store.dispatch

既然 Action Creator 可以返回函数，当然也可以返回其他值。另一种异步操作的解决方案，就是让 Action Creator 返回一个 Promise 对象。

这就需要使用redux-promise中间件。

### react-redux
React-Redux 将所有组件分成两大类：UI 组件（presentational component）和容器组件（container component）。

UI 组件有以下几个特征。
只负责 UI 的呈现，不带有任何业务逻辑
没有状态（即不使用this.state这个变量）
所有数据都由参数（this.props）提供
不使用任何 Redux 的 API

容器组件的特征恰恰相反。
负责管理数据和业务逻辑，不负责 UI 的呈现
带有内部状态
使用 Redux 的 API

    `
		import { connect } from 'react-redux'
		const VisibleTodoList = connect(
  			mapStateToProps,
 		 	mapDispatchToProps
		)(TodoList)`

mapStateToProps是一个函数。它的作用就是像它的名字那样，建立一个从（外部的）state对象到（UI 组件的）props对象的映射关系。
		`

	const mapStateToProps = (state) => {
  		return {
    		todos: getVisibleTodos(state.todos, state.visibilityFilter)
  		}
	}`

mapStateToProps会订阅 Store，每当state更新的时候，就会自动执行，重新计算 UI 组件的参数，从而触发 UI 组件的重新渲染。

mapStateToProps的第一个参数总是state对象，还可以使用第二个参数，代表容器组件的props对象。

mapDispatchToProps是connect函数的第二个参数，用来建立 UI 组件的参数到store.dispatch方法的映射。也就是说，它定义了哪些用户的操作应该当作 Action，传给 Store。它可以是一个函数，也可以是一个对象。

如果mapDispatchToProps是一个函数，会得到dispatch和ownProps（容器组件的props对象）两个参数。
	`

	const mapDispatchToProps = (
  		dispatch,
  		ownProps
	) => {
  		return {
    		onClick: () => {
      			dispatch({
        			type: 'SET_VISIBILITY_FILTER',
        			filter: ownProps.filter
      			});
    		}
  		};
	}`

React-Redux 提供Provider组件，可以让容器组件拿到state。

	`
		import { Provider } from 'react-redux'
		import { createStore } from 'redux'
		import todoApp from './reducers'
		import App from './components/App'

		let store = createStore(todoApp);

		render(
  			<Provider store={store}>
    			<App />
  			</Provider>,
  		document.getElementById('root')
		)`

上面代码中，Provider在根组件外面包了一层，这样一来，App的所有子组件就默认都可以拿到state了。

它的原理是React组件的context属性，请看源码。
	`

		class Provider extends Component {
  		getChildContext() {
    		return {
      			store: this.props.store
    		};
  		}
  		render() {
    		return this.props.children;
  		}
		}

		Provider.childContextTypes = {
  			store: React.PropTypes.object
		}`

## redux-saga
redux-saga的实质是类似redux-thunk的中间件，作用是为redux提供额外的功能
其次，我们都知道，在 reducers 中的所有操作都是同步的并且是纯粹的，即 reducer 都是纯函数，纯函数是指一个函数的返回结果只依赖于它的参数，并且在执行过程中不会对外部产生副作用，即给它传什么，就吐出什么。但是在实际的应用开发中，我们希望做一些异步的（如Ajax请求）且不纯粹的操作（如改变外部的状态），这些在函数式编程范式中被称为“副作用”。

Redux 的作者将这些副作用的处理通过提供中间件的方式让开发者自行选择进行实现。

redux-thunk 和 redux-saga 是 redux 应用中最常用的两种异步流处理方式。

### redux-thunk
redux-thunk 的任务执行方式是从 UI 组件直接触发任务。redux-thunk 的主要思想是扩展 action，使得 action 从一个对象变成一个函数。
    `

	// fetchUrl 返回一个 thunk
	function fetchUrl(url) {
  		return (dispatch) => {
    		dispatch({
      			type: 'FETCH_REQUEST'
    		});

    	fetch(url).then(data => dispatch({
      		type: 'FETCH_SUCCESS',
      		data
    	}));
  		}
	}

	// 如果 thunk 中间件正在运行的话，我们可以 dispatch 上述函数如下：
	dispatch(
  		fetchUrl(url)
	):
`
redux-thunk 的缺点：
（1）action 虽然扩展了，但因此变得复杂，后期可维护性降低；
（2）thunks 内部测试逻辑比较困难，需要mock所有的触发函数；
（3）协调并发任务比较困难，当自己的 action 调用了别人的 action，别人的 action 发生改动，则需要自己主动修改；
（4）业务逻辑会散布在不同的地方：启动的模块，组件以及thunks内部。

### redux-saga
sages 采用 Generator 函数来 yield Effects（包含指令的文本对象）。Generator 函数的作用是可以暂停执行，再次执行的时候从上次暂停的地方继续执行。Effect 是一个简单的对象，该对象包含了一些给 middleware 解释执行的信息。你可以通过使用 effects API 如 fork，call，take，put，cancel 等来创建 Effect。
与 redux-thunk 不同的是，在 redux-saga 中，UI 组件自身从来不会触发任务，它们总是会 dispatch 一个 action 来通知在 UI 中哪些地方发生了改变，而不需要对 action 进行修改。redux-saga 将异步任务进行了集中处理，且方便测试。
所有的东西都必须被封装在 sagas 中。sagas 包含3个部分，用于联合执行任务：
1、worker saga
做所有的工作，如调用 API，进行异步请求，并且获得返回结果
2、watcher saga
监听被 dispatch 的 actions，当接收到 action 或者知道其被触发时，调用 worker saga 执行任务
3、root saga
立即启动 sagas 的唯一入口
具体使用姿势：
    `

	class UserComponent extends React.Component {
 	 ...
  		onSomeButtonClicked() {
    		const { userId, dispatch } = this.props
    		dispatch({type: 'USER_FETCH_REQUESTED', payload: {userId}})
  		}
  	...
	}
	//sagas.js

	import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
	import Api from '...'

	// worker Saga: will be fired on USER_FETCH_REQUESTED actions
	function* fetchUser(action) {
   		try {
      		const user = yield call(Api.fetchUser, action.payload.userId);
      		yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   		} catch (e) {
      		yield put({type: "USER_FETCH_FAILED", message: e.message});
   		}
	}

	/*
  	Starts fetchUser on each dispatched `USER_FETCH_REQUESTED` action.
  	Allows concurrent fetches of user.
	*/
	function* mySaga() {
  		yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
	}

	/*
  	Alternatively you may use takeLatest.
  	Does not allow concurrent fetches of user. If "USER_FETCH_REQUESTED" gets
  	dispatched while a fetch is already pending, that pending fetch is cancelled
  	and only the latest one will be run.
	*/
	function* mySaga() {
  		yield takeLatest("USER_FETCH_REQUESTED", fetchUser);
	}

	export default mySaga;
	//main.js
	import { createStore, applyMiddleware } from 'redux'
	import createSagaMiddleware from 'redux-saga'

	import reducer from './reducers'
	import mySaga from './sagas'

	// create the saga middleware
	const sagaMiddleware = createSagaMiddleware()
	// mount it on the Store
	const store = createStore(
  		reducer,
  		applyMiddleware(sagaMiddleware)
	)

	// then run the saga
	sagaMiddleware.run(mySaga)

	// render the application
	`

最后，总结一下 redux-saga 的优点：

（1）声明式 Effects：所有的操作以JavaScript对象的方式被 yield，并被 middleware 执行。使得在 saga 内部测试变得更加容易，可以通过简单地遍历 Generator 并在 yield 后的成功值上面做一个 deepEqual 测试。
（2）高级的异步控制流以及并发管理：保持 action 的简单纯粹，aciton 不再像原来那样五花八门，让人眼花缭乱。task 的模式使代码更加清晰。可以使用简单的同步方式描述异步流，并通过 fork 实现并发任务。
（3）架构上的优势：将所有的异步流程控制都移入到了 sagas，UI 组件不用执行业务逻辑，只需 dispatch action 就行，增强组件复用性。扩展性强。


## dva
先考虑一下redux-saga的缺点
1、编辑成本高，需要在 reducer, saga, action 之间来回切换
2、不便于组织业务模型 (或者叫 domain model) 。比如我们写了一个 userlist 之后，要写一个    	productlist，需要复制很多文件。
还有一些其他的：
saga 书写太复杂，每监听一个 action 都需要走 fork -> watcher -> worker 的流程
entry 书写麻烦
...
而 dva 正是用于解决这些问题。
他最核心的是提供了 app.model 方法，用于把 reducer, initialState, action, saga 封装到一起，比如：
	`

	app.model({
  		namespace: 'products',
  			state: {
    			list: [],
    			loading: false,
  			},
  		subscriptions: [
    		function(dispatch) {
      			dispatch({type: 'products/query'});
    		},
  		],
  		effects: {
    		['products/query']: function*() {
      			yield call(delay(800));
      			yield put({
        			type: 'products/query/success',
        			payload: ['ant-tool', 'roof'],
      			});
    		},
  		},
  		reducers: {
    		['products/query'](state) {
      			return { ...state, loading: true, };
    		},
    		['products/query/success'](state, { payload }) {
      			return { ...state, loading: false, list: payload };
    		},
  		},
	});`
在有 dva 之前，我们通常会创建 sagas/products.js, reducers/products.js 和 actions/products.js，然后在这些文件之间来回切换,使用dva以后可以将所有的逻辑操作在model中完成。
