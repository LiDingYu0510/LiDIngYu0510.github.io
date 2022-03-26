---
layout: post
title: "[技術筆記]Vue3中，Typescript和Webpack5的路徑命名問題"
subtitle: "在Vue3的專案中使用Typescript和Webpack5，在路徑重命名的小issue"
description: "因為在專案中，原本只有在webpack中使用路徑重命名，發現在vscode中還是沒辦法找到該路徑，最後發現是typescript的編譯問題，因此來紀錄一下"
date: 2022-03-13T12:00:00
author: "李定宇"
image: "/img/vue_post_bg.jpg"
tags:
  - Vue
URL: "/2022/03/13/vue_typescript_webpack5"
categories: [Vue]
---

# Typescript 和 Webpack5 的路徑命名問題

## 前言

因為在專案中，原本只有在 webpack 中使用路徑重命名，發現在 vscode 中還是沒辦法找到該路徑，最後發現是 typescript 的編譯問題，因此來紀錄一下

## Typescript 定義路徑別名：

### tsconfig.json 設定

```
{
    "compilerOptions": {
		"paths": {
			"@base/*": ["src/*"],
			"@plugins/*": ["src/plugins/*"],
			"@themes/*": ["src/themes/*"],
			......
		}
	}
}
```

## webpack5 的別名

### webpack.config.js 設定

```
module.exports = {
	...
	resolve:{
		extensions: ['.ts', '.js', '.json', '.css', '.scss', '.vue'],
		alias: {
			'@base': path.resolve(__dirname, 'src'),
			'@plugins': path.resolve(__dirname, 'src/plugins'),
			'@themes': path.resolve(__dirname, 'src/themes'),
			...
		}
	},
	...
}
```

## PS，如果在 webpack5 中的專案有使用 JWT 的 library

會報一個**polyfill**的錯

### 解決辦法

在 **webpack.config.js**檔案中加上：

```
const NodePolyfillPlugin = require('node-polyfill-webpack-plugin');
module.exports = {
	...
	plugins:[
		new NodePolyfillPlugin()
	]
	...
}
```
