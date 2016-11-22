---
layout: post
title:  "Opening Robot Framework log failed"
categories: Jenkins
tags: Jenkins robot-framework-plugin
---

[Source Link](https://issues.jenkins-ci.org/browse/JENKINS-32118)

#### Error Info:

```
Opening Robot Framework log failed
Verify that you have JavaScript enabled in your browser.
Make sure you are using a modern enough browser. Firefox 3.5, IE 8, or equivalent is required, newer browsers are recommended.
Check are there messages in your browser's JavaScript error log. Please report the problem if you suspect you have encountered a bug.
```

#### Solution:
To workaround in firefox:
1. Go to page about:config
2. Set security.csp.enable = false

For chrome we can use this plugin: Disable Content-Security-Policy
