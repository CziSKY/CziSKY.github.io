+++
title = "在 Kotlin 用 36 行实现 when 表达式"
date = 2023-01-24
[taxonomies]
categories = ["coding"]
tags = ["kotlin"]
+++

近期我在看 [CS61A](https://github.com/CziSKY/CS61A)，对高阶函数 (Higher Order Function) 有了新的直觉和理解。所以我就突然想在 Kotlin 中实现函数式语言系的 Pattern Matching。我不知道这两个想法是怎么产生关联的，但它产生了。

## 来自失败者的讲解和他的一生

Pattern Matching 是什么呢？它就类似于 Java 中的 `Switch` 或者是 Kotlin 中的 `when`，这是一个 Haskell 中的简单例子：

```haskell
sayMe :: (Integral a) => a -> String  
sayMe 1 = "One!"  
sayMe 2 = "Two!"  
sayMe 3 = "Three!"  
sayMe 4 = "Four!"  
sayMe 5 = "Five!"  
sayMe x = "Not between 1 and 5"
```

即使你不懂 Haskell 大概也能猜出来这段代码的意思。可以理解成你传入一个参数 `Int` 然后它给你返回一个 `String`。但是 Pattern Matching 比这更强大，强大在哪了呢？下面是一个来自 Scala 官网的例子：

```scala
sealed trait Notification

case class Email(sender: String, title: String, body: String) extends Notification

case class SMS(caller: String, message: String) extends Notification

case class VoiceRecording(contactName: String, link: String) extends Notification

def showNotification(notification: Notification): String = {
  notification match {
    case Email(sender, title, _) =>
      s"You got an email from $sender with title: $title"
    case SMS(number, message) =>
      s"You got an SMS from $number! Message: $message"
    case VoiceRecording(name, link) =>
      s"You received a Voice Recording from $name! Click the link to hear it: $link"
  }
}

println(showNotification(SMS("12345", "Are you there?")))  // prints You got an SMS from 12345! Message: Are you there?

println(showNotification(VoiceRecording("Tom", "voicerecording.org/id/123")))  // prints You received a Voice Recording from Tom! Click the link to hear it: voicerecording.org/id/123
```

它支持内部参数的判断和绑定（绑定参数提供给下文），即你还可以把 `case Email(sender, title, _) =>` 改成 `case Email("闲蛋", title, _) =>`。这样即同时表示如果 `notification` 是一个 `Email` 类型加它的第一个参数还为 "闲蛋" 就执行下一步的操作。

但实现中途发现我太菜然后写挫了，因为我没能实现参数绑定。只实现了一个加强版的 when。

## 失败者的实现

我们可以把匹配体抽象成一个列表，这是最简单的实现方法。每一个 case 即是在列表内添加一个元素。当需要匹配的时候在列表里面寻找第一个元素（表示匹配顺序）然后返回。我们新建一个 `MatcherBranch` 类：

```kotlin
class MatcherBranch<R, O>(val value: O, val result: O.() -> R) {

    fun get() = result.invoke(value)
}
```

范型 `R` 表示的是最终我们返回的结果类型，`O` 表示传入的匹配类型。这两个类型都由更上层提供给我们，我们再新建一个 Matcher 类：

```kotlin
class Matcher<R, O>(val value: O) {

    val branches = mutableListOf<MatcherBranch<R, *>>()
}
```

`branches` 的第二个类型参数为 `*` 的原因是我们可能需要 case 到不同的类型上去。也就是说这个类型并不唯一。我们编写第一个匹配方法：

```kotlin
fun case(block: (O) -> Boolean, result: O.() -> R) {
    if (!block.invoke(value)) {
        return
    }
    branches += MatcherBranch(value = value, result = result)
}
```

第一个 case 是同类型的模式匹配，相当于对传进来的 `value` 变量用 if 进行判断。如果符合结果则加入分支。我们编写第二个匹配参数：

```kotlin
@JvmName("caseType")
inline fun <reified T> case(block: (T) -> Boolean = { true }, noinline result: T.() -> R) {
    (value as? T)?.apply {
        if (!block.invoke(this)) {
            return
        }
        branches += MatcherBranch(value = this, result = result)
    }
}
```

我们引入了一个新的范型 `T`，这个 `T` 代表了我们需要转换到的新类型。如转换成功则进行条件判断，条件判断成功则加入分支。

因为 JVM 底层的范型擦除机制，正常情况下我们并不能把这个 `T` 直接传过来然后跟参数转换，我们使用 Kotlin 的 `reified` 特性来解决这个问题。

此外，我们还需要一个分支，这个分支代表所有结果都没有匹配成功的返回。在 Java 中它为 `Switch` 中的 `default:`，在 Kotlin 中它为 `when` 中的 `else`。

```kotlin
fun default(result: R) {
    branches += MatcherBranch(value = value, result = { result })
}
```

此时还有一个问题，那就是我们并不能确定使用者会把 `default` 放在匹配体的什么位置（虽然并没有人会用），因为我们的逻辑是匹配第一个元素返回结果，如果你把这个 `default` 提到前面的话就非常糟糕了。它只会匹配到 `default`。就像这样：

```kotlin
interface Notification

data class Email(val sender: String, val title: String, val body: String) : Notification

data class SMS(val caller: String, val message: String) : Notification

data class VoiceRecording(val contactName: String, val link: String) : Notification

matcher(VoiceRecording("Joe", "voicerecording.org/id/123")) {
    default("Match error.")
    case<Email> { "You got an email from $sender with title: $title" }
    case<SMS> { "You got an SMS from $caller! Message: $message" }
    case<VoiceRecording> { "You received a Voice Recording from $contactName! Click the link to hear it: $link" }
}.match()
```

我们给 `MatcherBranch` 类加上一个变量来解决这个问题：

```kotlin
class MatcherBranch<R, O>(val value: O, val result: O.() -> R, val isDefault: Boolean = false) {

    fun get() = result.invoke(value)
}
```

然后我们修改一下 `default` 方法，再把 `match` 方法写下来：

```kotlin
fun default(result: R) {
    branches += MatcherBranch(value = value, result = { result }, isDefault = true)
}

fun match(): R {
    return branches
        .filter { !it.isDefault }
        .firstNotNullOfOrNull { it.get() } ?: branches.find { it.isDefault }?.get() ?: error("Match error.")
}
```

我们完成了，但直接这样使用非常蛋疼，我们在类外面编写两个顶层方法（在这里我有点作弊，因为 36 行并不包含下面的两个方法）：

```kotlin
fun <R, O> matcher(value: O, func: Matcher<R, O>.() -> Unit): Matcher<R, O> {
    return Matcher<R, O>(value).apply {
        func.invoke(this)
    }
}

@JvmName("extensionMatcher")
fun <R, O> O.matcher(func: Matcher<R, O>.() -> Unit): Matcher<R, O> {
    return Matcher<R, O>(this).also { matcher ->
        func.invoke(matcher)
    }
}
```

接下来我们就能愉快的使用它们了：

```kotlin
interface Notification

data class Email(val sender: String, val title: String, val body: String) : Notification

data class SMS(val caller: String, val message: String) : Notification

data class VoiceRecording(val contactName: String, val link: String) : Notification

VoiceRecording("Joe", "voicerecording.org/id/123").matcher {
    case<Email> { "You got an email from $sender with title: $title" }
    case<SMS> { "You got an SMS from $caller! Message: $message" }
    case<VoiceRecording> { "You received a Voice Recording from $contactName! Click the link to hear it: $link" }
    default("Match error.")
}.match()
// Result: You received a Voice Recording from Joe! Click the link to hear it: voicerecording.org/id/123

matcher(VoiceRecording("Joe", "voicerecording.org/id/123")) {
    case<Email> { "You got an email from $sender with title: $title" }
    case<SMS> { "You got an SMS from $caller! Message: $message" }
    case<VoiceRecording> { "You received a Voice Recording from $contactName! Click the link to hear it: $link" }
    default("Match error.")
}.match()
// Result: You received a Voice Recording from Joe! Click the link to hear it: voicerecording.org/id/123
```