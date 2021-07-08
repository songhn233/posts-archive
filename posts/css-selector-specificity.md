---
title: CSS 选择器优先级有权重的说法吗？
date: 2021-04-01T10:23:37+08:00
description: CSS Selector Specificity 的计算有很多文章都提到过，但其中经常出现权重的概念，这种说法是从哪里开始的，这种理解是否合理呢。
featured_image: https://img.songhn.com/img/cssspecificity-calc-1_kqzhog.webp
categories:
- 前端
tags:
- CSS
---
## 起因
假如你搜索过选择器优先级计算相关文章，那一定见过这样的说法：

- 1：代表内联样式，权值为1000
- 2：代表ID选择器，权值为100
- 3：代表类，伪类和属性选择器，权值为10
- 4：代表元素选择器和伪元素选择器，权值为1

**那很多人应该会有这样的疑问：**

class 权值为 10， id 权值为 100，那 11 个 class 选择器优先级不就比 id 高了吗，但写了个 demo 发现并不是这样，id 依然优先级更高。

那这样的说法是否合适呢？

## 规范
首先让我们打开现在的 [Selectors Speciation](https://www.w3.org/TR/selectors/#specificity)
> Specificities are compared by comparing the three components in order: the specificity with a larger A value is more specific; if the two A values are tied, then the specificity with a larger B value is more specific; if the two B values are also tied, then the specificity with a larger C value is more specific; if all the values are tied, the two specificities are equal.

可以看到现在的规范将选择器划分为 3 类来组成三元组 (A, B, C)。并没有常见的权重的概念，那之前的规范呢？

在 [Selectors Level 3](https://drafts.csswg.org/selectors-3/#specificity) 中可以发现
> Concatenating the three numbers a-b-c (in a number system with a large base) gives the specificity.

这个描述很类似于权重的概念，但是特别强调了这是一个大进制的表示方法，不能直观的把他理解为十进制

在 CSS-TRICKS [关于计算优先级的文章](https://css-tricks.com/specifics-on-css-specificity/) 中可以找到
> You can generally read the values as if they were just a number, like 1,0,0,0 is “1000”, and so clearly wins over a specificity of 0,1,0,0 or “100”. The commas are there to remind us that this isn’t really a “base 10” system, in that you could technically have a specificity value of like 0,1,13,4 – and that “13” doesn’t spill over like a base 10 system would.

大致意思是在读时可以用类似数字的读法，但书写时的逗号提醒我们这并不是一个十进制数。不过在很多文章中都采用了这种权重表示方法但是并没有使用逗号隔开，同时也没有强调这并不是一个十进制表示，会对很多人产生误导。

## 大进制

所以开头提到的问题也就能解决了，因为这并不是一个十进制数，而是一个很大的基数，所以自然也就不能进位了。那有没有可能进位呢，StackOverflow 上就有 [这样一个问题](https://stackoverflow.com/questions/2809024/how-are-the-points-in-css-specificity-calculated) 

其中 Faust 的回答这样描述
> It turns out that the "very large base" employed (at least by the 4 most commonly-used browsers*) to implement this standard algorithm is 256 or 28.

具体细节可以看完整评论。但该评论是 2012 年时候编辑的，在 14 年的更新这就说明浏览器普遍采用更大的基数，甚至是 infinity。

不过在 12 年的当时普遍采用 256 进制，并且是允许溢出的，这也是很多文章说明基数是 256 的论据之一。

## 溢出

那到底会不会溢出呢？

有一种 [说法](https://www.runoob.com/w3cnote/css-style-priority.html)
> 权重的进制是并不是十进制，CSS 权重进制在 IE6 为 256，后来扩大到了 65536，现代浏览器则采用更大的数量

这个说法是不准确的。站在今天的视角上，实际上溢出的问题在现代浏览器上已经修复了，即使扩大到 65536 或者更大的浏览器的阙值也不会发生溢出。在 Selectors Level 4 Working Draft 也是现在的 [Selectors Speciation](https://www.w3.org/TR/selectors/#specificity) 中特意加注了说明
> Due to storage limitations, implementations may have limitations on the size of A, B, or C. If so, values higher than the limit must be clamped to that limit, and not overflow.

也有 [文章](https://www.jianshu.com/p/1ea4c6578c39) 提到 webkit 中对于选择器优先级的计算细节，是不会产生溢出的。

## 计算
再次回到现在。那么选择器优先级该如何计算呢？其实标准已经写的很清楚了

- 计算 ID 选择器的数量 (= A)
- 计算类选择器，属性选择器，伪类的数量 (= B)
- 计算标签选择器，伪元素的数量（= C）

需要注意一些伪类选择器为其他选择器提供了一种 evaluation contexts，所以优先级被特别定义，如 `:is()`，`:not()` 等，可具体参考规范。

之后会对三元组 (A, B, C) 进行比较，优先级 A > B > C，先比较 A 的大小，谁大谁优先级高，相等则比较 B，以此类推。

## 总结
站在今天的视角上有不少说法都是有问题的，但其实以前的很多文章其实也强调了进制问题，只是在传播中很多文章并没有强调这一问题。包括对于 256 进制的说法，原回答中也指出这是 12 年的视角，现代浏览器是不存在溢出的问题的。其实规范对于优先级计算的描述还是很清楚的，在遇到这种迷惑的定义问题时可以优先参考规范描述。
