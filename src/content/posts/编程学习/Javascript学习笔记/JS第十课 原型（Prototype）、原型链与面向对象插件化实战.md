---
title: "第十课"
published: 2026-10-09
description: This is the first post of my new Astro blog.
tags: [JavaScript]
author: jia
---
# JS 第十课：原型（Prototype）、原型链与面向对象插件化实战

  

> 本课对应练习文件：`原型.html`（原型的本质、增删改查规则、原型铁三角关系、重写原型的时序深坑、原型链引用类型陷阱、IIFE 计算器插件化封装）、`test.html`（多参累加器实现、消费者挑车构造函数联动）。

  

# 一、原型的本质：所有实例对象的公共祖先

  

在课时8 中，我们发现如果把方法直接写在构造函数内部，每 `new` 一个实例就会在堆内存中重复生成一份函数对象，造成巨大的内存浪费。JavaScript 通过**原型（`prototype`）**优雅地解决了这个问题。

  

![原型的本质：所有实例对象的公共祖先](https://img.215454.xyz/file/blog/jia/1791442418669_01-prototype-ancestor.png)

  

## 1. 什么是 prototype？

  

在 JavaScript 中，**每一个函数天生自带一个 `prototype` 属性**。打印出来看，它本质上就是一个普通的空对象 `{}`。

  

```js

function Handphone(color, brand) {

this.color = color; // 个性属性：放在构造函数内部

this.brand = brand;

}

  

// 共有属性与方法：挂载到原型上

Handphone.prototype.rom = '64G';

Handphone.prototype.ram = '6G';

Handphone.prototype.screen = '16:9';

Handphone.prototype.call = function () {

console.log('I am calling somebody');

};

  

var hp1 = new Handphone('red', '小米');

var hp2 = new Handphone('black', '华为');

  

console.log(hp1.rom); // '64G'（无条件继承自公共祖先原型）

console.log(hp2.ram); // '6G'

hp2.call(); // 两个实例共同调用同一个 call 函数！

```

  

## 2. 原型的核心定义与工程职责

  

- **公共祖先**：`prototype` 是由该构造函数创建出来的所有实例对象的**公共祖先**，所有被该构造函数构造出来的对象都可以无条件继承原型上的属性和方法；

- **分工明确**：

- 需要通过传参动态配置的**个性化属性**（如颜色、品牌）→ 放在构造函数中通过 `this.xxx` 赋值；

- 固定不变的**公共常量与行为方法**（如屏幕比例、通话方法）→ 挂载到构造函数的 `prototype` 上。

  

# 二、原型属性读写不对称性：查得到，但改不掉！

  

很多初学者容易以为：既然实例继承了原型，那通过实例修改属性就能改变原型。这是完全错误的！

  

![原型属性读写不对称性：查得到，但改不掉](https://img.215454.xyz/file/blog/jia/1791442428478_02-prototype-read-write-rule.png)

  

## 1. 属性读取：顺藤摸瓜（读穿）

  

当访问 `instance.prop` 时，JS 引擎沿着原型链向上查找：

1. 先看实例自身有没有该属性（`hasOwnProperty`）；

2. 自身没有，就顺着隐藏指针 `__proto__` 查构造函数的原型对象；

3. 原型上还没有，继续查原型的原型，直到顶层 `null`；全都没有才返回 `undefined`。

  

```js

function Test() {}

Test.prototype.name = 'prototype';

  

var test = new Test();

console.log(test.name); // 'prototype'（自身没有，顺着原型读到了！）

```

  

## 2. 属性修改/删除：只能操作自身，伤不到原型毫发！

  

```js

test.name = 1; // 属性遮蔽（Shadowing）

console.log(Test.prototype.name); // 'prototype'（原型毫发无伤！）

console.log(test.name); // 1（读的是实例自身的属性）

  

delete test.name; // 删除实例自身的属性

console.log(test.name); // 'prototype'（原型的属性再次显露！）

```

  

> **核心铁律：**

> **通过实例绝对无法修改或删除原型上的属性！**

> `test.name = 1` 只是在实例自身上强行新增了一个同名属性并遮蔽（Shadowing）了原型同名属性；`delete test.name` 删掉的也只是自身的遮蔽属性。

> **要想修改原型，必须显式通过构造函数操作：`Test.prototype.name = '新名字'`！**

  

# 三、铁三角关系：构造函数、prototype 原型与实例 `__proto__`

  

这是理解 JavaScript 原型链寻路机制最关键的一张地图。

  

![铁三角关系：构造函数、prototype 原型与实例 __proto__](https://img.215454.xyz/file/blog/jia/1791442431013_03-prototype-triangle.png)

  

## 1. 铁三角三者的指针羁绊

  

```text

构造函数 (Car)

│ ▲

.prototype │ │ .constructor

▼ │

原型对象 (Car.prototype)

▲

│ .__proto__

│

实例对象 (car = new Car())

```

  

1. **构造函数 → 原型**：`Car.prototype`；

2. **原型 → 构造函数**：`Car.prototype.constructor === Car`（原型天生自带反向指回构造函数的指针）；

3. **实例 → 原型**：`car.__proto__ === Car.prototype`（由 `new` 运算在后台隐式搭建的连接管道）。

  

## 2. `constructor` 属性是谁的？

  

当你写 `car.constructor` 时，由于实例 `car` 自身根本没有 `constructor` 属性，它沿着 `__proto__` 顺流而上借读了原型的 `constructor`，最终指向 `Car`。

  

```js

var car = new Car();

console.log(car.constructor); // ƒ Car()

```

  

# 四、原型重写的时序深坑：new 在前 vs new 在后

  

开发中为了省事，经常会直接使用对象字面量一次性给 `prototype` 批量赋值。但这里隐藏着一个致命的时间序列陷阱！

  

![原型重写的时序深坑：new 在前 vs new 在后](https://img.215454.xyz/file/blog/jia/1791442442801_04-prototype-override-timing.png)

  

## 1. 先 new 后重写原型（实例依然指着旧原型！）

  

```js

function Person() {}

Person.prototype.name = 'A';

  

var p = new Person(); // 实例化！p.__proto__ 已经固化指向了原型 A

  

// 后来人直接把构造函数的 prototype 引用切到了全新的对象 B

Person.prototype = {

name: 'B'

};

  

console.log(p.name); // 输出依然是：'A' ！！并没有变成 'B'！

```

  

### 底层指针快照原理

- `new` 运算发生的那一刻，实例内部的 `p.__proto__` 就已经拿到了当时旧原型对象的实际内存地址并固定下来；

- 之后你修改 `Person.prototype = { name: 'B' }`，只是让构造函数的变量指针指向了一个新开辟的堆对象，**旧原型对象在内存中并没有被销毁**（因为还被 `p.__proto__` 强引用着）；

- 所以老实例 `p` 依然死心塌地指着老祖先 `A`！

  

## 2. 先重写原型后 new（新实例才能认祖归宗）

  

```js

function Person() {}

Person.prototype.name = 'A';

  

// 先把新原型切好！

Person.prototype = {

name: 'B'

};

  

var p2 = new Person(); // p2 诞生时连接到了新原型 B

console.log(p2.name); // 'B'

```

  

> **规约：** 如需重写原型，**必须在任何实例实例化之前完成！**

  

# 五、原型链继承的引用类型陷阱：共享与篡改危机

  

![原型链继承的引用类型陷阱：共享与篡改危机](https://img.215454.xyz/file/blog/jia/1791442450441_05-prototype-ref-trap.png)

  

## 1. 踩坑代码

  

```js

function Student() {}

// 错误写法：把可变引用类型（数组/对象）挂在原型上

Student.prototype.hobbies = ['篮球'];

  

var s1 = new Student();

var s2 = new Student();

  

s1.hobbies.push('足球'); // 顺着原型链找到了同一个数组并修改！

console.log(s2.hobbies); // ['篮球', '足球'] ！！s2 的兴趣爱好被篡改！

```

  

`s1.hobbies.push()` 并不是在给 `s1` 自身赋值，而是读出原型上的数组并调用方法修改了该堆内存。所有兄弟实例全部受到波及！

  

## 2. 规避范式：状态私有，行为共享

  

```js

function Student() {

this.hobbies = ['篮球']; // 状态数据独立放入构造函数

}

Student.prototype.say = function () {}; // 公共方法才挂原型

```

  

# 六、插件化架构综合实战：IIFE 封装独立计算器插件

  

结合课件 `原型.html` 中的末尾实战，展示工业级前端库封装的标准形态：

  

![插件化架构综合实战：IIFE 封装独立计算器插件](https://img.215454.xyz/file/blog/jia/1791442458051_06-plugin-compute-prototype.png)

  

```js

;(function () {

// 1. 构造函数定义私有状态

function Compute(opt) {

this.x = opt.firstNum;

this.y = opt.secondNum;

}

  

// 2. 原型对象集中式挂载四则运算方法

Compute.prototype = {

constructor: Compute, // 必须手动补齐 constructor 反向指针！

plus: function () {

return this.x + this.y;

},

minus: function () {

return this.x - this.y;

},

mul: function () {

return this.x * this.y;

},

div: function () {

return this.x / this.y;

}

};

  

// 3. 挂载到 window 暴露唯一入口

window.Compute = Compute;

})();

  

// 业务调用侧：

var compute = new Compute({

firstNum: 10,

secondNum: 2

});

  

console.log(compute.plus()); // 12

console.log(compute.minus()); // 8

console.log(compute.mul()); // 20

console.log(compute.div()); // 5

```

  

## 架构要点

  

1. **IIFE 插件沙箱**：防止内部变量污染外部全局；

2. **字面量重写原型时手动补齐 `constructor: Compute`**：维护铁三角反向指针完整性；

3. **数据层与逻辑层彻底分离**：`firstNum` 实例独立独享，四则运算逻辑全量共享。

  

# 本课小结

  

- **原型的本质**：`prototype` 是构造函数的公共祖先对象，所有实例通过隐式指针 `__proto__` 继承其上的属性和方法；

- **读写不对称性**：读取会沿原型链往上找（读穿）；写操作（`.xxx = yyy`）只会给实例自身增加遮蔽属性，**无法改动原型**；

- **铁三角关系**：`F.prototype`、`F.prototype.constructor === F`、`instance.__proto__ === F.prototype`；

- **原型重写时序**：重写 `prototype` 必须发生在 `new` 之前，否则提前 `new` 出的实例依然死死锁定旧原型快照；

- **状态与行为分离**：引用类型状态（数组、对象）放构造函数，函数方法放原型；

- **插件封装标准范式**：`IIFE 沙箱` + `构造函数传参` + `原型集中定义方法` + `挂载 window 暴露`。