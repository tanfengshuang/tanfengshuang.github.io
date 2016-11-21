---
layout: post
title:  "一行的if语句"
categories: Python
tags: Python 
---


a, b, c = 1, 2, 3


*    常规
<pre><code>
if a>b:
    c = a
else:
    c = b
</code></pre>

*    表达式
<pre><code>
c = a if a>b else b
</code></pre>

*    二维列表
<pre><code>
c = [b,a][a>b]
</code></pre>

*    传说是源自某个黑客
<pre><code>
c = (a>b and [a] or [b])[0]
</code></pre>
