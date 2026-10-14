---
title: "第十二课"
published: 2026-10-09
description: This is the first post of my new Astro blog.
tags: [JavaScript]
author: jia
---
# JS 第十二课：圣杯模式（Holy Grail Pattern）与企业级继承架构

  

> 本课对应练习文件：`index.html`（原型共享的双向污染陷阱、中介者推导、圣杯模式标准四行实现、IIFE 闭包缓存优化、程序员继承体系实战、模块化自执行函数）、`index1.html`（CSS 经典圣杯布局）。

  

# 一、原型共享模式的致命缺陷：牵一发而动全身

  

在学习面向对象继承时，很多初学者最容易想到的就是直接将父类原型赋给子类：

  

![原型共享模式的致命缺陷：牵一发而动全身](https://img.215454.xyz/file/blog/jia/1791644085319_01-prototype-sharing-trap.png)

  

```js

function Teacher() {}

Teacher.prototype.skill = 'JS';

  

function Student() {}

// 错误做法：直接共享同一个原型对象

Student.prototype = Teacher.prototype;

  

// 学生想扩展自己的专属属性：

Student.prototype.age = 18;

  

console.log(Teacher.prototype.age); // 18 ！！老师的原型被污染了！

```

  

## 为什么不能直接赋值？

  

- `Student.prototype = Teacher.prototype` 只是将同一个堆内存对象的地址拷贝了一份；

- 两个构造函数实际上在**共用同一个原型对象**；

- 子类新增任何属性，父类立刻受到波及；父类修改，子类也随之错乱 —— **两者完全丧失了继承层级应有的独立性！**

  

> **核心诉求：单向继承**

> 我们需要的是：**子类可以向上访问父类的属性和方法，但子类自身的扩展绝不能反向渗透污染父类！**

  

# 二、圣杯模式推导：引入空函数中介者 Buffer

  

为了打破父子原型之间的双向强绑定，JavaScript 先驱们发明了一种极其巧妙的模式 —— **引入一个干净的空函数作为中介者（Buffer）**。

  

![圣杯模式推导：引入空函数中介者 Buffer](https://img.215454.xyz/file/blog/jia/1791644090222_02-holy-grail-deduction.png)

  

## 推导三步法

  

1. **第一步：创建空中介者**

定义一个内部没有任何自身属性的纯净空函数 `function Buffer() {}`，将其原型指向父类原型：

`Buffer.prototype = Origin.prototype;`

2. **第二步：实例化中介者作为子类原型**

执行 `Target.prototype = new Buffer();`，产生一个空实例赋给子类原型。

3. **第三步：完美隔离与单向穿透**

- 当子类给原型加属性时（`Target.prototype.age = 18`），属性只挂在 `new Buffer()` 这个空实例自身上，**父类原型毫发无损**；

- 当子类查找父类属性时，顺着 `new Buffer().__proto__` 依然能**无缝向上穿透找到 `Origin.prototype`**！

  

> **为什么不直接用 `Target.prototype = new Origin()`？**

> 因为直接 `new Origin()` 会真实执行父类的构造函数代码，如果父类构造函数非常沉重（比如包含大量 DOM 操作、发请求或有很多无用的私有属性），会导致子类原型上充满无用的垃圾属性。而 `new Buffer()` 的构造函数体是全空的，性能极致纯净！

  

# 三、圣杯模式标准实现：四行核心代码深度解剖

  

![圣杯模式标准实现：四行核心代码深度解剖](https://img.215454.xyz/file/blog/jia/1791644098775_03-holy-grail-code-anatomy.png)

  

```js

// 经典圣杯模式 inherit 封装

function inherit(Target, Origin) {

function Buffer() {}

Buffer.prototype = Origin.prototype; // ① 中介者连接父类原型

Target.prototype = new Buffer(); // ② 子类原型指向中介者实例

Target.prototype.constructor = Target; // ③ 认祖归宗：修正构造函数反向指针

Target.prototype.super_class = Origin; // ④ 超类索引：保留对真正父类的引用

}

```

  

## 深度解密第 ③ 行与第 ④ 行

  

- **为什么必须修正 `constructor`？**

执行完第 ② 步后，子类原型的 `constructor` 沿着链条会一路找到 `Origin.prototype.constructor`（指向父类 `Origin`！）。如果不显式修正为 `Target`，那么子类实例执行 `s.constructor` 就会错误地以为自己是由父类构造函数创建的；

- **为什么记录 `super_class`？**

给子类预留一个回溯超类的通道。当子类未来重写了父类同名方法、又想在特定场景下调用父类原始逻辑时，可以通过 `this.constructor.super_class.prototype.xxx.call(this)` 显式调用（类似于 ES6 的 `super.xxx()`）。

  

# 四、圣杯模式进阶：闭包沙箱化（极致性能优化）

  

![圣杯模式进阶：闭包沙箱化（极致性能优化）](https://img.215454.xyz/file/blog/jia/1791644115362_04-holy-grail-closure-optimization.png)

  

普通版 `inherit` 在每次被调用时，都会在栈内存中临时重新声明一个 `function Buffer() {}`。在大型工程或频繁继承的场景下，可以利用 **IIFE 闭包** 进行终极性能优化：

  

```js

var inherit = (function () {

// 私有闭包缓存：只在内存中创建一次！

var Buffer = function () {};

  

return function (Target, Origin) {

Buffer.prototype = Origin.prototype;

Target.prototype = new Buffer();

Target.prototype.constructor = Target;

Target.prototype.super_class = Origin;

};

})();

```

  

## 闭包沙箱化的三大收益

  

1. **内存零抖动**：全系统复用同一个 `Buffer` 函数，无论调用几万次继承，都不会重复创建函数对象；

2. **彻底私有化**：`Buffer` 被牢牢锁在 IIFE 内部，外界完全看不见，杜绝全局命名空间污染；

3. **安全隔离**：每次继承只是临时改写 `Buffer.prototype` 的指针，`new Buffer()` 产出的实例立刻被各自的 `Target.prototype` 独立接管，绝不冲突。

  

# 五、企业级模块化实战：程序员岗位继承体系

  

结合课件 `index.html` 中的综合实战，看看现代企业级前端库是如何运用圣杯模式搭建模块继承体系的：

  

![企业级模块化实战：程序员继承体系（前端 vs 后端）](https://img.215454.xyz/file/blog/jia/1791644127709_05-programmer-inheritance-demo.png)

  

```js

var initProgrammer = (function () {

// 1. 基类：抽象公共契约与打卡行为

var Programmer = function () {};

Programmer.prototype = {

name: '程序员',

tool: '计算机',

work: '编写应用程序',

duration: '10个小时',

say: function () {

console.log('我是一名' + this.myName + this.name +

'，工作是用' + this.tool + this.work +

'，每天工作' + this.duration +

'，常用语言是' + this.lang.toString());

}

};

  

// 2. 子类岗位

function FrontEnd() {}

function BackEnd() {}

  

// 3. 圣杯模式安全单向继承

inherit(FrontEnd, Programmer);

inherit(BackEnd, Programmer);

  

// 4. 差异化扩展，互不打扰

FrontEnd.prototype.lang = ['HTML', 'CSS', 'JS'];

FrontEnd.prototype.myName = '前端';

  

BackEnd.prototype.lang = ['Node', 'Java', 'SQL'];

BackEnd.prototype.myName = '后端';

  

// 5. 模块导出

return {

FrontEnd: FrontEnd,

BackEnd: BackEnd

};

})();

  

// 业务调用侧：

var fe = new initProgrammer.FrontEnd();

var be = new initProgrammer.BackEnd();

fe.say(); // 我是一名前端程序员...常用语言是HTML,CSS,JS

be.say(); // 我是一名后端程序员...常用语言是Node,Java,SQL

```

  

# 六、圣杯模式的双重维度：JS 原型圣杯 vs CSS 经典圣杯布局

  

在学习本课时，练习文件里的 `index1.html` 还包含了一套 CSS 的圣杯布局。为什么它们都被计算机界冠以「圣杯（Holy Grail）」之名？

  

![圣杯模式的双重维度：JS 原型圣杯 vs CSS 经典圣杯布局](https://img.215454.xyz/file/blog/jia/1791644134506_06-js-vs-css-holy-grail.png)

  

| 领域 | 解决的百年难题 | 核心解决哲学 |

| :--- | :--- | :--- |

| **JS 原型继承圣杯** | 原型直接赋值导致父子双向污染，或直接 new 父类导致引入冗余垃圾属性 | 引入一个纯净空的中间代理函数 `Buffer()` 充当防火墙，达成**单向继承** |

| **CSS 布局圣杯（index1.html）** | 既要中间主要内容在 HTML 结构中最优先加载，又要左右两栏定宽自适应三栏等高 | 利用浮动 + 负外边距（`margin-left: -100%` / `-100px`）+ 相对定位回流 |

  

> **“圣杯”之名的真谛：**

> 在编程与计算机科学中，“Holy Grail”象征着**经过无数先驱艰苦摸索、终于找到的那个最完美、无懈可击、成为行业标准的终极解决方案**！

  

# 本课小结

  

- **直接共享原型是大坑**：`Child.prototype = Parent.prototype` 会导致子类修改反向污染父类；

- **圣杯模式的核心**：利用一个干净的空函数 `Buffer` 充当中间隔离带（`Target.prototype = new Buffer()`），实现完美的**单向继承**；

- **四行标准代码**：

1. `Buffer.prototype = Origin.prototype;`

2. `Target.prototype = new Buffer();`

3. `Target.prototype.constructor = Target;`（修正反向指针）

4. `Target.prototype.super_class = Origin;`（保留超类索引）

- **闭包沙箱化**：使用 IIFE 把 `Buffer` 缓存起来，实现单例中介者复用，内存零冗余；

- **ES6 映射**：现代的 `class Sub extends Super` 底层在降级编译后，采用的依然是圣杯模式的思想！