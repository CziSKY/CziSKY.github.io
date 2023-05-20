+++
title = "了解 Dependent Type"
date = 2023-05-21
[taxonomies]
categories = ["garbage"]
tags = ["programming language"]

+++

> 前几天给 [Anqur](https://github.com/rowscript/rowscript) 大佬的 `RowScript` 提了个 Issue，然后 QQ 突然被加好友并被介绍了最简单的依值类型 (Dependent Type) 是什么样的。所以归纳整理产出点垃圾强化下印象。

> 在这里也安利一下，[RowScript](https://github.com/rowscript/rowscript) 是还在前期开发中 (In heavy development)，能转译到 JavaScript 的纯函数式编程语言。目标用户群体是 JS 社区。

以 Kotlin 举例，我们能在常见的静态类型编程语言写出这种东西：

```kotlin
fun <T> id(t: T): T = t
```

意思是说，它能够接收一个任何类型 (即泛型 T) 的参数然后返回参数本身，例 `id(114514)` 返回 `114514`。

用 Haskell 重写一下：

```haskell
id :: forall T. T -> T
id x = x
```

在 Haskell 中，该函数显式标注的类型 (或类型签名) 可以让我们很直观的观察该函数的性质。实际上，System-F 明确规定了类型是分为 `Value Type` 跟 `Type Scheme` 的。

拿以上例子来说，`forall T.` 就是一个 `Type Scheme`。它将输入的类型参数化作为一个占位符，并让后面的 `Value Type` 就是 `T -> T` 用上这个参数 (类型)。

但在 Dependent Type 中，我们没有 `Value Type` 和 `Type Scheme` 。假如有一个支持 Dependent Type 的语言，上述函数的类型应该会被改写成这样子：

```
(T : Type) -> T -> T
```

在这里可以将 `(T : Type)` 理解为约束传进来的 `T` 为任意类型。(就跟 Kotlin 中的 `<T>` 或者 Haskell 中的 `forall T.` 很像) ，实际上这个 `Type` 是一个叫 `Universe` 的东西。意为所有类型的类型 (Type of type)。

然而在上述的表示中，其实 `T -> T` 的定义是加了语法糖的。在去糖之后它长这个样子：

```
(T : Type) -> (a : T) -> T
```

你可能看着会很懵逼，为什么第二个 `T` 也能存在参数？但谜底在此时揭晓了，在 Dependent Type 中你可以写出任意类型的定义，也可以写出值的定义。但是它们都算某种表达式 (Expression) 的定义。

```
let ID : Type = (T : Type) -> (a : T) -> T
```

我们将类型或是这个表达式绑在了 `F` 上，它称为 `Function Type` 或 `Dependent Function Type` 或 `Pi Type`。同样的，有类型也要有值。可以理解为我们可以编写对应的实现函数 (Lambda)。

```
let ID : Type = (T : Type) -> (a : T) -> T

let id : ID = lambda (T : Type) . lambda (a : T) . a
```

这就是依值类型 (Dependent Type)。就像我们日常编程使用函数中参数定义了什么，那么函数的内部就能使用那个参数。Dependent Type 多了一个你在 `Function Type` 的 "参数" 里定义了什么，那么在它的 Body 中就能用那个参数。

## Pushing door to the future

注意，`(T : Type)` 在 Dependent Type 中是一个实参，在没有编译器额外 Handling 的情况下调用该函数需要传入一个类型实参，例 `f number 114514`

但你可以实现类型推导：[dependent type 下的类型推导 (meta variables)
](https://zhuanlan.zhihu.com/p/74410702)。