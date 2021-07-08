---
title: Async 在 forEach、map 等各种迭代中的行为
date: 2021-07-07T21:47:13+08:00
description: 在迭代中使用 async 是一种常见的需求，包括要求迭代中的异步顺序执行和并行执行，在不同的迭代模式下有不同的实现方式和原因。
categories:
- 前端
tags:
- Javascript
---
## 问题引入
首先假设有这样一个场景，对于一个数组中的所有文件读取内容，同时输出文件内容。

假如我们写出这样的代码（本文示例代码部分参考 Stackoverflow 中的 [回答](https://stackoverflow.com/questions/37576685/using-async-await-with-a-foreach-loop)）

```js
async function printFiles () {
  const files = await getFilePaths()

  files.forEach(async (file) => {
    const contents = await fs.readFile(file, 'utf8')
    console.log(contents)
  })
}
```
可以看到执行是符合预期的，但这其实不是一种正确的写法，因为这个函数`forEach` 后还是同步的，`async` 没有起到作用，假如要求在全部读取后输出所有内容，这样的写法就只会输出一个空数组，之后函数就会返回了

```js
function get(i) {
  return Promise.resolve(i)
}
async function printFiles () {
  const files = [1,2,3]
  const ans = []
  files.forEach(async (file) => {
    const contents = await get(file)
    console.log(contents)
    ans.push(contents)
  })
  console.log(ans)
}
printFiles()
// will console: [] 1 2 3
```
在有些场合可以看见 **`forEach` 中的异步是并行的** 这一说法，不过这种说法有一定问题，虽然他的表现是异步的，但是不能依赖这种异步带来的特性，因为这实际上起不到异步预期的效果，就像上文中的例子，假如后续还有相关操作就会受到影响。`forEach` 只是对于数组中所有的元素执行回调函数，本身并没有返回值，所以想要用 `await` 之类的方法加以控制也不行。

## 并行执行
那怎样达到真正的并行执行异步的效果呢，可以利用 `async` 函数的 `Promise` 返回值，当 `list` 中所有的 `Promise` 都成功返回后再往下执行即可，这就需要借助 `map` 来利用返回值

```js
function get(i) {
  return Promise.resolve(i)
}
async function printFiles () {
  const files = [1,2,3]
  const ans = []
  await Promise.all(files.map(async (file) => {
    const contents = await get(file)
    console.log(contents)
    ans.push(contents)
  }))
  console.log(ans)
}
printFiles()
// will console: 1 2 3 [1, 2, 3] 
```
可以看到这时候就符合预期了。还有一些要注意的细节，由于 `map` 返回结果是一个 `Promise` 数组，而 `await` 只适用于单个，所以需要借助 `Promise.all` 来实现，当然配合 `Promise.race` 和 `Promise.allSettled` 等内置的静态方法也能实现对应的效果。

## 顺序执行
那假如迭代本身也有依赖关系，期望是前一个 `async` 执行结束后在执行下一个呢？

套用前面 `map` 代替 `forEach` 利用返回值来 `await` 的思路，不难想到通过 `reduce` 来实现这种效果，因为 `reduce` 的累加器需要前一个返回的结果，所以会等待 `async` 执行完毕。

```js
async function printFiles () {
  const files = await getFilePaths();
  await files.reduce(async (acc, file) => {
    await acc;
    const contents = await fs.readFile(file, 'utf8');
    console.log(contents);
  }, Promise.resolve());
}
```

这样就能保证顺序执行，由于 `reduce` 每次执行都会返回一个结果作为累加器作用，我们可以利用 `async` **隐式返回** 的 `Promise`，每次首先 `await` 等前一个执行完毕后在进行当前的工作，以保证顺序的一致。

当然在大部分情况下这种场景无需关心相对顺序，只需要保证所有任务异步执行，那就可以使用并行的写法。

除了 `forEach` 和 `map` 这些以外，还有更基本 `for...in`, `for...of` 和 `for` 语句来实现迭代遍历，那么他们行为有什么区别呢？
### for-of
```js
async function printFiles () {
  const files = await getFilePaths();

  for (const file of files) {
    const contents = await fs.readFile(file, 'utf8');
    console.log(contents);
  }
}
```

在 `for...of` 中表现出的行为是顺序执行的，而 `forEach` 中却没有这种行为，这是因为两种内部实现方式不一样，`forEach` 上文已经提到只是对于所有的元素执行回调函数，不会等待前一个执行结果的返回，可以参考 [ES 规范](https://tc39.es/ecma262/#sec-array.prototype.foreach) 中的定义

![es_foreach](https://img.songhn.com/img/es_foreach_spec.png)

而在 `for...of` 中其实引入了迭代器机制，具体细节可以参考 [MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators)，所以每次都会等待当前当前执行结束后返回 next，这样 await 就能起到作用了。

### for

而在 `for...in` 和 `for` 中虽然并不使用新的 iterator 来执行，但在内部实现上其实会去等待当前执行的结果，之后再执行下一个，同样参考 [ES 规范](https://tc39.es/ecma262/#sec-forbodyevaluation) 中的描述，对于 `for` 中循环体的执行机制
![es_for_body](https://img.songhn.com/img/es_for_body.png)
可以看到 3.b 对于当前执行求 result，3.c 通过 result 判断是否需要跳出当前循环，可以看出 await 会对 `for` 循环进行作用

### for-in

`for...in` 中 [参考规范](https://tc39.es/ecma262/#sec-runtime-semantics-forinofloopevaluation)
![es_for_in_state](https://img.songhn.com/img/es_for_in_state.png)

可以看出 `for...in` 和 `for...of` 虽然在使用上有很多感知上的区别，而且社区普遍不推荐使用 `for..in`，但是两者在循环体的执行机制上是很像的，用的是同一个抽象方法，只是入参有些区别，最大的区别应该在循环头的处理上，`for...of` 会对 iterator 进行处理，有兴趣可以参考对应规范。
所以在执行 `for...in` 循环体时也会依赖迭代器进行执行（这里的迭代器是一种规范抽象定义，和 JS 中的具体迭代器实作不是一回事），具体细节可以参考 ForIn/OfBodyEvaluation 的 [定义](https://tc39.es/ecma262/#sec-runtime-semantics-forin-div-ofbodyevaluation-lhs-stmt-iterator-lhskind-labelset)

### for...await...of
另外值得一提的是，在 ES2018 中引入了新的 `for-await-of` 语法，对应的 [规范](https://tc39.es/ecma262/#sec-for-in-and-for-of-statements) 和当时的 [proposal](https://github.com/tc39/proposal-async-iteration)。

所以上述的函数也可以改写为
```js
async function printFiles () {
  const files = await getFilePaths()
  for await (const contents of files.map(file => fs.readFile(file, 'utf8'))) {
    console.log(contents)
  }
}
```
同时该语法配合生成器等有更多的用法，有兴趣可以自行查阅。

### polyfill
当然觉得规范阅读起来太过复杂，一个更直观的方法是参考 对应的 polyfill，例如对于 forEach，可以参考 MDN 中给出的 [polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach#polyfill)

其中关键在于
```js
while (k < len) {
  var kValue;
  if (k in O) {
    kValue = O[k];
    callback.call(T, kValue, k, O);
  }
  k++;
}
```
可以看到只是执行了这一函数，所以在异步函数的情况下也会继续执行下一个，也就造成了并行的情况。不需要查阅规范也就对于这种现象有了直观的理解。

## 总结
通过查阅规范其实大致就能理解这种不同迭代之间的区别了，尤其之前对于 forEach 和 for 行为的不一致感到困惑，后来看到规范中抽象的定义就明白这种区别了。

在查阅资料的过程中遇到最多的一种情况就是只说现象，不说原因。比如 **如果想要顺序执行不能使用 forEach** 或 **没必要用 for-of 来顺序执行，其实直接 for 就可以** 等等的描述。

当然这是很正常的，比如我只是单纯的需要顺序执行或是并行执行的效果，但是用 `forEach` 遇到了问题，参考这种结论来保证代码的正确性就好了，不过了解不同迭代方法执行表现出的不同现象在规范中的表现也是一件有趣的事情。

## References

- [javascript - Using async/await with a forEach loop](https://stackoverflow.com/questions/37576685/using-async-await-with-a-foreach-loop)
- [JavaScript async and await in loops](https://zellwk.com/blog/async-await-in-loops/)
- [JavaScript中遍历数组的方式：forEach 相对于 for 有什么优势](https://segmentfault.com/q/1010000007977209)
- [当 async/await 遇上 forEach](http://objcer.com/2017/10/12/async-await-with-forEach/)
- [ECMAScript 2022 Language Specification](https://tc39.es/ecma262)
- [MDN Web Docs](https://developer.mozilla.org/en-US/)