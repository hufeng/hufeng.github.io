---
layout: post
title: "babel vs react jsx"
date: 2015-04-17
categories: React
---

# Babel VS jsx

在[babel](babeljs.io)的官网，很惊奇的看到一点，就是babel对jsx的开箱即用的支持（怀疑jsx可能内置了babel的一些功能）。这样就可以在项目中直接使用babel，而不是jsx。怎么migrate呢？


## 在浏览器中
以前：

```javascript
<script type="text/jsx"></script>
```

现在：

```javascript
<script type="text/babel"></script>
```

## browserify的亲们注意了
以前：

```javascript
browserify -t reactify main.js
```
现在：

```javascript
browserfiy -t babelify main.js
```

## Node的小伙伴看这里
以前：

```javascript
require('node-jsx').install();
```

现在：

```javascript
require('babel/register'); //类型于coffeescript的方案
```

##最后的最后肯定是webpack君
从前：

```javascript
loaders: [
  {test: /\.js$/, exclude: /node_modules/, loader: 'jsx-loader'}
]
```

现在:

```javascript
loaders: [
  {test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader'}
]
```


>最后，无论喜欢or不喜欢，ES6要来了。




