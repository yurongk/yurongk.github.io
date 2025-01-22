---
title: vuedraggable(基于sortable.js的拖拽组件)
date: 2023-11-16 +0800
categories: [工作日志, 指南]
tags: [vuedraggable, vue]
author: <yurongku>  
mermaid: true
---


基于sortable.js的拖拽组件
yarn add vuedraggable
import draggable from "vuedraggable";
components: {
  draggable,
},
```html
<draggable
  :list="list"
  class="catalog-list"
  handle=".drag"
  ghost-class="ghost"
  :move="dragMove"
  @start="dragging = true"
  @end="dragging = false"
>
  <div class="catalog-item" v-for="item in list" :key="item.id">
    <div class="drag">
      <a-icon type="menu" />
    </div>
    <div class="title">
      {{ item.title }}
    </div>
    <div class="operator">
      <span>编辑</span>
      <span>删除</span>
    </div>
  </div>
</draggable>
```