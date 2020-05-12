---
title: canvas文本高度
date: 2020-05-12 14:00:06
tags:
---

> 多看官方文档，拨云见日

在使用`canvas`绘制文本的时候，我们可能会需要知道文本的宽高，来达到给文本增加边框，或者确定相对位置。

```js
function draw() {
  var ctx = document.getElementById('canvas').getContext('2d');
  ctx.font = "48px serif";
  ctx.fillText("Hello world", 10, 50);
}
```
