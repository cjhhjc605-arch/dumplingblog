---
title: "第十七课"
published: 2026-10-15
description: This is the first post of my new Astro blog.
tags: [JavaScript]
author: jia
---
# JS 第十七课：算法思维首秀 —— 去重、封装 typeof 与查表法

  

> 本课对应练习文件：`index.html`（数组去重登记表法、封装 myTypeof 与 typeof 三大软肋、字符串去重、找第一个不重复字符、闭包计数器、函数表达式的名字、数组查表法优化 switch、稀松数组复习）。

  

# 一、数组去重：登记表法（上节课作业正解）

  

![数组去重：登记表法实战（上节课作业正解）](https://img.215454.xyz/file/blog/jia/1792059334428_01-dedup-unique.png)

  

```js

var arr = [0,0,1,1,2,3,3,4,4,5,5,6];

  

Array.prototype.unique = function(){

var temp = {}, newArr = []; // 登记表 + 结果

for (var i = 0; i < this.length; i++){

if (!temp.hasOwnProperty(this[i])){

temp[this[i]] = this[i]; // 登记

newArr.push(this[i]); // 只收第一次见到的

}

}

return newArr;

};

console.log(arr.unique()); // [0,1,2,3,4,5,6]

```

  

## 为什么必须用 hasOwnProperty 判重？

  

数组里**特意放了 0**，这是这题的两颗地雷：

  

```js

// 坑 1：0 是"假值"！

temp[0] = 0; // 第一个 0 登记进去

!temp[0]; // → true，被当成"没见过"

// 第二个 0 又被收进来 —— 去重失败！

  

// 坑 2：'toString' 这类键名，会撞上

// 从 Object.prototype 继承来的属性

```

  

`hasOwnProperty` 只检查"**自己身上**有没有这个键"，完美避开两个坑。所以正规写法必须 `temp.hasOwnProperty(key)`，不能图省事写 `!temp[key]`。

  

> **课堂版一处小修：** 循环里写的是 `arr.length`（写死了全局数组），挂到原型上应该用 `this.length` —— "挂谁改谁"。

  

## 解法家族（温故知新）

  

| 解法 | 思路 | 特点 |

| :--- | :--- | :--- |

| **登记表法（本课正解）** | 对象记"见过没" | 一趟遍历 O(n)，最快最正统 |

| indexOf 法 | 新数组里查存在性 | O(n²)，最直观 |

| 排序邻居法 | 排序后相同元素必相邻 | 会打乱原顺序 |

| 就地双循环 splice | 发现重复删后面那个 | 直接改原数组，练 `splice` |

| ES6 `[...new Set(arr)]` | Set 天生不存重复 | 一行流，毕业后的写法 |

  

# 二、封装 typeof：typeof 失灵现场与修复

  

![封装 typeof：typeof 失灵现场与修复方案](https://img.215454.xyz/file/blog/jia/1792059337174_02-mytypeof.png)

  

## 1. typeof 只认识"基础类型"

  

```js

console.log(typeof 'str'); // "string" ✓

console.log(typeof 123); // "number" ✓

console.log(typeof undefined); // "undefined" ✓

console.log(typeof function(){}); // "function" ✓

  

console.log(typeof new String); // "object" —— 包装类失灵！

console.log(typeof [1,2,3]); // "object" —— 分不出数组

console.log(typeof null); // "object" —— 经典 bug

```

  

**typeof 的三大软肋：** `null`、数组、包装类（`new String` / `new Number` / `new Boolean`）全部报 `'object'`。

  

## 2. 精准识别：Object.prototype.toString

  

```js

var toStr = Object.prototype.toString;

toStr.call(new String); // "[object String]"

toStr.call([1,2,3]); // "[object Array]"

toStr.call(null); // "[object Null]"

```

  

它返回 `"[object Xxx]"` 形式的**精确标签**，拿它当键去映射表里"翻译"就行：

  

```js

var res = {

'[object Array]': 'array',

'[object Object]': 'object',

'[object Number]': 'object number',

'[object String]': 'object string',

'[object Boolean]': 'object boolean'

};

```

  

## 3. 完整封装 myTypeof

  

```js

function myTypeof(val){

var type = typeof(val);

var res = {

'[object Array]': 'array',

'[object Object]': 'object',

'[object Number]': 'object number',

'[object String]': 'object string',

'[object Boolean]': 'object boolean'

};

var toStr = Object.prototype.toString;

  

if (val === null){

return 'null'; // ① 先堵 null

} else if (type === 'object'){

var ret = toStr.call(val); // ② 是 object 的查表翻译

return res[ret];

} else {

return type; // ③ 基础类型直接返回

}

}

  

console.log(myTypeof(new String)); // "object string"

```

  

封装它的意义 = 做出 jQuery `$.type()` 那种"一处判断、全类型通吃"的工具函数。

  

# 三、字符串去重：同一张登记表，换个大厨

  

![字符串去重：同一张登记表，换个大厨](https://img.215454.xyz/file/blog/jia/1792059346184_03-string-unique.png)

  

思路和数组去重一模一样，只换"收集"的方式：`push` 变 `+=` 拼接。

  

```js

var str = '111222000aabb';

  

String.prototype.unique1 = function(){

var temp = {}, newStr = ''; // 登记表 + 结果串

for (var i = 0; i < this.length; i++){

if (!temp.hasOwnProperty(this[i])){

temp[this[i]] = this[i]; // 登记

newStr += this[i]; // 只拼第一次见到的

}

}

return newStr;

};

console.log(str.unique1()); // "120ab"

```

  

| 步骤 | 数组去重 | 字符串去重 |

| :--- | :--- | :--- |

| 登记表 | `var temp = {}` | `var temp = {}` |

| 遍历 | `i < this.length` | `i < this.length` |

| 判重 | `!temp.hasOwnProperty(x)` | `!temp.hasOwnProperty(x)` |

| 收集 | `newArr.push(x)` | `newStr += x` |

| 返回 | 新数组 | 新字符串 |

  

> **一鱼两吃：** 同一个思路，换个结果容器和收集方式就能移植 —— 这就是"学思路不背题"的价值。（课堂版同样把 `str.length` 写死了，规范写法用 `this.length`。）

  

# 四、找第一个不重复的字符：两轮遍历套路

  

![找第一个不重复的字符：两轮遍历套路](https://img.215454.xyz/file/blog/jia/1792059355658_04-first-norepeat.png)

  

```js

var str = 'trujaiogfagljgajljdalrtdbujdlajgcgadgdagl';

  

function test(str){

var temp = {};

// 第一轮：建计数表

for (var i = 0; i < str.length; i++){

if (temp.hasOwnProperty(str[i])){

temp[str[i]]++; // 又见面 → 计数 +1

} else {

temp[str[i]] = 1; // 第一次见 → 记 1

}

}

// 第二轮：找计数为 1 的那一个

for (var key in temp){

if (temp[key] === 1){

return key; // 第一个"只出现一次"的字符

}

}

}

  

console.log(test(str)); // 'i'

```

  

**为什么是 'i'？** `for...in` 按"首次出现顺序"遍历字母键：t、r、u、j、a 全都重复过，轮到 `'i'` 时计数恰好为 1 —— 第一个不重复的字符就此诞生。

  

> **套路总结：两轮遍历 = "先统计，再挑选"。** 想找"出现过不止一次的字符"？把第二轮条件改成 `temp[key] > 1` 即可 —— **换条件、不换骨架**。这类"统计表 + 二次筛选"的写法以后会反复遇到（词频统计、投票计数都是它）。

>

> 小提醒：`for...in` 对字母键按插入顺序遍历，但**数字字符**（`'0'`~`'9'`）会被引擎当作整数键排在前面 —— 统计含数字的字符串时要留意这个细节。

  

# 五、闭包计数器：私有变量 d 的自白

  

![闭包计数器：私有变量 d 的自白](https://img.215454.xyz/file/blog/jia/1792059357760_05-closure-counter.png)

  

```js

function Test(a,b,c){

var d = 0; // 私有变量：外部看不见

function e(){ // 引用 d → 形成闭包

d++;

console.log(d);

}

this.f = e; // 暴露一条"访问通道"

}

  

var test1 = new Test();

test1.f(); // 1

test1.f(); // 2 —— d 被闭包保住，一直活着

var test2 = new Test(); // 全新的一套 d + e

test2.f(); // 1 —— 重新数起

```

  

**为什么 d 不归零？** 函数 `e` 引用了 `d`、`test1.f` 又引用着 `e` → 引用链不断，`d` 就在内存里常驻。**每 new 一次都生成独立的一套闭包**，test1 和 test2 各数各的、互不干扰。

  

- `d` 是构造函数里的局部变量，外面 `test1.d` 永远是 `undefined`；

- 只有通过 `f()` 这条通道才能碰到它 —— 这就是"**私有变量**"的雏形。

  

> **考点串联：** 闭包 = 函数 + 它能记住的环境。看到"多个实例各自持有一份私有状态"，条件反射想到闭包 —— 圣杯模式（第十二课）、this 绑定（第十三课）之后，这里是闭包的又一次实战。

  

# 六、函数表达式的名字：内部可用，外部蒸发

  

![函数表达式的名字：内部可用，外部蒸发](https://img.215454.xyz/file/blog/jia/1792059363183_06-func-expression-name.png)

  

```js

var test = function a(){

a(); // 函数内部能看见 a（递归靠它）

return 'a';

};

// a(); // 外部调用 → ReferenceError！

console.log(test.name); // "a"

```

  

- 函数表达式会自动"**吞掉**"函数名：名字 `a` 只在函数体内部可见（像一个内部小名，专门给递归和调试用）；

- 出了函数，外部谁也别想找到它；

- `test.name` 读出来仍是 `'a'`。

  

## typeof 的安全探测

  

```js

console.log(test.name); // "a"

console.log(typeof a); // "undefined" —— 不报错！

console.log(a); // ReferenceError: a is not defined —— 直接引用必炸

```

  

> **对比记忆：** 函数**声明** `function a(){}` 会把 `a` 注册到外层作用域（内外都能用）；函数**表达式**里的名字 `a` 只活在自己体内。顺带夹着一条冷知识：`typeof 未声明变量` 返回 `"undefined"` 而不报错，所以它常在"检查某个库是否加载"的场景里当探针（如 `typeof jQuery !== 'undefined'`）。

  

# 七、数组查表法：一行干掉十五行 switch

  

![数组查表法：一行干掉十五行 switch](https://img.215454.xyz/file/blog/jia/1792059380306_07-lookup-array.png)

  

```js

function test(day){

var weekday = [, 'Mon','Tue','Wed','Thu',

'Fri','Sat','Sun']; // 0 号位留空

weekday[day] != undefined

? console.log(weekday[day])

: console.log("I don't know");

}

test(3); // "Wed"

test(0); // 空位 → "I don't know"

```

  

- 下标当"键"、数组当"字典"：`day` 是多少就取第几项；

- 空位和越界一律返回 `undefined`，正好用三元运算兜底"不知道"；

- 对比原版 `switch`（15 行起步）：**能把分支变成"查表"的场合，优先查表** —— 数据驱动代替流程控制，逻辑更短、更好改（想加一天？表里补一项就行）。

  

## 稀松数组复习

  

```js

var arr = [,1,2,,,3,4,]; // 稀松数组

console.log(arr[0]); // undefined

console.log(arr.length); // 7 —— 末尾逗号不算一项

```

  

查表法正是靠"**故意留空 0 号位、让下标对齐语义**"这个技巧站稳的 —— 空位不是 bug，是设计。

  

# 本课小结

  

- **登记表法**：一套思路三处开花（数组去重、字符串去重、计数统计）；判重必须用 `temp.hasOwnProperty(key)`，防 0 假值与继承属性两个坑；

- **封装 typeof**：typeof 三大软肋（null / 数组 / 包装类全报 'object'）；`Object.prototype.toString.call()` 拿精确标签，查映射表翻译成人话；

- **两轮遍历**：先统计再挑选，换条件不换骨架；

- **闭包计数器**：私有变量 + 每个实例独立的一套闭包；`test1.d` 拿不到、`test1.f()` 却能持续自增；

- **函数表达式的名字**：内部可用（递归、`test.name`）、外部蒸发；`typeof` 是唯一能安全探测未声明变量的运算符；

- **查表法**：数据驱动代替流程控制；稀松数组的"空位"是设计不是瑕疵。

  

> **下一站预告：** JS 的五大错误类型 —— SyntaxError / ReferenceError / RangeError / TypeError / URIError，以及 encodeURI / decodeURI 编解码。