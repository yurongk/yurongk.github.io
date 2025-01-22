---
title: antd table 自定义表头
date: 2024-09-16 +0800
categories: [工作日志, 指南]
tags: [antdv]
author: <yurongku>  
mermaid: true
---

```
<template slot="run">
<span>船名船号</span>
<a-dropdown :trigger="['click']">
    <a-icon
    style="color: #bfbfbf; margin-left: 8px"
    type="interaction"
    @click.stop
    />
    <a-menu slot="overlay">
    <a-menu-item key="0">
        <a-radio>按中文排序</a-radio>
    </a-menu-item>
    <a-menu-item key="1">
        <a-radio>按数字排序</a-radio>
    </a-menu-item>
    </a-menu>
</a-dropdown>
</template>


{
    // title: "船名船号", -------->必须注释
    dataIndex: "name",
    key: "name",
    width: "20%",
    slots: { title: "run" },
    sorter: (a, b) => a.name.localeCompare(b.name, "zh-CN"),
},
```