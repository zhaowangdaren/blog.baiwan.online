---
title: canvas文本高度
date: 2020-05-12 14:00:06
tags:
---

> 多看官方文档，拨云见日

{% asset_img header-bg.jpg canvas-text %}
在使用`canvas`绘制文本的时候，我们可能会需要知道文本的宽高，来达到给文本增加边框，或者确定相对位置。下面我们来一步步实现一个在文本绘制边框的例子。

## 初始化

初始化`canvas`,首先在html中插入一个`canvas`节点。

```html
<canvas id="canvas" style="width: 100%;"></canvas>
```

在绘图前，我们应该先设置分辨率，否则会糊。

{% asset_img blur.jpg blur %}

先不设置分辨率，直接绘制`canvas text`

```js
function draw() {
  var ctx = document.getElementById('canvas').getContext('2d');
  ctx.font = "40px serif";
  ctx.fillText("canvas text", 10, 50);
}
```

得到的效果是

{% asset_img uninit-pixel.png 未设置分辨率 %}

设置分辨率主要是设置`canvas`的宽高。

```js
var fontSize = 40
var dpr = window.devicePixelRatio || 1
/**
 * 设置分辨率
 */
function initPixel () {
  var canvas = document.getElementById('canvas')
  canvas.width = canvas.width * dpr
  canvas.height = canvas.height * dpr
  var ctx = canvas.getContext('2d');
  ctx.scale(dpr, dpr)
}

function draw() {
  var ctx = document.getElementById('canvas').getContext('2d');
  ctx.font = fontSize + "px serif";
  var text = "canvas text"
  ctx.fillText(text, 10, 50);
}
```

设置分辨率后的文字更清晰。

{% asset_img init-pixel.png 设置分辨率后 %}

## 绘制边框

绘制边框，首先我们需要知道文本的宽高（假设文本只占一行）。文本的宽度我们很容易得到，通过`CanvasRenderingContext2D.measureText`来得到文字的宽高。

```js
var textMetrics = ctx.measureText("canvas text");
```

得到的`TextMetrics`包含了很多信息，根据其中一些属性可以得到文本的宽高，例如：`TextMetrics.width, TextMetrics.fontBoundingBoxAscent, TextMetrics.fontBoundingBoxDescent`，我们的目标主要就是得到宽高，现在都能得到了，这也太简单了，那画简单的矩形就更不用讲了，心疼本文结束。
{% asset_img fake-over.jpg 本文结束 %}

继续滑动页面，查看`TextMetrics`的其他描述，突然看到了**浏览器兼容性**
{% asset_img surprise.jpg 妙啊 %}

两个可以计算出文本高度的属性`TextMetrics.fontBoundingBoxAscent, TextMetrics.fontBoundingBoxDescent`还只是实验功能，很多浏览器不支持。

{% asset_img textmetrics-unsport.png TextMetrics不支持的属性 %}

所以，继续，我们基本只能通过TextMetrics得到文本的宽度，也就是`TextMetrics.width`。那么如何得到绘制文本的高度呢？

## 误入歧途

查看文档，搜索例子，均未得到结果。想到前些天分析ace.js富文本代码编辑的源码，可以通过DOM渲染文本，从而得到文本的高度。

```html
<div class="measure" style="font-size: 40px;font-family: serif;">canvas text</div>
```

首先获取`.measure` DOM的高度，由于设置分辨率的时候进行了放大操作，而DOM是没有进行放大设置的，所以这里传给canvas的高度要进行相应缩小。

```js
function strokeTextRect(ctx, text) {
  var textMetrics = ctx.measureText(text);
  var height = document.querySelector('.measure').clientHeight / dpr
  ctx.strokeStyle = 'red';
  ctx.strokeRect(10, 50 - height, textMetrics.width, height);
}
```

这样得到的边框效果如下

{% asset_img height-1.png measure高度未设置lineHeight %}

边框有点高。给`.measure` DOM设置`line-height`:

```html
<div class="measure" style="font-size: 40px;font-family: serif;line-height: 40px;">canvas text</div>
```

边框的高度变小了
{% asset_img line-height.png %}

`font-size`和`line-height`之间有什么联系吗？

{% asset_img renzhen.gif 认真 %}

## 正道

查询一系列关于`font-size`介绍的中文文档，均是字体大小，这个大小是指宽？还是指字体的高？

查询W3C草案，被自己浅薄的知识震惊到。

{% asset_img csswg_font-size.png font-size草案 %}

This property indicates the desired **height** of glyphs from the font.
此属性指示所需的字体字形高度。---有道词典

{% asset_img papapa.gif 啪啪打脸 %}

`font-size`是文本的高度。本文结束。

## 完整例子

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>canvas text height</title>
  <style>
    body, html {
      margin: 0;
    }
    .measure {
      /* position: absolute; */
      top: 300px;
      left: 10px;
      word-break: keep-all;
    }
  </style>
</head>
<body>
  <div>
    <canvas id="canvas" style="width: 100%;"></canvas>
    <div class="measure" style="font-size: 40px;font-family: serif;line-height: 40px;">canvas text</div>
  </div>
  <script>
    var fontSize = 40
    var dpr = window.devicePixelRatio || 1
    // var dpr = 1
    function initPixel () {
      var canvas = document.getElementById('canvas')
      console.log('canvas', canvas.width, canvas.height, dpr)
      canvas.width = canvas.width * dpr
      canvas.height = canvas.height * dpr
      var ctx = canvas.getContext('2d');
      ctx.scale(dpr, dpr)
    }
    function draw() {
      var ctx = document.getElementById('canvas').getContext('2d');
      ctx.font = "40px serif";
      var text = "canvas text"
      ctx.fillText(text, 10, 50);
      strokeTextRect(ctx, text)
    }
    function strokeTextRect(ctx, text) {
      var textMetrics = ctx.measureText(text);
      var height = document.querySelector('.measure').clientHeight / dpr
      // height = fontSize / dpr
      // dpr = 1
      ctx.strokeStyle = 'red';
      ctx.strokeRect(10, 50 - height, textMetrics.width, height);
    }
    window.onload = () => {
      initPixel()
      draw()
    }
  </script>
</body>
</html>
```
