---
title: "Must-known Kotlin Features for Functional Programming"
date: 2022-12-15T10:50:50+01:00
tags:
  - Kotlin
  - Functional Programing
---
Kotlin takes Java’s functional programming features, gives them a makeover, and makes them way cooler. It’s like Java went to a coding gym and came back stronger and sleeker. In this post, we’ll check out how Kotlin takes Java’s functional side to the next level, making your code cleaner and way more fun to write.

### Function Types
Functions are [_first class citizens_](https://en.wikipedia.org/wiki/First-class_function) in Kotlin. To facilitate this, Kotlin has a [_Fuction Type_](https://kotlinlang.org/docs/lambdas.html#function-types) - a type which specifies that an object needs to be a function.

To decalre it, one should wrap the types of the arguments this function expected to recieve in brackets, an arrow (`->`) and then the result type of the function (if the function doesn't return anything _significant_ it should return a `Unit`), something like this `() -> Unit` or this `(Int, Int) -> Int` (recall how cumbersome this in Java, aka. `@FunctionalInterface`).

A function type offers only one method `invoke()` - which has the same arguments and result type as the function type that defines it. Since `invoke()` is an operator, we can call it implcitly:

{{< highlight kotlin >}}
fun x(y: (Int) -> Int) {
    y.invoke(3, 3) // explict invoke
    y(3, 3) // implict invoke
}
{{< /highlight >}}

{{< callout >}} It is a good practice to use Named Arguments to the function's parameter list types, especially when the list is long and its meaning is unclear{{< /callout >}}

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

### Function Reference
_Function Reference_ are complemtery to lambda expressions - instead of creating a new function object we can refer an existing function. 

We refer to a top-level function using a `::` and the function name - `val f = ::topLevelFunctionName`.

### Inline Functions
Using functions with functional parameters (_High Order-Functions_) has some performence penalties - when calling such a function, objects are created for each lambda expression, which can have a performance overhead. This a great oppurtuntiy to use Kotlin's `inline` keyword - when adding it to a function with functional parameters - the function together with its lambdad expression are inlined.

{{< highlight kotlin >}}
fun filterList(numbers: List<Int>, predicate: (Int) -> Boolean): List<Int> {
    val result = mutableListOf<Int>()
    for (number in numbers) {
        if (predicate(number)) {
            result.add(number)
        }
    }
    return result
}

val isEven: (Int) -> Boolean = { it % 2 == 0 }

fun main() {
  val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
  val evenNumbers = filterList(numbers, isEven)
  println(evenNumbers) // Output: [2, 4, 6, 8, 10]
}
{{< /highlight >}}

Another benefit of inlining a function with a functional parameters is having a _non-local_ return.
