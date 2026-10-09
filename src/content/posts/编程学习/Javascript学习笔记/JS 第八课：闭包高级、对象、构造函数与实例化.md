---
title: "第八课"
published: 2026-10-09
description: This is the first post of my new Astro blog.
tags: [JavaScript]
author: jia
---
# JS 第八课：闭包高级、对象、构造函数与实例化

  

> 本课对应练习文件：`index.html`（对象的增删改查、方法中的 this、构造函数与实例化、Options 模式、Calc 不定参计算器）、`monichajian.js`（模拟插件库构造函数）、`monihoutai.html`（模拟业务调用后台）。

  

# 一、对象的本质与增删改查四部曲

  

在 JavaScript 中，**对象是一组无序属性和方法的集合**（引用值）。对象本身具有完全动态的特性，可以在运行时随时挂载、修改或剔除属性。

  

![对象字面量：增、删、改、查四部曲](https://img.215454.xyz/file/blog/jia/1791428090330_01-object-crud.png)

  

## 1. 对象的四种基础操作

  

```js

var teacher = {

name: '张三',

age: 32,

sex: 'male',

height: 176,

weight: 130,

teach: function () {

console.log("I am teaching JavaScript");

}

};

```

  

| 操作 | 语法写法 | 行为说明 |

| :--- | :--- | :--- |

| **查（Read）** | `teacher.name`<br>`teacher['age']` | 读取属性值；访问不存在的属性得到 `undefined` 而不会报错 |

| **增（Create）** | `teacher.address = '北京'`<br>`teacher.drink = function(){}` | 动态挂载新属性或方法到对象实例上 |

| **改（Update）** | `teacher.height = 190`<br>`teacher.teach = function(){}` | 对已存在键重新赋值，直接覆盖旧值 |

| **删（Delete）** | `delete teacher.address`<br>`delete teacher.teach` | 使用 `delete` 操作符彻底抹除该键值对 |

  

## 2. 点语法 `.` 与中括号语法 `[]` 的差异

  

- **点语法 `obj.prop`**：要求 `prop` 必须符合合法的标识符命名规范（不能有空格、不能以数字开头、不能为变量）；

- **中括号语法 `obj[expr]`**：中括号内接收一个字符串表达式。如果属性名保存在变量里、或者包含连字符等特殊字符，**必须使用中括号语法**：

  

```js

var attr = 'weight';

console.log(teacher[attr]); // 130（动态读取）

```

  

# 二、方法中的 this：动态绑定与对象解耦

  

当对象内部包含方法时，方法如果需要访问或修改对象自身的属性，应该怎么写？

  

![方法中的 this：指代当前调用该方法的对象](https://img.215454.xyz/file/blog/jia/1791428099340_02-this-in-methods.png)

  

## 1. 硬编码对象名的缺陷

  

```js

var teacher = {

weight: 130,

smoke: function () {

teacher.weight--; // ✗ 硬编码了变量名 teacher

}

};

```

  

这种写法的致命问题在于**强耦合**：

- 如果外部将变量重命名为 `var master = teacher; teacher = null;`，执行 `master.smoke()` 将直接抛出 `TypeError` 崩溃；

- 如果把该方法借给别人复用 `student.smoke = teacher.smoke;`，它修改的依然是 `teacher` 的体重，而不是 `student` 的体重！

  

## 2. 使用 this 动态绑定

  

```js

var teacher = {

weight: 130,

smoke: function () {

this.weight--; // ✓ 动态指代调用者

console.log(this.weight);

},

eat: function () {

this.weight++;

console.log(this.weight);

}

};

  

teacher.smoke(); // 129（此时 this 严格指向 teacher）

```

  

> **核心原则：** 当函数作为对象的方法被调用（`obj.fn()`）时，函数体内的 `this` **永远指向紧挨着点号左侧的那个对象**。

  

## 3. 课堂考勤案例（方法内操作复杂数据）

  

```js

var attendance = {

students: [],

total: 6,

join: function (name) {

this.students.push(name);

if (this.students.length === this.total) {

console.log(name + '到课，学生已经到齐');

} else {

console.log(name + '到课，学生未到齐');

}

},

leave: function (name) {

var idx = this.students.indexOf(name);

if (idx !== -1) {

this.students.splice(idx, 1);

}

console.log(name + '早退');

},

classOver: function () {

this.students = [];

console.log('已经下课');

}

};

```

  

方法通过 `this.students`、`this.total` 统一操作对象内部的属性状态，语义高度自洽。

  

# 三、构造函数与 new 实例化机制

  

如果我们想要创建 10 个不同的教师、100 个不同的学生，使用对象字面量手动复制粘贴显然不可行。为此，JavaScript 提供了**构造函数（Constructor）**来作为生产对象的模具。

  

![new 操作符的底层执行过程（四步法）](https://img.215454.xyz/file/blog/jia/1791428105462_03-new-operator-steps.png)

  

## 1. 系统自带与自定义构造函数

  

- **系统自带**：`var obj = new Object();`（等价于对象字面量 `{}`）；

- **自定义构造函数**：按照行业约定，**构造函数必须采用大驼峰命名法（PascalCase）**，首字母大写：

  

```js

function Teacher() {

this.name = '张三';

this.sex = '男';

this.weight = 130;

this.smoke = function () {

this.weight--;

console.log(this.weight);

};

this.eat = function () {

this.weight++;

console.log(this.weight);

};

}

  

var t1 = new Teacher(); // 实例化出一个对象

var t2 = new Teacher();

```

  

## 2. new 运算符在幕后做的 4 件事

  

当你写下 `new Teacher()` 的瞬间，JS 引擎底层隐式执行了以下四步标准流程：

  

1. **创建空对象**：在堆内存中隐式开辟一个全新的空对象：`var this = {};`；

2. **关联原型链**：将新对象的原型指针连接到构造函数的原型对象：`this.__proto__ = Teacher.prototype;`；

3. **绑定 this 并执行**：执行构造函数代码，将函数体内的所有 `this.xxx = yyy` 属性赋给这个新对象；

4. **隐式返回**：构造函数末尾隐式返回刚刚组装完毕的实例：`return this;`。

  

# 四、构造函数传参：Options 配置对象模式

  

![构造函数传参：多参数列表 vs 配置对象（Options）](https://img.215454.xyz/file/blog/jia/1791428111681_04-constructor-options-pattern.png)

  

## 1. 传统多参数列表的弊端

  

```js

function Teacher(name, sex, weight, course, age, city) {

this.name = name;

this.sex = sex;

// ...

}

var t = new Teacher('张三', '男', 145, 'JS', 30, '北京');

```

  

- 必须严格死记每一个参数的位置顺序；

- 若想让 `sex` 或 `weight` 使用默认值，调用方被迫传入 `undefined` 占位：`new Teacher('张三', undefined, undefined, 'JS')`，极其反人类。

  

## 2. Options 配置对象模式（现代前端行业规范）

  

```js

function Teacher(opt) {

this.name = opt.name;

this.sex = opt.sex || '男'; // 极其便于指定默认兜底值

this.weight = opt.weight;

this.course = opt.course;

this.smoke = function () {

this.weight--;

console.log(this.weight);

};

this.eat = function () {

this.weight++;

console.log(this.weight);

};

}

```

  

在 `monichajian.js` 与 `monihoutai.html` 中，正是模拟了插件化组件封装的标准形态：

  

```js

// 业务调用侧：随意调换键值顺序，只传关心的字段

var t1 = new Teacher({

name: '张三',

weight: 145,

course: 'Javascript'

});

```

  

# 五、多实例独立性与面向对象性能思考

  

![构造函数实战：多实例独立性与方法共享雏形](https://img.215454.xyz/file/blog/jia/1791428116665_05-constructor-multi-instances.png)

  

## 1. 实例之间数据的绝对隔离

  

```js

var t1 = new Teacher({ name: '张三', weight: 145 });

var t2 = new Teacher({ name: '李四', weight: 90 });

  

t1.smoke(); // 144

t1.smoke(); // 143

t2.smoke(); // 89（t2 的体重丝毫不受 t1 影响）

  

t1.name = '王五';

console.log(t2.name); // '李四'

```

  

- 每次 `new` 都会在内存堆中申请一个崭新的独立对象地址；

- `t1` 独占 `0x101` 堆内存，`t2` 独占 `0x202` 堆内存，两者的属性变量绝不互相干扰。

  

## 2. 当前写法的性能痛点

  

如果通过 `new Teacher()` 实例化出 1000 个老师对象，内存中就会生成 **1000 个一模一样的 `smoke` 函数与 1000 个一模一样的 `eat` 函数**。

  

> **下节预告：** 属性应该各自独立（个性），但方法应该共享一套逻辑（共性）—— 解决这个性能瓶颈的机制，正是下一课的核心灵魂：**原型与原型链（Prototype & Prototype Chain）**！

  

# 六、综合实战：不定参计算器与 new + apply 黑科技

  

综合运用闭包、伪数组 `arguments`、实例化与改变 `this` 指向的经典面试大题。

  

![综合实战：不定参计算器与 new + apply 技巧](https://img.215454.xyz/file/blog/jia/1791428130497_06-calc-constructor-apply.png)

  

## 1. 题目需求

设计一个计算器构造函数 `Calc`，支持接收任意数量的数字参数，并提供累加 `sum()` 与累乘 `multiply()` 的方法。

  

## 2. 代码实现

  

```js

function Calc() {

// 闭包私有变量保存参数集合

var nums = [];

for (var i = 0; i < arguments.length; i++) {

nums[nums.length] = arguments[i];

}

  

// 累加方法

this.sum = function () {

var total = 0;

for (var i = 0; i < nums.length; i++) {

total += nums[i];

}

return total;

};

  

// 累乘方法

this.multiply = function () {

var product = 1;

for (var i = 0; i < nums.length; i++) {

product *= nums[i];

}

return product;

};

}

```

  

## 3. 如何把动态长度的输入参数塞给构造函数？

  

```js

// 模拟获取用户输入：'1, 2, 3, 4'

var input = "1, 2, 3, 4";

var parts = input.split(/[\s,]+/); // 正则支持逗号或多空格分割

var nums = [];

for (var i = 0; i < parts.length; i++) {

var n = parseFloat(parts[i]);

if (!isNaN(n)) {

nums[nums.length] = n;

}

}

  

// 核心技巧：ES3 时代的 new + apply

var c = new Calc();

Calc.apply(c, nums); // 借用 apply 把数组平铺作为 arguments 注入到实例 c 中！

  

console.log("相加结果：", c.sum()); // 10

console.log("相乘结果：", c.multiply()); // 24

```

  

# 本课小结

  

- **对象是键值对的集合**：通过点语法或中括号语法进行增删改查；

- **方法中的 `this`**：动态指向调用该方法的宿主对象（点号左边的对象），实现方法与数据的优雅解耦；

- **构造函数与 `new`**：

1. 隐式创建空对象 `this = {}`；

2. 隐式连接原型链 `this.__proto__ = F.prototype`；

3. 执行代码挂载属性 `this.prop = val`；

4. 隐式返回 `return this`；

- **配置对象（Options Pattern）**：工业界构造函数传参的标准范式，解耦参数顺序，极易扩展；

- **实例独立性**：每个 `new` 出来的对象在堆内存中独占空间，属性互不影响；

- **不定参处理**：结合闭包封装数据沙箱，借用 `apply(instance, arr)` 解决未知长度参数灌入实例的构造过程。