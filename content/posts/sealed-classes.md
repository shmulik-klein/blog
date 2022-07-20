---
title: "To Seal or not to Seal: Sealed Classes"
date: 2022-07-20T10:42:33+02:00
tags:
- Development
- Java
- OOP
categories:
- Development
---

### Sealed with a Kiss

Java 15 introduced a preview feature called [Sealed Classes and Intefaces](https://openjdk.org/jeps/360), which was finalized in Java 17.
Sealed classes and interfaces restrict which other classes or interfaces may extend or implement them.

You seal a class/interface by applying the `sealed` modifier before the `class`/`interface` keyword. Than, you have to provide which classes/interface are allowed to inherit/implement the sealed class/interface using the `permits` keyword.

```java
public sealed class Beer permits Ale, Lager, Porter, Stout {
    ...
}

public final class Ale extends Beer {

}
```

### To Seal or not to Seal?

