---
title: aggrid valueGetter有时取不到值
date: 2024-04-16 +0800
categories: [工作日志, 指南]
tags: [aggrid]
author: <yurongku>  
mermaid: true
---

```vue
valueGetter: (params) => {
    if (params && params.data && params.data.standardPrice) {
        return number_format(params.data.standardPrice);
    } else {
        return null; // 或者返回其他适当的值
    }
},
```