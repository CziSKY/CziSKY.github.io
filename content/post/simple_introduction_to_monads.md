+++
title = "[译] Monads 简单引言"
date = 2022-11-13
[taxonomies]
categories = ["coding"]
tags = ["functional programming", "java"]

+++

> [原文](https://www.codeproject.com/Articles/5290753/Simple-Introduction-to-Monads)以 GPLv3 许可发布，ChristianNeumanns 版权所有。

## 简介

Monads 在大多数纯函数式编程语言中都被大量使用。例如在 Haskell 中，它们是必不可少的，并且出现在各种应用和库 （轮子）中。

另一方面，Monads 却很少用于流行的、非纯函数式的编程语言中，例如 C#、Java 和 Python 等。

为什么会有这么大的差异？

要找到答案，我们首先要知道：

- 什么是 Monad？
- 为什么它们会被用在函数式编程语言中？它们解决了什么问题？
- 我们如何在 C#、Java、Python 等语言中使用 Monad？我们应该这么做么？

看完这篇文章，希望你能对这些问题对答如流。

>本文针对的是具有良好的非函数式编程语言背景的软件开发人员。
>
>文章中的例子使用 Java 编写，但不要求你是 Java 专家，因为它们仅涉及基本的 Java 知识。这些例子在 C# 中看起来非常相似，而且可以容易地用其他支持范型 （类型参数）和高阶函数（可以将一个函数作为输入参数的函数，以及返回函数）的语言来重写它们。
>
>[Gitlab](https://gitlab.com/ppl-lang/blog/-/tree/master/2020-03-Monad_Intro/Java_examples/monadtests) 上提供了完整的源代码

## 简单的问题

假设我们必须编写一个简单的函数，它会对输入的字符串进行如下转换：

- 移除两侧空格。
- 将所有字母转换为大写。
- 在尾部附加一个感叹号。

举个例子，用 `" Hello bob"` 作为函数输入应返回 `"HELLO BOB!"`。

## Java 里的简单解决方案

在大多数编程语言中，这是很容易做到的。这里是一个 Java 的解决方案：

```java
static String enthuse(String sentence) {
    return sentence.trim().toUpperCase().concat("!");
}
```

为了编写一个包含基本测试的完整 Java “应用程序”，我们将以下代码放在文件 *MonadTest_01.java* 里：

```java
public class MonadTest_01 {
    
    static String enthuse(String sentence) {
        return sentence.trim().toUpperCase().concat("!");
    }
    
    public static void main(String[] args) {
        System.out.println(enthuse("  Hello bob  "));
    }
}
```

然后我们可以通过以下命令编译和运行该程序：

```bash
javac MonadTest_01.java
java MonadTest_01
```

输出看起来符合预期：

```
HELLO BOB!
```

到目前为止还不错。

## 只要函数！

在之前的 Java 例子中，我们使用例如像 `sentence.trim()` 的对象方法。然而，由于这篇文章是关于 Monads 的，我们必须意识到纯函数式语言并没有在对象上执行的方法。基于 [Lambda 演算](https://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97)的函数式编程语言只有无副作用的函数，接受一个输入，然后返回一个结果。

因此，让我们仅使用纯函数来重写之前的代码，依然使用 Java。这一点很重要，因为我们必须使用函数，才能最终理解为什么要发明 Monads。

这里是重写后的代码：

```java
static String trim(String string) {
    return string.trim();
}

static String toUpperCase(String string) {
    return string.toUpperCase();
}

static String appendExclam(String string) {
    return string.concat("!");
}

static String enthuse(String sentence) {
    return appendExclam(toUpperCase(trim(sentence)));
}

public static void test() {
    System.out.println(enthuse("  Hello bob  "));
}
```

代码从三个纯函数 （`trim`、`toUppercase`、`appendExclam`）开始，它们接收一个字符串作为输入，并返回一个字符串作为结果。也许你会觉得我在作弊，因为我仍然在函数体内使用对象方法（例如：`string.trim()`）。但这在这里并不重要，因为在这个练习中，我们并不关心这个函数的实现，我们关注的是它的**类型签名**。

有趣的部分是 `enthuse` 函数体。

```java
return appendExclam(toUpperCase(trim(sentence)));
```

我们可以看到里面只有函数调用（就像在纯函数式语言中一样）。调用是嵌套的，并像这样执行。

- 第一步：执行 `trim(sentense)`。
- 第二步：将第一步的结果输入给 `toUpperCase`。
- 第三步：将第二步的结果输入给 `appendExclam`。
- 第四步：将第三步的结果作为函数 `enthuse` 的结果返回。

![](../hello_bob_functions_chains.png)

为了查看一切是否仍然正常，我们可以执行测试，结果仍然是一样的。

```
HELLO BOB!
```

## 函数复合

在函数式编程语言中，嵌套的函数调用（例如我们的：`appendExclam(toUpperCase(trim(sentence)))`）被称为**函数复合**。

>函数复合 (*Function composition*) 是函数式编程语言里的面包和黄油。在 Lambda 演算中，一个函数体是一个单一的表达式，复杂的表达式可以通过函数复合来创建。
>
>![](../function_composition.png)

正如我们将在后面看到的，Monad 为我们复合函数时可能出现的问题提供了一个解决方案。但是在深入之前，让我们看看在不同环境下使用函数复合的变化。熟悉这一重要概念的读者可以跳到下一章。

### Unix 管道

首先值得注意的是，函数复合的想法与 Unix/Linux 中的管道类似。第一条命令的输出作为输入被送入第二条命令。然后，第二条命令的输出被作为输入送到第三个命令中以此类推。在 Unix/Linux 中，符号 `|` 被用于管道命令。下面是一个计算文件名中含有 "page" 的文件数量的命令例子（借用自 [How to Use Pipes on Linux](https://www.howtogeek.com/438882/how-to-use-pipes-on-linux/)）。

```bash
ls - | grep "page" | wc -l
```

### 管道操作符 (Pipe Operator)

因为管道在许多情况下都十分有用，一些编程语言有专门的管道操作符。例如 F# 使用 ` |>` 来链化函数调用。如果 Java 有这个操作符，则函数 enthuse 就可以写成下面这种形式：

```
static String enthuse(String sentence) {
    return trim(sentence) |> toUpperCase |> appendExclam;
}
```

它们拥有相同的语意，但这种形式比使用嵌套函数进行调用的 Java 更易读。

```java
static String enthuse(String sentence) {
    return appendExclam(toUpperCase(trim(sentence)));
}
```

### 函数复合操作符

由于函数复合是必不可少的，大多数函数式编程语言都有一个专门的函数复合操作符，使函数的复合变得非常简单。

例如在 Haskell 中，点 (`.`) 被用来复合函数 （源自数学中的环形运算符号 `∘`）。点本身就是一个函数，其类型签名定义如下：

```haskell
(.) :: (b -> c) -> (a -> b) -> a -> c
```

该函数接收两个函数作为输入 (`b → c` 和 `a → b`)，并返回另一个函数 (`a → c`)，它是两个输入函数的复合。

因此，要说明函数 `h` 是函数 `f` 和 `g` 的复合，在 Haskell 可以这样表示：

```haskell
h = f . g
```

注意，点运算符在 Haskell 和像 C#、Java 等面向对象语言中的语义完全不同。在 Java 中，`f.g` 意味着在对象 `f` 上应用 `g` (例如：`person.name`)。在 Haskell 中，它意味着将函数 `f` 和 `g` 进行复合。

F# 使用 `>>` 来复合函数，它的定义是这样的：

```F#
let (>>) f g x = g(f(x))
```

而他的使用方法如下：

```F#
let h = f >> g
```

>不要把 F# 的 `>>` 操作符与 Haskell 中的 Monad 排序操作符相混淆，后者也使用符号 `>>`。

如果 Java 有一个类似于 F# 的函数复合语法，那么函数 `enthuse` 可以简单地写成这样：

```
static String enthuse (String sentence) = trim >> toUpperCase >> appendExclam;
```

## 错误，但不是异常

为了本教程的目的，假设我们的函数可能会因以下几点导致执行失败：

- 如果输入的字符串是空的或仅包含空格，则函数 `trim` 执行失败 （即结果并不能是一个空字符串）。
- 如果输入的字符串是空的或者包含字母或空格以外的字符，则函数 `toUpperCase` 执行失败。
- 如果输入的字符串超过二十个字符，则函数 `appendExclam` 执行失败。

在 Java 中，我们习惯性的抛出异常 (*Exception*) 来表示一个错误。但是纯函数式语言不支持异常，因为函数不能有副作用。一个可能失败的函数必须将错误信息作为函数返回结果的一部分。例如函数在执行成功的情况下返回一个字符串，在出现错误的情况下返回一个错误数据。

所以让我们用 Java 来实现吧！首先，我们定义一个简单的错误类，其中有一个描述错误信息的字段。

```java
public class SimpleError {
    
    private final String info;

    public SimpleError(String info) {
        this.info = info;
    }
    
    public String getInfo() { return info; }

    public String toString() { return info; }
}
```

如前所述，这些函数必须能够在执行成功的情况下返回一个字符串，否则就是一个错误对象。为了实现这一点，我们可以定义 `ResultOrError` 类：

```java
public class ResultOrError {
    
    private final String result;
    private final SimpleError error;
    
    public ResultOrError(String result) {
        this.result = result;
        this.error = null;
    }

    public ResultOrError(SimpleError error) {
        this.result = null;
        this.error = error;
    }
    
    public String getResult() { return result; }
    
    public SimpleError getError() { return error; }
    
    public boolean isResult() { return error == null; }
    
    public boolean isError() { return error != null; }
    
    public String toString() {
        if (isResult()) {
            return "Result: " + result; 
        } else {
            return "Error: " + error.getInfo(); 
        }
    }
}
```

正如我们所看到的：

- 该类有两个不可变的字段，用来保存结果或错误。
- 有两个构造函数 (*Constructor*)：
  - 第一个构造函数用于成功的情况（例：`return new ResultOrError("hello");`）。
  - 第二个构造函数用于失败的情况（例：`return new ResultOrError(new Error("Something went wrong"));`）。
- `isResult` 和 `isError` 是工具函数。
- `toString` 是为了调试目的 (`Debugging`) 而复写的。

为了包含错误处理，我们重写上面的三个工具函数：

```java
public class StringFunctions {

   public ResultOrError trim(String string) {
        String result = string.trim();
        if (result.isEmpty()) {
            return new ResultOrError(new SimpleError("String must contain non-space characters."));
        }
        return new ResultOrError(result);
    }

    public ResultOrError toUpperCase(String string) {
        if (!string.matches("[a-zA-Z ]+")) {
            return new ResultOrError(new SimpleError("String must contain only letters and spaces."));
        }
        return new ResultOrError(string.toUpperCase());
    }

    public ResultOrError appendExclam(String string) {
        if (string.length() > 20) {
            return new ResultOrError(new SimpleError("String must not exceed 20 characters."));
        }
        return new ResultOrError(string.concat("!"));
    }
}
```

> 为了使这个练习代码变得简单，我们不会像在生产环境下检查和处理空值。假设上述代码中其中一个函数被调用，输入值为 `null`，我们只需接受。即使它会抛出 `NullPointerException`。

重要的是，之前返回字符串的三个函数现在都返回了一个 `ResultOrNull` 对象。

因此，函数 `enthuse` 的定义如下：

```java
static String enthuse(String sentence) {
    return appendExclam(toUpperCase(trim(sentence)));
}
```

······然而并没有起任何作用。

不幸的，现在的函数复合无效了，因为现在的函数返回一个 `ResultOrError` 对象。但需要一个字符串作为输入，输入和输出类型不再匹配了，这些函数不能再被链接起来了。

在之前的代码中，当函数返回字符串时，一个函数的输出可以输入到下一个函数中。

![](../string_functions_chains.png)

但现在这样做是不行的了。

![](../string_ROE_functions.png)

然而，我们仍然在 Java 中像这样实现 `enthuse`：

```java
static ResultOrError enthuse(String sentence) {
    ResultOrError trimmed = trim(sentence);
    if (trimmed.isResult()) {
        ResultOrError upperCased = toUpperCase(trimmed.getResult());
        if (upperCased.isResult()) {
            return appendExclam(upperCased.getResult());
        } else {
            return upperCased;
        }
    } else {
        return trimmed;
    }
}
```

不太好！最开始一行实现的代码已然变成一个丑陋的怪物，我们可以改进一下。

```java
static ResultOrError enthuse_2(String sentence) {
    ResultOrError trimmed = trim(sentence);
    if (trimmed.isError()) return trimmed;

    ResultOrError upperCased = toUpperCase(trimmed.getResult());
    if (upperCased.isError()) return upperCased;

    return appendExclam(upperCased.getResult());
}
```

这种代码在 Java 与其他许多编程语言中都可以使用，但它肯定不是我们想要反复编写的代码。错误处理和正常流程代码混在一起，使得代码难以阅读、编写和维护。

更重要的是，我们根本无法在纯函数式编程语言中编写这样的代码。**一个函数返回的表达式只能由函数复合组成**。

我们很容易想象出其他导致同样困境的案例。那么，在同一问题出现多种变化的情况下，我们应该怎么做呢？是的，我们应该尝试找到一个可以在最多情况下使用的一般解决方案。

Monads 来拯救我们了！ 它为这种问题提供了一个通用的解决方案，而且它们还有其他的好处。正如我们在后面所看到的，Monad 使所谓的 Monadic 函数能够被复合。这些函数不能被直接复合，因为它们的类型是不兼容的。

有些人说：“如果 Monads 不存在，你可以发明它们”（Brian Beckman 在他的精彩演讲 [Don't Fear the Monad](https://www.youtube.com/watch?v=ZhuHCtR3xq8) 中提到），这是真的！

因此，我们先抛下 Monad，让我们自己一步步地思考来找到解决方案。

## “绑定” 函数 (bind)

在函数式编程语言中，一切都由函数来完成。所以我们已经知道必须创建一个函数来解决我们的问题。

我们把这个函数称为 `bind`，因为它的作用是绑定两个不能直接复合的函数。

接下来我们必须决定 `bind` 的输入是什么，以及它应该返回什么。让我们来考虑链接函数 `trim` 和 `toUpperCase` 的情况。

![](../trim_toUppercase_functions_chains.png)

要实现的逻辑必须按以下方式工作：

- 如果 `trim` 返回一个字符串，那么就可以调用 `toUpperCase`。因为它需要一个字符串作为输入，所以最后的输出将会是 `toUpperCase` 的输出。
- 如果 `trim` 返回一个错误，那么 `toUpperCase` 就不能被调用。必须简单地转发这个错误。所以最后的输出将会是 `trim` 的输出。

我们可以推断，`bind` 需要两个参数。

- `trim` 的返回结果，其类型为 `ResultOrError`。
- 函数 `toUpperCase`，因为如果 `trim` 是一个字符串，那么 `bind` 必须调用 `toUpperCase`。

`bind` 的输出类型很容易确定。如果 `trim` 返回一个字符串，那么 `bind` 的输出就是 `toUpperCase` 的输出，它的类型是 `ResultOrError`。如果 `trim` 失败了，那么 `bind` 的输出就是 `trim` 的输出，它也是 `ResultOrError` 类型的。由于两种情况下的输出类型都是 `ResultOrError`，所以 `bind` 的输出类型也必须是 `ResultOrError`。

所以现在我们知道了 `bind` 的类型签名：

![](../string_ROE_bind_function.png)

在 Java 里，我们可以这样写：

```java
ResultOrError bind(ResultOrError value, Function<String, ResultOrError> function)
```

实现 `bind` 很简单，因为我们清楚地知道我们要做什么：

```java
static ResultOrError bind(ResultOrError value, Function<String, ResultOrError> function) {
    if (value.isResult()) {
        return function.apply(value.getResult());
    } else {
        return value;
    }
}
```

现在可以将函数 `enthuse` 重写成这样：

```java
static ResultOrError enthuse(String sentence) {
    ResultOrError trimmed = trim(sentence);

    ResultOrError upperCased = bind(trimmed, StringFunctions::toUpperCase);
    // alternative:
    // ResultOrError upperCased = bind ( trimmed, string -> toUpperCase(string) );

    ResultOrError result = bind(upperCased, StringFunctions::appendExclam);
    return result;
}
```

但这依旧是命令式的代码 （一连串的语句）。如果我们做的很好，那么我们一定能够通过函数复合来重写 `enthuse`。而事实上，我们可以这么做：

```java
static ResultOrError enthuse_2(String sentence) {
    return bind(bind(trim(sentence), StringFunctions::toUpperCase), StringFunctions::appendExclam);
}
```

乍一看，`bind` 的函数体可能有点混乱。我们会在后面改变这一点，重点是我们在 `enthuse` 函数体内只使用了函数复合。

> 如果你从未见过这种风格的代码，那么请你慢慢消化并充分理解在这个函数内发生了什么。理解 `bind` 是理解 Monads 的关键！
>
> `bind` 是在 Haskell 中的函数名，还有一些替代名称。例如：`flatMap`、`chain` 和 `andThen`。

它是否能够正常工作？让我们来测试一下。这里有一个包含 `bind` 的类，`enthuse` 的两种编码风格版本（命令式与函数式）。以及一些简单的测试，涵盖了成功和所有失败的情况：

```java
public class MonadTest_04 {
    
    static ResultOrError bind(ResultOrError value, Function<String, ResultOrError> function) {
        if (value.isResult()) {
            return function.apply(value.getResult());
        } else {
            return value;
        }
    }

    static ResultOrError enthuse(String sentence) {
        ResultOrError trimmed = trim(sentence);

        ResultOrError upperCased = bind(trimmed, StringFunctions::toUpperCase);
        // alternative:
        // ResultOrError upperCased = bind ( trimmed, string -> toUpperCase(string) );

        ResultOrError result = bind(upperCased, StringFunctions::appendExclam);
        return result;
    }

    static ResultOrError enthuse_2(String sentence) {
        return bind(bind(trim(sentence), StringFunctions::toUpperCase), StringFunctions::appendExclam);
    }

    private static void test(String sentence) {
        System.out.println(enthuse(sentence));
        System.out.println(enthuse_2(sentence));
    }

    public static void tests() {
        test("  Hello bob  ");
        test("   ");
        test("hello 123");
        test("Krungthepmahanakhon is the capital of Thailand");
    }
}
```

`test` 函数的运行输出：

```
Result: HELLO BOB!
Result: HELLO BOB!
Error: String must contain non-space characters.
Error: String must contain non-space characters.
Error: String must contain only letters and spaces.
Error: String must contain only letters and spaces.
Error: String must not exceed 20 characters.
Error: String must not exceed 20 characters. 
```

上面定义的 `bind` 函数仅为我们的需求服务。为了让它成为 Monad 的一部分，我们必须让它变得更加通用。我们将很快做到这一点。

> 如上所示，使用 `bind` 是解决函数复合问题的常见方法。但这并不是唯一的方法。另一种方法叫做 Kleisli 复合 (*Kleisli composition*)，这不在本文的讨论范围内。

## 终于，一个 Monad！

现在我们有了 `bind`，剩下的步骤就很容易了。我们只需要做一些改进，以便有一个可以适用于其他情况的解决方案。

在这一章内我们的目标很明确，掌握这一模式，改进 `ResultOrError`。

### 第一步改进

在之前的章节中，我们将 `bind` 声明成一个独立的函数，满足了我们的特殊需求。第一步改进是将 `bind` 函数移到 `ResultOrError` 类中，函数 `bind` 必须是 Monad 类的一部分。原因是 `bind` 的实现基于使用 `bind` 的 Monad。虽然 `bind` 的类型签名总是相同的，但对于不同的 Monad 我们使用不同的实现 (*Implementions*) 。

### 第二步改进

在我们的范例代码中，复合的函数都需要一个字符串作为输入，并返回一个字符串或一个错误对象。如果我们想要复合的函数接收一个整数 (*Integer*)，并返回一个整数或一个错误对象呢？我们可以改进 `ResultOrError` 使其适用于任何类型的结果吗？当然可以。我们只需要给 `ResultOrError` 加上一个类型参数 (*Type Parameter*)。

这是将 `bind` 移入类并加上类型参数后的新版本：

```java
public class ResultOrErrorMona<R> {

    private final R result;
    private final SimpleError error;

    public ResultOrErrorMona(R result) {
        this.result = result;
        this.error = null;
    }

    public ResultOrErrorMona(SimpleError error) {
        this.result = null;
        this.error = error;
    }

    public R getResult() {
        return result;
    }

    public SimpleError getError() {
        return error;
    }

    public boolean isResult() {
        return error == null;
    }

    public boolean isError() {
        return error != null;
    }

    static <R> ResultOrErrorMona<R> bind(ResultOrErrorMona<R> value, Function<R, ResultOrErrorMona<R>> function) {
        if (value.isResult()) {
            return function.apply(value.getResult());
        } else {
            return value;
        }
    }

    public String toString() {
        if (isResult()) {
            return "Result: " + result;
        } else {
            return "Error: " + error.getInfo();
        }
    }
}
```

`ResultOrErrorMona`，注意看这个类名。我并没有拼错它，因为这个类目前还不是一个 Monad，这么命名它仅是为了好玩。

### 第三步改进

假设我们必须将下面两个函数链接起来：

```java
ResultOrError<Integer> f1(Integer value)
ResultOrError<String>  f2(Integer value)
```

这是一张用来说明这一点的图片：

![](../integer_string_functions_chain.png)

我们的 `bind` 函数还不能处理这种情况，因为两个函数的输出类型不同 （`ResultOrError<Integer>` 和 `ResultOrError<String>`）。我们必须使 `bind` 更加通用，以便不同值类型的函数能够被链接起来。`bind` 的类型签名必须改变，从：

```java
static <R> Monad<R> bind(Monad<R> monad, Function<R, Monad<R>> function)
```

······到：

```java
static <R1, R2> Monad<R2> bind(Monad<R1> monad, Function<R1, Monad<R2>> function)
```

`bind` 的实现也必须调整，这里是新的类：

```java
public class ResultOrErrorMonad<R> {

    private final R result;
    private final SimpleError error;

    public ResultOrErrorMonad(R result) {
        this.result = result;
        this.error = null;
    }

    public ResultOrErrorMonad(SimpleError error) {
        this.result = null;
        this.error = error;
    }

    public R getResult() {
        return result;
    }

    public SimpleError getError() {
        return error;
    }

    public boolean isResult() {
        return error == null;
    }

    public boolean isError() {
        return error != null;
    }

    static <R1, R2> ResultOrErrorMonad<R2> bind(ResultOrErrorMonad<R1> value, Function<R1, ResultOrErrorMonad<R2>> function) {
        if (value.isResult()) {
            return function.apply(value.result);
        } else {
            return new ResultOrErrorMonad<R2>(value.error);
        }
    }

    public String toString() {
        if (isResult()) {
            return "Result: " + result.toString();
        } else {
            return "Error: " + error.toString();
        }
    }
}
```

注意看类名，`ResultOrErrorMonad`。

是的，这是一个 Monad。

>在现实中，我们不会为属于 Monad 的类型添加 "Monad" 后缀，之所以把这个类命名为 `ResultOrErrorMonad` 是为了给读者表明这个类是一个 Monad。

我们怎么能确认这个类就是一个 Monad 呢？

虽然 Monad 这个词在数学中拥有非常精确的定义 （就像在数学中的所有东西一样），但是在编程语言的世界里，这个词还没有明确的定义。然而，维基百科上有一个[常见的定义](https://zh.wikipedia.org/wiki/%E5%8D%95%E5%AD%90_(%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B))。一个 Monad 由三部分组成：

> 译者著：下面的定义摘抄自维基百科，其中的单子指 Monad。

- **[类型构造器](https://zh.wikipedia.org/wiki/类型构造子) (*Type Constructor*) `M`，建造一个单子类型`M T`。**

  换句话说，Monad 中包含的值有一个类型参数，在我们的例子中，它是该类声明中的类型参数 `R`。

  ```java
  class ResultOrErrorMonad<R>
  ```

- **[类型转换子](https://zh.wikipedia.org/wiki/类型转换) (Type Converter)，经常叫做 `unit` 或 `return`，将一个对象 `x` 嵌入到单子中。**

  在 Haskell 中，类型转换器的定义是：`return :: a -> m a`

  在类 Java 这样的语言中，这意味着必须有一个构造函数，它接收一个 `R` 类型的值，并返回一个包含该值，类型参数为 `M<R>` 的 Monad。

  在我们的代码中，它是 `ResultOrErrorMonad` 类的构造函数。

- **[组合子](https://zh.wikipedia.org/wiki/组合子) (*Combinator*)，典型的叫做 `bind`（[约束变量](https://zh.wikipedia.org/wiki/约束变量)的那个 bind），并表示为[中缀算子](https://zh.wikipedia.org/wiki/中缀表示法) `>>=`，去包装一个单体变量，接着把它插入到一个单体函数/表达式之中，结果为一个新的单体值：**

  ```haskell
  (mx >>= f) :: (M T, T -> M U) -> M U
  ```

  在 Haskell 中，`bind` 定义为 `(>>=) :: m a -> (a -> m b) -> m b`。

  在我们的代码中，它是 `bind` 函数：

  ```java
  <R1, R2> ResultOrErrorMonad<R2> bind(ResultOrErrorMonad<R1> value, Function<R1, ResultOrErrorMonad<R2>> function)
  ```

维基百科上随后指出：“但要完全具备单子资格，这三部分还必须遵守一些定律：······”。

在 [Haskell](https://hackage.haskell.org/package/base-4.12.0.0/docs/Control-Monad.html#t:Monad) 中，三条定律定义如下：

- ```haskell
  return a >>= k = k a
  ```

- ```haskell
  m >>= return = m
  ```

- ```haskell
  m >>= (\x -> k x >>= h) = (m >>= k) >>= h
  ```

讨论这些定律超出了本文的范围 （本篇文章仅是关于 Monad 的介绍），这些定律确保了 Monads 在任何情况都能良好运行。违反这些定律会导致微妙和痛苦的 Bug，[这里](https://www.reddit.com/r/haskell/comments/16iakr/what_happens_when_a_monad_violates_monadic_laws/)、[这里](https://stackoverflow.com/questions/12617664/a-simple-example-showing-that-io-doesnt-satisfy-the-monad-laws)、和[这里](https://www.quora.com/What-would-be-the-practical-implications-if-an-implementation-of-Haskells-Monad-typeclass-didnt-satisfy-the-monadic-laws)都有解释。据我所知，目前还没有任何一个编译器能够强制执行 Monad 定律。因此保证这些定律是否正确的被应用成为了开发者的责任，我只想说上面的 `ResultOrErrorMonad` 满足了 Monad 定律。

## 最大程度的提高复用性

除了给结果提供一个类型参数外，我们还可以为错误提供一个类型参数。这使得 Monad 更加可重用，因为现在你可以自由决定他们想要哪种类型的错误。举个例子，你可以看一下 F# 的 [`Result`](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/results) 类型。

最后，我们通过让程序员定义两个值的含义来让 Monad 更加可重复使用。在我们的例子中，一个值代表结果，而另一个代表错误。但是我们可以更抽象一点，我们可以创建一个 Monad，简单地容纳两个可能出现的值中的其中一个 —— `value_1` 或 `value_2`。而每个值的类型都可以由一个类型参数自由定义，这个 Monad 确实被一些函数式编程语言所支持。在 Haskell 中，它被称为 `Either`。它的构造函数被这样定义：

```haskell
data Either a b = Left a | Right b
```

以我们的 `ResultOrErrorMonad` 作为起点，在 Java 实现一个 `Either Monad` 是轻松的。

>有些项目对可能执行失败的函数使用 `Either Monad`。在我看来，使用 `ResultOrError` 类型是一个更好的、不容易出错的选择（原因不在这里解释）。

## 面向对象化

现在我们知道了 Monad 在函数式编程语言的作用，让我们回到 OOP （*Object-Oriented Programming*，面向对象编程）的世界。我们能不能创建一个类似 OO-Monad 的东西？

如果我们看一下 `ResultOrErrorMonad` 类，我们可以看到这个类里的所有东西都十分标准。只有一个例外，函数 `bind` 是该类的一个静态成员。这意味着我们不能对 `bind` 使用对象方法的点语法来调用。目前调用 `bind` 的语法为 `bind(v, f)`。但如果 `bind` 如果是类的非静态成员，我们就可以写成 `v.bind(f)`。这将使语法在嵌套调用的情况下更具可读性。

幸运的是，使 `bind` 变为非静态成员是容易的。

为了使 Monad 的功能更全面，我们也为错误值引入第二个类型参数。那么程序员就不需要使用 `SimpleError` 了，他们可以定义并使用自己的错误类。下面是面向对象风格的 `ResultOrError` 类：

```java
public class ResultOrError<R, E> {

    private final R result;
    private final E error;

    private ResultOrError(R result, E error) {
        this.result = result;
        this.error = error;
    }
    
    public static <R, E> ResultOrError<R, E> createResult(R result) {
        return new ResultOrError<R, E>(result, null);
    }

    public static <R, E> ResultOrError<R, E> createError(E error) {
        return new ResultOrError<R, E>(null, error);
    }

    public R getResult() {
        return result;
    }

    public E getError() {
        return error;
    }

    public boolean isResult() {
        return error == null;
    }

    public boolean isError() {
        return error != null;
    }

    public <R2> ResultOrError<R2, E> bind(Function<R, ResultOrError<R2, E>> function) {
        if (isResult()) {
            return function.apply(result);
        } else {
            return createError(error);
        }
    }

    public String toString() {
        if (isResult()) {
            return "Result: " + result.toString();
        } else {
            return "Error: " + error.toString();
        }
    }
}
```

现在 `enthuse` 函数体中使用 `bind` 的代码变得更加可读了，下面是之前的代码：

```java
return bind(bind(trim(sentence), v -> toUpperCase(v)), v -> appendExclam(v));
```

我们可以避免嵌套写法，下面是现在的代码：

```java
return trim(sentence).bind(v -> toUpperCase(v)).bind(v -> appendExclam(v));
```

所以 Monad 在面向对象语言的世界中有用么？是的，它们**可以有用**。

需要强调一下 “可以” 这个词，因为这通常取决于我们想要实现什么。比方说，我们有一些很好的理由来 “不使用异常” 来处理错误，还记得我们在 *错误，但不要异常*章节中不得不写的丑陋错误处理代码么？：

```java
static ResultOrError enthuse(String sentence) {
    ResultOrError trimmed = trim(sentence);
    if (trimmed.isResult()) {
        ResultOrError upperCased = toUpperCase(trimmed.getResult());
        if (upperCased.isResult()) {
            return appendExclam(upperCased.getResult());
        } else {
            return upperCased;
        }
    } else {
        return trimmed;
    }
}
```

使用 Monad 来干掉样板代码：

```java
    static ResultOrError enthuse(String sentence) {
        return trim(sentence).bind(v -> toUpperCase(v)).bind(v -> appendExclam(v));
    }
```

太棒了！

## 摘要

理解 Monad 的关键是理解 `bind`（也叫 `chain`、`andThen` 等等）。函数 `bind` 是用来复合两个 Monadic 函数的。一个 Monadic 函数是一个接收 `T` 类型的值并返回一个包含该值对象的函数，Monadic 函数不能直接复合，因为调用的第一个函数类型输出类型与第二个函数的输入类型不兼容。`bind` 解决了这个问题。

函数 `bind` 本身已经很有用了，但它仅是 Monad 的一部分。

像 Java 的世界里，一个 Monad 是具有以下特征的类 （类型）`M`。

- 一个类型参数 `T`，它定义了存储在 Monad 中的值类型 （例如：`M<T>`）。

- 一个构造函数，接收一个 `T` 类型的值，并返回一个 Monad 且包含了 `M<T>` 值。

  - 在类似于 Java 的语言中：

    ```java
    M<T> create(T value)
    ```

    ![](../create_function.png)

  - 在 Haskell 中：

    ```haskell
    return :: a -> m a
    ```

- 用来复合两个 Monadic 函数的 `bind` 函数：

  - 在类似于 Java 的语言中：

    ```java
    M<T2> bind(M<T1> monad, Function<T1, M<T2>> function)
    ```

    ![](../generic_bind_function.png)

  - 在 Haskell 中：

    ```haskell
    (>>=) :: m a -> (a -> m b) -> m b
    ```

一个 Monad 必须遵守三条 Monad 定律，这些定律确保 Monad 在所有情况下都能良好运行。

Monad 主要用于函数式编程语言，因为这些语言依赖函数复合。但是它们也可以在其他范式的编程语言发挥作用，例如支持范型和高阶函数的面向对象编程语言。

## 结语

正如标题所提到的，这篇文章仅是对 Monad 的介绍。它没有涵盖 Monad 的所有范围，没有展示 Monad 其他有用的例子（例：`Maybe Monad`、`IO Momad`、`State Monad` 等）。而且它完全忽略了范畴论 (*Category Theory*)。即 Monad 的数学背景，对于那些想了解更多的人来说，互联网上有大量信息可以去查阅学习它们。

希望这篇文章帮助大家掌握 Monad 的要领，看到它们的魅力，并了解它们如何改进代码和简化你的生活。

**"HAPPY MONADING!"**
