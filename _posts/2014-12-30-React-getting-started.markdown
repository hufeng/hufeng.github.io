---
layout: post
title: "react - getting started"
date: 2014-12-30
categories: React
---

> 工欲善其事，必先利其器


React官方的Getting started感觉不够pragmatic，通过在浏览器中加载jstransform可以很简单的试验React，但是真正的项目中并不会这样用，我们可以使用更加现代，更加的爽直的方式来构建我们的React应用。


工具组合： React + Commonjs(模块化规范) + webpack（打包，合并，压缩，我们的应用）


来来来，give me five minutes.

安装webpack

```sh
sudo npm install -g webpack webpack-dev-server
```



创建项目

```sh
mkdir demo #创建项目名称
cd demo
npm init       #npm创建一个node工程
npm install react —save #安装依赖
npm install babel-loader —save-dev #安装babel依赖
```

项目目录结构

```sh
tree -L 2
├── assets                              #我们的应用目录
│   └── js
├── build                                #应用打包目录
│   └── bundle.js                   #打包合成文件
├── node_modules                #node依赖的模块
│   ├── jsx-loader
│   └── react
├── package.json                 #node项目信息，类似maven的pom
└── webpack.config.js         #webpack的配置文件

6 directories, 3 files
```

开始coding
hello.js

```javascript
import React from 'react';

export default class Hello extends React.Component {
	render() {
		return (<div>hello</div>);
	}
};
```

world.js

```javascript
import React from 'react';

export default class World extends React.Component {
	render() {
		return (<div>world.</div>);
	}
}
```

app.js

```javascript
import React from 'react';
import Hello from './hello';
import World from './world';

class App exntends React.Component {
	render() {
		return (
			<div>
				<Hello/>
				<World/>
			<div>
		)
	}
}

React.render(<App/>, document.body);
```

webpack配置文件
webpack.config.js

```javascript
module.exports = {
	entry: './assets/js/app.js',
	output: {
		path: './build',
		filename: 'bundle.js'
	},
	module: {
		loaders: [
			{test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader?stage=0'}
		]
	}
};
```

> 完事具备，只欠东风

```sh
webpack-dev-server -p 3000 -w
visit: http://localhost:3000/webpack-dev-server
```