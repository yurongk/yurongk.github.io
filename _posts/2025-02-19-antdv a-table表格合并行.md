---
title: antd vue a-table表格合并行
date: 2025-02-19 +0800
categories: [工作日志, 指南]
tags: [antdv]
author: <yurongku>  
mermaid: true
---

[参考](https://juejin.cn/post/6973696602435223565)

1. 合并行

```js
 combineRow(key) {
      const tableData = this.recordList;
      for (var i = 0; i < tableData.length; i++) {
        const item = tableData[i];
        let count = 1;
        for (let j = i + 1; j < tableData.length; j++) {
          // 如果是同一个值，往后递增
          if (item[key] === tableData[j][key]) {
            count++;
            // 往后相同的值都设为空单元格
            tableData[j][`${key}RowSpan`] = 0;
            // 只有同值第一个才设置合并的单元格数
            item[`${key}RowSpan`] = count;
            // 所有都是为同一个值的情况
            // 如果到了尾部，则循环结束
            if (j === tableData.length - 1) {
              return;
            }
          } else {
            // 指针跳转到下一个，从下一排开始
            i = j - 1;
            count = 1;
            tableData[j][`${key}RowSpan`] = 0;
            break;
          }
        }
      }
      this.recordList = tableData;
    },
```


使用的时候


```js

    this.combineRow("fleetId");
    this.columns = [
      {
        title: "船队",
        dataIndex: "fleetId",
        key: "fleetId",
        align: "center",
        customRender: (text, row) => {
          return {
            children: `${text}`,
            attrs: {
              rowSpan: row.fleetIdRowSpan,
            },
          };
        },
      },
      {
        title: "监测点位",
        dataIndex: "gateId",
        key: "gateId",
        align: "left",
        scopedSlots: {
          customRender: "gateId",
        },
        customRender: (text, row) => {
          return {
            children: this.$createElement(
              "span",
              {
                class: "",
              },
              this.gateList.length
                ? this.gateList.find((item) => item.gateId == text).gate
                : "暂无点位信息"
            ),
            attrs: {
              rowSpan: row.fleetIdRowSpan,
            },
          };
        },
      },
      {
        title: "上下行",
        key: "direction",
        dataIndex: "direction",
        align: "center",
        customRender: (text, row) => {
          return {
            children: this.$createElement(
              "span",
              {
                class:
                  text == 1 ? "ant-tag ant-tag-green" : "ant-tag ant-tag-blue",
              },
              text == 1 ? "下行" : "上行"
            ),
            attrs: {
              rowSpan: row.fleetIdRowSpan,
            },
          };
        },
      },
      {
        title: "船名船号",
        dataIndex: "shipName",
        key: "shipName",
        align: "left",
      },
      {
        title: "过境时间",
        dataIndex: "passTime",
        key: "passTime",
        align: "left",
        defaultSortOrder: "descend", // 默认上到下为由大到小的顺序
        sorter: (a, b) => {
          return a.passTime > b.passTime ? 1 : -1;
        },
      },
      {
        title: "船长(米)",
        key: "shipLength",
        dataIndex: "shipLength",
        align: "center",
      },
      {
        title: "船宽(米)",
        key: "shipWidth",
        dataIndex: "shipWidth",
        align: "center",
      },
      {
        title: "船高(米)",
        key: "shipHigh",
        dataIndex: "shipHigh",
        align: "center",
      },
      {
        title: "图片",
        key: "pic",
        dataIndex: "pic",
        align: "center",
        scopedSlots: {
          customRender: "pic",
        },
      },
    ];

```