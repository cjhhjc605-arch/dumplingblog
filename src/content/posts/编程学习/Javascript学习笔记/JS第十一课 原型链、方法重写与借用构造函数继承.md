---
title: "第十一课"
published: 2026-10-09
description: This is the first post of my new Astro blog.
tags: [JavaScript]
author: jia
---
# JS 第十一课：原型链、方法重写与借用构造函数继承

  

> 本课对应练习文件：`index.html`（字符串字节数复习、三级原型链继承、原型链上的 this、Object.create、方法重写 Override、call/apply 与借用构造函数继承）。

  

# 一、原型链：沿着 `__proto__` 一层一层往上找

  

上一课我们搞清了单个构造函数的 `prototype`。这一课把它串起来 —— 当原型本身也是一个实例时，就形成了**原型链**。

  

![原型链：沿着 __proto__ 一层一层往上找](https://img.215454.xyz/file/blog/jia/1791509341289_01-prototype-chain-levels.png)

  

## 1. 三级继承的准备代码

  

```js

function Professor() {}

Professor.prototype.tSkill = 'JAVA'; // 教授会 Java

var professor = new Professor();

  

function Teacher() {

this.mSkill = 'JS/JQ';

this.students = 500;

}

Teacher.prototype = professor; // 老师继承教授

  

var teacher = new Teacher();

  

function Student() {

this.pSkill = 'HTML/CSS';

}

Student.prototype = teacher; // 学生继承老师

  

var student = new Student();

  

console.log(student.tSkill); // 'JAVA'（隔了两层也能读到！）

console.log(student.mSkill); // 'JS/JQ'（来自老师）

console.log(student.pSkill); // 'HTML/CSS'（自己的）

```

  

## 2. 原型链的定义与查找规则

  

> **沿着 `__proto__` 一层一层向上查找属性，形成的那条「链条」就叫原型链。**

  

查找顺序：

  

```text

实例自身 → 构造函数原型 → 上层原型 → …… → Object.prototype → null

```

  

- 找到就**立即返回**，不再继续往上找；

- 全部找完还没有才返回 `undefined`；

- 链条最后一定停在 `Object.prototype`，再往上是 `null`（链条的尽头）。

  

## 3. 为什么隔两层还能读到？

  

因为 `student.__proto__` 指向 `teacher`，而 `teacher.__proto__` 指向 `professor` —— 引擎会一路顺着这根链条往上爬，直到找到 `tSkill` 为止。

  

# 二、原型链上的原始值 vs 引用值

  

同样是通过原型链继承来的属性，**原始值改不动原型，引用值却能污染原型**。

  

![原型链上的原始值 vs 引用值：改法完全不同](https://img.215454.xyz/file/blog/jia/1791509348330_02-primitive-vs-ref-on-chain.png)

  

## 1. 原始值：读得到，但改的永远是「自己」

  

```js

// teacher 上有 students = 500（原始值）

student.students++; // 等价于 student.students = student.students + 1

// ① 读：自身没有 → 顺着原型链读到 500

// ② 加 1 得 501

// ③ 写：在【student 自己身上】新建 students = 501

  

console.log(student.students); // 501（自身属性）

console.log(teacher.students); // 500（原型毫发无伤！）

```

  

`++` 是「先读再写」，写的时候在实例自身新建了同名属性，**遮蔽**了原型上的值。

  

## 2. 引用值：读到的就是同一个对象，改了就全变

  

```js

// teacher 上有 success = { alibaba: '28', tencent: '30' }

student.success.baidu = '100';

  

// ① 读：自身没有 → 读到原型上的那个对象

// ② 在【那个对象内部】新增 baidu 属性

// （没有发生赋值遮蔽，改的是堆里同一个对象）

  

console.log(teacher.success.baidu); // '100'（原型也变了！）

console.log(student.success === teacher.success); // true

```

  

> **一句话总结：**

> 改「**对象内部的属性**」会污染所有实例（大家共用同一个引用）；

> 改「**变量本身**」只影响自己（在实例上新建了遮蔽属性）。

> **所以引用类型状态千万不能挂在原型上！**

  

# 三、原型链上的 this：永远指向调用者

  

同一个方法，`car.intro()` 和 `Car.prototype.intro()` 的输出为什么不一样？

  

![原型链上的 this：永远指向调用者](https://img.215454.xyz/file/blog/jia/1791509357344_03-this-on-prototype-chain.png)

  

```js

function Car() {

this.brand = 'Benz'; // 实例自身属性

}

Car.prototype = {

brand: 'Mazda', // 原型上的同名属性

intro: function () {

console.log('我是' + this.brand + '车');

}

};

  

var car = new Car();

car.intro(); // '我是 Benz 车'（this 是 car）

Car.prototype.intro(); // '我是 Mazda 车'（this 是原型）

```

  

## 原理解析

  

- `car.intro()` 里 `this` 指向点号左边的 `car`，查 `this.brand` 时**先从 `car` 自身找**，找到了 `'Benz'` 就直接返回，不需要再去原型上找；

- `Car.prototype.intro()` 的点号左边是原型对象，所以 `this` 指向原型，读到的是 `'Mazda'`。

  

> **口诀：** 谁调用，`this` 就指向谁；属性查找是**就近原则**。

> 方法的 `this` 与你**把方法写在哪里无关**，只与**「谁来调用」**有关！

  

# 四、Object.create：手工指定原型

  

![Object.create：手工指定原型 & 原型链的尽头](https://img.215454.xyz/file/blog/jia/1791509365605_04-object-create.png)

  

## 1. `Object.create(obj)` —— 手工指定祖先

  

```js

var obj1 = Object.create(null);

obj1.num = 1;

  

var obj2 = Object.create(obj1); // 以 obj1 为原型

console.log(obj2.num); // 1（继承成功！）

```

  

不经过构造函数，**在创建时由引擎内置指定原型**，永久生效。

  

## 2. `Object.create(null)` —— 无根的孤儿对象

  

```js

var obj = Object.create(null);

obj.num = 1;

console.log(obj); // { num: 1 }，没有 __proto__

  

obj.toString();

// ✗ TypeError: obj.toString is not a function

  

document.write(obj);

// ✗ Cannot convert object to primitive value

```

  

因为 `Object.create(null)` 造出的对象 `__proto__` 直接是 `null`，**原型链上一无所有**，自然找不到 `toString`。而 `document.write` 必须把对象转成字符串，转换失败就抛错。

  

想让它能被输出，只能自己补一个 `toString`：

  

```js

obj.toString = function () { return '你好'; };

```

  

## 3. 自定义 `__proto__` 是无效的

  

```js

var obj3 = Object.create(null);

obj3.__proto__ = { count: 2 };

console.log(obj3.count); // undefined（改不动！）

```

  

> **结论：不是所有对象都继承于 `Object.prototype`！**

> `Object.create(null)` 常被用作「**纯字典容器**」（缓存表、Map 的替代品），因为它没有任何继承属性，能**避免原型污染攻击**。

  

# 五、方法重写（Override）：两个同名 `toString` 的区别

  

![方法重写（Override）：两个同名 toString 的区别](https://img.215454.xyz/file/blog/jia/1791509366084_05-method-override.png)

  

```js

// Object 版本：返回统一格式 [object Xxx]

Object.prototype.toString.call(1); // '[object Number]'

Object.prototype.toString.call('a'); // '[object String]'

Object.prototype.toString.call([1, 2]); // '[object Array]'

Object.prototype.toString.call({}); // '[object Object]'

  

// Number 版本：把数字转成字符串

Number.prototype.toString.call(1); // '1'（就是它自己）

(255).toString(); // '255'

(255).toString(16); // 'ff'（转十六进制！）

(255).toString(2); // '11111111'

```

  

## 1. 两个 `toString` 是完全不同的两个函数

  

- **`Object.prototype.toString`**：返回「你是哪种类型」的标签，是业内公认的**精确类型检测方案**（`typeof []` 只能得到 `'object'`，而它能准确得到 `'[object Array]'`）；

- **`Number.prototype.toString`**：把数字转成字符串，返回数字本身。

  

## 2. 为什么普通调用命中的是 Number 版本？

  

```js

(255).toString(); // '255'，而不是 '[object Number]'

```

  

因为 `Number.prototype` 在原型链上**更靠前**（就近原则），所以命中的是 Number 版本，把 Object 版本「遮蔽」掉了。

  

> **方法重写（Override）的本质：** 子级原型上定义了与祖先同名的属性或方法，查找时会**就近命中子级的版本**，把祖先的版本遮蔽掉 —— 这就是原型链上的方法重写！

  

# 六、call / apply 与借用构造函数继承

  

![借用构造函数继承：用 call / apply 偷来别人的初始化代码](https://img.215454.xyz/file/blog/jia/1791509374010_06-call-apply-inherit.png)

  

## 1. `call` / `apply` 的唯一作用

  

两者都是**改变函数执行时的 this 指向**并立即执行：

  

| 方法 | 语法 | 参数传递方式 |

| --- | --- | --- |

| `call` | `fn.call(thisArg, 参1, 参2, ...)` | 参数逐个传入 |

| `apply` | `fn.apply(thisArg, [参1, 参2, ...])` | 参数以数组传入 |

  

## 2. 用法一：给普通对象「灌」属性

  

```js

function Car(brand, color) {

this.brand = brand;

this.color = color;

this.run = function () {

console.log('running');

};

}

  

var newCar = { displacement: '3.0' };

  

Car.call(newCar, 'Benz', 'red');

// 等价写法：Car.apply(newCar, ['Benz', 'red']);

  

console.log(newCar);

// { displacement: '3.0', brand: 'Benz', color: 'red', run: ƒ }

```

  

`Car.call(newCar, ...)` 把 `Car` 体内的 `this` 强行指向了 `newCar`，于是 `this.brand = brand` 就成了 `newCar.brand = 'Benz'` —— 属性被「灌」进了 newCar！

  

## 3. 用法二：构造函数之间的继承（借用构造函数）

  

```js

function Compute() {

this.plus = function (a, b) {

console.log(a + b);

};

this.minus = function (a, b) {

console.log(a - b);

};

}

  

function FullCompute() {

Compute.apply(this); // 👈 借用父构造函数的初始化代码！

this.mul = function (a, b) {

console.log(a * b);

};

this.div = function (a, b) {

console.log(a / b);

};

}

  

var compute = new FullCompute();

compute.plus(1, 2); // 3

compute.minus(1, 2); // -1

compute.mul(1, 2); // 2

compute.div(1, 2); // 0.5

```

  

`Compute.apply(this)` 里的 `this` 是 `new FullCompute()` 造出的新实例，所以 `plus` / `minus` 被挂到了子类实例上 —— **子类成功继承父类的属性！**

  

## 4. 优缺点

  

- **优点**：可以继承属性，且**不会污染父类原型**（属性都是复制到子类实例上的）；

- **缺点**：**无法继承父类原型上的方法**（因为原型链并没有连起来）。

  

> **业界最终方案是「组合继承」：**

> **用 `call` / `apply` 借用构造函数继承属性 + 用原型链继承方法**，两者配合才是完整的继承方案。

  

# 本课小结

  

- **原型链**：沿 `__proto__` 逐层向上查找属性形成的链条，顺序为「自身 → 构造函数原型 → 上层原型 → … → `Object.prototype` → `null`」；

- **原始值 vs 引用值在原型链上的差异**：改「对象内部属性」会污染所有实例，改「变量本身」只在自身新建遮蔽属性 —— **引用类型状态绝不能挂原型**；

- **`this` 与书写位置无关**，只由「谁调用」决定（就近原则）；

- **`Object.create`**：`Object.create(obj)` 手工指定原型；`Object.create(null)` 创建无原型链的「纯字典」对象，且**不是所有对象都继承自 `Object.prototype`**；

- **方法重写 Override**：子级原型上同名方法会遮蔽祖先版本；`Object.prototype.toString` 是精确类型检测的终极方案；

- **借用构造函数继承**：`Parent.call(this)` 把父构造函数体内的初始化逻辑「借」过来，实现属性继承；配合原型链即成「组合继承」。