---
title: "To Seal or not to Seal: Sealed Classes"
description: "This post discuss on seal classes feature and when to use it, including pattern matching"
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
public sealed class Sensor permits TempSensor, ForceSensor, PiezoSensor {
    ...
}

public final class TempSensor extends Sensor {

}
```

### To Seal or not to Seal?
Is sealing a class/interface is really a necssary feature? what does its purpose?

>The purpose of sealing a class is to let client code reason clearly and conclusively about all permitted subclasses.  _JEP-360: Sealed Classes_

The JEP states that before sealed classes, the only way the reason about an object sub-type, was to chain `instanceof` if-conditions. This old way required the code to have a catch-all `else` case, since the compiler couldn't know if all the subclasses were taken into account. If such a catch-all clause was missing, a compile time error would have been thrown. On the other hand such a clause is redundant, since the user knows which classes are subclasses of the superclasses, he just doesn't have a way to pass that for the compiler.

```java
void react(Sensor sensor) {
    if (sensor instanceof TempSensor) {
        return ... // do something with ((TempSensor) sensor)
    } else if (sensor instanceof ForceSensor) {
        return ... // do something with ((ForceSensor) sensor)
    } else if (sensor instanceof PiezoSensor) {
        return ... // do something with ((PiezoSensor) sensor)
    }

    // Compiler, are there any other subclasses that I need to take into accout here?
}
```
That's where _Pattern Matching_ and _Test Patterns_ fits in.

>Instead of inspecting an instance of a sealed class with if-else, client code will be able to switch over the instance using type test patterns ([JEP 375](https://openjdk.org/jeps/375)). This allows the compiler to check that the patterns are _exhaustive_.   _JEP-360: Sealed Classes_

No catch-all clause is required and the compiler can warn us if we missed a subclass of the sealed superclass.

```java
int react(Sensor sensor) {
    return switch (shape) {
        case TempSensor t    -> ... t.getTemp() ...
        case ForceSensor f -> ... f.getMass ...
        case PiezoSensor p   -> ... s.getElectricCharge() ...
    };
}
```
A more compact and expressive code + no need for the explict subclass cast<br>
