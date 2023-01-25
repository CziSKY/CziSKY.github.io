+++
title = "The Little Schemer 笔记 —— 零"
date = 2023-01-24
[taxonomies]
categories = ["notes"]
tags = ["programming language", "scheme"]
+++

> 如果你的朋友说话后面经常带 `（`，那他就已经掌握了 Scheme 这门语言的精髓。（并不是，我瞎说的。我只是想开个玩笑

# Toys

`Atom` 和 `List` 都属于 `S-Expression`，`List` 是由 `S-Expression` 构成的集合（这意味着你可以构成 `((abc) ((cba)), ())` 这样的 `List`），而 `Atom` 可以看作是单个元素。例如 `abc`、`True`
和 `123` 都是 `Atom`。

- `()` 也是一个 `List`，被称为 `Null List`。
- 在命名上，通过操作符 `'` (Quote) 号来区分对象是否是一个 `S-Expression` 还是一个可被执行的函数体。例如 `(quote (+ 2 2))` 或者 `'(+ 2 2)` 是一个 `List`，而不会被求值 (Evaluate) 成 `4`。

车 (`car`) 和 成都人 (`cdr`)：

- `(car l)` 表示取 `l` 这个 `List` 中的头元素，就像 Haskell 的 `head` 函数。例如：`(car '((a b c) x y z)) => '(a b c)`
- `(cdr l)` 表示取 `l` 这个 `List` 中除了头元素的所有元素，就像 Haskell 的 `tail` 函数。当 `List` 只有一个元素时返回 `()`。就像 Haskell 中的 `tail`
  函数，例如：`(cdr '((a b c) x y z)) => '(x y z)`。

其中 `car` 和 `cdr` 都不能对 `()` 或 `Atom` 进行操作。

`(cons x l)` 表示把一个元素加在 `l` 的头部。就像 Haskell 的 `:` 操作符。例如 `(cons 'a (car '((b) c d))) => '(a b)`。第一个参数为 `S-Expression`，第二个参数必须为 `List`。

任意 `'a` 为 `S-Expression`，`'b` 为 `List`，可推出：

- `(car (cons 'a 'b)) => 'a`
- `(cdr (cons 'a 'b)) => 'b`

`(null? l)` 表示参数是否为 `Null List`，其中 `l` 必须为 `List`。例：`(null? '(a)) => #F`

`(atom? s)` 表示参数是否为 `Atom`，其中 `s` 为 `S-Expression`。它的定义如下：

```scheme
(define atom?
  (lambda (x)
    (and (not (pair? x))
         (not (null? x)))))
```

将 `and` 翻译成 `&&`，`not` 翻译成 `!` (即颠倒布尔值)。我就能理解这个定义了：对于参数 `x`，如果它既不满足 `pair? x` 的条件（这里没有给出来）也不是 `Null List`
，那它则是一个 `Atom`。

`(eq? a b)` 用来判断两个 `Atom` 参数是否相等，**其中两个参数不可以是数字**。

# Do It, Do It Again, and Again, and Again…

`lat` 表示为一个只有 `Atom` 的集合，而 `lat? l` 可以判断参数 `List` 是否满足这个条件。`lat?` 的定义：

```scheme
(define lat?
  (lambda (l)
    (cond
     ((null? l) #t)
     ((atom? (car l)) (lat? (cdr l)))
     (else #f))))
```

它是用我的老朋友递归实现的，在 Haskell 系列教程和 CS61A 我就已经见过 114514 次了。为了帮助理解我决定用简单的语言再解释一遍：

- 它接收一个参数 `l`，必须为 `List`。主体由两个条件语句组成。
- 第一个条件即判断 `l` 是否为一个空集合 `Null List`。如果是的话即返回 `True`。
- 第二个条件判断 `l` 的第一个元素是否是 `Atom`。
    - 如果是 `Atom`，即把移除头元素的集合再递给 `lat?` 进行调用。每次递归调用集合都会少去一个头元素，如果这个 `List` 都由 `Atom`
      组成的话，最后一次递归调用即会触发前面第一个条件。则返回 `True`。
    - 如果某一个元素为 `List`，则会跳转到下面 else 的部分，即返回 `False`。

同时，这里的 `cond` 是 Pattern Matching。

`(or? a b)` 接收两个布尔值参数，类似于常见编程语言中的 `||`。

`(member? a lat)` 来表示参数 `a` 是否为 `List lat` 中的一部分 (Contain)，它的实现同样使用了递归，定义如下：

```scheme
(define member?
  (lambda (a lat)
    (cond
     ((null? lat) #f)
     (else (or (eq? (car lat) a) (member? a (cdr lat)))))))
```

# Cons the Magnificent

`(rember a lat)` 表示从 `List lat` 里面移除第一个存在的 `a` 元素，定义如下：

```scheme
(define rember
  (lambda (a lat)
    (cond
     ((null? lat) (quote ()))
     ((eq? a (car lat)) (cdr lat))
     (else (cons (car lat)
                 (rember a (cdr lat)))))))
```

`(firsts l)` 表示取出 `List l` 中的每一个元素 `List` 中的第一个元素构成集合，定义如下：

```scheme
(define firsts
  (lambda (l)
    (cond
     ((null? l) (quote ()))
     (else (cons (car (car l)) (firsts (cdr l)))))))
```

`(cons (car (car l)) (firsts (cdr l)))` 的前部分 `cons (car (car l))` 可以称为 `Typical Element`。后半部分 `(firsts (cdr l))` 被称成 `Natural Recursion`。这种一种结构化思考递归程序的方式，按我自己的话来说这种方式可以提醒我们确认函数什么时候终止。（即什么时候该停止递归返回结果）

比如说针对 `((a b) (c d) (e f))`，它最开始的 `Typical Element` 只是 `'a`（第一次执行）。然后通过 `Natural Recursion` 又求出 `Typical Element` `'c` 和 `'e`。最后一次结束递归返回 `'()`。通过 `cons` 又把它们合在一起成为了 `'(a c e)`。

![](http://i.stack.imgur.com/QhBcl.png)

`(insertR a b lat)` 表示在 `List lat` 里，把 `Atom a` 插入到 `Atom b` 后。如 `(insertR 'a 'b '(c d b)) => '(c d b a)`。`(insertL a b lat)` 是它的对偶，即将 `Atom a` 插入到 `Atom b` 前面，定义如下：

```scheme
(define insertR
  (lambda (a b lat)
    (cond
     ((null? lat) (quote ()))
     ((eq? b (car lat)) (cons b
                              (cons a (cdr lat))))
     (else (cons (car lat)
                 (insertR a b (cdr lat)))))))
```

```scheme
(define insertL
  (lambda (a b lat)
    (cond
     ((null? lat) (quote ())))
     ((eq? b (car lat)) (cons a lat))
     (else (cons (car lat)
                 (insertL a b (cdr lat))))))
```

`(subst a b lat)` 表示在 `List lat` 中，用 `Atom a` 替代第一个出现的 `Atom b`。如 `(subst 'a 'b '(c b)) => '(c a)`，定义如下：

```scheme
(define subst
  (lambda (a b lat)
    (cond
     ((null? lat) (quote ()))
     ((eq? b (car lat)) (cons a (cdr lat)))
     (else (cons (car lat)
                 (subst a b (cdr lat)))))))
```

`(subst2 a b c lat)` 表示在 `List lat` 中，用 `Atom a` 替代第一个出现的 `Atom b` 或 `Atom c` 。如 `(subst2 'a 'b '(c b)) => '(c a)`，定义如下：

```scheme
(define subst2
  (lambda (a b c lat)
    (cond
     ((null? lat) (quote ()))
     ((or (eq? b (car lat))
          (eq? c (car lat))) (cons a (cdr lat)))
     (else (cons (car lat)
                 (subst2 a b c (cdr lat)))))))
```

`(multirember a lat)` 表示在 `List lat` 中删除所有的 `Atom a`。定义如下：

```scheme
(define multirember
  (lambda (a lat)
    (cond
     ((null? lat) (quote ()))
     ((eq? a (car lat)) (multirember a (cdr lat)))
     (else (cons (car lat)
                 (multirember a (cdr lat)))))))
```

`(multiinsertR a b lat)` 表示在 `List lat` 中，在所有的 `Atom b` 后面插入一个 `Atom a`。定义如下：

```scheme
(define multiinsertR
  (lambda (a b lat)
    (cond
     ((null? lat) (quote ()))
     ((eq? b (car lat)) (cons b
                              (cons a (multiinsertR a b (cdr lat)))))
     (else (cons (car lat)
                 (multiinsertR a b (cdr lat)))))))
```

`(multiinsertL a b lat)` 表示在 `List lat` 中，在所有的 `Atom b` 前面插入一个 `Atom a`。定义如下：

```scheme
(define multiinsertL
  (lambda (a b lat)
    (cond
     ((null? lat) (quote ()))
     ((eq? b (car lat)) (cons a (cons b (multiinsertL a b (cdr lat)))))
     (else (cons (car lat)
                 (multiinsertL a b (cdr lat)))))))
```

`(multisubst a b lat)` 表示在 `List lat` 中，把所有的 `Atom b` 替换成 `Atom a`。定义如下：

```scheme
(define multisubst
  (lambda (a b lat)
    (cond
     ((null? lat) (quote ()))
     ((eq? b (car lat)) (cons a (multisubst a b (cdr lat))))
     (else (cons (car lat)
                 (multisubst a b (cdr lat)))))))
```