---
layout: post
title:  "442-strace-ltrace"
categories: Linux
tags: RHCA 442
---

# strace -c uname
# strace -e open iptables -L

# strace -fc -S calls elinks -dump http://www.baidu.com
# strace -fc -e open elinks -dump http://www.baidu.com

-S for strace:
	-S sortby   Sort the output of the histogram printed by the -c option by the specified criterion.  Legal values are time, calls, name, and nothing (default time).

# ltrace -Sfc elinks -dump http://www.baidu.com

-S for ltrace:
	Display system calls as well as library calls

-f fork -c count
