---
title: "JavaScript第六课"
published: 2026-09-26
description: This is the first post of my new Astro blog.
tags: [JavaScript]
author: jia
---
//AO Go 作用域

// 作用域链相关所产生的一切问题

  

//AO是紧密和function联合在一起，funciton独立的仓库

  

// //对象

// var obj = {

// name:'蓝轨迹',

// address:'北京',//属性

// teach:function(){//方法

  

// }

// }

// console.log(obj.name);

  

function test(a,b){

  

}

console.log(test.name);

console.log(test.length);

  

//函数也会一种对象类型，引用类型 引用值

//test.name test.length test.prototype

//对象 -》 有些属性是我们无法访问的

// JS引擎内部固有的隐式属性

//[[scope]]

//1.函数创建时，生成的一个JS内部的隐式属性、

//2.函数存储作用域链的容器，作用域链

  

//AO/GO

//AO:函数的执行期上下文

//GO：