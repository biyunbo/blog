---
title: socket.io+react实时聊天
date: 2017-10-14 12:06:21
categories: 
	- react
tags:
	- react
	- socket.io
---

# socket.io+react实现实时聊天功能。

心血来潮，想要写一个实时聊天的小项目，于是找到了socket.io。

Socket.io用于浏览器与node.js之间实现实时通信。

socket.io是一个跨浏览器的支持webSocket的实时通讯的js。

他主要分为服务器端和客户端。

服务器一般使用node.js。

而客户端可以使用传统的html也可以使用现在流行的框架如react或者vue。

而我在搜索相关文章的时候发现，普遍都是html的，基本都是官方给的例子。

客户端用react或vue的并不多，可以说是很少。

我在这里选择了react。（完全个人喜好）

由于可参考的文章有限，基本都只自己摸索的，也踩了很多坑，这里总结一下，希望对您可以有一点小小的帮助。

### 项目地址https://github.com/biyunbo/react-chat  。

### 项目展示网址          [实时聊天软件](http://chat.byb224.top)

### 手机端扫描二维码

![er](https://github.com/biyunbo/react-chat/raw/master/show/chat.png)

## 服务端

首先安装socket.io`npm install socket.io`

我用的node框架是express；接着安装express `npm install express`

开启服务：

```
var express = require('express');
var app = express();
var server = require('http').createServer(app);
var io = require('socket.io').listen(server);

server.listen(3000);
//监听3000端口
```

创建链接：

```
io.sockets.on('connection', function (socket) {
	//获取当前房间id
	var user = '';
  	var roomId = socket.request._query.roomId;
  	socket.on('join',function(userName){
  		//socket.emit('info');
		user = userName;
    	// 将用户昵称加入房间名单中
    	if (!roomInfo[roomId]) {
	     	roomInfo[roomId] = [];
	    }
    	roomInfo[roomId].push(user);
    	socket.join(roomId);    // 加入房间
    	// 通知房间内人员
    	//io.to(roomId).emit('sys', user + '加入了房间', roomInfo[roomId]);  
    	console.log(user + '加入了' + roomId);
    	//console.log(roomInfo);
    	var roomPer = roomInfo[roomId];
    	io.to(roomId).emit('info',roomPer)
	});
	socket.on('leave', function () {
		socket.emit('disconnect');
	});
	socket.on('disconnect', function () {
		// 从房间名单中移除
		var index = roomInfo[roomId].indexOf(user);
		if (index !== -1) {
			roomInfo[roomId].splice(index, 1);
		}
		socket.leave(roomId);    // 退出房间
		//io.to(roomId).emit('sys', user + '退出了房间', roomInfo[roomId]);
		console.log(user + '退出了' + roomId);
		//console.log(roomInfo)
		var roomPer = roomInfo[roomId];
    	io.to(roomId).emit('info',roomPer)
	});

	// 接收用户消息,发送相应的房间
	socket.on('message', function (msg) {
		// 验证如果用户不在房间内则不给发送
		//console.log(msg,user)
		if (roomInfo[roomId].indexOf(user) === -1) {
			return false;
		}else{
			io.to(roomId).emit('msg',msg);
		}
	});
	
});
```

看注释应该就可以看懂了，重点是客户端。

## 客户端

这里react，redux就不详细说了。不太懂得可以去看的的[在react项目中使用redux总结。](http://www.bobyb.top/2017/09/28/redux%E4%BD%BF%E7%94%A8%E6%80%BB%E7%BB%93/)

首先在组件中引入socket.io`import io from 'socket.io-client';`

重点是在组件中的什么地方和服务端链接。

我最开始是在每一个用到的生命周期函数中都`socket = io.connect(`localhost:3000`,{query:`roomId=${roomId}`});`

但是发现这样并不对，会出现多次触发链接，发送消息也会多次触发。

于是我在：

```
export default class Chat extends React.Component {
	constructor(props) {
		super(props);
		this.send = this.send.bind(this)
		this.msg = this.msg.bind(this)
		this.socket = io.connect(`localhost:3000`,{query:`roomId=${roomId}`});
	}
```

链接了服务端。

在生命周期函数中就可以用`this.socket.on('msg',this.msg);`来监听了。

```
componentWillMount(){
        this.socket.on('msg',this.msg);
	}
```

```
msg(data){
        this.props.msg(data);
        this.mylistdiv.scrollTop = this.mylistdiv.scrollHeight //滚动条保持在最下方
    }
```

监听msg函数，接收消息。

send函数发送消息

```
send(){
		let value = this.myvalue.value
        let {userName} = this.props.global
        let data = {userName:userName,msg:value}
        if(value != ""){
        	this.socket.send(data);
        }
        this.myvalue.value = '';
	}
```

这就完成了整个发送接收消息的过程。

下面是完整代码：

```
import React from 'react';
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import PropTypes from 'prop-types';
import io from 'socket.io-client';
/*actions*/
import * as global from 'actions/global';

/*组件*/


@connect(
    state => state,
    dispatch => bindActionCreators({...global}, dispatch)
)
export default class Chat extends React.Component {
	constructor(props) {
		super(props);
		this.send = this.send.bind(this)
        this.msg = this.msg.bind(this)
        this.info = this.info.bind(this)
        this.side = this.side.bind(this)
        let {roomId} = props.match.params;
        this.socket = io.connect(`localhost:3000`,{query:`roomId=${roomId}`});
	}
	componentWillMount(){
        let {userName} = this.props.global
        let {roomId} = this.props.match.params;
        this.props.roomId(roomId);
        this.props.join(userName,this.socket)
        this.socket.on('msg',this.msg);
        this.socket.on('info',this.info);
	}
	componentWillReceiveProps(newProps) {
        //props改变触发此函数
        if (newProps.global.roomId !== this.props.global.roomId) {
        	var msg = [];
            this.props.msgC(msg);
        }
    }
	send(){
		let value = this.myvalue.value
        let {userName} = this.props.global
        let data = {userName:userName,msg:value}
        if(value != ""){
        	this.socket.send(data);
        }
        this.myvalue.value = '';
	}
    info(roomPer){
        this.props.roomInfo(roomPer);
    }
    side(e){
        if(this.props.global.side){
            this.props.side(false);
        }else{
            this.props.side(true);
        }
    }
    msg(data){
        this.props.msg(data);
        this.mylistdiv.scrollTop = this.mylistdiv.scrollHeight
    }
	render() {
        var {msgList,userName,roomId} = this.props.global;
		return(
			<div className="chat">
				<Header title={roomId} leftto="fanhui" {...this.props} socket={this.socket} side={this.side}/>
				<div className="chat-main" ref={(ref) => this.mylistdiv = ref}>
                    {
                        msgList.map((ele,index) => {
                            return (
                                <Chatli key={index} {...ele} user={userName}/>
                            )
                        })
                    }
				</div>
				<div className="send">
					<input ref={(ref) => this.myvalue = ref} />
					<div className="send-in" onClick={this.send}>发送</div>	
				</div>
                <Side {...this.props} side={this.side} />
			</div>
		)
	}
}

class Header extends React.Component {
    constructor(props) {
        super(props);
        this.handleClick = this.handleClick.bind(this)
    }
    handleClick() {
        // this.props.history.push('/')
        var {socket} = this.props;
        history.go(-1);
        this.props.leave(socket)
    }
    render() {
        let { title,leftto } = this.props;
        let left = null;
        if(leftto == "kong"){
            left = (<div></div>)
        }else if(leftto == "fanhui"){
            left = (
                <div className="fanhui" onClick={this.handleClick}>
                    <i className="iconfont icon-fanhui"></i>
                </div>
            )
        }
        return(
            <div className="top">
                {left}
                <div className="title">{title}</div>
                <div className="gengduo" onClick={this.props.side}>更多</div>
            </div>     
        )
    }
}

class Chatli extends React.Component {
    constructor(props) {
        super(props);
    }
    render() {
        let {userName,msg} = this.props.msgList
        let {user} = this.props
        var name = '';
        var cmsg = '';
        if(user == userName){
            name = 'name1'
            cmsg = 'cmsg1'
        }else{
            name = 'name'
            cmsg = 'cmsg'
        }
        return(
            <div className="li clear">
                <div className={name}>
                    {userName}
                </div>
                <div className={cmsg}>
                    {msg}
                </div>
            </div>
        )
    }
}

class Side extends React.Component {
    constructor(props) {
        super(props);
    }
    render() {
        let {roomInfo,side} = this.props.global;
        console.log(roomInfo)
        let sidelist = '';
        let sideb ='';
        if(side){
            sidelist = 'side-list'
            sideb = 'side'
        }else{
            sidelist = 'side1'
            sideb = 'side1'
        }
        return(
            <div>
                <div className={sideb} onClick={(e) => this.props.side(e)}></div>
                <div className={sidelist}>
                    {
                        roomInfo == null ? <div></div> : <Sideli roomInfo={roomInfo}/>
                    }
                </div>
            </div>
        )
    }
}

class Sideli extends React.Component {
    constructor(props) {
        super(props);
    }
    render() {
        let {roomInfo} = this.props
        return(
            <div className="sideli">
                <div className="sideli-top">房间在线人数：{roomInfo.length}</div>
                <div className="roomPer">房间在线成员</div>
                <div className="sideli-box">
                    {
                        roomInfo.map((ele,index) => {
                            return(
                                <div className="box-li" key={index}>{ele}</div>
                            )
                        })
                    }
                </div>
            </div>
        )
    }
}
```

其中配合了redux的使用。

希望对大家有所帮助。