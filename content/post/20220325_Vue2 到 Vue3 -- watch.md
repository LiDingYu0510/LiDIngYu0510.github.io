---
layout: post
title: "[技術筆記]Vue中使用watch來監聽data變化"
subtitle: "總結了在Vue2.x和Vue3中各自如何使用watch"
description: "Vue2.x和Vue3在實現watch上有一些不同，因此紀錄一下。"
date: 2022-03-25T12:00:00
author: "李定宇"
image: "/img/vue_post_bg.jpg"
tags:
  - Vue
URL: "/2022/03/255/vue_watch"
categories: [Vue]
---

## 前言

目前我在手上的專案，要從 Vue2.6 轉換到 Vue3.0，剛好利用這個機會紀錄一下兩者的差別。

## Vue2.x 的 watch 寫法

### 簡單寫法

假設欲觀察的值為` testData`

```
data(){
return {
		testData:0
	}
},
watch:{
	testData:(newValue, oldValue){
		console.log("data changed")
	}
}
```

### 完整寫法

假設欲觀察的值為` testData`

```
data(){
return {
		testData:0
	}
},
watch:{
	testData:{
		immediate:true, //立即監聽（初次渲染時就執行）
		handler:(newValue, oldValue){
			//監視的回調
			console.log(newValue)
		}
	}
}
```

## Vue3.0 的 watch 寫法

### 監視一個 ref 定義的響應式數據

假設欲觀察的值為` testData`

```
setup(){
	const testData = ref(0)
	watch(testData, (newValue, oldValue)=>{
		console.log("testData changed", newValue)
	})
	return {testData}
}
```

ps. `immediate`和`deep`的配置也可以使用；不過大多數情況 setup 中的 watch 都是 immediate 的

```
watch(testData, (newValue, oldValue)=>{...}, {immediate:true, deep:true})
```

### 監視多個 ref 定義的響應式數據

假設欲觀察的值為` testData01`、`testData02`

```
setup(){
	const testData01 = ref(0)
	const testData02 = ref("Hello Vue3")

	watch([testData01, testData02], (newValue, oldValue)=>{
		console.log("testData01 changed", newValue[0])
		console.log("testData02 changed", newValue[1])
	})
	return {testData01, testData02}
}
```

### 監視 reactive 所定義的 object 的全部 keyValue

假設欲觀察的值為` tesObject`

```
setup(){
	const testOject = reactive({
		testData01 : 0,
		tsetData02 : "hello Vue3"
	})
	watch(testOject, (newValue, oldValue)=>{
		console.log("data changed", newValue, oldValue);
	})
	return {testOject}
}

```

**不過此寫法沒辦法獲取到 oldValue**

### 監視 reactive 所定義的 object 的某個 keyValue

假設欲觀察的值為` tesObject`中的`testData01 `

```
setup(){
	const testOject = reactive({
		testData01 : 0,
		tsetData02 : "hello Vue3"
	})
	watch(()=>testOject.testData01,
		(newValue, oldValue)=>{
			console.log("data changed", newValue, oldValue);
		})

	return {testObject}
}
```

### 監視 reactive 所定義的 object 的某個 keyValue,而該值也是個 object

假設欲觀察的值為` tesObject`中的`testDataObject `

```
setup(){
	const testOject = reactive({
		testData01 : 0,
		testData02 : "hello Vue3",
		testDataObject : {
			testData03:2022,
			testData04:"Thursday"
		}
	})
	watch(()=>testOject.testDataObject,
			(newValue, oldValue)=>{
				console.log("data changed", newValue, oldValue);
			},
			{deep : true}
		)
	return {testObject}
}
```

**要開啟深度監聽**

### 監視父組件傳過來的 props

假設欲觀察的值為`testProps`

```
props:{
	testProps:{
		type:string,
		required:true
	}
},
setup(){
		watch(()=> testProps,
			(newValue, oldValue)=>{
				console.log("data changed", newValue, oldValue);
			},
			{deep : true}
		)
	return {testProps}
}
```

**要開啟深度監聽**
