---
title: redux使用总结
date: 2017-09-28 13:34:19
categories: redux
tags: 
      - redux
      - react
---

# 在react项目中使用redux总结。

这是在写完一个自己的小项目后所写的。

###项目地址https://github.com/biyunbo/react-cnode 。

### 项目展示网址[cnode中文社区](http://cnode.byb224.top)

### 手机端扫描二维码

![er](https://github.com/biyunbo/react-cnode/raw/master/show/cnode.png)

下面开始进入正题。

1. 首先要知道redux是什么有什么作用。

   引用一个别人说的一句话

   > redux就像眼镜，需要用他的时候你自然知道

   redux是一个状态管理机制，他的作用就是将所有的状态，保存在一个对象里面。

2. 如何使用redux。

   在使用redux中，你一定要了解这几个东西。

   1. store

      store是保存数据的地方，在整个应用中用且只能有一个store。

      所以的数据都放在这个容器中，用的时候从这个容器中去拿，你也可以把他看作是一个后端的数据库。

   2. state

      说有数据的状态，获取当前状态可以通过store.getState()拿到。

   3. action

      改变state的唯一方法。

   4. reducer

      接收action然后返回一个新的state。

以上这些是我的理解，要看这些概念的东西我觉得[redux中文教程](http://www.redux.org.cn/)已经写的很好了。

下面我直接说我在项目中的用法。

在react项目中首先要引入redux

```
npm install redux
npm install react-redux
```



```
import { createStore, applyMiddleware } from 'redux';
import { Provider } from 'react-redux';
import thunk from 'redux-thunk';
```

因为react 的 render()函数只能返回一个容器。

```
ReactDOM.render(
  <Provider store={store}>
  	<Component />
  </Provider>,
  document.getElementById('root')
    );
```

Provider里包含的是这个容器的说要数据，store。

```
const store = createStore(rootReducer,applyMiddleware(thunk))
```

createStore创建一个store，applyMiddleware加入中间件thunk

rootReducer是将store和reducer链接起来的，用combineReducers方法，将所有的reducer都注册到store中。

```
//rootReducer
import { combineReducers } from 'redux';
import { routerReducer } from 'react-router-redux'
import { global } from './global';
//注册reducer，每个自定义的reducer都要来这里注册！！！不注册会报错。
const rootReducer = combineReducers({
  routing: routerReducer,
  /* your reducers */
  global,
});

export default rootReducer;

```



有了这个中间件，我们就可以在action里用异步去dispatch一个reducer。

这样就引入成功了。

之后是写action。

就拿登录来说。

```
export const successLogin = (accesstoken, loginname, id ) => ({
	type: 'SUCCESS_LOGIN',
	accesstoken,
	loginname,
	id
})
```

这就是一个简单的action。

```
export const postAccessToken = (access_token) => async (dispatch, getState) =>{
	try {
		let response = await instance.post(`accesstoken/?accesstoken=${access_token}`)
		await response.data.success ? dispatch(successLogin(access_token ,response.data.loginname ,response.data.id)):dispatch(failLogin(response.data.error_msg))
	} catch(error) {
		console.log('error: ', error)
	}
}
```

由于我们用了thunk这个中间件，所以我们可以直接在action里传入dispathch，直接触发另一个action。

这里我用到了async  await  语法。

首先发起请求，触发`postAccessToken`这个action，在这个action中，有dispathch触发了`successLogin`这个action。

action里的type是唯一的，他对应了一个reducer。

```
// 初始化状态
let Initialization = {
    success : false
}

export function global(state = Initialization , action) {
	switch (action.type) {
		case 'SUCCESS_LOGIN':
			//console.log('登录成功');
			return Object.assign({},state,{
				success : true,
				loginname : action.loginname,
				id : action.id,
				accesstoken : action.accesstoken
			})
		default:
			return state
	}
}
```

这个reducer返回一个新的对象，改变了state的状态。

这就是一个登录状态的变化过程。

然后我们再看在react中怎么去触发这个`postAccessToken`action。

```
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import * as global from 'actions/global';

@connect(
    state => state,
    dispatch => bindActionCreators({...global}, dispatch)
)
export default class Login extends React.Component {
	constructor(props) {
		super(props);
		this.handleClick = this.handleClick.bind(this)
	}
	componentWillMount(){

	}
	handleClick(){
		let token = this.myvalue.value
		this.props.postAccessToken(token);
	}
	render() {
		return(
			<div className="main">
				<Header title="登录" leftto="kong"/>
				<div className="denglu">
					<input placeholder="cnode社区accesstoken" ref={(ref) => this.myvalue = ref} />
					<div className="login" onClick={this.handleClick}>登录</div>
				</div>
			</div>
		)
	}
}
```

引入connect方法。

这个方法可以将你用到的redux挂载到你这个组件上。

@是es7的用法。

这个必须写在你的组件相邻的上边。

引入bindActionCreators方法。

这个方法有一个隐式调用。

`postAccessToken() `只要用这个action就是直接dispatch(postAccessToken)。

dispatch了这个action就开始了之前我们说的那个流程，数据就是这样流动的。







这些就是我个人所用到的redux，希望对新手有所帮助。