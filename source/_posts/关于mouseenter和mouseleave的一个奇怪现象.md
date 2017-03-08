---
title: '关于 mouseenter, mouseleave 的一个奇怪现象'
date: 2017-03-08 22:28:56
tags:
	- trick
categories:
	- trick
---

今天在处理 `mouseenter`, `mouseleave` 时，发现一个奇怪的现象。

reproduction，[点击这里](https://jsfiddle.net/kuangcaibao/bhpbwnzu/)

触发 `mouseenter` 事件，是在鼠标进入 `box1` 框中，符合预期。但是触发 `mouseleave` 事件，却是在鼠标离开下方边框，然后再下面一段距离才行，这个就不符合预期了。

{% qnimg mouseleave.png title:现象截图 alt:waiting  %}

但是如果把代码中的，这行注释掉，那么2个事件的表现就都符合预期了。

```
$("#box3").stop().fadeIn(100);
// $("#box3").css({"display": "inline-block"});
```