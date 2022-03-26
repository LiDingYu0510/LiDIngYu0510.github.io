---
layout: post
title: "[技術筆記]Vue中的Event Bus"
subtitle: "總結了在Vue2.x和Vue3中各自如何使用event bus"
description: "如果要實現跨組件的通訊，除了Vuex之外，在小型專案也很常使用event bus。而Vue2.x和Vue3在實現event bus上也有一些不同。"
date: 2022-03-24T12:00:00
author: "李定宇"
image: "/img/vue_post_bg.jpg"
tags:
  - Vue
URL: "/2022/03/24/vue_event_bus"
categories: [Vue]
---

# 從 Vue2.x 到 Vue3 中的 EventBus

## 前言

在 Vue2 中，如果要實現跨組件的通訊，除了 `Vuex`之外，在小型專案也很常使用 `event bus`；但是在 Vue3 中，移除了 `$on`、`$off`、`$emit`等語法，沒辦法讓 event bus 使用其 api 來監聽。不過，Vue3 中可以用第三方 library [mitt](https://www.npmjs.com/package/mitt)

## Vue2.x 中的 Event Bus

### 第一種方法：在 main.js 中建立 event bus 屬性

**main.js** 中：

```
//在Vue實例中新增一個屬性
Vue.prototype.$eventBus = new Vue();
```

在要監聽事件的組件中：

```
this.$on("event name",(*args)=>{
	...
})
```

在要觸發事件的組件中：

```
this.$emit("event name",*args)
```

### 第二種方法：直接建立新的 Vue 實例，當作 EventBus

在**eventBus.js**中

```
import Vue from "vue";
export const EventBus = new Vue();
```

在要監聽事件的組件中：

```
import { EventBus } from "@/eventBus";
EventBus.$on("event name",(...args)=>{
	...
})
```

在要觸發事件的組件中：

```
import { EventBus } from "@/eventBus";
EventBus.$emit("event name",...args)
```

### 第三種方法：把 eventBus 改造成 plugin

這是公司在前專案中使用的，因為要同時是用 Vue 中和**window**的 EventBus。

在**EventBus.js**中

```
import Vue from 'vue';

class EventBus {
    constructor() {
        this.bus = new Vue();
    }
    on(event, handler) {
        this.bus.$on(event, handler);
    }
    once(event, handler) {
        this.bus.$once(event, handler);
    }
    off(event, handler) {
        this.bus.$off(event, handler);
    }
    emit(event, ...args) {
        this.bus.$emit(event, ...args);
    }
}
export default {
    install(Vue) {
        const eventBus = new EventBus();
        Vue.prototype.$EventBus = eventBus;
        window.EventBus = eventBus;
    }
};
```

然後到**main.js**中註冊

```
import Vue from 'vue';
import EventBus from '@/plugins/EventBus';

Vue.use(EventBus);
```

之後就可以用上述的方式來監聽和觸發事件～

## Vue3 中的 Event Bus

由於 vue3 移除了 `$on`、`$off`、`$emit`等語法，所以用第三方 library [mitt](https://www.npmjs.com/package/mitt)來代替。

新增一個 eventBus 的 entry（**entry.ts**）

```
import mitt, { Emitter } from 'mitt';
const EventBus: Emitter<any> = mitt();

export default EventBus;
```

而我目前的專案是把 event bus 當 hook 來用，所以用以下寫法  
（**useDataHook.ts**）

```
import EventBus from '@/hooks/eventBus/entry';
import { onMounted } from 'vue';
export default function () {
    onMounted(() => {
        EventBus.on("event name",(...args)=>{
			...
		})
    });
}
```

（**DemoListener.vue**）

```
import useDataHook from '@/hooks/eventBus/useDataHook';
setup(){
	useDataHook()
}
```

（**DemoTrigger.vue**）

```
import EventBus from '@/hooks/eventBus/entry';
setup(){
	function demoTrigger(){
		EventBus.emit("event name", ...args)
	}
	return {demoTrigger}
}
```
