---
title: leaflet wmts图层使用
date: 2023-11-16 +0800
categories: [工作日志, 指南]
tags: [gis, leaflet]
author: <yurongku>  
mermaid: true
---


[OGC WebGIS常用服务标准](https://zhuanlan.zhihu.com/p/543257223)

1. 引入 leaflet.tilelayer.wmts.js

```js
(L.TileLayer.WMTS = L.TileLayer.extend({
  defaultWmtsParams: {
    service: "WMTS",
    request: "GetTile",
    version: "1.0.0",
    layer: "",
    style: "",
    tilematrixset: "",
    format: "image/jpeg",
  },
  initialize: function (a, b) {
    this._url = a;
    var c = {},
      d = Object.keys(b);
    d.forEach((a) => {
      c[a.toLowerCase()] = b[a];
    });
    var e = L.extend({ allowHeader: b.allowHeader }, this.defaultWmtsParams),
      f = c.tileSize || this.options.tileSize;
    for (var g in ((e.width =
      c.detectRetina && L.Browser.retina ? (e.height = 2 * f) : (e.height = f)),
    c))
      e.hasOwnProperty(g) && "matrixIds" != g && (e[g] = c[g]);
    (this.wmtsParams = e),
      (this.matrixIds = b.matrixIds || this.getDefaultMatrix()),
      L.setOptions(this, b);
  },
  onAdd: function (a) {
    (this._crs = this.options.crs || a.options.crs),
      L.TileLayer.prototype.onAdd.call(this, a);
  },
  getTileUrl: function (a) {
    var b = this.options.tileSize,
      c = a.multiplyBy(b);
    (c.x += 1), (c.y -= 1);
    var d = c.add(new L.Point(b, b)),
      e = this._tileZoom,
      f = this._crs.project(this._map.unproject(c, e)),
      g = this._crs.project(this._map.unproject(d, e));
    tilewidth = g.x - f.x;
    var h = this.matrixIds[e].identifier,
      i = this.wmtsParams.allowHeader
        ? this.wmtsParams.tilematrixset + ":" + h
        : h,
      j = this.matrixIds[e].topLeftCorner.lng,
      k = this.matrixIds[e].topLeftCorner.lat,
      l = Math.floor((f.x - j) / tilewidth),
      m = -Math.floor((f.y - k) / tilewidth),
      n = L.Util.template(this._url, { s: this._getSubdomain(a) });
    return (
      n +
      L.Util.getParamString(this.wmtsParams, n) +
      "&tilematrix=" +
      i +
      "&tilerow=" +
      m +
      "&tilecol=" +
      l
    );
  },
  setParams: function (a, b) {
    return L.extend(this.wmtsParams, a), b || this.redraw(), this;
  },
  getDefaultMatrix: function () {
    for (var a = Array(22), b = 0; 22 > b; b++)
      a[b] = {
        identifier: "" + b,
        topLeftCorner: new L.LatLng(20037508.3428, -20037508.3428),
      };
    return a;
  },
})),
  (L.tileLayer.wmts = function (a, b) {
    return new L.TileLayer.WMTS(a, b);
  });

```
1. 调用
自己增加的配置：
allowHeader 设置为 true，允许 tilematrix 带上tilematrixSet

```js
this.map = L.map("leaflet", {
  crs: L.CRS.EPSG4326, //设置坐标系4326
  center: [33.483253, 120.163961], //中心坐标
  zoom: 8, //缩放级别
  minZoom: 5,
  maxZoom: 18,
  zoomControl: true, //缩放组件
  scrollWheelZoom: true,
  attributionControl: false, //去掉右下角logol
});
var matrixIds = [];
for (var i = 0; i < 22; ++i) {
  matrixIds[i] = {
    identifier: "" + i,
    topLeftCorner: new L.LatLng(90, -180),
  };
}
this.wmtsLayer = L.tileLayer
  .wmts("http://10.22.4.83:8599/geoserver/smartdzhd/gwc/service/wmts", {
    layer: "smartdzhd:tcz",
    format: "image/png",
    tilematrixSet: "EPSG:4326",
    version: "1.0.0",
    request: "GetTile",
    tilematrix: "EPSG:4326:8",
    matrixIds: matrixIds, //可缩放
    allowHeader: true,
    zIndex: 3,
  })
  .addTo(this.map);
```

如果增加多个图层，修改不同的zindex

```js
L.tileLayer
  .wmts(
    "http://10.1.30.123:8090/iserver/services/map-JSGISTSLDTtdtsl/wmts100",
    {
      layer: "JSGIST-SLDTtdtsl",
      format: "image/png",
      tilematrixSet: "ChinaPublicServices_JSGIST-SLDTtdtsl",
      version: "1.0.0",
      request: "GetTile",
      tilematrix: "8",
      allowHeader: false,
      matrixIds: matrixIds, //可缩放
      zIndex: 1,
    }
  )
  .addTo(this.map);
```
