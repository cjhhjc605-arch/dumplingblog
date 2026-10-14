---
title: "第十三课"
published: 2026-10-14
description: This is the first post of my new Astro blog.
tags: [JavaScript]
author: jia
---
# JS 第十三课：链式调用、属性检测、函数内省与 this 终极法则

  

> 本课对应练习文件：`index.html`（面向对象链式调用 return this、对象枚举与属性检测 hasOwnProperty/in/instanceof、数组精准类型判断、arguments.callee 与 fn.caller、this 绑定四重奏法则、阿里高频手撕笔试题）。

  

# 一、链式调用模式（Chaining）：每一个方法末尾 return this

  

在 jQuery 中经常能见到形如 `$('#box').css('color','red').show().addClass('active')` 的连缀代码。这种行云流水的流畅接口（Fluent Interface）是如何在 JavaScript 底层实现的？

  

![链式调用模式：每一个方法末尾 return this](https://img.215454.xyz/file/blog/jia/1791814177114_01-method-chaining-this.png)

  

## 1. 核心实现秘诀：显式返回宿主实例

  

```js

var sched = {

wakeup: function () {

console.log('Running');

return this; // 👈 把当前宿主对象暴露出去！

},

morning: function () {

console.log('Going shopping');

return this;

},

noon: function () {

console.log('Having a rest');

return this;

},

night: function () {

console.log('Sleeping');

return this;

}

};

  

// 畅通无阻的流水线连缀调用：

sched.wakeup()

.morning()

.noon()

.night();

```

  

## 2. 底层运行机制

  

- `sched.wakeup()` 执行结束后，返回了 `this`（即 `sched` 对象本身）；

- 后续的 `.morning()` 就相当于紧接着在执行 `sched.morning()`；

- **如果某一步漏写了 `return this`**：该方法将默认返回 `undefined`，下一步调用就会变成 `undefined.noon()`，直接抛出 `TypeError: Cannot read properties of undefined` 致命报错！

  

# 二、属性检测三剑客：hasOwnProperty vs in vs instanceof

  

当对象接入了原型链之后，属性的来源可能来自“自身挂载”，也可能来自“祖先原型”。如何准确区分与检测？

  

![属性检测三剑客：hasOwnProperty vs in vs instanceof](https://img.215454.xyz/file/blog/jia/1791814182332_02-hasown-in-instanceof.png)

  

| 检测方式 | 语法形式 | 检测范围 | 典型应用场景 |

| :--- | :--- | :--- | :--- |

| **`hasOwnProperty`** | `obj.hasOwnProperty(prop)` | **仅限自身挂载的属性**（排除原型） | `for...in` 循环中过滤原型自定义污染属性 |

| **`in` 运算符** | `prop in obj` | **自身 + 原型链全局穿透** | 只要对象能访问到该属性（哪怕在祖先上）即为 `true` |

| **`instanceof`** | `A instanceof B` | **检测 `B.prototype` 是否出现在 `A` 的原型链上** | 鉴别对象的原型血统关系 |

  

```js

function Car() {

this.color = 'red'; // 自身属性

}

Car.prototype.wheels = 4; // 原型属性

  

var c = new Car();

  

console.log(c.hasOwnProperty('color')); // true（自身亲生）

console.log(c.hasOwnProperty('wheels')); // false！！（来自原型，不是亲生的）

  

console.log('color' in c); // true

console.log('wheels' in c); // true（在原型链上即可！）

  

console.log(c instanceof Car); // true

console.log(c instanceof Object); // true

```

  

> **避坑铁律：** 在使用 `for...in` 遍历对象键名时，**必须配合 `obj.hasOwnProperty(key)` 进行守卫拦截**，否则原型上任何自定义的扩展都会被一并循环遍历出来，引发脏数据风险！

  

# 三、类型判断大阅兵：如何精准判断一个变量是不是数组？

  

![类型判断大阅兵：如何精准判断一个变量是不是数组](https://img.215454.xyz/file/blog/jia/1791814190060_03-is-array-comparison.png)

  

## 1. 为什么不能用 `typeof`？

  

```js

typeof []; // 'object'

typeof {}; // 'object'

typeof null; // 'object'

```

  

`typeof` 针对所有复杂对象和历史遗留的 `null` 一律返回 `'object'`，在区分数组和普通对象时彻底失效。

  

## 2. `instanceof` 和 `constructor` 的潜在瑕疵

  

- `[] instanceof Array === true` 且 `[].constructor === Array`；

- **跨环境缺陷**：在包含 `iframe` 的复杂 Web 应用中，子窗口和父窗口各自拥有完全独立的 `Array` 构造函数全局对象。从子 iframe 传过来的数组，用父窗口的 `instanceof Array` 校验时会判定为 `false`！

  

## 3. 终极工业解决方案：`Object.prototype.toString.call(a)`

  

```js

var a = [];

var isArr = Object.prototype.toString.call(a) === '[object Array]';

console.log(isArr); // true ！！绝对精准、跨环境免疫

```

  

- 原生借用顶层根原型的标准内部类型标签识别；

- 现代 ES5+ 官方提供的 `Array.isArray(a)` 原生 API，底层也正是基于这一原理实现的封装。

  

# 四、函数内省两件套：arguments.callee vs fn.caller

  

JavaScript 在运行时为函数体内部提供了两个强大的反射内省属性：

  

![函数内省两件套：arguments.callee vs fn.caller](https://img.215454.xyz/file/blog/jia/1791814204388_04-callee-and-caller.png)

  

## 1. `arguments.callee`：指向当前正在执行的函数本体

  

当我们在使用立即执行函数（IIFE）或匿名函数时，函数自身没有名字，如何完成**自递归调用**？

  

```js

// 用 IIFE 计算 1 ~ 100 累加，无需显式命名函数：

var sum = (function (n) {

if (n <= 1) {

return 1;

}

// callee 精准指向该匿名函数本身！

return n + arguments.callee(n - 1);

})(100);

  

console.log(sum); // 5050

```

  

## 2. `fn.caller`：追溯当前函数的调用者

  

```js

function test1() {

test2();

}

function test2() {

console.log(test2.caller); // 输出调用者函数：ƒ test1()

}

test1();

test2(); // 全局环境下直接调用，caller 输出 null

```

  

> **严格模式（'use strict'）警示：**

> 在现代 JavaScript 严格模式下，为了执行安全与引擎内联优化考量，访问 `arguments.callee` 与 `fn.caller` 会被直接抛出 `TypeError` 禁用限制！但它们依然是理解经典运行时调用栈的经典考点。

  

# 五、this 绑定法则四重奏与优先级判定

  

![this 绑定法则四重奏：默认、隐式、显式与 new](https://img.215454.xyz/file/blog/jia/1791814209366_05-this-binding-rules.png)

  

JavaScript 中函数体内的 `this` 是在**调用执行时动态绑定**的，归纳为以下 4 大场景：

  

1. **默认独立调用**：`test()` → 非严格模式下指向全局 `window`（严格模式下为 `undefined`）；

2. **隐式对象方法调用**：`obj.test()` → `this` 指向紧挨着点号前面的宿主对象 `obj`；

3. **显式强行绑定**：`test.call(obj)` / `test.apply(obj)` → 强行指定 `this` 为目标对象；

4. **`new` 构造函数绑定**：`new Test()` → `this` 永远指向刚刚开辟诞生的新实例。

  

> **终极裁决优先级：**

> **`new 构造绑定` > `显式 call/apply 绑定` > `隐式对象方法绑定` > `默认独立调用（window）`**

  

# 六、大厂高频面试手撕题：预编译与 this 的联合绞杀

  

![大厂高频面试手撕题：预编译与 this 的联合绞杀](https://img.215454.xyz/file/blog/jia/1791814227184_06-interview-traps-this-scope.png)

  

结合课件 `index.html` 末尾的高频真题，一步步推演：

  

## 考题 1：普通调用 vs new 实例化的 this.a

  

```js

var a = '1';

function test() {

var a = '2';

this.a = '3';

console.log(a);

}

test(); // 输出 2

new test(); // 输出 2

console.log(a); // 输出 3！

```

- `test()` 普通独立调用时，`this === window`，所以 `this.a = '3'` 直接改写了全局的 `window.a`；

- 但函数体内的 `console.log(a)` 读的是局部变量 `var a = '2'`，所以输出 2；

- `new test()` 时，`this` 指向新实例，修改的是实例属性；函数内部读局部变量依然输出 2；

- 最后打印全局 `console.log(a)`，由于在第一步被改成了 3，所以输出 `'3'`！

  

## 考题 2：变量提升遮蔽与 this.a 穿透

  

```js

var a = 5;

function test() {

a = 0;

console.log(a); // 0

console.log(this.a); // 5

var a; // 👈 变量声明提升到函数顶端！

console.log(a); // 0

}

test();

```

- 函数底部的 `var a` 预编译提升到了 `test->AO` 顶端，因此 `a = 0` 改的是局部 `AO.a`，**全局没有被污染**；

- 执行 `console.log(this.a)` 时，普通调用 `this === window`，直奔全局读取到原始的 `window.a === 5`！

  

## 考题 3：引用值比较 `{} == {}` 为什么是 false？

  

```js

console.log({} == {}); // false ！！

```

- 每个 `{}` 都在堆内存中申请了全新的独立物理地址空间；

- `==` 与 `===` 对比引用类型时，比对的是**内存指针地址**而不是内容；指针不同，判定必然为 `false`。

  

# 本课小结

  

- **链式调用精髓**：方法内部显式 `return this`，将自身对象交接给下一环继续调用；

- **属性检测分界**：`hasOwnProperty` 查自己；`in` 查自身加原型链整条链；`instanceof` 沿着 `__proto__` 查构造函数血统；

- **判断数组最佳实践**：优先使用 `Object.prototype.toString.call(a) === '[object Array]'` 或 `Array.isArray(a)`；

- **函数反射**：`arguments.callee` 获取当前执行函数本身（匿名递归）；`fn.caller` 追溯上层调用者；

- **`this` 优先级定律**：`new 绑定 > call/apply 显式 > 对象方法隐式 > 独立调用默认`；

- **复合分析法则**：任何复杂的变量与 `this` 混合题，**先画预编译 AO / GO 分析变量归属，再看函数调用方式裁决 `this` 指向**，两线并行必出正确答案。