---
title: "JavaScript第六课"
published: 2026-09-26
description: This is the first post of my new Astro blog.
tags: [JavaScript]
author: jia
---
# JS 第六课：深入作用域链 [[scope]]、闭包产生本质与实战模块化

> 本课对应练习文件：`index.html`（函数的隐式属性 [[scope]]、作用域链生命周期、三层作用域演变 a→b→c、闭包产生的本质、闭包对父级 AO 的修改机制、日程管理模块实战）。

# 一、函数也是对象：可见属性 vs 隐式属性 `[[scope]]`

在 JavaScript 中，函数不仅仅是一段可以调用的代码块，**函数本质上是一种特殊的引用对象（Object）**。

![函数也是对象：可见属性 vs 隐式属性 scope](https://img.215454.xyz/file/blog/jia/1791531414632_01-function-as-object-scope.png)

## 1. 我们可以直接访问的「可见属性」

```js
function test(a, b) {}

console.log(test.name);      // 'test'（函数名）
console.log(test.length);    // 2（形参的个数）
console.log(test.prototype); // { constructor: ƒ }（原型对象）
```

函数可以像普通对象一样通过点号读取属性，甚至也能人为挂载自定义属性（如 `test.custom = 123`）。

## 2. 引擎内部专属的「隐式属性」`[[scope]]`

除了可见属性外，JS 引擎在底层还为每一个函数赋予了一组**隐式属性**，其中最为关键的就是 **`[[scope]]`**：

- **何时生成？** 当函数被**创建定义**的那一瞬间，由 JS 引擎自动创建；
- **存的是什么？** 它就是**存储该函数作用域链（Scope Chain）的容器**！
- 函数在执行时能看到的所有变量，全都是顺着这个容器里的链条逐级向上寻找的。

# 二、`[[scope]]` 的生命周期：定义、执行与销毁

以一个最简单的单层函数 `a` 为例，看看在不同生命周期阶段，它的 `[[scope]]` 容器是如何动态变化的：

![scope 的生命周期：定义、执行与销毁](https://img.215454.xyz/file/blog/jia/1791531426072_02-scope-lifecycle.png)

## 1. 第一步：函数定义时（出生）
函数 `a` 在全局环境中被声明。它复制并记录了当前所处环境的作用域链（此时只有全局 `GO`）：
```text
a.[[scope]]:
  0: GO
```

## 2. 第二步：函数执行前一刻（顶端压入 AO）
当 `a()` 被调用时，JS 引擎进行预编译，生成专属于 `a` 的执行期上下文（`a->AO`），并**把它插入到作用域链的最顶端（第 0 位）**：
```text
a.[[scope]]:
  0: a->AO （最优先查找）
  1: GO
```

## 3. 第三步：函数执行完毕（弹出并销毁 AO）
函数执行结束，顶端的 `a->AO` 立即被弹出并销毁，内存归还给垃圾回收系统，作用域链回退到最初定义时的状态：
```text
a.[[scope]]:
  0: GO
```

# 三、三层嵌套作用域链演变全景：a → b → c

当函数出现多层内部嵌套（`a` 里面有 `b`，`b` 里面有 `c`）时，作用域链的层层传递与层层剥离体现得淋漓尽致：

![三层嵌套作用域链演变全景：a → b → c](https://img.215454.xyz/file/blog/jia/1791531440986_03-nested-scope-chain-full.png)

```js
function a() {
    function b() {
        function c() {
            console.log(b);
        }
        c();
    }
    b();
}
a();
```

| 执行阶段 | 动作与作用域链变化 | 说明 |
| :--- | :--- | :--- |
| **a 定义** | `a.[[scope]] = [GO]` | 诞生在全局，拿到全局 GO |
| **a 执行** | `a.[[scope]] = [a->AO, GO]` | 顶端压入自己的 a->AO |
| **b 定义** | `b.[[scope]] = [a->AO, GO]` | **关键：直接继承 a 执行时的完整链条！** |
| **b 执行** | `b.[[scope]] = [b->AO, a->AO, GO]` | 顶端压入自己的 b->AO |
| **c 定义** | `c.[[scope]] = [b->AO, a->AO, GO]` | **直接继承 b 执行时的完整链条！** |
| **c 执行** | `c.[[scope]] = [c->AO, b->AO, a->AO, GO]` | 顶端压入 c->AO，此时链条最长，能看到所有祖先变量 |
| **c 结束** | `c->AO` 弹出销毁 | 回退到 `[b->AO, a->AO, GO]` |
| **b 结束** | `b->AO` 弹出销毁 | 回退到 `[a->AO, GO]`，内部的 c 彻底失联销毁 |
| **a 结束** | `a->AO` 弹出销毁 | 全局回退到 `[GO]` |

> **核心定律：**  
> 内部函数的 `[[scope]]` **继承的是父函数执行时的实时作用域链**！父函数内部的变量，天然对子函数完全开放可见。

# 四、闭包产生的本质：内部函数被带到了外部保存

有了前面的 `[[scope]]` 模型，理解**闭包（Closure）**就会变得极其直观。

![闭包产生的本质：内部函数被带到了外部保存](https://img.215454.xyz/file/blog/jia/1791531442209_04-closure-essence.png)

对比两个只有一字之差的经典函数：

## 1. 立即调用：未产生闭包

```js
function test1() {
    function test2() {
        console.log(a);
    }
    var a = 1;
    return test2(); // 立即执行，返回的是 undefined！
}
var res = test1();  // 输出 1，但 res === undefined
```

`test1` 执行时顺带把 `test2` 跑完了，随后 `test1->AO` 和 `test2->AO` 随着调用栈退出**正常被垃圾回收**，内存全部释放。

## 2. 返回函数本体：闭包诞生！

```js
function test1() {
    function test2() {
        console.log(a);
    }
    var a = 1;
    return test2;   // 返回函数对象本身！
}
var test3 = test1(); // 全局变量 test3 牢牢接住了 test2 函数对象！
test3();            // 1
test3();            // 1（随时随地可以反复调用）
```

### 闭包的底层死锁机制
1. `test1` 执行结束，按常规逻辑它的 `test1->AO` 应该被销毁；
2. 但注意：返回出来的 `test2` 被全局变量 `test3` 引用着，处于存活状态；
3. 而 `test2.[[scope]]` 内部**依然牢牢牵着 `test1->AO` 这根链条**！
4. 垃圾回收器发现 `test1->AO` 仍有外部指针依赖，**不能释放它**！
5. 结果：**`test1->AO` 像被装进了一个随身保险箱，永久寄生在内存中** —— 这就是**闭包**！

# 五、闭包常见困惑：`a = 2` 是修改外层 AO 还是变成全局变量？

![闭包常见困惑：a = 2 究竟是修改外层 AO 还是变成全局变量](https://img.215454.xyz/file/blog/jia/1791531452795_05-closure-assign-scope.png)

初学者看到没写 `var` 的赋值容易产生误解，以为只要没写 `var` 就会变成全局变量：

```js
function test1() {
    function test2() {
        var b = 2;
        a = 2; // 没写 var！它会污染 window 吗？
        console.log(a);
    }
    var a = 1; // 👈 外层声明了 var a
    return test2;
}

var test3 = test1();
test3();               // 2
console.log(window.a); // undefined ！！全局丝毫不受影响！
```

## 寻路机制推演
- 变量查找严格遵守**「作用域链自上而下匹配」**：
  1. 先查 `test2->AO`：里面有 `b`，但没有 `a`；
  2. 顺着链条继续查 `test1->AO`：**找到了 `a`（初始值为 1）**！
  3. 既然找到了，赋值操作立即针对 `test1->AO.a` 进行，把它改写为 `2`，**查找立刻终止**！
- 绝不会冒泡到全局 `GO`，全局的 `window.a` 依然是 `undefined`。

> **对照：** 只有当整条链条上**所有祖先 AO 都没有声明过该变量**时，赋值操作才会一路漏到底部，沦为挂在 `window` 上的暗示全局变量（Imply Global）。

# 六、闭包实战：日历日程管理模块（Module Pattern）

闭包在真实工程中绝不是为了炫技，而是 JavaScript 实现**数据私有化与模块封装**的核心基石。

![闭包实战：日历日程管理对象的模块化封装](https://img.215454.xyz/file/blog/jia/1791531461473_06-closure-schedule-module.png)

```js
function sunSched() {
    var sunSched = ''; // 私有日程数据，外界绝对无法直接读取或篡改

    var operation = {
        setSched: function (thing) {
            sunSched = thing;
        },
        showSched: function () {
            console.log('My schedule on sunday is ' + sunSched);
        }
    };

    return operation; // 导出控制接口对象
}

var myDay = sunSched();
myDay.setSched('walking');
myDay.showSched(); // 'My schedule on sunday is walking'

myDay.setSched('coding');
myDay.showSched(); // 'My schedule on sunday is coding'
```

## 核心设计模式解析

1. **信息隐藏与安全沙箱**：外界根本拿不到变量 `sunSched` 的直接指针，彻底杜绝了外部随意重置数据的风险；
2. **多方法共享同一个 AO**：`setSched` 和 `showSched` 在同一次执行中诞生，它们共同持有着同一个私有作用域链，一个修改、另一个即时可见；
3. **面向对象私有属性的鼻祖**：在 ES6 类私有字段（`#field`）诞生前，这是 JavaScript 实现私有属性的唯一工业标准。

# 本课小结

- **函数也是对象**：拥有可见属性（`name`、`length`、`prototype`）以及引擎专属隐式属性 `[[scope]]`；
- **`[[scope]]` 是存储作用域链的容器**，在函数**创建定义时生成**；
- **执行期动态压栈**：函数执行前一刻，把新生成的 `AO` 插入到链条**最顶端第 0 位**；执行完毕立即弹出销毁；
- **嵌套作用域传递**：内部子函数在定义时，直接**继承父函数执行时的完整链条**；
- **闭包产生的本质**：内部函数被保存到了外部，导致其所依赖的原生作用域链无法被回收，父级 `AO` 被永久保留；
- **赋值寻路机制**：沿作用域链自顶向下查找，命中哪个 AO 就改哪个，整条链全未命中才会沦为全局变量；
- **模块化实践**：利用闭包返回携带操作方法的对象，实现私有状态保护（Module Pattern）。
