---
title: "Must-known Kotlin Features for Functional Programming"
date: 2022-12-15T10:50:50+01:00
tags:
  - Kotlin
  - Functional Programing
---
Coming from Java, these are the features that let's Kotlin to be a functional programming language:
### Function Types
Functions are _first class citizens_ in Kotlin. To facilitate this, Kotlin has a _Fuction Type_ - a type which specifies the an object need to be a function.

To decalre it, one should wrap the types of the arguments this function expected to recieve in brackets, an arrow (`->`) and then the result type of the function (if the function doesn't return anything _significant_ it should return a `Unit`), something like this `() -> Unit` or this `(Int, Int) -> Int`. 

A function type offers only one method `invoke()` - which has the same arguments and result type as the function type that defines it. Since `invoke()` is an operator, we can call it implcitly:

{{< highlight kotlin >}}
fun x(y: (Int) -> Int) {
    y.invoke(3, 3) // explict invoke
    y(3, 3) // implict invoke
}
{{< /highlight >}}

{{< callout >}} It is a good practice to use Named Arguments for function arguments, espcially when their are many of them and their meaning is unclear. {{< /callout >}}

{{< callout >}} When the function type is long (lots of arguments or just using named arguments) and repetitive, it's nicer to use Kotlin's  type aliases. {{< /callout >}}

### Lambda Expressions
Lambda expressions are a shorter alternative to Anonnymous Functions, but also a bit more powerful. To declare a lambda expression, curly brackets are needed (`{ }` is a valid lambda expression, which its function type is `() -> Unit`).

A full lambda expression will include a list of parameters seperated again with an arrow `->`, followed by the lambda expression body: 
{{< highlight kotlin >}} val myFun: (Int, Int) -> Int = { x: Int, y: Int -> return x + y } {{< /highlight >}}

To make lambdas shorter - an exlict `return` isn't needed and the last statement considered to be the return value.

#### Trailing Lambdas
A _Trailing Lambda_ is a convention in Kotlin, that when the last parameter in a function call is a lambda expression, it can be declared outside of the function's parameters brackets:

{{< highlight kotlin >}}
callingMyFunction(param1: Int, param2: String) {
    _ -> println("My lambda is outside the called function's parameters brackets)
}
{{< /highlight >}}

When a lambda expression is the only parameter for a function, the brackets are needed at all and can be omitted.

### Methods Reference
_Methods Reference_ are complemtery to lambda expressions - instead of creating a new function object we can refer an existing function. We refer to a function using a `::` and the function name - `val f = ::functionName`.
