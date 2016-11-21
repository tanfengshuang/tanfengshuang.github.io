---
layout: post
title:  "一行的if语句"
categories: Python
tags: Python 
---

```
a, b, c = 1, 2, 3
```


*    常规

```
if a>b:
    c = a
else:
    c = b
```

*    表达式

```
c = a if a>b else b
```

*    二维列表

```
c = [b,a][a>b]
```

*    传说是源自某个黑客

```
c = (a>b and [a] or [b])[0]
```
