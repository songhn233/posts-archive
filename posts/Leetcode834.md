---
title: Leetcode 834. 树中距离之和 题解
katex: true
date: 2020-10-06T23:30:00+08:00
tags:
- 换根dp
categories:
- 算法与数据结构
enableMathJax: true
katex: true
description: Leetcode 也能出换根我是没想到（
summary: Lazyload 懒加载作为一项前端优化技术能够有效加快首屏加载、提升用户使用体验等，本文是基本的 lazyload 和在图片、JavaScript插件等方面的实践指北
---
[原题链接](https://leetcode-cn.com/problems/sum-of-distances-in-tree/)

## 题解

这是一道换根dp，具体来讲是通过两次dfs来求解。

首先考虑一下只求一个根节点的所有点到它距离之和，可以写出这样的递推式：
$$ f_u = \sum (f_v + size_v) $$

首先解释一下这个方程：

$f_v$ 表示 $v$ 所在的子树的所有点到 $v$ 的距离之和，$u$ 是 $v$ 的父节点。
$size_v$ 表示 $v$ 所在的子树的节点个数。

现在 $v$ 的所有子节点到 $v$ 的距离和已经知道了，那么这 $size_v -1$ 个 $v$ 的子节点到 $u$ 的距离就要都加上 $v$ 到 $u$ 的距离也就是 $1$，最后再加上 $v$ 到 $u$ 的 $1$ 即得到了 $size_v$，从而就得到了这个方程。

$size$ 我们可以通过一次 DFS 求出，同时也能顺便求出 $f$，这样时间复杂度是 $O(N)$，但是如果我们对每个点都这样求的话，复杂度就会上升到 $O(N^2)$，而这并不理想。

之后可以通过换根来优化复杂度，这也就是第二次 DFS 做的操作，我们得到了 $u$ 作为根时的答案，那么儿子 $v$ 的答案便可以通过公式求解，然后把 $v$ 当成根继续递归求解得到所有答案。那么怎么计算这个公式呢？

现在 $v$ 是 $u$ 的儿子节点，那么 $ans_v$ 的值便是原有的 $f_v$ 的值加上 $ans_u$ 减去 $v$ 的贡献 $(f_v + size_v)$ 后这一块值再加上接到 $v$ 上产生的贡献 $(size_u - size_v)$，可以这样写：
$$ans_v = f_v + (ans_u - (f_v + size_v) + (size_u - size_v)$$

即

$$ans_v = ans_u + size_u - 2\cdot size_v$$

由于 $ans_u$ 是所有节点到 $u$ 的距离和，此时 $u$ 相当于根节点，所以 $size_u$ 就是 $N$，最后就是 $ans_v = ans_u + N - 2\cdot size_v$ 。

我们一开始通过 $f$ 求出一个根节点的答案后 $ans_{root} = f_{root}$，知道了 $ans_{root}$ 后我们就可以递归求出 $ans$ ，同时发现 $f$ 经过第一次递归得出一个根的答案后就没用了，于是我们可以直接用 $f$ 来计算 $ans$ 节省一个数组开销。

## 代码

```js
const sumOfDistancesInTree = (N, edges) => {

  const map = new Array(N).fill(0).map(t => []);
  const nodeSize = new Array(N).fill(0);
  const dp = new Array(N).fill(0);

  const init = edges => {
    edges.forEach(edge => {
      const [u, v] = edge;
      map[u].push(v);
      map[v].push(u);
    });
  };

  const dfs = (u, fa) => {
    nodeSize[u] = 1;
    const vMap = map[u];
    vMap.forEach(v => {
      if (v === fa) {
        return ;
      }
      dfs(v, u);
      nodeSize[u] += nodeSize[v];
      dp[u] += dp[v] + nodeSize[v];
    });
  };

  const calc = (u, fa) => {
    const vMap = map[u];
    vMap.forEach(v => {
      if (v === fa) {
        return ;
      }
      dp[v] = dp[u] + N - 2 * nodeSize[v];
      calc(v, u);
    });
  };

  init(edges);
  dfs(0, -1);
  calc(0, -1);
  return dp;
};
```