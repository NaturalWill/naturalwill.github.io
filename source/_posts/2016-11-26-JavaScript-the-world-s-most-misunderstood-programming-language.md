layout: blog
title: "JavaScript: 世界上最被误解的语言"
date: 2016-11-26 14:23:53
tags: 
  - JavaScript
  
---

# JavaScript: 世界上最被误解的语言


[JavaScript](http://www.crockford.com/javascript) , 亦称为 Mocha 、 LiveScript 、 JScript 或 ECMAScript ，是世界上流行的编程语言之一。事实上世界上差不多每台个人电脑都至少安装了一个 JavaScript 解释器。JavaScript 的流行完全是由于他在 WWW 脚本语言领域中的地位决定的。

不管它有多么流行，极少有人了解 JavaScript 是一个十分动态的通用面向对象编程语言。这怎能成为一个秘密呢？为什么这个语言如此被误解？

## 关于名字

这个 Java- 前缀暗示了 JavaScript 和 Java 的关系，也就是 JavaScript 是 Java 的一个子集也就是不如 Java 强大。看上去这个名称就故意制造混乱，然后随之而来的是误解。 JavaScript 并不是解释型的 Java 语言。 Java 是解释型的 Java ， JavaScript 是另一种语言。

JavaScript 和 Java 的语法很相似，就象 Java 和 C 的语法相似一样。但它也不是 Java 的子集就像 Java 也不是 C 的子集一样。在应用上， Java 要远比原先设想的好得多（ Java 原称 Oak ）。

JavaScript 并不是由 Sun 公司开发的。 JavaScript 是由 Netscape 公司开发。它本来叫做 LiveScript ，这个名字并不是那样容易混淆。

-Script 后缀暗示了它不是一个真正的编程语言——脚本语言好象不是真正的编程语言。但其实这是一个专长的问题。相对 C 而言， JavaScript 牺牲性能但带来更强的表达力和动态性。

<!-- more -->

## 披着 C 外衣的 Lisp

JavaScript 的 C 风格的语法，包括大括号和复杂的 for  语句，让它看起来好象是一个普通的过程式语言。这是一个误导因为 JavaScript 和函数式语言如 Lisp 和 Scheme 有更多的共同之处。它用数组代替了列表，用对象代替了属性列表。函数是第一位的，而且有闭包。你不需要平衡那些括号就可以用 lambda 算子。

## 思维定势

JavaScript 是原被设计在 Netscape Navigator 中运行的。它的成功让它成为几乎所有浏览器的标准配置。这导致了思维定势。 JavaScript 简直就是程序语言中的 George  Reeves （一位曾扮演超人的演员，但后来死于枪杀，被官方认为自杀，细节不详──译注）。其实， JavaScript 也适合很多和 Web 无关的应用程序。

## 不断改变的目标

JavaScript 的第一个版本功能十分弱。它缺少异常处理、内部函数和继承。而它的现在的形态，它已经是一套完整的面向对象语言。但很多看法都是基于认为它的形式不成熟。

管理这个语言的 ECMA 委员正在开发扩展 ，原意是很好，而这样却会加剧这个语言最严重的问题：版本太多了。这也造成了混淆。

## 设计错误

没有什么编程语言是完美的， JavaScript 也有它的设计上的错误，如“+”的重载同时表示“加法”与“带类型转换的字符串连接”，和有错误倾向的 with 语句应该避免使用。保留字策略过于严格。分号的加入是一个很大的错误，正则表达式的记号也是。这些错误会导致编程错误，并把整个语言的设计视为问题。幸运的是，这些问题可以用一个很好和 [lint](http://www.jslint.com/) 程序来避免。

这个语言的设计从整体上看还是十分健全的。但很令人惊讶的是， ECMAScript 委员会好象根本不想修正这些错误。也许他们对重新制作一个更感兴趣。

## 肮脏的实现

JavaScript 早期实现错误百出。这对该语言带来了很恶劣的影响。更糟糕的是，这些实现还被嵌入的更错误百出的浏览器中。

## 拙劣的书籍

几乎所有的书籍都是相当可怕的。他们包含错误，不好的例子，并促进坏做法。语言的重要特征经常被解释得很差，或者完全干脆完全不写。我回顾了JavaScript的几十本书，我只能推荐其一： [JavaScript: The Definitive Guide (5th Edition)](http://www.amazon.com/exec/obidos/ASIN/0596101996/wrrrldwideweb) by David Flanagan. 

## 不够标准的标准

JavaScript 的[官方标准规格说明书](http://www.ecma-international.org/publications/standards/Ecma-262.htm)由 [ECMA](http://www.ecma-international.org/) 发布。该规格书也是质量奇差。它难以阅读也难以理解。它也对拙劣书籍的问题作出了自己的一份“贡献”，因为作者无法使用这个标准文档来增加他们对语言的认识。 ECMA 和 TC 39委员会应该为此感到深深的羞愧。

## 业余爱好者

大部分写 JavaScript 的人都不是程序员。他们缺乏训练写好程序的修养。 JavaScript 有如此丰富的表达能力，他们可以任意用它来写代码，以任何形式。这给 JavaScript 带来了一个名声──它是专门为外行设计的，不适合专业的程序员。这显然不是事实。

## 面向对象

JavaScript 是不是面向对象的？它拥有对象，可以包含数据和处理数据的方法。对象可以包含其它对象。它没有类，但它却有构造器可以做类能做的事，包括扮演类变量和方法的容器的角色。它没有基于类的继承，但它有基于原型的继承。

两个建立对象系统的方法是通过继承 (is-a) 和通过聚合 (has-a)。 JavaScript 两个都有，但它的动态性质让它可以在聚合上超越。

一些批评说 JavaScript 不是真正面向对象的因为它不能提供信息的隐藏。也就是，对象不能有私有变量和私有方法：所有的成员都是公共的。

但 [Private Members in JavaScript](http://www.crockford.com/javascript/private.html) 又证明了 JavaScript 对象可以拥有私有变量和私有方法。当然，极少有人认识到，因为 JavaScript 是世界是最受误解的语言嘛！

另外还有批评说 JavaScript 不能提供继承，但 [Classical Inheritance in JavaScript](http://javascript.crockford.com/inheritance.html) 却证明了 JavaScript 不仅能支持传统的继承还能应用其它的代码复用模式。


原文链接： [JavaScript: The World's Most Misunderstood Programming Language](http://javascript.crockford.com/javascript.html)
