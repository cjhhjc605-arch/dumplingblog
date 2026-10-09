---
title: JavaScript第五课
published: 2026-09-26
description: This is the first post of my new Astro blog.
tags:
  - JavaScript
author: jia
---
# JS 第五课：形参默认值、预编译与 AO / GO

> 本课对应练习文件：`index.html`（形参默认值、预编译、AO 与 GO 演算）、`test.html`（综合例题的完整输出）。

# 一、形参的默认值

## 1. 不传参就是 undefined

形参只是一个占位的变量，**调用时没传值，它就是 `undefined`**。所以要么在参数列表里给默认值，要么在函数体里手动兜底。

![01-param-default.png](https://img.215454.xyz/file/1790392819527_01-param-default.png)

## 2. ES6 的写法

```js
function test(a = 1, b = 2) {
    console.log(a, b);
}
test();        // 1 2
test(3);       // 3 2
```

## 3. ES6 之前的三种兜底写法

**写法一：用 `||` 兜底**

```js
function test(a, b) {
    var a = arguments[0] || 1;
    var b = arguments[1] || 2;
    console.log(a + b);
}
```

**写法二：`if` + `typeof` 判断**

```js
function test(a, b) {
    if (typeof arguments[0] === 'undefined') {
        a = 1;
    } else {
        a = arguments[0];
    }
    // b 同理
    console.log(a + b);
}
```

**写法三：三元 + `typeof`（最常用）**

```js
function test(a, b) {
    var a = typeof arguments[0] !== 'undefined' ? arguments[0] : 1;
    var b = typeof arguments[1] !== 'undefined' ? arguments[1] : 1;
    console.log(a + b);
}
test(3, 4);   // 7
```

> 这里用 `arguments[0]` 和直接用形参 `a` 是等价的 —— 它们**一一对应**，指向同一份数据。

## 4. 为什么不能无脑用 `||`

```js
test(0);   // arguments[0] 是 0，而 0 是假值 → a 被赋成默认值 1
```

**明明传了 0，却被当成「没传」。** `||` 只判断真假，不判断「有没有传」，所以只有写法二、三才是严格正确的。

# 二、预编译：代码还没执行，JS 先做了两件事

![02-precompile-hoisting.png](https://img.215454.xyz/file/1790392821690_02-precompile-hoisting.png)
## 1. 一段 JS 的加载执行流程

1. **检查通篇的语法错误** —— 有语法错误，整个脚本块都不执行；
2. **预编译**（第 1.5 步）—— 把变量声明、函数声明提前登记好，准备出 AO / GO；
3. **解释一行，执行一行** —— 从上到下按顺序执行真正的代码。

## 2. 提升规则对照

| 代码里写的 | 提升的是什么 | 执行到那行之前，它是 |
| --- | --- | --- |
| `function test(){}` | **整个函数，连函数体一起** | 可以直接调用 |
| `var a = 1;` | 只提升 `var a` 这个声明 | `undefined`（1 还没赋上去） |
| `a = 1;`（没写 var） | 不提升 | 报错 `a is not defined` |

```js
test();                  // ✓ 先调用也没问题
function test() {
    console.log(1);
}

console.log(a);          // undefined
var a = 1;
```

> 一句话记住：**函数声明整体提升，变量只有声明提升，赋值是不提升的。**

注意 `var a = 1` 其实是**两个动作**：一个是声明 `var a`（会被提升），一个是赋值 `a = 1`（留在原地执行）。

# 三、AO：函数里的活跃对象

**AO（Activation Object，活跃对象）** 就是预编译在函数里的产物 —— 它是一个对象，决定了函数里每一行读到的是什么。

![03-ao-steps.png](https://img.215454.xyz/file/1790392817258_03-ao-steps.png)
## 1. 四步建立过程

| 步骤 | 做什么 |
| --- | --- |
| ① | **寻找形参和变量声明** —— 形参和 `var` 声明的变量都记为 `undefined` |
| ② | **实参值赋给形参** |
| ③ | **找函数声明并赋值** —— 函数声明优先级最高，会覆盖前面同名的值 |
| ④ | **执行** —— 从上到下按顺序执行代码 |

> 注意第 ③ 步的威力：**函数声明会盖住第 ② 步传进来的实参**；但执行阶段的赋值，又能盖回来。

## 2. 一个完整的演算

```js
function test(a) {
    console.log(a);        // ① function a() {}
    var a = 1;
    console.log(a);        // ② 1
    function a() {}
    console.log(a);        // ③ 1
    var b = function () {};
    console.log(b);        // ④ function () {}
    function d() {}
}
test(2);
```

AO 的变化过程：

```text
① 找形参和变量声明 → { a: undefined, b: undefined }
② 实参赋给形参     → { a: 2 }
③ 找函数声明并赋值 → { a: function a(){}, b: undefined, d: function d(){} }
④ 执行             → a = 1 → a 变成 1；b = function(){} → b 变成函数
```

所以第三个 `console.log(a)` 输出的是 **1** 而不是函数 —— 因为 `function a(){}` 在第 ③ 步就处理完了，执行阶段不会再看它一眼。

## 3. 再看一个例子：赋值顺序决定最终结果

![04-ao-example.png](https://img.215454.xyz/file/1790392815746_04-ao-example.png)

```js
function test(a, b) {
    console.log(a);      // 1
    c = 0;
    var c;
    a = 5;
    b = 6;
    console.log(b);      // 6
    function b() {}
    function d() {}
    console.log(d);      // function d(){}
}
test(1);
```

**两个关键点：**

- `c = 0` 写在 `var c` **前面**，但因为 `var c` 被提升进了 AO，所以 c 是**局部变量**，不会污染全局；
- `function b(){}` 在第 ③ 步就把 b 变成了函数，但执行到 `b = 6` 时又被改回数字 —— **预编译只能决定起点，执行才决定终点**。

# 四、GO 与 window：它们其实是同一个东西


![05-go-window.png](https://img.215454.xyz/file/1790392818941_05-go-window.png)

## 1. GO 的三步

**GO（Global Object，全局上下文）** 的建立过程和 AO 一模一样，只是**少了「形参」那一步**：

1. 找变量（`var` 声明的）
2. 找函数声明
3. 执行

**`GO === window`** —— 全局的变量和函数，都是 window 上的属性。

## 2. 暗示全局变量（imply global variable）

**没有声明就直接赋值 → 这个变量自动成为全局变量**，会挂到 window 上：

```js
a = 1;                     // 前面没写 var
b = 2;
console.log(window.a);     // 1
console.log(window.b);     // 2

// 相当于
// window = { a: 1, b: 2 }
```

> 访问 window 上**不存在**的属性，不会报错，直接得到 `undefined`。

## 3. 坑：`var a = b = 1` 到底声明了谁？

```js
function test() {
    var a = b = 1;      // 相当于 var a = (b = 1)
}
test();

console.log(window.b);  // 1         ← b 成了全局变量
console.log(window.a);  // undefined ← a 是局部的，不在 window 上
```

连写时**只有紧挨 `var` 的那个才是局部变量**，后面的会被「漏」到全局。

> 这也是一个 bug 高发区：两个不同的函数里都写 `i = 0`，就会互相踩。**结论：变量一定先声明再用。**

# 五、三个最容易踩的坑


![06-traps.png](https://img.215454.xyz/file/1790392814749_06-traps.png)

## 1. `if` 里的 `var` 也会被提升

```js
function test() {
    console.log(b);      // undefined
    if (a) {
        var b = 2;
    }
    c = 3;
    console.log(c);      // 3
}
var a;
test();
a = 1;
console.log(a);          // 1
```

- `var b` 被提升到函数顶部，但 `if` 没进去，**赋值没发生** → `undefined`；
- `c` 没声明过 → 成为全局变量 → `3`。

## 2. `return` 之后的代码不执行，但预编译照做

```js
function test() {
    return a;
    a = 1;
    var a = 2;
    function a() {}
}
console.log(test());     // function a(){}
```

- 预编译照样把 `function a(){}` 赋给了 AO.a；
- 但 `return` 之后的 `a = 1`、`var a = 2` 根本执行不到，所以返回的是那个函数。

## 3. 函数声明写在最后，也会先执行

```js
function test() {
    a = 1;               // 改的是 AO.a
    function a() {}
    var a = 2;
    return a;
}
console.log(test());     // 2
```

AO 的演变：`undefined → function a(){}`（预编译）`→ 1`（执行 a = 1）`→ 2`（执行 var a = 2），所以返回 **2**。

> 三个坑的共同解法：**先按「AO 三步」把对象建出来，再从上往下执行**。遇到 `var` 就找 AO 里有没有同名属性 —— 有就是改局部，没有才是全局。

# 六、综合例题：把 AO / GO 完整走一遍


![07-walkthrough.png](https://img.215454.xyz/file/1790392824029_07-walkthrough.png)

```js
a = 1;
function test(e) {
    function e() {}
    arguments[0] = 2;
    console.log(e);        // 2
    if (a) { var b = 3; }
    var c;
    a = 4;
    var a;
    console.log(b);        // undefined
    f = 5;
    console.log(c);        // undefined
    console.log(a);        // 4
}
var a;
test(1);
console.log(a);            // 1
console.log(f);            // 5
```

## 1. 先建 GO

```text
GO = {
    a:    undefined → 1        （var a，执行时被赋成 1）
    test: function test(e){}
    f:    5                    （函数里 f = 5，没声明 → 隐式全局）
}
```

## 2. 再建 AO（test 的）

```text
AO = {
    e: undefined → 1 → function e(){} → 2
    b: undefined
    c: undefined
    a: undefined → 4
}
```

## 3. 逐行执行

| 代码 | 发生了什么 |
| --- | --- |
| `function e() {}` | 第 ③ 步已处理，跳过 |
| `arguments[0] = 2` | **arguments 和形参联动** → AO.e 变成 2 |
| `console.log(e)` | 输出 **2**（不是那个空函数） |
| `if (a)` | 读的是 **AO.a**（`undefined`）→ 假，跳过 if |
| `var b` / `var c` / `var a` | 预编译都处理过了，等于空语句 |
| `a = 4` | 改的是 **AO.a**（局部） |
| `console.log(b)` | 提升了但没赋过值 → **undefined** |
| `f = 5` | 没声明 → 挂到 **GO** |
| `console.log(c)` | 同上 → **undefined** |
| `console.log(a)` | 读 AO.a → **4** |
| `console.log(a)`（函数外） | 读 GO.a → **1**（函数里的 4 没影响到它） |
| `console.log(f)`（函数外） | 读 GO.f → **5** |

## 4. 四个关键点

1. **`arguments[0] = 2` 会改到形参 e** —— 它们指向同一份数据；
2. **函数里只要写了 `var a`，AO 里就有 a，会遮住全局的 a** —— 所以函数里读到的是 AO.a，函数外的 1 毫发无损；
3. **`f = 5` 没声明 → 挂到 GO**，不在 AO 里；
4. 最终输出：**2、undefined、undefined、4、1、5**。

# 本课小结

- 形参不传值就是 `undefined`；ES6 直接在参数列表写默认值，ES6 之前用 `typeof arguments[i] !== 'undefined' ? arguments[i] : 默认值`；
- **`||` 兜底有坑**：传 `0`、`''`、`false` 都会被当成「没传」；
- JS 的执行分三步：**检查语法错误 → 预编译 → 解释一行执行一行**；
- **函数声明整体提升，变量只有声明提升，赋值不提升**；
- **AO 四步**：① 找形参和变量声明 → ② 实参赋给形参 → ③ 找函数声明并赋值 → ④ 执行；
- **GO 三步**（少了形参那步），**GO === window**；
- **隐式全局变量**：没声明就赋值 → 自动挂到 window 上；`var a = b = 1` 里只有 a 是局部的；
- 遇到任何输出题，**先建 AO/GO，再从上到下执行** —— 有 `var` 就看 AO 里有没有同名属性，有就是改局部，没有才是全局。

> 这一课是后面所有「变量到底是谁的」问题的基础。学到闭包、this、作用域链时，用的还是这套 AO / GO 的思考方式。
