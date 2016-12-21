---
layout: post
title:  "AutoReply项目中遇到的问题"
date:   2016-11-28 12:00:00
categories: web
excerpt:
---

* content
{:toc}

## 一

这次涉及到了`Chart.js`一个新模块，坑如下：

- 难以直接调整`canvas`的大小，使用如下设置：

```javascript
// Boolean - whether or not the chart should be responsive and resize when the browser does.

responsive: true,

// Boolean - whether to maintain the starting aspect ratio or not when responsive, if set to false, will take up entire container

maintainAspectRatio: false,
```
- `label`等字体颜色设置直接在全局option里设置，图的颜色在数据data里设置

```javascript
var ctx = document.getElementById("sparkline" + i).getContext("2d");
var chartData = {  
    labels: label.map(function(item){return item.substr(5,5)}),
    datasets: [{
        fillColor: "rgba(255,255,255,.3)",
        strokeColor: "#fff",
        pointStrokeColor: "#fff",
        data: this.overview[i-1],
    }]
};
var myLine = new Chart(ctx).Line(chartData, {
  scaleFontColor: "#fff",
});
```

其他：

- 用flask当后台时，出现页面点击后不刷新部件的情况，用node之后没有这个问题，除了js的配合上，考虑是否是数据用string传输影响了效率？
