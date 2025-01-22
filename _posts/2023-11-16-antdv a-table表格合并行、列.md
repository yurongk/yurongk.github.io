---
title: antd vue a-table表格合并行、列
date: 2023-11-16 +0800
categories: [工作日志, 指南]
tags: [antdv]
author: <yurongku>  
mermaid: true
---


1. 合并行

```js
// 方法
const renderCells = (text, data, key, index) => {
  if (data.length < 1) {
    return 1;
  }
  if (text === "" || text === null) {
    data[index].rowNum = 1;
    return 1;
  }
  // 上一行该列数据是否一样
  if (
    index !== 0 &&
    text === data[index - 1][key] &&
    index % this.pagination.pageSize != 0
  ) {
    data[index].rowNum = 0;
    return 0;
  }
  let rowSpan = 1;

  // 判断下一行是否相等
  for (let i = index + 1; i < data.length; i++) {
    if (text !== data[i][key]) {
      break;
    }
    rowSpan++;
  }
  data[index].rowNum = rowSpan;

  return rowSpan;
};

// 使用
customRender: (text, row, index) => {
  const obj = {
    children: text !== null ? text : "",
    attrs: {
      rowSpan: 1,
    },
  };
  obj.attrs.rowSpan = renderCells(text, recordList, "date", index);
  return obj;
},
```

2. 合并列

```js
// 方法
const renderContent = (value, row, index) => {
  const obj = {
    children: value,
    attrs: {},
  };
  if (row.point) {
    obj.attrs.colSpan = 0;
  }
  return obj;
};

// 使用
customRender: renderContent,
```


### 完整demo

```vue
<template>
  <div id="app">
    <a-card>
      <div class="one">
        <a-row>
          <a-col :span="21">
            <span>时间段：</span>
            <a-range-picker
              @change="changeTime"
              show-time
              v-model="queryForm.time"
              :locale="locale"
              :placeholder="['开始时间', '结束时间']"
              class="range"
            />
            <span>监测点位：</span>
            <a-select
              class="selects"
              show-search
              v-model="queryForm.gateId"
              placeholder="请选择"
              @change="init"
            >
              <a-select-option
                v-for="(item, index) in gateList"
                :key="index"
                :value="item.gateId"
              >
                {{ item.gate }}
              </a-select-option>
            </a-select>
            <span>上下行：</span>
            <a-select
              show-search
              class="selectss"
              v-model="queryForm.direction"
              placeholder="请选择"
              @change="init"
            >
              <a-select-option
                v-for="(item, index) of type"
                :key="index"
                :value="item.value"
              >
                {{ item.label }}
              </a-select-option>
            </a-select>
            <span>船名：</span>
            <a-input
              class="inputs"
              v-model="queryForm.shipName"
              @change="init"
              placeholder="请输入船号"
            />
            <a-button @click="empty" class="btnns"> <b>重置</b></a-button>
          </a-col>
          <a-col :span="3">
            <a-button
              style="background: rgb(22, 106, 193); border: 0; float: right"
              type="primary"
              @click="download"
            >
              <img
                src="@/assets/img/biaogekuang.png"
                alt=""
                style="
                  width: 22px;
                  height: 22px;
                  margin-right: 5px;
                  vertical-align: middle;
                "
              />
              数据导出
            </a-button>
          </a-col>
        </a-row>
      </div>
      <div class="ship-table two">
        <a-table
          bordered
          size="middle"
          :columns="columns"
          :data-source="recordList"
          :pagination="pagination"
          :scroll="{ y: `calc(100vh - 1.3rem)` }"
        >
          <template slot="gateId" slot-scope="text">
            <div v-if="gateList.length">
              {{ gateList.find((item) => item.gateId == text).gate }}
            </div>
            <div v-else>暂无点位信息</div>
          </template>
          <template slot="direction" slot-scope="text">
            <div>
              <span>
                <a-tag color="blue" v-if="text == 0"> 上行 </a-tag>
                <a-tag color="green" v-if="text == 1"> 下行 </a-tag>
              </span>
            </div>
          </template>
          <template slot="pic">
            <viewer>
              <img
                class="tab-pic-list"
                src="http://10.24.213.184/imgs/Pictures/detail/2_20250116145240_2.jpg"
                alt=""
              />
            </viewer>
          </template>
        </a-table>
      </div>
    </a-card>
  </div>
</template>

<script>
// 列
const renderContent = (value, row, index) => {
  const obj = {
    children: value,
    attrs: {},
  };
  return obj;
};
// 行
const renderCells = (text, data, key, index) => {
  if (data.length < 1) {
    return 1;
  }
  if (text === "" || text === null) {
    data[index].rowNum = 1;
    return 1;
  }
  // 上一行该列数据是否一样
  if (
    index !== 0 &&
    text === data[index - 1][key] &&
    index % this.pagination.pageSize != 0
  ) {
    data[index].rowNum = 0;
    return 0;
  }
  let rowSpan = 1;

  // 判断下一行是否相等
  for (let i = index + 1; i < data.length; i++) {
    if (text !== data[i][key]) {
      break;
    }
    rowSpan++;
  }
  data[index].rowNum = rowSpan;

  return rowSpan;
};
const columns = [
  {
    title: "船队",
    dataIndex: "fleetId",
    key: "fleetId",
    align: "center",
    customRender: (text, row, index) => {
      const obj = {
        children: text !== null ? text : "",
        attrs: {
          rowSpan: 1,
        },
      };
      obj.attrs.rowSpan = renderCells(text, recordList, "fleetId", index);
      return obj;
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
    title: "监测点位",
    dataIndex: "gateId",
    key: "gateId",
    align: "left",
    scopedSlots: {
      customRender: "gateId",
    },
  },
  {
    title: "上下行",
    key: "direction",
    dataIndex: "direction",
    align: "center",
    scopedSlots: {
      customRender: "direction",
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
const recordList = [
  {
    fleetId: "1",
    direction: 1,
    shipHigh: "4.5",
    shipWidth: "12.3",
    shipLength: "43.5",
    gateId: "1",
    passTime: "2024-12-19 23:54:40",
    shipName: "无船号",
  },
  {
    fleetId: "1",
    direction: 1,
    shipHigh: "4.5",
    shipWidth: "12.3",
    shipLength: "43.5",
    gateId: "1",
    passTime: "2024-12-19 23:54:40",
    shipName: "无船号",
  },
  {
    fleetId: "1",
    direction: 1,
    shipHigh: "4.5",
    shipWidth: "12.3",
    shipLength: "43.5",
    gateId: "1",
    passTime: "2024-12-19 23:54:40",
    shipName: "无船号",
  },
  {
    fleetId: "1",
    direction: 1,
    shipHigh: "4.5",
    shipWidth: "12.3",
    shipLength: "43.5",
    gateId: "1",
    passTime: "2024-12-19 23:54:40",
    shipName: "无船号",
  },
  {
    fleetId: "2",
    direction: 0,
    shipHigh: "4.5",
    shipWidth: "12.3",
    shipLength: "43.5",
    gateId: "1",
    passTime: "2024-12-19 23:54:40",
    shipName: "无船号",
  },
  {
    fleetId: "2",
    direction: 0,
    shipHigh: "4.5",
    shipWidth: "12.3",
    shipLength: "43.5",
    gateId: "1",
    passTime: "2024-12-19 23:54:40",
    shipName: "无船号",
  },
];
import { GetCheckPoint } from "@/api/components/CheckPoint/GetCheckPoint";
export default {
  components: {},
  data() {
    return {
      //监测点位
      gateList: [],
      queryForm: {
        time: null,
        shipName: "",
        direction: "",
        endTime: "",
        gateId: "",
        startTime: "",
      },
      columns: columns,
      recordList: recordList,
      pagination: {
        current: 1,
        defaultPageSize: 18,
        total: 0,
      },
    };
  },
  mounted() {
    GetCheckPoint({ pageSize: 30 }).then((res) => {
      this.gateList.splice(0, this.gateList.length);
      this.gateList = res.data.list.map((r) => ({
        gate: r.gate,
        gateId: r.id,
      }));
      this.gateList.unshift({
        gate: "所有",
        gateId: 0,
      });
    });
  },
  methods: {},
};
</script>
<style scoped lang="scss">
#app {
  width: 100vw;
  height: calc(100vh - 64px) !important;
  overflow: auto !important;
  background: linear-gradient(
    to bottom,
    rgb(5, 96, 186),
    rgb(225, 237, 251)
  ) !important;
}
/deep/ .ant-card {
  display: flex;
  height: calc(100vh - 73px);
  margin: 0 8px !important;
  border-radius: 10px;
  margin-bottom: 8px !important;
}
/deep/ .ant-card-body {
  padding: 0 !important;
  margin: 0 !important;
  height: 100%;
  width: 100%;
}
/deep/ .ant-table-row {
  height: 34px;
}
/deep/ .ant-table-column-has-actions {
  text-align: center;
}

/deep/.ant-table .light {
  background-color: white;
}
/deep/.ant-table .dark {
  background-color: #d3d7d441;
}
/deep/.ant-table-tbody > tr > td {
  padding: 6px 6px !important;
}
/deep/.ant-table-thead > tr > th {
  background: rgb(234, 239, 245) !important;
  padding: 6px 6px !important;
}
.trace-info {
  margin: 10px 20px;
}
.ship-table {
  margin: 10px;
}
.tab-pic-list {
  width: 120px;
}

.range {
  width: 200px;
  margin-right: 20px;
}
.selects {
  width: 150px;
  margin-right: 20px;
}
.selectss {
  width: 100px;
  margin-right: 20px;
}
.inputs {
  width: 150px;
  margin-right: 20px;
}
.btnns {
  border: 1px solid rgb(73, 131, 196);
  background: rgb(255, 255, 255);
  margin-left: 10px;
  color: rgb(73, 131, 196);
}
.one {
  border-radius: 5px;
  height: 9%;
  // min-height: 9%;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  padding: 15px 20px;
  background: rgb(247, 250, 255);
}
.two {
  height: 91%;
}
</style>

<style lang="scss">
//  scoped 里面，第三方组件样式
.viewer-flip-horizontal,
.viewer-flip-vertical {
  display: none;
}
.viewer-toolbar > ul > .viewer-large {
  height: inherit;
  margin-bottom: 0;
  margin-top: 0;
  width: inherit;
}
.viewer-toolbar > ul > .viewer-large::before {
  margin: 2px;
}
.viewer-next {
  position: fixed !important;
  right: 40px !important;
  top: 50% !important;
  scale: 3;
}
.viewer-prev {
  position: fixed !important;
  left: 40px !important;
  top: 50% !important;
  scale: 3;
}
.viewer-title {
  display: none;
}

::-webkit-scrollbar {
  width: 10px;
  height: 1px;
}
::-webkit-scrollbar-thumb {
  border-radius: 5px;
  background-color: rgb(0, 108, 180);
}
::-webkit-scrollbar-track {
  /* box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2); */
  background: rgb(255, 255, 255);
  border-radius: 5px;
}
</style>

```