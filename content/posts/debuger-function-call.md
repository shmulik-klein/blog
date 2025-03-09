---
title: "Debuger Function Call"
date: 2025-03-08T23:37:19+01:00
draft: true
tags:
    - go
    - debugger
---

### High-Level
It all began with this [PR](https://github.com/golang/go/issues/21678).

The Go runtime includes mechanisms (since go 1.11) that enable a debugger to call functions during a debugging session. In a very high level, this is done by modifying the program's stack and instruction pointer to simulate a call to the desired function. The debugger sets up the function call by preparing the arguments and placing them in the appropriate registers or stack locations as per the calling convention used by Go. The function call is then executed as if it were part of the original program.