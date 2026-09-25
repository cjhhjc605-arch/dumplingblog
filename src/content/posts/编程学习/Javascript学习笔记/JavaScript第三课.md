---
title: JavaScript第三课
published: 2026-09-25
description: JavaScript第三课
tags:
  - JavaScript
author: jia
---
# JS 第三课：循环结构、引用值与类型转换

  

> 本课对应练习文件：`index.html`（for / while / do…while 循环、break 与 continue、引用值、typeof 与类型转换）、`test.html`（函数练习题与递归）。

  

# 一、for 循环

  

## 1. 基本写法

  

```js

for (初始化; 判断条件; 变量更新) {

// 循环体

}

```

  

`for` 把循环需要的三件事都写在括号里，一眼就能看清「从哪里开始、什么时候停、每一轮怎么变」。

  

![for 循环执行流程：① 初始化 → ② 判断 → 循环体 → ③ 更新 → 回到 ②](images/01-for-flow.png)

  

## 2. 执行顺序

  

- **① 初始化**：只在最开始执行一次；

- **② 判断条件**：为真 → 执行循环体；为假 → 整个循环结束；

- **③ 变量更新**：更新完再回到 ② 重新判断。

  

以 `for (var i = 0; i < 3; i++) { console.log(i); }` 为例：

  

| 轮次 | ① i 的值 | ② i < 3 ? | ③ 输出 | ④ i++ 之后 |

| ------ | -------- | --------- | ------ | ---------- |

| 第 1 轮 | 0 | true | 0 | i = 1 |

| 第 2 轮 | 1 | true | 1 | i = 2 |

| 第 3 轮 | 2 | true | 2 | i = 3 |

| 第 4 轮 | 3 | false | —— | 循环结束 |

  

> 循环次数 = 3 次，但**判断执行了 4 次** —— 最后一次判断为假，正是循环结束的出口。

  

## 3. for 与 while 可以互相改写

  

初始化和变量更新都可以挪到括号外面，`for` 就变成了 `while`：

  

```js

// 初始化和更新都挪出去

var i = 0;

for (; i < 10; ) {

console.log(i);

i++;

}

  

// 等价的 while 写法

var i = 0;

while (i < 10) {

console.log(i);

i++;

}

```

  

> 三个部分都可以省略，但**两个分号不能省**。`for (;;)` 就是死循环，等价于 `while (1)`。

  

## 4. 两个小练习

  

**练习一：倒着打印 0 ~ 99**

要求：括号里只能写一句、不能写判断语句、`{}` 里不能出现 `i++` / `i--`。

  

```js

var i = 100;

for (; i--; ) {

console.log(i);

}

```

  

`i--` 是**先返回旧值、再自减**：

  

- `i = 100` 时，`i--` 返回 100（真），`i` 变成 99 → 打印 99；

- ……一路打印 98、97 …… 1、0；

- `i = 0` 时，`i--` 返回 0（假）→ 循环结束。

  

所以最终打印 99 → 0，正好是 0 ~ 99 这 100 个数。

  

**练习二：10 的 n 次方**

  

```js

var n = 5;

var num = 1;

for (var i = 0; i < n; i++) {

num = num * 10; // 循环 5 次，每次乘 10

}

console.log(num); // 100000

```

  

# 二、while 与 do…while

  

![while 与 do…while 的区别：先判断 vs 先执行](images/02-while-dowhile.png)

  

## 1. while：先判断，后执行

  

```js

var i = 0;

while (i < 10) {

console.log(i);

i++;

}

```

  

**条件一开始就不满足时，循环体一次都不会执行。**

  

## 2. do…while：先执行，后判断

  

```js

var i = 0;

do {

console.log('我要开始循环了');

i++;

} while (i < 10); // ← 注意这里有分号

```

  

**不管条件如何，循环体至少执行一次**，然后才去判断要不要再来一轮。

  

> 写法上最容易漏的是 `while (条件)` 后面的那个**分号**。

  

## 3. 三者的区别

  

| 循环 | 判断位置 | 循环体最少执行次数 |

| ------------ | -------- | ------------------ |

| `for` | 每次循环体之前 | 0 次 |

| `while` | 每次循环体之前 | 0 次 |

| `do…while` | 每次循环体之后 | **1 次** |

  

## 4. 死循环

  

```js

while (1) { // 条件永远为真 → 死循环

console.log(i);

i++;

}

```

  

死循环本身不报错，但页面会卡住，必须靠 `break` 跳出来。

  

# 三、break 与 continue

  

![break 与 continue：一个跳出整个循环，一个只跳过本次](images/03-break-continue.png)

  

## 1. break —— 立刻跳出整个循环

  

从 0 开始累加，加到 `sum >= 100` 就停：

  

```js

var sum = 0;

for (var i = 0; i < 100; i++) {

sum += i;

if (sum >= 100) {

break; // 循环到此结束

}

console.log(i, sum);

}

```

  

- `i = 13` 时 `sum = 91`，还没到 100，继续；

- `i = 14` 时 `sum = 105 ≥ 100` → `break`，循环结束；

- `i = 15、16、…` 再也不会执行。

  

## 2. continue —— 只跳过本次，进入下一轮

  

打印 0 ~ 100 中，跳过 7 的倍数、以及个位数是 7 的数：

  

```js

for (var i = 0; i <= 100; i++) {

if (i % 7 == 0 || i % 10 == 7) {

continue; // 跳过本次，进入下一轮

}

console.log(i);

}

```

  

被跳过的数：0、7、14、17、21、27、28……其余正常打印。

  

> **关键区别**：`continue` 只是结束**本轮**剩下的语句，`i++` 和下一轮判断照常进行；`break` 则是**整个循环**就此中断。

  

| | 作用范围 | 循环变量还会继续变化吗 |

| --- | --- | --- |

| `break` | 整个循环立即结束 | 不会 |

| `continue` | 只结束本轮剩余语句 | 会（`for` 里的 `i++` 照常执行） |

  

# 四、循环练习

  

## 1. n 的阶乘

  

```js

var n = 5;

var num = 1;

for (var i = 1; i <= n; i++) {

num *= i; // 1×2×3×4×5

}

console.log(num); // 120

```

  

## 2. 数字倒序输出：789 → 987

  

用 `%` 取余拿个位，用 `/` 和减法拿高位：

  

```js

var num = 789;

var a = num % 10; // 9 → 个位

var b = (num - a) % 100 / 10; // 8 → 十位

var c = (num - a - b * 10) / 100; // 7 → 百位

console.log("" + a + b + c); // "987"

```

  

> 前面加一个空字符串 `""` 是为了让 `+` 做**字符串拼接**，否则 9 + 8 + 7 会算成 24。

  

## 3. 三个数中的最大值（嵌套 if）

  

```js

var a = 1, b = 2, c = 3;

if (a > b) {

if (a > c) {

console.log(a);

} else {

console.log(c);

}

} else {

if (b > c) {

console.log(b);

} else {

console.log(c);

}

}

```

  

思路：先让 `a` 和 `b` 比，胜出的一方再去和 `c` 比，最后胜出的就是最大值。

  

## 4. 100 以内的质数（双重循环）

  

质数：**只能被 1 和自身整除**的数。让内层循环用 `j` 从 1 数到 `i`，数一数 `i` 能被几个数整除。

  

```js

var c = 0;

for (var i = 2; i < 100; i++) {

for (var j = 1; j <= i; j++) {

if (i % j == 0) {

c++; // i 能被 j 整除，计数 +1

}

}

if (c == 2) {

console.log(i); // 只有 1 和自身能整除 → 质数

}

c = 0; // 换下一个 i 前必须归零

}

```

  

![双重循环判定质数：外层 i 与内层 j，统计约数个数](images/04-nested-loop-prime.png)

  

**容易踩的坑**：每检查完一个 `i` 都要把计数器 `c = 0` 归零，否则 `c` 会一直累加，后面的数全被判错。

  

# 五、引用值：数组与对象

  

JS 里的数据类型分为两大家族：

  

- **原始值（基本类型）**：number、string、boolean、undefined、null；

- **引用值**：array（数组）、object（对象）、function（函数）、Date、RegExp。

  

## 1. 数组

  

```js

var arr = [1, 2, 3, 4, 5, 6, 7];

console.log(arr[5]); // 6 —— 下标从 0 开始

console.log(arr.length); // 7

arr[3] = null; // 可以直接改某个位置的值

  

// 配合循环批量处理

for (var i = 0; i < arr.length; i++) {

arr[i] += 2; // 每个元素都加 2

}

console.log(arr);

```

  

## 2. 对象

  

```js

var person = {

name: '小城', // 键名（属性名）: 属性值（键值）

age: 18,

height: 180,

weight: 140,

job: 'WEB开发工程师'

};

person.name = '小王'; // 改属性

console.log(person.name); // 小王

```

  

## 3. 原始值与引用值最大的区别：赋值时复制的是什么

  

![原始值与引用值的赋值区别：复制值 vs 复制地址](images/05-primitive-vs-reference.png)

  

```js

// 原始值：复制的是「值」

var a = 1;

var b = a; // b 拿到一份拷贝

b = 2;

console.log(a, b); // 1 2 —— a 完全不受影响

  

// 引用值：复制的是「地址」

var arr = [1, 2, 3];

var arr2 = arr; // arr2 和 arr 指向同一个数组

arr2[0] = 99;

console.log(arr); // [99, 2, 3] —— 通过 arr2 改动，arr 也变了

```

  

> 记住一句话：**原始值变量里装的是值本身，引用值变量里装的是地址**。两个变量存着同一个地址，改哪个都算数。

  

# 六、typeof 与数据类型

  

`typeof` 用来查看一个值到底是什么类型，它返回的结果是一个**字符串**。

  

| 表达式 | 结果 | 说明 |

| ----------------------- | ----------- | ------------------------ |

| `typeof(123)` | `'number'` | 数字 |

| `typeof('abc')` | `'string'` | 字符串 |

| `typeof(true)` | `'boolean'` | 布尔值 |

| `typeof([])` | `'object'` | 数组属于对象 |

| `typeof({})` | `'object'` | 对象 |

| `typeof(null)` | `'object'` | 历史遗留问题，记住即可 |

| `typeof(undefined)` | `'undefined'` | 未定义 |

| `typeof(function(){})` | `'function'` | 函数 |

| `typeof(typeof(123))` | `'string'` | 因为 typeof 返回的是字符串 |

  

其他几个容易忽略的：

  

```js

console.log(typeof(1 - 1)); // number

console.log(typeof(1 - "1")); // number —— 减法把字符串转成了数字

console.log(typeof(a)); // undefined（a 未声明过）

```

  

> 两个特殊情况：**`null` 的 typeof 是 `'object'`**，**`NaN` 的 typeof 是 `'number'`**（NaN 虽然是「非数」，但它属于数字类型）。

  

# 七、显式类型转换

  

显式转换就是我们主动调用函数去改变类型：`Number()`、`parseInt()`、`parseFloat()`、`String()`、`Boolean()`。

  

![Number / parseInt / parseFloat 对同一个输入的不同结果](images/06-number-parseint.png)

  

## 1. Number()：最严格的转换

  

| 传入的值 | 结果 | 说明 |

| ------------- | ------- | ----------------------------- |

| `Number('123')` | `123` | 纯数字字符串 |

| `Number('3.14')`| `3.14` | 小数也可以 |

| `Number('1a')` | `NaN` | 整串必须都是数字 |

| `Number('true')`| `NaN` | 字符串 'true' 不是数字 |

| `Number(true)` | `1` | 布尔值有定义：true = 1，false = 0 |

| `Number(null)` | `0` | null 转成 0 |

| `Number(undefined)` | `NaN` | 转不成数字 |

  

## 2. parseInt()：从左往右读，取整数

  

```js

parseInt('123'); // 123

parseInt('3.12'); // 3 —— 直接砍掉小数，不四舍五入

parseInt('3.98'); // 3

parseInt('123abc'); // 123 —— 读到不能读为止

parseInt('abc123'); // NaN —— 开头就不是数字

parseInt(true); // NaN —— parseInt 不认布尔值

parseInt(null); // NaN

parseInt(undefined); // NaN

parseInt(NaN); // NaN

```

  

**第二个参数是进制**：

  

```js

parseInt('11', 16); // 17 —— 16 进制里的 11

parseInt('b', 16); // 11 —— 16 进制里的 b

parseInt('100', 2).toString(16); // '4' —— 二进制 100 → 十进制 4 → 十六进制 '4'

```

  

> 顺带一提：网页里的十六进制颜色值 `#fff`、`#f0f0f0` 用的就是这套 16 进制计数法。

  

## 3. parseFloat() 与 toFixed()：保留小数

  

```js

parseFloat('3.1465926'); // 3.1465926（保留小数）

parseFloat('3.1465926').toFixed(2); // '3.15' —— 四舍五入到 2 位小数

```

  

> `toFixed(n)` 的结果是**字符串**，只负责让你看清楚，不参与后续计算时可以直接用。

  

## 4. String() 与 toString()

  

```js

String(123); // '123'

String(true); // 'true'

  

var str = '3.14';

str.toString(); // '3.14'

  

null.toString(); // ❌ 报错

undefined.toString(); // ❌ 报错

```

  

> 区别：`String()` 什么都能转；`toString()` 遇到 `null` 和 `undefined` 会直接报错。

  

## 5. Boolean()：转成真假值

  

```js

Boolean(1); // true

Boolean(null); // false

```

  

**总结：`undefined`、`null`、`NaN`、`""`（空字符串）、`0`（含 -0）、`false` 这 6 个值转成布尔值都是 `false`，其他一律是 `true`。**

  

## 6. isNaN()：判断「是不是非数」

  

```js

isNaN(NaN); // true

isNaN(123); // false

isNaN('123'); // false —— 会先把字符串转成数字 123

isNaN('a'); // true —— 转不成数字

isNaN(null); // false —— null 转成 0

isNaN(undefined); // true —— Number(undefined) 是 NaN

```

  

> `isNaN(x)` 相当于先做 `Number(x)`，再看结果是不是 `NaN`。

  

# 八、隐式类型转换

  

不写转换函数，运算符也会**自动**帮你转，这就是隐式转换。

  

![隐式类型转换的触发场景与常见反直觉结果](images/07-implicit-conversion.png)

  

## 1. 什么时候会隐式转换

  

| 运算符 | 转换规则 | 例子 |

| ----------------- | ------------------------------------------ | ----------------------------- |

| `+` | 只要有一边是字符串 → 拼接成字符串 | `'a' + 1` → `'a1'` |

| `-` `*` `/` `%` | 两边都转成数字再计算 | `'3' * 2` → `6` |

| `++` `--` | 先转成数字，再自增 / 自减 | `'123'` 自增后 → `124`（数字）|

| `>` `<` `>=` `<=` | 转成数字比大小；两边都是字符串则按 ASCII 码比 | `1 > '2'` → `false` |

| 一元 `+` `-` | 直接转成数字 | `+'123'` → `123`；`-'abc'` → `NaN` |

  

```js

var a = '123';

a++; // a 变成数字 124

  

a = a + '4'; // '1234' —— 加号做拼接

var b = '3' * 2; // 6 —— 乘号做数学运算

  

var c = typeof(+'123'); // 'number'

var d = +'abc'; // NaN

```

  

## 2. 那些「反直觉」的结果

  

| 表达式 | 结果 | 为什么 |

| ------------------------- | ------- | --------------------------------------------- |

| `1 == '1'` | `true` | `==` 会先把两边转成同一类型 |

| `1 != '2'` | `true` | 同上 |

| `1 === '1'` | `false` | `===` 不转换类型，类型不同直接为假 |

| `NaN == NaN` | `false` | NaN 不等于任何值，**连自己都不等** |

| `undefined == null` | `true` | 特殊规定，只有它俩互相相等 |

| `undefined == 0` | `false` | undefined 转数字是 NaN |

| `undefined == NaN` | `false` | 同 NaN 有关的一律不等 |

| `null == 0` | `false` | 虽然 `Number(null)` 是 0，但比较时不相等 |

| `null > 0` / `null < 0` | `false` | 都成立不了 |

| `undefined > 0` / `< 0` | `false` | 都成立不了 |

| `null > undefined` | `false` | 都成立不了 |

| `'a' > 'b'` | `false` | 字符串按 ASCII 码比：'a'(97) < 'b'(98) |

| `2 > 1 == 1` | `true` | 先算 `2 > 1` 得 `true`，`true == 1` → `1 == 1`|

| `2 > 1 > 3` | `false` | 先算 `2 > 1` 得 `true`，`true > 3` → `1 > 3` |

  

> 两个重点：

> 1. **比较一律用 `===` / `!==`**，避免 `==` 的隐式转换带来意外；

> 2. **关系运算符从左到右算**，`2 > 1 > 3` 不是数学上的连续比较，而是 `(2 > 1) > 3`。

  

# 九、函数与递归

  

## 1. 四个练习任务（对应 test.html）

  

1. 定义函数，通过 `window.prompt` 接收一个饮料名称，返回对应价格；

2. 定义函数，接收一个运算符（`+` `-` `*` `/` `%`）和第二个数，做运算并返回结果；

3. 接收 n，算出 n 的阶乘 —— **不能用 for 循环**；

4. 定义函数，接收 n，算出斐波那契数列的第 n 位 —— **不能用 for 循环**。

  

前两题用 `switch` + `window.prompt` 完成：

  

```js

var drink = window.prompt('请输入饮料名称');

function price(drink) {

switch (drink) {

case '可乐':

document.write('可乐：¥4');

break;

case '雪碧':

document.write('雪碧：¥5');

break;

case '矿泉水':

document.write('矿泉水：¥2');

break;

default:

document.write('没找到');

}

}

price(drink);

```

  

后两题不能用循环，就得用**递归**。

  

## 2. 递归的两个要素

  

![递归：调用栈的「递」与「归」，以及斐波那契数列](images/08-recursion.png)

  

- **规律**：怎么把大问题变成同样的小问题；

- **出口**：最小、已知答案、能让递归停下来的情况。

  

**没有出口的递归就是死循环**，会一直调用下去直到报错。

  

## 3. 阶乘的递归写法

  

```js

function fact(n) {

if (n === 1) { // 出口

return 1;

}

return n * fact(n - 1); // 规律

}

console.log(fact(5)); // 120

```

  

执行过程是**先递下去、再归回来**：

  

```text

fact(5) = 5 × fact(4) → 120

fact(4) = 4 × fact(3) → 24

fact(3) = 3 × fact(2) → 6

fact(2) = 2 × fact(1) → 2

fact(1) = 1 ← 到达出口，开始往回算

```

  

## 4. 斐波那契数列

  

数列规律：**1, 1, 2, 3, 5, 8, 13, 21 ……** 前两位固定是 1，从第 3 位起，每一位 = 前两位之和。

  

```js

function fb(n) {

if (n <= 0) {

return 0;

}

if (n <= 2) { // 出口：第 1、2 位都是 1

return 1;

}

return fb(n - 1) + fb(n - 2); // 规律：前两位相加

}

console.log(fb(3)); // 2

```

  

展开来看：

  

```text

fb(5) = fb(4) + fb(3) = 3 + 2 = 5

fb(4) = fb(3) + fb(2) = 2 + 1 = 3

fb(3) = fb(2) + fb(1) = 1 + 1 = 2

fb(2) = 1 ← 出口

fb(1) = 1 ← 出口

```

  

> 递归的写法比循环短，但每调用一次函数都会占用一层「调用栈」，n 很大时会明显变慢。

  

# 本课小结

  

- **for** 的三个部分：初始化只做一次，判断 → 循环体 → 更新反复进行；循环判断次数 = 循环次数 + 1；

- **while** 先判断后执行（可能一次都不执行），**do…while** 先执行后判断（至少执行一次，注意结尾分号）；

- **break** 跳出整个循环，**continue** 只跳过本轮、`i++` 照常执行；

- 双重循环常用来「逐个检查 + 逐个试除」，注意内层计数器每轮要**归零**；

- **原始值**赋值复制的是值，**引用值**赋值复制的是地址，两个变量指向同一对象时改哪个都算数；

- `typeof` 返回字符串；`typeof(null)` 是 `'object'`，`typeof(NaN)` 是 `'number'`；

- **显式转换**：`Number()` 最严格，`parseInt()` / `parseFloat()` 从左往右读到不能读为止，`toFixed()` 四舍五入保留小数位；

- **6 个假值**：`undefined`、`null`、`NaN`、`""`、`0`、`false`，其余都是真；

- 比较用 **`===`**；`NaN == NaN` 是 `false`，`undefined == null` 是 `true`；

- **递归** = 规律 + 出口，先递下去找到出口，再一层层归回来算结果。