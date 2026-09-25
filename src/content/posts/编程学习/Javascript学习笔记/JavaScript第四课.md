---
title: Java Script第四课
published: 2026-09-25
description: Java Script第四课
tags:
  - JavaScript
author: jia
---
# JS 第四课：嵌套 switch、斐波那契循环与函数入门

  

> 本课对应练习文件：`index.html`（嵌套 switch 日程表、用循环算斐波那契、函数基础、形参与实参、return 与作用域）。

  

# 一、嵌套 switch：一天的日程安排

  

## 1. 为什么要套两层

  

「今天是星期几」和「现在是早上还是晚上」是**两个独立的判断**，所以要用两层 switch：外层决定哪一天，内层决定哪个时段。

  

![嵌套 switch 的判断顺序，以及七天重复写出来的结果](https://img.215454.xyz/file/blog/jia/1790347449232_01-switch-week-plan.png)

  

程序的判断顺序是：先问星期几 → 匹配外层 case → 再问时段 → 匹配内层 case → 最后输出安排。

  

```js

var week = window.prompt('请输入星期几');

var time;

  

switch (week) {

case '星期一':

time = window.prompt('请输入早上中午晚上');

switch (time) {

case '早上':

document.write('今天是星期一早上：记单词');

break;

case '中午':

document.write('今天是星期一中午：吃烤肉');

break;

case '晚上':

document.write('今天是星期一晚上：看电影');

break;

}

break;

// 星期二到星期日：一模一样的三个分支，只是星期换了个字

default:

document.write('输入异常');

}

```

  

## 2. 七天写一遍，问题就出现了

  

上面这段代码，从「星期一」到「星期日」要**复制 7 遍**，一共 21 条 `document.write`。它们除了星期几的名字不一样，逻辑完全相同。

  

> 这就是**重复代码**。重复代码的坏处是：想改一个地方（比如把「吃烤肉」改成「吃食堂」），得改 7 处，漏一处就出 bug。

  

**解决办法就是本课的主角 —— 函数**：把重复的代码提取出来，用一个单独的函数去完成，这就是所谓的**「低耦合」**。

  

## 3. 简化写法：switch 里套 if

  

如果每个分支里只是「上午做 A、下午做 B」这种二分情况，用 `if` 会比再套一层 switch 清爽得多：

  

```js

var weekday = window.prompt('请输入星期几');

var time = window.prompt('请输入上午或下午');

  

switch (weekday) {

case '星期一':

if (time === '上午') {

console.log('看书');

} else if (time === '下午') {

console.log('逛街');

}

break;

}

```

  

> 选择口诀还是那句：**有范围用 `if`，固定值用 `switch`**。两者可以互相嵌套，但嵌套层数越多，代码越难读。

  

# 二、用循环算斐波那契数列

  

课时 3 用递归算过斐波那契（1, 1, 2, 3, 5, 8, 13, 21……）。这一课换一种思路：**用循环，从前往后一位一位推**。

  

## 1. 三个变量，滚动前进

  

只需要三个变量：

  

- `n1`：当前这一对里的前一个数；

- `n2`：当前这一对里的后一个数；

- `n3`：算出来的新数，也就是「再往后一位」。

  

每一轮做两件事：先算出 `n3 = n1 + n2`，再让窗口整体往后挪一格（`n1` 接替 `n2`，`n2` 接替 `n3`）。

  

![斐波那契循环解法：n1、n2 一步步往后滚](https://img.215454.xyz/file/blog/jia/1790347449815_02-fibonacci-loop.png)

  

| 轮次 | 本轮开始时 n1 | 本轮开始时 n2 | 算出 n3 = n1 + n2 | 本轮结束后的 n1、n2 |

| --- | --- | --- | --- | --- |

| 开始前 | 1 | 1 | —— | n1 = 1，n2 = 1 |

| i = 2 | 1 | 1 | 2 | n1 = 1，n2 = 2 |

| i = 3 | 1 | 2 | 3 | n1 = 2，n2 = 3 |

| i = 4 | 2 | 3 | 5 | n1 = 3，n2 = 5 |

| i = 5 | 3 | 5 | 8 | n1 = 5，n2 = 8 |

  

## 2. 代码

  

```js

var n1 = 1;

var n2 = 1;

var n3;

  

if (n <= 2) {

console.log(1); // 第 1、2 位固定是 1

} else {

for (var i = 2; i < n; i++) {

n3 = n1 + n2;

n1 = n2; // 窗口右移

n2 = n3;

}

console.log(n3);

}

```

  

**循环跑了几轮？** `n = 6` 时 `i` 取 2、3、4、5 共 **4 轮**，最后 `n3 = 8`，正好是第 6 位。也就是**循环轮数 = n − 2**。

  

> 移位顺序不能反：一定要**先 `n1 = n2`，再 `n2 = n3`**。如果先写 `n2 = n3`，那么 `n1 = n2` 拿到的就是刚算出来的 `n3`，窗口就滚不起来了。

  

## 3. 输入校验：光有 isNaN 还不够

  

`window.prompt` 拿到的永远是**字符串**，所以要先 `parseInt` 转成数字。但用户可能乱输，得有四道关卡。

  

![输入校验：isNaN 与 String(n) !== input 的区别](https://img.215454.xyz/file/blog/jia/1790347463605_03-input-validation.png)

  

```js

var n = parseInt(window.prompt('请输入第几位：'));

  

if (isNaN(n)) {

console.log('输入异常');

} else {

if (n <= 0) {

console.log('输入错误');

} else {

// ……正常计算

}

}

```

  

`isNaN(n)` 能挡住 `'abc'` 这种「完全转不成数字」的输入，但**挡不住 `'5a'`**：

  

```js

parseInt('5a'); // 5 —— 只转了一半，isNaN(5) 是 false，校验直接放行

```

  

## 4. 更严格的一版：把数字转回字符串再比一次

  

```js

var input = window.prompt('请输入第几位：');

var n = parseInt(input);

  

// 严格要求输入内容整体就是数字，不能混杂字母

if (isNaN(n) || String(n) !== input) {

console.log('输入异常');

} else if (n <= 0) {

console.log('输入错误');

} else {

// 斐波那契逻辑不变

}

```

  

思路是：`parseInt` 的结果再 `String()` 转回去，如果和用户原始输入**一模一样**，说明整串都是数字；只要不一样（比如 `'5a'` 转出来是 `'5'`），就是输入有问题。

  

> 小技巧：`String(n) !== input` 这一招只适用于「纯整数输入」。像 `' 5 '`（带空格）这种其实合法的输入也会被判为异常，需要先 `trim()` 处理。

  

## 5. 循环 vs 递归

  

| | 循环版 | 递归版（课时 3） |

| --- | --- | --- |

| 写法 | 三个变量滚动，`for` 循环 | `return fb(n-1) + fb(n-2)` |

| 代码量 | 略长，但一眼能看懂流程 | 短，接近数学定义 |

| 效率 | 高，算第 n 位只跑 n−2 轮 | 低，同一个值被反复计算很多次 |

| 适合 | 工程里的实际计算 | 理解「规律 + 出口」的递归思想 |

  

# 三、函数是什么

  

## 1. 从数学里的函数说起

  

数学里的函数：对于**任意一个 x 的值，都有一个唯一确定的 y 与之对应**，记作 `y = f(x)`。

  

- x 叫自变量，y 叫函数值；

- 关键词是**确定性** —— 输入一样，输出就一定一样。

  

计算机里的函数借用了这个概念：**给它同样的参数，它总是给你同样的结果**。这种「把计算过程封装起来」的思路，就是函数式编程的起点。

  

## 2. 为什么要用函数：高内聚、低耦合

  

这是本课最重要的一句设计原则：

  

- **高内聚**：代码相关性强、独立性强 —— 一个函数只干一件事，干得完整、干净；

- **低耦合**：重复的代码提取出来，用一个单独的函数来完成 ——模块之间尽量不互相依赖。

  

换个说法就是**模块的单一责任制**：一个功能的代码只负责一个功能，是一个独立的模块，不要依赖其他模块。这个「解耦合」的过程，靠的就是函数。

  

看看这个例子：

  

```js

if (2 > 0) {

test();

}

if (3 > 0) {

test();

}

if (1 > 0) {

test();

}

  

function test() {

for (var i = 0; i < 10; i++) {

console.log(i);

}

}

```

  

那段循环本来要写三遍，现在只写一遍，需要的地方**叫一下函数名**就行。以后想改逻辑，也只改一处。

  

> 注意这个例子里 `test()` 写在函数声明**之前**，却依然能调用 —— 因为「函数声明」会先被 JS 处理（可以先调用后声明），而「函数表达式」不行，见下一节。

  

## 3. 函数的基本写法

  

```js

function test(参数) {

// 函数的执行语句

}

```

  

![函数的结构：入口是参数，出口是 return](https://img.215454.xyz/file/blog/jia/1790347466057_04-function-basics.png)

  

| 部分 | 作用 |

| --- | --- |

| `function` | 关键字，告诉 JS「我要声明一个函数」 |

| 函数名 | 调用时用它，遵循命名规则 |

| `( 参数 )` | **入口**：接收外面传进来的数据 |

| `{ 执行语句 }` | 函数真正要干的活 |

| `return 值` | **出口**：把结果交出去，顺便结束函数 |

  

## 4. 写完 ≠ 执行

  

```js

function test() {

console.log('我被执行了');

}

// 到这里为止，什么都不会打印 —— 函数只是被「定义」了

  

test(); // ← 写了这一句，函数体才会跑

```

  

**函数定义好之后一直处于「待命」状态，只有被调用才会执行。**

  

## 5. 函数名的命名规则

  

- 不能以数字开头；

- 可以包含字母、下划线 `_`、美元符号 `$`；

- 中间可以包含数字；

- 多个单词用小驼峰命名法：`myWonderfulTest`。

  

# 四、函数的三种写法

  

## 1. 函数声明

  

```js

function test() {

var a = 1,

b = 2;

console.log(a, b);

}

test(); // 1 2

```

  

最常用的写法，函数名就是 `test`。

  

## 2. 具名函数表达式

  

```js

var test = function test1() {

var a = 1,

b = 2;

console.log(a, b);

};

console.log(test.name); // 'test1'

test(); // 1 2

```

  

**注意名字有俩**：赋给变量的名字是 `test`（调用要用它），函数自己内部还有一个名字 `test1`。所以 `test.name` 打印出来是 `'test1'`。

  

## 3. 匿名函数表达式

  

```js

var test = function () {

var a = 1,

b = 2;

console.log(a, b);

};

```

  

函数自己没写名字，也叫**函数字面量**，全靠变量名 `test` 来调用。

  

## 4. 三者的区别

  

| 写法 | 样子 | 能不能先调用后声明 |

| --- | --- | --- |

| 函数声明 | `function test(){}` | 可以（声明会被提前处理） |

| 具名函数表达式 | `var test = function test1(){}` | 不行（本质是赋值语句） |

| 匿名函数表达式 | `var test = function(){}` | 不行 |

  

# 五、形参、实参和 arguments

  

![形参、实参、arguments 的对应关系](https://img.215454.xyz/file/blog/jia/1790347475402_05-params-arguments.png)

  

## 1. 形参：形式上占位

  

```js

function test(a, b) { // a、b 就是形参

console.log(a + b);

}

```

  

形参写在函数定义的括号里，只是**占个位置**（相当于两个待赋值的变量），它自己并不知道将来会收到什么。

  

## 2. 实参：真正传进去的数据

  

```js

test(1, 2); // 1、2 就是实参

```

  

实参写在**调用**的括号里，是实际参与运算的数据。

  

- **实参按顺序一一对应形参**：第 1 个实参给第 1 个形参，第 2 个给第 2 个；

- 参数**没有数据类型的区分**：`test('false', NaN)` 也完全合法；

- 形参和实参的**数量可以不一样**：

- 实参多了 → 多出来的没有形参接收（可以用 `arguments` 拿到）；

- 实参少了 → 没收到值的形参就是 `undefined`。

  

```js

test(1, 2, 3);

function test(a, b) {

console.log(a, b); // 1 2 —— 第三个实参没人接

}

```

  

## 3. arguments：装着全部实参

  

函数内部随时可以使用 `arguments`（不用提前声明），它是一个**像数组的对象**，装着这次调用传进来的所有实参：

  

```js

function test(a, b) {

console.log(arguments); // [1, 2, 4]

console.log(arguments[1]); // 2

console.log(arguments.length); // 3 —— 实参的个数

console.log(test.length); // 2 —— 形参的个数

  

for (var i = 0; i < arguments.length; i++) {

console.log(arguments[i]); // 挨个打印每个实参

}

}

test(1, 2, 4);

```

  

> 对比记牢：**`arguments.length` 是实参个数，函数名 `.length` 是形参个数。**

  

## 4. 形参和实参是联动的

  

```js

function test(a, b) {

a = 3; // 改形参

console.log(arguments[0]); // 3 —— arguments[0] 也跟着变了

console.log(arguments[2]); // 7

}

test(1, 2, 7);

```

  

在函数内部**改形参的值，实参也会跟着变**（它们指向同一份数据）。而 `arguments[2]`（多传的那个 7）没有形参和它对应，只能通过 `arguments` 取。

  

## 5. 练习：累加任意个数的实参

  

```js

function sum() {

var a = 0;

for (var i = 0; i < arguments.length; i++) {

a += arguments[i];

}

console.log(a);

}

sum(1, 2, 3, 4, 5, 6); // 21

```

  

形参列表可以是空的，参数个数完全由调用方决定 —— 这就是 `arguments` 最大的用处。

  

> 顺带一提：`Number(parseInt(window.prompt('a')))` 这种写法是多余的，`parseInt` 的返回值已经是数字类型了，不用再套一层 `Number`。

  

# 六、return：结束函数 + 交出结果

  

![return 的两个作用：立刻结束函数、把结果交出去](https://img.215454.xyz/file/blog/jia/1790347480891_06-return.png)

  

## 1. 遇到 return，函数立刻停下

  

```js

function test() {

console.log('我正在执行');

return 0;

console.log('我执行完了就结束这个函数'); // 永远不会执行

}

test();

```

  

执行过程：

  

1. `console.log('我正在执行')` → 打印；

2. `return 0;` → 把 0 交回去，函数到此结束；

3. 第三行属于**死代码**，写了也白写。

  

## 2. 把结果交出去

  

调用一个函数，得到的就是它的返回值：

  

```js

function add(a, b) {

return a + b; // 把结果交出去

}

console.log(add(1, 2)); // 3

```

  

**没有写 return 的函数，调用结果一律是 `undefined`。**

  

## 3. 常见用法：给参数配一个默认值

  

```js

function test(name) {

return name || '您没有填写姓名！';

}

console.log(test('程小野')); // 程小野

console.log(test()); // 您没有填写姓名！

```

  

`||` 的规则是**有真则真**：左边是假值就取右边的值。

  

- 传了 `'程小野'` → 真 → 返回 `'程小野'`；

- 没传参数 → `name` 是 `undefined` → 假 → 返回右边的默认文案。

  

> 这段代码比 `if (!name) { return '您没有填写姓名！'; } return name;` 短得多，是实际开发里很常用的写法。

  

# 七、作用域：变量在哪里能被看见

  

![作用域链：里面能看见外面，外面看不见里面](https://img.215454.xyz/file/blog/jia/1790347493513_07-scope-chain.png)

  

## 1. 全局变量与局部变量

  

- **全局变量**：没写在任何函数里面的变量，程序里到处都能用；

- **局部变量**：写在函数里面的变量（`var` 声明的），只在函数内部有效。

  

```js

var g = 1; // 全局变量

  

function test1() {

var b = 2; // 局部变量，只有 test1 里能用

console.log(g, b); // 1 2 —— 里面有外面的 g

function test2() {

var c = 3; // 局部变量，只有 test2 里能用

console.log(g, b, c); // 1 2 3 —— 一层层往外都能访问

}

test2();

// console.log(c); // ✗ c is not defined，test1 里看不到 test2 的 c

}

test1();

// console.log(b); // ✗ b is not defined，外面看不到里面的 b

```

  

## 2. 作用域链：一层一层往外找

  

函数里用到一个变量时，查找顺序是：

  

1. 先看**自己函数里**有没有；

2. 没有 → 去**外层函数**里找；

3. 一直找到**全局作用域**还没有 → 报错 `xxx is not defined`。

  

这个逐层向外查找的链子，就叫**作用域链**。

  

## 3. 并列的函数互不可见

  

```js

function test1() {

var a = 1;

console.log(b); // ✗ b is not defined

}

function test2() {

var b = 2;

console.log(a); // ✗ a is not defined

}

test1();

test2();

```

  

两个函数是**并列关系**（不存在谁包含谁），所以各自的局部变量**谁也看不见谁**。

  

> 一句话记住：**里面能看见外面，外面看不见里面。**

  

## 4. 不写 var 会怎样？

  

```js

function test() {

var a = b = 1; // 相当于 var a = (b = 1)

console.log(a, b); // 1 1

}

test();

console.log(b); // 1 —— b 没写 var，变成了全局变量

console.log(a); // ✗ a is not defined —— a 是局部变量

```

  

`var a = b = 1` 这种连写方式要小心：**只有 `a` 被 `var` 声明成了局部变量，`b` 悄悄变成了全局变量**。

  

## 5. 回到出发点：封装

  

> 一个固定的功能或者程序段被封装的过程，就是**函数**。在这个封装体里需要一个**入口**和一个**出口**：入口就是参数，出口就是返回。

  

这句总结正好把本课串起来了：从「重复的 switch 日程表」这个痛点出发，用函数把重复代码封装起来（低耦合），而函数的对外接口就是「参数进、return 出」。

  

# 本课小结

  

- 嵌套 switch 用来处理**两个独立的判断**（星期 + 时段），但同样的分支写七遍就是重复代码，应该提取成函数；

- 斐波那契的循环解法：`n3 = n1 + n2` 之后，**先 `n1 = n2` 再 `n2 = n3`**，窗口逐格右移；循环轮数 = n − 2；

- 输入校验：`isNaN` 只能挡住「完全不是数字」的输入，`'5a'` 这种要用 `String(n) !== input` 才能发现；

- 用函数的意义是**高内聚、低耦合**：一个函数只干一件事，重复代码只写一遍；

- 函数**只有被调用才会执行**；声明可以先调用后声明，函数表达式不行；

- 形参是占位的变量，实参是真正传进去的数据，**按顺序一一对应**，数量可以不等；

- **`arguments.length` 是实参个数，函数名 `.length` 是形参个数**；改形参，实参会跟着变；

- `return` 有两个作用：**交出结果** + **立刻结束函数**；没写 return 的函数返回 `undefined`；

- 作用域链：**里面能看见外面，外面看不见里面**；不写 `var` 的赋值会变成全局变量。