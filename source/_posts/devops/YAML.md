---
title : ansible第四篇 ：YAML
date : 2017-11-17 21:00:00
author : imkindu
categories : devops
tags :
- YAML
- ansible
---

　　ansible中的playbook配置使用YAML格式写的，所以在介绍playbook之前，需要单独介绍一下YAML。YAML 是专门用来写配置文件的语言，非常简洁和强大，远比 JSON 格式方便。

<!--more-->


## 简介

　　YAML（/ˈjæməl/，尾音类似camel骆驼）是一个可读性高，用来表达数据序列的格式。YAML参考了其他多种语言，包括：C语言、Python、Perl，并从XML、电子邮件的数据格式（RFC 2822）中获得灵感。Clark Evans在2001年首次发表了这种语言[1]，另外Ingy döt Net与Oren Ben-Kiki也是这语言的共同设计者[2]。目前已经有数种编程语言或脚本语言支持（或者说解析）这种语言。

　　YAML是"YAML Ain't a Markup Language"（YAML不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）[3]，但为了强调这种语言以数据做为中心，而不是以标记语言为重点，而用反向缩略语重命名。

　　不说别的，其实我们早就见过YAML格式的配置文件了，使用hexo搭建博客时，其配置文件`_conf.yml`就是YAML配置的。

## 语法规则


YAML 语言（发音 /ˈjæməl/ ）的设计目标，就是方便人类读写。它实质上是一种通用的数据串行化格式。

它的基本语法规则如下。

``` text
大小写敏感
使用缩进表示层级关系
缩进时不允许使用Tab键，只允许使用空格。
缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
```

`#` 表示注释，从这个字符一直到行尾，都会被解析器忽略。

YAML 支持的数据结构有三种。

> `对象`：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
`数组`：一组按次序排列的值，又称为序列（sequence） / 列表（list）
`纯量（scalars）`：单个的、不可再分的值



## 对象

　　对象其实就是一组键值对，使用冒号表示

``` yaml
name : imkindu
age : 28
```

　　对象可以嵌套对象

``` yaml
user : { name : imkindu, age : 28 }
```

## 数组

　　个人比较喜欢称呼为列表，在hexo 的tags配置就是一串列表，以`-`开头

``` yaml
- Cat
- Dog
- Fish
```

　　数据结构的子成员是一个数组，则可以在该项下面缩进一个空格。

``` yaml
-
 - Cat
 - Dog
 - Goldfish
```

　　转换为javascript格式为

``` yaml
[ [ 'Cat', 'Dog', 'Goldfish' ] ]
```


## 组合结构

　　对象和数组可以结合使用，形成复合结构。

``` yaml
languages:
 - Ruby
 - Perl
 - Python 
websites:
 YAML: yaml.org 
 Ruby: ruby-lang.org 
 Python: python.org 
 Perl: use.perl.org 
```

转为 JavaScript 如下。

``` yaml
{ languages: [ 'Ruby', 'Perl', 'Python' ],
  websites: 
   { YAML: 'yaml.org',
     Ruby: 'ruby-lang.org',
     Python: 'python.org',
     Perl: 'use.perl.org' } }
```


## 纯量

纯量是最基本的、不可再分的值。以下数据类型都属于 JavaScript 的纯量。

> - 字符串
- 布尔值
- 整数
- 浮点数
- Null
- 时间
- 日期

数值直接以字面量的形式表示。

``` yaml
number: 12.30
```

## 参考


[阮一峰 YAML语言教程] http://www.ruanyifeng.com/blog/2016/07/yaml.html
[维基百科 YAML] https://zh.wikipedia.org/wiki/YAML