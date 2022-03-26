---
layout: post
title: "[技術筆記]Flutter 中的 StatlessWidget 和 StatefulWidget "
subtitle: "Flutter中最基本的兩種Widget"
description: "在Flutter開發中，可以說“一切皆widget”，在前端框架中很常見的名詞如View、View Controller、Activity、Application、Layout等等，在Flutter中都是widget。 
而在Flutter中的widget，有`StatelessWidget` 和 `StatefulWidget` 兩種類型"
date: 2022-02-21T12:00:00
author: "李定宇"
image: "/img/flutter_post_bg.jpg"
tags:
  - Flutter
URL: "/2022/02/21/flutter_widget"
categories: [Flutter]
---

# StatelessWidget 和 StatefulWidget

## 前言

在 Flutter 開發中，可以說“一切皆 widget”，在前端框架中很常見的名詞如 View、View Controller、Activity、Application、Layout 等等，在 Flutter 中都是 widget。
而在 Flutter 中的 widget，有`StatelessWidget` 和 `StatefulWidget` 兩種類型；而兩者的差別，是在於：

- StatefulWidget 是處理有交互、畫面視覺變化的場景；
- StatelessWidget 則是處理沒有變化的靜態場景；

## StatelessWidget

- 使用時機：如果渲染初期就可以獲得所需參數，那就可以直接用 StatelessWidget；換句話說，當 widget 建構出來後，不再關心也不再響應數據變化，那麼就用 StatelessWidget

## StatefulWidget

- 有狀態的 widget
- 透過 `setState((){})` 來更新數據並觸發 build()，重新渲染 widget
- 生命週期（目前我專案中最常用到的就是 initState 和 dispose）
- initState: 在整個生命週期中只會被調用一次，可以用來初始化一些數據
- didChangeDependencies：專門處理 State 依賴關係變化
- build：return 一個 widget，為該畫面 UI
- didUpdateWidget(Widget oldWidget)：只要父 widget 調用`setState`，該函數就會被觸發
- deactivate：當該 widget 的可見狀態發生變化時會調用，依照定義，是在`dispose`、也就是 widget 被銷毀之前調用。（不過目前筆者還沒有該使用場景，會再研究一下）
- dispose：當 widget 被永久地從 widget tree 中被移除時調用，通常會拿來“移除監聽”

## 有 StatefulWidget，為何還需要 StatelessWidget

雖然說使用 StatefulWidget 可以應付所有場景，但調用`setState`來更新視圖時，該 StatefulWidget 的所有子 widget 也會跟著 rebuild，這代表了巨大的資源消耗，會讓 Flutter 的渲染性能造成很大的影響。  
如果一個頁面的 root layout 是一個 StatefulWidget，在其 State 中每調用一次更新 UI，都將是一整個頁面所有 Widget 的銷毀和重建。  
所以要慎用～
