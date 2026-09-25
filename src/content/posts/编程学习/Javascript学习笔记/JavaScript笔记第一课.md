---
title: JavaScript笔记第一课
published: 2026-09-25
description: 这是Javascript第一课
tags:
  - JavaScript
author: jia
---
# JS 第一课：浏览器内核、JavaScript 的由来与变量入门

  

> 本课对应练习文件：`js.html`（浏览器内核、JS 历史、ECMA、编译/解释型、JS 三大块、单线程）、`index.html`（变量与数据类型）、`js/index.js`（外链脚本）。

  

# 一、五大浏览器与它们的浏览器内核

  

## 1. 内核 = 渲染引擎 + JS 引擎

  

早期浏览器只有「渲染引擎」，2001 年 IE6 之后，浏览器内核被拆成了两部分：

  

![浏览器内核的组成与五大浏览器的对照表](images/01-browser-engines.png)

  

- **渲染引擎**：把 HTML 和 CSS 画成你看到的页面 —— 排版、颜色、字体、动画都归它管；

- **JS 引擎**：逐行解释执行 JavaScript 代码。

  

同一个网页脚本，引擎越好就跑得越快。

  

## 2. 五大浏览器的内核对照

  

| 浏览器 | 渲染引擎 | JS 引擎 | 说明 |

| --- | --- | --- | --- |

| IE | Trident | JScript | JScript 性能较差，带来不少性能问题和安全漏洞 |

| Chrome | WebKit → Blink | V8 | 2008 年谷歌发布，V8 把 JS 直接翻译成机器码 |

| Safari | WebKit | JavaScriptCore | 苹果自家浏览器 |

| Firefox | Gecko | SpiderMonkey | Mozilla 公司 2003 年发布，性能大幅提升 |

| Opera | Presto → Blink | V8 | 早期用自家的 Presto，后来改用 Chromium 内核 |

  

## 3. V8 引擎厉害在哪

  

V8 的两个特点，直接改变了整个 Web 生态：

  

1. **直接把 JS 翻译成机器码**（而不是先转成中间代码再解释），速度大幅提升；

2. **可以独立于浏览器运行** —— 这也是后来 Node.js 能在服务器上跑 JS 的前提。

  

> 微软的 Edge 也走过同样的路：2016 年用自己的 EdgeHTML + Chakra 引擎，2020 年转向 Chromium 内核，改用 Blink + V8，兼容性和性能都上了一个台阶。

  

# 二、浏览器历史与 JavaScript 的诞生

  

![JavaScript 的诞生与浏览器大战时间线](images/02-js-history.png)

  

## 1. 一切从「分享文档」开始

  

| 年份 | 事件 | 意义 |

| --- | --- | --- |

| 1990 | 蒂姆·伯纳斯·李开发了 World Wide Web | 让超文本能在网上互相分享 |

| 1993 | 美国伊利诺大学 NCSA 组织（马克·安德森）开发 Mosaic 浏览器 | 能显示图片，真正的图像浏览器，普通人也能上网 |

| 1994 | 马克·安德森和吉姆·克拉克成立 Mosaic 通信公司 | 后来改名为 **Netscape**，网景公司 |

  

## 2. 网景时代：Netscape Navigator

  

1995 年网景发布了 Netscape Navigator 的第一个版本，迅速成为当时最受欢迎的浏览器，**市场占有率一度超过 90%**。

  

也是在网景，**Brendan Eich 开发了 LiveScript 语言** —— 这就是 JavaScript 的前身。

  

## 3. LiveScript 为什么改名 JavaScript

  

1996 年 Java 火了起来，网景公司为了蹭热度，把 LiveScript 改名成了 **JavaScript**，造成了很多人的误解。

  

> **要记住：JavaScript 和 Java 是两种完全不同的编程语言** —— 设计目标、语法和用途都不一样，只是名字像而已。

  

## 4. 浏览器大战

  

1996 年，微软收购了 Spyglass 公司，拿到 Mosaic 浏览器的授权，开始开发 Internet Explorer，并**捆绑在 Windows 里发布**。IE 3.0 里出现了 **JScript**，与 Netscape 展开激烈的浏览器大战，IE 逐渐占领市场。

  

| 年份 | 事件 |

| --- | --- |

| 1996 | IE 3.0 出现 JScript，浏览器大战开始 |

| 2001 | IE6 随 XP 诞生，**浏览器内核分成渲染引擎和 JS 引擎**；IE6 的 JScript 性能较差 |

| 2003 | Mozilla 发布 Firefox，Gecko 渲染引擎 + SpiderMonkey JS 引擎，性能大幅提升 |

| 2008 | 谷歌发布 Chrome，带来 **V8 引擎** |

| 2015 | ECMA 国际组织发布 **ECMAScript 6（ES6）**，成为现代 JavaScript 的基础 |

| 2020 | 微软宣布 Edge 转向 Chromium 内核 |

  

## 5. 近年版本

  

- 2021 年 Chrome 90 继续优化 V8 引擎；

- 2022 年 Firefox 100 改进 Gecko 与 SpiderMonkey；

- 2023 年 Safari 15 采用 WebKit + JavaScriptCore；

- 2024 年 Edge 基于 Chromium 内核继续更新。

  

# 三、ECMA、ECMAScript 与 JS 的三大块

  

![JavaScript 的三大块：ECMAScript、DOM、BOM](images/04-js-three-parts.png)

  

## 1. ECMA 是什么

  

**ECMA**（European Computer Manufacturers Association，**欧洲计算机制造商协会**）负责评估、开发、认可电信和计算机标准。

  

**ECMA-262** 是脚本语言的规范，这套规范就叫 **ECMAScript**。

  

> 关键一点：**ES5、ES6 只是标准，不是语言**。标准规定「应该有什么语法」，各家引擎去实现这套标准。

  

## 2. JavaScript 的三大块

  

| 组成部分 | 全称 | 管什么 | 谁制定标准 |

| --- | --- | --- | --- |

| **ECMAScript** | —— | 语言本身：语法、变量、关键字、保留字、值、对象、继承、函数、引用类型 | ECMA（ECMA-262） |

| **DOM** | Document Object Model 文档对象模型 | 操作网页内容（把网页变成对象树） | W3C |

| **BOM** | Browser Object Model 浏览器对象模型 | 操作浏览器本身（窗口、地址栏、前进后退） | 没有规范，各家自己实现 |

  

> 记住这条分界线：**ECMAScript 是 JS 的语言核心，DOM 和 BOM 是浏览器额外提供的**。

> 所以同一段 JS，在浏览器里能操作 `document`、`window`，在别的环境里就不一定能用了。

  

# 四、编译型与解释型

  

![编译型与解释型的翻译过程对比](images/03-compiled-vs-interpreted.png)

  

## 1. 两种语言的翻译过程

  

| 类型 | 过程 | 特点 |

| --- | --- | --- |

| **编译型** | 源码 → 编译器（编译） → 机器码（可执行文件） → CPU 执行 | 先整体翻译成可执行文件，再执行 |

| **解释型** | 源码 → 解释器（逐行解释执行） → CPU 执行 | 不生成可执行文件，边翻译边执行 |

  

## 2. JavaScript 属于解释型

  

**JavaScript 是一种解释型语言**：代码在运行时由浏览器的 JavaScript 引擎逐行解释执行。

  

- 好处：**开发周期快、跨平台兼容性好**（有浏览器就能跑）；

- 代价：**性能上不如编译型语言**。

  

> 这里说的「逐行解释」，在 V8 里做了很大优化（会把热点代码直接编译成机器码），所以现代 JS 的速度已经快了很多 —— 但语言分类上它仍然属于解释型。

  

## 3. 高级语言

  

不管是编译型还是解释型，写出来的源码都是**高级语言** —— 给人看的。最终都得变成机器码，CPU 才能执行。

  

# 五、单线程与轮转时间片

  

![JS 的单线程模型与轮转时间片](images/05-single-thread.png)

  

## 1. JS 是单线程的

  

- **单线程**：同一时间只能执行一个任务，任务之间需要排队等待；

- **多线程**：同一时间可以执行多个任务，任务之间可以并行处理。

  

**JavaScript 是单线程的** —— 一个任务在执行时，其他任务都要等着，所以有可能出现**阻塞**现象（比如某个操作特别慢，页面就像卡死了一样）。

  

## 2. 用「轮转时间片」模拟多线程

  

为了在单线程里实现「同时做几件事」的效果，JS 用了**轮转时间片**：短时间内轮流执行多个任务的片段。

  

1. 拿到任务 1、任务 2；

2. 把每个任务**切分成小片段**；

3. **随机排列**这些任务片段，组成一个队列；

4. 按照队列顺序，把任务片段依次送进 JS 进程；

5. JS 线程执行一个又一个任务片段。

  

因为切换速度极快，人感觉不到中间的空档 —— 看起来就像几个任务在「同时」跑。

  

> 这也解释了为什么「一段代码写得太慢」会拖垮整个页面：单线程下没有别的线程能顶上来。

  

# 六、变量：声明、赋值与命名规范

  

![变量的声明与赋值，以及命名规范](images/06-variables.png)

  

## 1. 声明与赋值

  

变量的使用由两个部分组成：**声明变量** + **变量赋值**。

  

```js

var a; // ① 声明：只是先占个位置

a = 3; // ② 赋值：把 3 装进去

var a = 3; // ③ 声明并赋值：两步合一（最常用）

```

  

## 2. `=` 是「赋值」，不是数学里的「等于」

  

```js

var x = 1;

x = 2; // 把 2 装进 x，原来的 1 被覆盖

document.write(x); // 2

```

  

数学里 `x = 2` 是「x 等于 2」这个判断；在 JS 里它是**一条动作**：把右边的值装进左边的变量。

  

## 3. 一次声明多个变量

  

用逗号隔开：

  

```js

var x = 3,

y = 4;

var z = x + y;

document.write(z); // 7

```

  

## 4. 命名规范（企业开发要求）

  

| # | 规则 | 例子 |

| --- | --- | --- |

| 1 | 不能以数字开头 | ✗ `1abc`　　✓ `abc1`（数字放中间或结尾可以） |

| 2 | 可以用字母、下划线 `_`、美元符 `$` 开头 | ✓ `_abc`　`$abc`　`abc` |

| 3 | 中间可以包含字母、下划线、`$` 和数字 | ✓ `my_name1`　`$price2` |

| 4 | **关键字、保留字**不能用来命名 | ✗ `var`　`function`　`for`　`if` |

| 5 | **语义化、结构化**：一眼看懂装的是什么 | ✓ `userName`　✗ `a1`、`aa`、`bbb` |

| 6 | 多个单词用**小驼峰命名法** | ✓ `myEnglishName`（首字母小写，后面每个单词首字母大写） |

| 7 | 尽量用英文和缩写 | 不要用拼音和毫无意义的字母组合 |

  

# 七、数据类型：弱类型、原始值与引用值

  

![弱类型语言与 JS 的两类值](images/07-weak-typing.png)

  

## 1. JavaScript 是弱类型语言

  

按语言的分类链条来看：

  

- JavaScript 这一类：**动态语言 → 脚本语言 → 解释型语言 → 弱类型语言**；

- 另一类（如 Java、C++）：**静态语言 → 编译型语言 → 强类型语言**。

  

「弱类型」的表现就是：**变量本身不挑类型，装什么完全由赋的值决定**。

  

```js

var a = 1; // 先装一个数字

var a = 3.14; // 改成小数，没问题

var str = '我爱编程'; // 换个变量装字符串，也没问题

```

  

> 强类型语言要先说清楚「这个变量只能装整数」，之后想换类型就得重新声明 —— JS 不用。

  

## 2. 原始值（基本类型）· 5 种

  

| 类型 | 例子 | 说明 |

| --- | --- | --- |

| `Number` | `1`、`3.14` | 数字（含小数） |

| `String` | `'我爱编程'` | 字符串 |

| `Boolean` | `true` / `false` | 布尔值 |

| `undefined` | —— | 未定义 |

| `null` | —— | 空 |

  

## 3. 引用值 · 5 种

  

**object（对象）、array（数组）、function（函数）、date（日期）、RegExp（正则）**

  

```js

var arr = [1, 2, 3];

arr.push(5); // 数组自带的方法，往后追加一个元素

document.write(arr); // 1,2,3,5

```

  

# 八、栈内存与堆内存

  

这是本课最核心的一个概念 —— 原始值和引用值**存储方式不同**，所以「赋值」的行为完全不同。

  

![栈内存与堆内存：原始值和引用值的区别](images/08-stack-heap.png)

  

## 1. 原始值：值就存在栈里

  

**栈内存**的特点是：先进后出，空间小、速度快；原始数据是直接**覆盖**，旧值不会留着。

  

```js

var a = 2;

var b = a; // 复制一份「值」给 b

a = 3;

document.write(a, b); // 3 2

```

  

`b = a` 的那一刻，b 拿到的是 a 当时那份值的**拷贝**。之后 a 再改成 3，和 b 已经没有关系了 —— 所以 b 还是 2。

  

## 2. 引用值：变量在栈里，值在堆里

  

引用值的存储方式不一样：

  

> **引用值存储在栈内存，值存储在堆内存，栈里的变量存的是「指向堆内存的指针（地址）」。**

  

```js

var arr1 = [1, 2, 3, 4];

var arr2 = arr1; // 复制的是「地址」

arr1.push(5); // 往堆里的数组加一个 5

document.write(arr1); // 1,2,3,4,5

document.write(arr2); // 1,2,3,4,5 ← 一起变

```

  

两个变量里存着**同一个地址**，指向堆里的同一个数组 —— 通过谁改都算数。

  

## 3. 两种「改」的区别

  

这里有个很容易混的地方：**改数组内容**和**给变量换一个新数组**，结果完全不同。

  

```js

// 情况一：改数组内容（push）→ 两个变量一起变

var arr1 = [1, 2, 3, 4];

var arr2 = arr1;

arr1.push(5);

document.write(arr1); // 1,2,3,4,5

document.write(arr2); // 1,2,3,4,5

  

// 情况二：给 arr1 换一个新数组 → arr2 不受影响

arr1 = [1, 2];

document.write(arr1); // 1,2

document.write(arr2); // 1,2,3,4,5 ← arr2 还是原来那个

```

  

- **情况一**：`push` 是在**原来那块堆内存**上改，两个变量看的是同一个数组，所以一起变；

- **情况二**：`arr1 = [1, 2]` 是让 arr1 **指向了一块新的堆内存**，arr2 还指着老地址，所以纹丝不动。

  

> 一句话总结：**改「堆里的东西」会互相影响，改「变量指向哪」不会。**

  

# 九、把 JS 写进网页

  

## 1. 外链脚本

  

把 JS 单独写在一个文件里，用 `src` 引进来：

  

```html

<script type="text/javascript" src="js/index.js"></script>

```

  

`js/index.js` 里就一句话：

  

```js

document.write('Hello World');

```

  

## 2. 内嵌脚本

  

直接写在 HTML 里：

  

```html

<script type="text/javascript">

var a = 2;

var b = a;

a = 3;

document.write(a); // 3

</script>

```

  

## 3. 一个容易踩的坑

  

**如果一个 `<script>` 标签带了 `src`，那么它标签中间写的内容会被忽略**：

  

```html

<!-- 这样写，中间的 document.write 不会执行 -->

<script type="text/javascript" src="js/index.js">

document.write('I am a inner JS');

</script>

```

  

想要两个都执行，就老老实实写成两个标签：

  

```html

<script src="js/index.js"></script>

<script>

document.write('I am a inner JS');

</script>

```

  

# 本课小结

  

- **浏览器内核 = 渲染引擎 + JS 引擎**，五大浏览器的内核分别是 Trident/JScript、Blink/V8、WebKit/JavaScriptCore、Gecko/SpiderMonkey、Presto（后来也用 Blink/V8）；

- 1990 年万维网诞生 → 1993 年 Mosaic → 1994 年网景 → 1995 年 Navigator 与 LiveScript → 1996 年改名 JavaScript 并爆发浏览器大战 → 2008 年 Chrome 与 V8 → 2015 年 ES6；

- **JavaScript 和 Java 没有任何关系**，只是改名的产物；

- **ECMA** 是欧洲计算机制造商协会，**ECMA-262** 是脚本语言规范，这套规范叫 **ECMAScript**；ES5、ES6 只是标准，不是语言；

- **JS 是解释型语言**：源码 → 解释器逐行解释执行 → CPU 执行，开发快、跨平台好，性能不如编译型；

- **JS 的三大块**：ECMAScript（语言核心）、DOM（操作网页，W3C 标准）、BOM（操作浏览器，没有规范）；

- **JS 是单线程的**，用「轮转时间片」把任务切碎、排队轮流执行，来模拟多线程；

- 变量由**声明**和**赋值**两部分组成，`=` 是赋值动作；命名要遵守 7 条规范（小驼峰、语义化、避开关键字）；

- **原始值** 5 种（Number / String / Boolean / undefined / null），**引用值** 5 种（object / array / function / date / RegExp）；

- **原始值存在栈内存**，赋值复制的是值；**引用值存在堆内存**，栈里的变量存的是地址，赋值复制的是地址；

- 改「堆里的内容」会互相影响（`push`），改「变量指向哪」不会（`arr1 = [...]`）。

  

> 最后一句关于编程语言的总结：**任何编程语言都离不开四样东西 —— 变量、数据结构、函数、运算能力**。

> 后面几课要学的，正是这四样在 JavaScript 里的具体写法。