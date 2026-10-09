---
title: "JavaScript第七课"
published: 2026-10-09
description: This is the first post of my new Astro blog.
tags: [JavaScript]
author: jia
---
# JS 第七课：立即执行函数（IIFE）、闭包实战与逗号运算符

  

> 本课对应练习文件：`index.html`（立即执行函数、5 种表达式写法、经典闭包循环问题、累加器、班级学生管理器、逗号运算符）、`index1.html`（DOM `li` 列表点击事件闭包踩坑与解决）。

  

# 一、函数声明 vs 函数表达式：为什么 `()()` 会报错？

  

想要真正搞懂**立即执行函数（IIFE）**，必须先搞清楚 JavaScript 里关于「执行符号」`()` 的底层规则。

  

![函数声明 vs 函数表达式：为什么 ()() 会报错](https://img.215454.xyz/file/blog/jia/1791426541851_01-decl-vs-expr.png)

  

## 1. 只有表达式才能被「执行符号」`()` 执行

  

在 JavaScript 中：

  

- **函数表达式**：放在赋值号右边、或者括号里的函数，本质是一个**值**，可以直接跟 `()` 执行：

  

```js

var a = function () {

console.log(1);

};

a(); // ✓ 1

  

// 直接把匿名函数包在括号里，括号把它强行变成了表达式

(function () { console.log(1); })(); // ✓ 1（最常见的 IIFE 写法）

(function () { console.log(1); }()); // ✓ 1（W3C 推荐写法，把执行括号包在里面）

```

  

- **函数声明**：以 `function` 开头的语句是一条**声明语句**，语句后面绝对不能直接跟 `()`：

  

```js

function test() {

console.log(1);

}(); // ✗ 抛出异常：Uncaught SyntaxError: Unexpected token ')'

```

  

## 2. 引擎为什么报错？

  

JS 引擎在解析上面这段代码时，看到了一个完整的函数声明 `function test() { ... }`；解析完之后，突然又看到了一个孤立的 `()`。

  

在 JS 语法里，`()` 作为分组操作符时**里面不能为空**（比如 `(1 + 2)` 是合法的，但单独写 `()` 是非法的）。引擎不知道你拿这个空的括号想干什么，于是直接报语法错误！

  

> **但如果括号里传了参数呢？**

> ```js

> function test(e) {

> console.log(e);

> }(6); // 不报错！但也没有执行 test(6)！

> ```

> 引擎会把它当成两句毫无关系的代码来解析：

> 1. 第一句：一个正常的函数声明 `function test(e) {}`；

> 2. 第二句：一个独立的表达式 `(6)`，值就是 6。

> 函数 `test` 根本没有被调用！

  

# 二、把函数声明变成表达式的 5 种写法

  

只要能打破 `function` 开头的「语句结构」，让引擎把它当成一个**表达式**，后面跟 `()` 就能立刻执行。常见的有以下 5 种方式：

  

![把函数声明变成表达式的 5 种写法](https://img.215454.xyz/file/blog/jia/1791426557797_02-five-ways-to-expr.png)

  

| 方式 | 代码示例 | 说明 |

| --- | --- | --- |

| **括号包裹** | `(function(){ console.log(1); })();`<br>`(function(){ console.log(1); }());` | **最推荐、最标准**的写法，两大流派都合法 |

| **一元正号 `+`** | `+function(){ console.log(1); }();` | 将函数当成操作数参与运算，强制转换为表达式 |

| **一元负号 `-`** | `-function(){ console.log(1); }();` | 运算结果为 `NaN`，但不影响函数体执行 |

| **逻辑非 `!`** | `!function(){ console.log(1); }();` | 压缩工具（如 UglifyJS）常用来省 1 字节 |

| **逻辑或** | `0 || function(){ console.log(1); }();` | 只要短路逻辑走到右边，就会变成表达式执行 |

  

## 立即执行函数（IIFE）的特点

  

1. **执行完立刻销毁**：执行结束后它的执行期上下文（AO）立刻被垃圾回收（除非产生了闭包）；

2. **函数名被忽略**：`(function test(){})()` 执行完后，外部直接访问 `test` 会报错 `test is not defined`。所以 **IIFE 普遍使用匿名函数**，起名没有意义；

3. **独立的作用域**：常用于模块封装，防止里面的变量污染全局命名空间。

  

# 三、经典闭包循环问题：为什么打印 10 个 10？

  

这是 JavaScript 面试中出现频率最高的一道经典题。

  

![经典闭包循环问题：打印 10 个 10 还是 0 到 9](https://img.215454.xyz/file/blog/jia/1791426557256_03-closure-loop-fix.png)

  

## 1. 踩坑代码

  

```js

function test() {

var arr = [];

for (var i = 0; i < 10; i++) {

arr[i] = function () {

document.write(i + ' ');

};

}

return arr;

}

  

var myArr = test();

for (var j = 0; j < 10; j++) {

myArr[j](); // 输出：10 10 10 10 10 10 10 10 10 10

}

```

  

### 为什么全是 10？

  

1. **定义 ≠ 执行**：在 `for (var i = 0; i < 10; i++)` 的循环过程中，只是把 10 个小函数**声明**出来塞进数组，函数体内的 `document.write(i)` **一次都没执行**；

2. **循环跑完，i 已经定格**：循环结束的条件是 `i < 10` 为假，所以循环结束时 `i = 10`；此时 `test` 函数的 AO 里：`{ arr: [...], i: 10 }`；

3. **调用时往外找**：等到循环结束后执行 `myArr[j]()` 时，这 10 个小函数各自创建自己的 AO，但自己里面都没有 `i`；

4. **共享同一个父级 AO**：它们沿着自己的作用域链 `[[scope]]` 往上找，全都找到了同一个 `test` 的 AO，读到的都是同一个 `i`（值已经是 10）。

  

## 2. 解决方案：用立即执行函数（IIFE）强行留住每一轮的 i

  

```js

function test() {

var arr = [];

for (var i = 0; i < 10; i++) {

(function (j) {

arr[j] = function () {

document.write(j + ' ');

};

})(i); // ← 每一轮把当时的 i 当成实参传给形参 j

}

return arr;

}

  

var myArr = test();

for (var j = 0; j < 10; j++) {

myArr[j](); // 输出：0 1 2 3 4 5 6 7 8 9

}

```

  

### 为什么加了 IIFE 就能正常打印 0 ~ 9？

  

- 每一轮循环都会**立刻执行一次 IIFE**，因此产生了 10 个互不相同的独立执行期上下文（AO_0 到 AO_9）；

- 每一轮的实参 `i` 被固定复制给了每一层 IIFE 的**形参 `j`**；

- 里面的小函数被赋值给 `arr[j]` 时，它闭包引用的不是外部的 `test.AO`，而是这层 **专属的 IIFE.AO**；

- 最终 10 个小函数各抱一把专属钥匙，分别打印属于自己的 `j`（0、1、2……9）！

  

## 3. DOM 列表点击事件的同一个坑（对应 `index1.html`）

  

在实际 Web 开发中，给一排 `<li>` 绑定点击事件会遇到完全一样的问题：

  

```html

<ul>

<li>1</li>

<li>2</li>

<li>3</li>

<li>4</li>

<li>5</li>

</ul>

<script type="text/javascript">

var oLi = document.querySelectorAll('li');

  

// 错误写法：点任何一个 li 打印的都是 5（因为绑定完之后 i 已经是 5 了）

// for (var i = 0; i < oLi.length; i++) {

// oLi[i].onclick = function () {

// console.log(i); // 永远打印 5

// };

// }

  

// 正确写法：用 IIFE 为每一个点击事件保留各自的索引

for (var i = 0; i < oLi.length; i++) {

(function (j) {

oLi[j].onclick = function () {

console.log(j); // 0, 1, 2, 3, 4

};

})(i);

}

</script>

```

  

> **现代替代方案**：ES6 中直接将 `var i = 0` 改为 `let i = 0`，`let` 具有块级作用域，引擎会在每一轮迭代中自动产生一个独立的作用域，不再需要手动套一层 IIFE。

  

# 四、逗号运算符：从左往右算，只返回最后一项

  

逗号运算符是 JavaScript 中优先级最低的运算符，面试中经常和括号、函数表达式混在一起考。

  

![逗号运算符的求值规则与典型坑点](https://img.215454.xyz/file/blog/jia/1791426567501_04-comma-operator.png)

  

## 1. 核心计算规则

  

```js

var a = (表达式1, 表达式2, 表达式3, ..., 表达式N);

```

  

- **从左往右依次计算**每一个子表达式；

- 前面表达式的副作用会照常发生（如 `i++`、函数调用等）；

- **整个括号的最终结果是最后一个表达式的值**。

  

```js

var a = (1 + 1, 2 * 3, 10);

console.log(a); // 10（前面的 2 和 6 都参与了计算，但最终只取 10）

```

  

## 2. 逗号运算符与函数表达式连环坑

  

```js

var fn = (

function f() { return "1"; },

function g() { return 2; }

)();

console.log(typeof f); // 'undefined'

console.log(fn); // 2

```

  

一步步拆解：

1. 括号里有两个函数声明，但因为放在了逗号运算符的括号里，**两个都被强制转换成了函数表达式**；

2. 函数一旦变成表达式，函数名 `f` 和 `g` 就会在外部丢失（自动忽略），所以在外部访问 `f` 是 `undefined`；

3. 逗号运算符从左到右取最后一项，所以括号返回的是函数 `g`；

4. 紧接着跟上执行括号 `()` 调用函数 `g`，返回值是数字 `2`，赋给 `fn`。

  

# 五、闭包实战一：累加器与私有变量保护

  

闭包最基础的工业级应用：**防止全局变量污染，实现真正的数据私有化**。

  

![闭包实战：累加器与私有变量保护](https://img.215454.xyz/file/blog/jia/1791426575752_05-accumulator-closure.png)

  

```js

function test() {

var count = 0; // 私有变量，外界无法直接读取或修改

  

function add() {

count++;

console.log(count);

}

  

return add; // 将操作私有变量的函数暴露出去

}

  

var add = test();

add(); // 1

add(); // 2

add(); // 3

  

// 重新调用 test()，造一个全新的累加器

var add2 = test();

add2(); // 1（完全独立的 count，不影响上面那个 add）

```

  

### 为什么两次创建的累加器互不影响？

  

- 每次执行外部函数 `test()` 时，都会在堆内存中新开辟一块独立的 `AO`；

- `add` 闭包保存的是 `AO_1`（里面的 count 累加到 3）；

- `add2` 闭包保存的是 `AO_2`（里面的 count 从 0 开始累加到 1）；

- 两块内存完全隔离，互不干扰。

  

# 六、闭包实战二：返回对象，暴露多组操作接口

  

在更复杂的场景下，内部闭包不需要只返回一个单一函数，而是可以**返回一个包含多个方法的对象**。

  

![闭包实战：返回操作对象管理班级名单](https://img.215454.xyz/file/blog/jia/1791426577626_06-class-students-closure.png)

  

```js

function myClass() {

var students = []; // 真正的学生数据，完全私有化

  

var operation = {

join: function (name) {

students.push(name);

console.log(students);

},

leave: function (name) {

var idx = students.indexOf(name);

if (idx !== -1) {

students.splice(idx, 1);

}

console.log(students);

}

};

  

return operation; // 暴露操作接口

}

  

var obj = myClass();

obj.join('张三'); // ['张三']

obj.join('李四'); // ['张三', '李四']

obj.leave('张三'); // ['李四']

```

  

## 这种模式的核心价值

  

1. **信息隐藏（数据安全）**：外界根本没有直接拿到 `students` 数组的引用指针，任何人都无法通过 `obj.students = null` 或 `obj.students.length = 0` 恶意或无意改坏底层数据；

2. **操作收拢与统一校验**：增删改查完全被收拢在 `join` / `leave` 两个方法内部，可以在方法中自由加入参数校验、日志打印或权限控制；

3. **共享同一个闭包**：`join` 和 `leave` 在同一个父级上下文内创建，它们的作用域链 `[[scope]]` 指向的是**同一个 AO**，因此一个方法对 `students` 的改动对另一个方法**完全可见**。

  

> **架构启示：** 这就是现代前端状态管理（如 Redux、Pinia）以及模块化（Module Pattern）最早的原型思想：私有 State + 显式暴露的 Actions。

  

# 本课小结

  

- **函数声明不能直接加 `()` 执行**，因为语句后面跟空括号是语法错误；

- **只有函数表达式才能被 `()` 执行**；用括号包裹 `(function(){})()` 是最推荐的 IIFE 写法，也可以用 `+`、`-`、`!`、`||` 将其强转为表达式；

- **IIFE 的函数名会被自动忽略**，通常直接使用匿名函数；

- **经典闭包循环问题**的根源：循环体内定义的函数共享父级 AO 中同一个最终定格的变量 `i`；

- **解决循环闭包**：在循环内部用 IIFE 传参，强行在中间多造一层专属的 AO 冻结当轮实参；

- **逗号运算符**：从左往右依次计算，整个表达式返回**最后一项的值**，前面的计算照常发生；

- **闭包实战价值**：

1. 累加器：私有化变量，防止全局污染；

2. 模块对象：通过返回包含多个方法的对象，多方法共享同一个私有闭包 State，实现严谨的面向对象数据封装。