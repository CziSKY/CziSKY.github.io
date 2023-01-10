+++
title = "小学生都能看懂的集合论笔记 —— 零"
date = 2023-01-08
[taxonomies]
categories = ["notes"]
tags = ["maths"]

[extra]
math = true
+++

> 自从把我的偶像千里冰封拉到我的群里后，我的焦虑感就更强了。因为每当他跟群里的其他大佬交流 PL 领域内容时都像领域展开一样让人无法理解和插嘴。（他依然是我的偶像，只是我太菜了）
>
> 而为了变强和缓解焦虑的我开了 [DaoFP 的坑](https://kirraobj.ink/post/2022-1-5-daofp-notes-01/)，一本针对程序员讲范畴论 (Category Theory)
> 的书。在找九零大佬问问题时提到他的踩坑经历（原话是范畴论不太推荐用纯编程角度理解，包括书中讲到一些定义的时候我也会觉得有点怪，就像莫名其妙冒出来的一样）并安利我可以去学一点抽象代数
> (Abstract Algebra) 来获得一些**严谨**的参照例子。
>
> 所以我要拐回去学抽象代数了，我的数学并不是特别好，简直跟我本人 18 岁前的经历一样垃圾。但我还是不做人了，JoJo！经过再次交流后确定我应该从集合论 (Set Theory) 的基础开始补起。
>
> 其实集合论（并集，交集等）高中就已经讲过了，但并没有讲太深（单射、满射、双射、定义域、陪域、值域、集合族交并等），而没有这些基础知识很难啃得动抽代。而因为不想漏过一些点所以我打算从头开始
> ，而课程我打算跟 [Maki's Lab](https://www.bilibili.com/video/BV1sL4y1H7Xa) 的。非常感谢他们无偿向所有人提供关于数学的高质量教学课程。

## 1.1 集合的介绍

### 枚举法

用枚举法你可以表示两种集合：

- 有限集：$\mathcal{A}$ {$a_1, a_2, ..., a_n$} 因为你能列举完这个集合的所有元素，所以称为有限集。
- (可数) 无限集：$\mathcal{B}$ {$b_1, b_2, ..., b_n, ...$} 你列举不完这个集合的所有元素，但这个集合里的元素具有明显的规律。可以拿有理数集和斐波那契数集来举例。为什么可数是因为你可以给集合里的元素加下表并把它们数成一列。在之后会提到。

### 元素

元素可以是任意的数学对象。可以是一个数、可以是一个函数、还可以是一个集合。表示元素 $\mathcal{a}$ 在集合 $\mathcal{X}$ 中即为 $a \in \mathcal{X}$ ，反之 $a \notin \mathcal{X}$。

来用枚举法来表示三个重要的集合：

- $\mathcal{ℕ_1}$ = {$\mathcal{1, 2, 3, 4}$, ...} 为正整数集。
- $\mathcal{ℕ_0}$ = {$\mathcal{0, 1, 2, 3, 4}$, ...} ($ℕ_1 \cup$ {$\mathcal{0}$}) 为非负整数集。
- $\mathcal{ℤ}$ = {..., $\mathcal{-4, -3, -2, -1, 0, 1, 2, 3, 4}$, ...} 为整数集。

有了已定义的这些集合我们就能定义一些关系，例：

- 元素 $a \in \mathcal{ℕ_1}$ 即说明 $\mathcal{a}$ 为正整数。
- 元素 $y \in \mathcal{ℕ_0}$ 即说明 $\mathcal{y}$ 为非负整数。
- 元素 $z \notin \mathcal{ℤ}$ 即说明 $\mathcal{z}$ 不属于整数。

### 描述法

如果集合中的任意元素 $\mathcal{x}$ 都满足命题 $\mathcal{P}$，那么该集合就可以写成 {$\mathcal{x}$ | $\mathcal{P(x)}$}。例如 $x \in \mathcal{A}$ 是一个命题。

然后我们就可以来用描述法来定义集合之间的关系或者一些集合了！下面是一些例子：

- 交集 {$\mathcal{x}$ | $x \in \mathcal{A}$ & $x \in \mathcal{B}$}：在 $\mathcal{A}$ 中也在 $\mathcal{B}$ 中的元素构成的集合。用 $\cap$ 表示。

- 差集 {$\mathcal{x}$ | $x \in \mathcal{A}$ & $x \notin \mathcal{B}$}：在 $\mathcal{A}$ 但不在 $\mathcal{B}$ 中的元素构成的集合。用 $\setminus$ 表示。我比较喜欢 $\mathcal{A} -
\mathcal{B}$ 这种表示方法。

- 并集 {$\mathcal{x}$ | $x \in \mathcal{A}$ || $x \in \mathcal{B}$}：在 $\mathcal{A}$ 或在 $\mathcal{B}$ 中的元素构成的集合。用 $\cup$ 表示。

- 补集 {$\mathcal{x}$ | $x \in \mathcal{X}$ && $x \notin \mathcal{A}$}：设 $\mathcal{X}$ 为一个包含 $\mathcal{A}$ 的集合（全集），该集合属于 $\mathcal{X}$ 但不属于 $\mathcal{A}$。则被记作在
$\mathcal{X}$ 中的 $\mathcal{A^C}$。

- 有理数集：{$\frac p q$ | $p, q \in \mathcal{ℤ}$, $q \ne \mathcal{0}$}：所有分数构成的集合。 $\mathcal{p}$ 和 $\mathcal{q}$ 均属于整数且 $\mathcal{q}$ 不等于 $\mathcal{0}$。用
$\mathcal{ℚ}$ 来表示。

- 实数集：{$\mathcal{x}$ | $x \in \mathcal{ℝ}$}：课中只讲解了 $\mathcal{x}$ 属于实数的模糊定义，这里用初高中的数学知识理解就是有理数与无理数的并集，用 $\mathcal{ℝ}$ 来表示。

- 包含：$\mathcal{A}$ 是 $\mathcal{B}$ 的一个子集（$\mathcal{B}$ 集合中包含了 $\mathcal{A}$ 集合），这样一种关系可以确认如果 $x \in \mathcal{A}$ 则同时也能推出 $x \in \mathcal{B}$。用
$\subseteq$ 表示。

- 相等：$\mathcal{A = B}$，即同时满足 $x \in \mathcal{A}$ 和 $x \in \mathcal{B}$ 的关系，用 $\mathcal{=}$ 来表示。

运用以上的知识观察一下之前讲到的所有集合我们便可推出这样一种关系，即 $\mathcal{ℕ_1} \subseteq \mathcal{ℕ_0} \subseteq \mathcal{ℤ} \subseteq \mathcal{ℚ} \subseteq \mathcal{ℝ}$。

解释一下就是实数集包含了有理数集，而有理数集又包含了整数集等以此类推的这样一个规律。

## 1.2 集合的介绍

空集 {$\mathcal{x}$ | $x \ne \mathcal{x}$}：一个空的集合，里面所有的元素都不等于它自己。这里的定义我觉得比较 Tricky 和好玩的地方，因为没有东西不等于它自己。所以它是空的。用 $\emptyset$ 来表示。

元素势：一个有限集中所有的元素总数 ({$a_1, a_2, ..., a_n$}, 这里的 $\mathcal{n}$ 即为总数)，通常记为 $\mathcal{|X|}$，也可以记作 $\mathcal{card(X)}$ 或 #$\mathcal{X}$。

幂集 {$\mathcal{A}$ | $\mathcal{A} \subseteq \mathcal{X}$}：原集合中所有中所有的子集（包括全集和空集）构成的集合（集族）。用 $\mathcal{P}$ 来表示。

> 例：假设集合 $\mathcal{X}$，它有 {$\mathcal{1, 2, 4}$} 这三个元素。
> 
> 则它的幂集 ($\mathcal{P(X)}$) 则为集合中所有元素可能出现的排列组合（作为一个集合）加上空集，则为：{$\emptyset$, {$\mathcal{1}$}, {$\mathcal{2}$}, {$\mathcal{4}$}, 
> {$\mathcal{1, 2}$}, {$\mathcal{1, 4}$}, {$\mathcal{2, 4}$}, {$\mathcal{1, 2, 4}$}}。
> 
> 再假设 $\mathcal{X}$ 是一个有限集（$\mathcal{|X|} = \mathcal{n} < \infty$），则我们可以说它幂集的势为 $2^n$。但这又是为什么呢？在这里用一种比较简单和直观的方法去证明它：
> 
> 定义集合 $\mathcal{X}$ 为 {$a_1, a_2, ..., a_n$}，再定义一个子集 $\mathcal{A} \subseteq \mathcal{X}$。那么对于 $\mathcal{X}$ 中的所有元素就有两种情况，
> 即从 $\mathcal{a_1}$ 到 $\mathcal{a_n} \in \mathcal{A}$ 或 $\mathcal{a_1}$ 到 $\mathcal{a_n} \notin \mathcal{A}$。每一种元素都对应着两个选择，即所有可能出现的选项则为 $2^n$。

### 集族

简单来说，以集合为元素的集合称为集族（$\mathcal{Set}$ $\mathcal{X} \in \mathcal{S}$），例如上面的幂集就可以称为一个集族（空集这里也纳入一个单独的集合）。

幂集的子集仍为一个集族（{{$\emptyset$},{$\mathcal{1}$}} 或 {{$\mathcal{1}$}}）。$\mathcal{A \subseteq P(X)}$，$\mathcal{B \in A}$，$\mathcal{B \in P(X)}$，$\mathcal{B \subseteq X}$ (As a subset)。

指标集：是一个带有索引集的集合（索引集即之前提到的全部元素下表所构成的集合），例如最开始讲的无限集 $\mathcal{B}$ {$b_1, b_2, ..., b_n, ...$}。其中元素的下表里的所有元素都 $\in$ $\mathcal{ℕ_1}$。那么我们就可以说正整数集就是这个集合的索引集，表示为 $\mathcal{(B_n)}_\mathcal{n \in ℕ_1}$ 。通常我们用指标集来描述集族。

那么集族之间也跟集合一样可以表示两个集族之间的交并关系：

集族并集：直觉上我最开始以为跟集合的并定义一样：{$\mathcal{x}$ | $x \in \mathcal{A}$ || $x \in \mathcal{B}$}，但对于集族的并我们需要用存在量词 $\exists$ 来连接元素。又因为上面提到可以用指标集来来描述集族，所以针对集族 $\mathcal{(A_x)}_\mathcal{x \in X}$ 的并我们这么定义：{$\mathcal{x}$ | $\exists$ $\mathcal{a} \in \mathcal{X}$, $\mathcal{x} \in \mathcal{(A_a)}$}。

其中集族 $\mathcal{(A_a)}$ 仍然是属于 $\mathcal{(A_x)}_{x \in X}$ 的，所以可以表示为集族的交。

针对集族的交我们可以沿用上面的定义，只需把存在量词改成全称量词。所以针对集族 $\mathcal{(A_x)}_\mathcal{x \in X}$ 的交我们这么定义 {$\mathcal{x}$ | $\forall$ $\mathcal{a} \in \mathcal{X}$, $\mathcal{x} \in \mathcal{(A_a)}$}。

# 摆烂，等考试回来再推进