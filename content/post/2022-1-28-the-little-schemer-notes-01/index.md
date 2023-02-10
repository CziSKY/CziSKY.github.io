+++
title = "The Little Schemer 笔记 —— 壹"
date = 2023-02-09
[taxonomies]
categories = ["notes"]
tags = ["programming language", "scheme"]
+++

# Number Games

整数集 ℤ 内的所有元素均为 `Atom`，在此基础上我们可以定义一些函数：

`(add1 n)` 表示为将整数 `Atom n` + 1，例如 `(add1 '68) => '69` 定义如下：

```scheme
(define add1
  lambda (n)
    (+ n 1)))))
```

`(sub1 n)` 表示为将整数 `Atom n` - 1，例如 `(sub1 '68) => '67` 定义如下：

```scheme
(define sub1
  lambda (n)
    (- n 1)))))
```

`(o+ m n)` 将整数 `Atom m` 与 `Atom n` 相加，例如 `(o+ '1 '2) => '3`。定义为：

```scheme
(define o+
  (lambda (m n)
   (cond
    ((zero? m) n)
    (else (add1 (o+ n (sub1 m)))))))
```

`(o- m n)` 将整数 `Atom m` 与 `Atom n` 相减，例如 `(o+ '2 '1) => '1`。定义为：

```scheme
(define o-
  (lambda (m n)
   (cond
    ((zero? m) n)
    (else (sub1 (o- n (sub1 m)))))))
```

`tup` 是由一个数字构成的 `List`，例如 `'(1, 2, 3)`，`()` 也可表示为 `Null tup`。

`(add tup)` 求 `tup` 中的所有数字之和，定义为：

```scheme
(define addtup
  (lambda (tup)
    (cond
     ((null? tup) 0)
     (else (o+ (car tup)
               (addtup (cdr tup)))))))
```

`(o* a b)` 返回 `a * b`

```scheme
(define o*
  (lambda (a b)
    (cond
     ((zero? b) 0)
     (else (o+ a (o* a (sub1 b)))))))
```

`(tup+ tup1 tup2)` 将 `tup1` 中元素与 `tup2` 元素进行相加，对于空元素则为 `0`。例如：`(tup+ '(1 2 3) '(3 4)) => '(4 6 3)`，定义如下：

```scheme
(define tup+
  (lambda (tup1 tup2)
    (cond
     ((and (null? tup1)
           (null? tup2)) '())
     ((null? tup1) tup2)
     ((null? tup2) tup1)
     (else (cons (o+ (car tup1) (car tup2))
                 (tup+ (cdr tup1) (cdr tup2)))))))
```

`(> a b)` 返回整数 `Atom a` 是否大于整数 `Atom b`，根据下面的定义我们也能轻松推出 `(< a b)`。定义如下：

```scheme
(define >
  (lambda (a b)
    (cond
     ((zero? a) #f)
     ((zero? b) #t)
     (else (> (sub1 a) (sub1 b))))))
```

`(= a b)` 返回整数 `Atom a` 是否等于整数 `Atom b`，定义如下：

```scheme
(define =
  (lambda (a b)
    (cond
     ((> a b) #f)
     ((< a b) #f)
     (else #t))))
```

`(o/ a b)` 返回整数 `Atom a` 除以 `Atom b` 的结果，定义如下：

```scheme
(define o/
  (lambda (a b)
    (cond
     ((< a b) 0)
     (else (add1 (o/ (o- a b) b))))))
```

`(length lat)` 计算 `lat` 的长度，定义如下：

```scheme
(define length
  (lambda (lat)
    (cond
     ((null? lat) 0)
     (else (add1 (length (cdr lat)))))))
```

`(pick n lat)` 返回计算 `lat` 中第 `n` 个元素，定义如下：

```scheme
(define pick
  (lambda (n lat)
    (cond
     ((zero? (sub1 n)) (car lat))
     (else (pick (sub1 n) (cdr lat))))))
```

`(rempick n lat)` 返回去掉第 `n` 个元素后的 `lat`，跟 Haskell 的 `choose` 函数等价，定义如下：

```scheme
(define rempick
  (lambda (n lat)
    (cond
     ((zero? (sub1 n)) (cdr lat))
     (else (cons (car lat)
                 (rempick (sub1 n) (cdr lat)))))))
```

`number? a` 用来判断 `a` 是不是一个整数 `Atom`，它是一个基础函数。

`(non-nums lat)` 去除 `lat` 中的所有数字，定义如下：

```scheme
(define non-nums
  (lambda (lat)
    (cond
     ((null? lat) '())
     ((number? (car lat)) (non-nums (cdr lat)))
     (else (cons (car lat)
                 (non-nums (cdr lat)))))))
```

`(eqan? a b)` 比较 `Atom a` 和 `Atom b` 是否相同，定义如下：

```scheme
(define eqan?
  (lambda (a b)
    (cond
     ((and (number? a)
           (number? b) (= a b)))
     ((or (number? a)
          (number? b)) #f)
     (else (eq? a b)))))
```

`(occur a lat)` 统计 `a` 在 `lat` 中出现的次数，定义如下：

```scheme
(define occur
  (lambda (a lat)
    (cond
     ((null? lat) 0)
     ((eq? (car lat) a) (add1 (occur a (cdr lat))))
     (else (occur a (cdr lat))))))
```

在上面的代码中，我们经常使用 `(zero? (sub1 a))` 来纠正判断下标，它的意思是整数 `Atom a` + 1 是否等于 0。我们可以使它更清晰一点：

```scheme
(define one?
  (lambda (n)
    (= n 1)))
```

然后我们可以改写 `rempick` 函数：

```scheme
(define rempick
  (lambda (n lat)
    (cond
     (one? n) (cdr lat))
     (else (cons (car lat)
                 (rempick (sub1 n) (cdr lat)))))))
```
