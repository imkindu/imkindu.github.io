---
title : git commit规范
date : 2017-11-09 11:00:00
author : imkindu
categories : git
tags :
- git
---

　　以前公司使用SVN和自己使用git提交写注释时，都是随手瞎写的，主要是备注给自己看的，但是接触到linux command这种命令行格式之后，觉得太low，也不太方便查看和查找。

<!--more-->


## commit message好处


- 可读性好，清晰，不必深入看代码即可了解当前commit的作用。
- 为 Code Reviewing做准备
- 方便跟踪工程历史
- 让其他的开发者在运行 git blame 的时候想跪谢
- 提高项目的整体质量，提高个人工程素质

## commit message格式

　　每次提交，Commit message 都包括三个部分：`header`，`body` 和 `footer`。

``` python
<type>(<scope>) : <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```


其中，header 是必需的，body 和 footer 可以省略。
不管是哪一个部分，任何一行都不得超过72个字符（或100个字符）。这是为了避免自动换行影响美观。

> Header

Header部分只有一行，包括三个字段：type（必需）、scope（可选）和subject（必需）。

> **type**

用于说明 commit 的类别，只允许使用下面7个标识。

- `feat`：新功能（feature）
- `fix`：修补bug
- `docs`：文档（documentation）
- `style`： 格式（不影响代码运行的变动）
- `refactor`：重构（即不是新增功能，也不是修改bug的代码变动）
- `test`：增加测试
- `chore`：构建过程或辅助工具的变动


如果type为feat和fix，则该 commit 将肯定出现在 Change log 之中。其他情况（docs、chore、style、refactor、test）由你决定，要不要放入 Change log，建议是不要。

> **scope**

scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

例如在Angular，可以是$location, $browser, $compile, $rootScope, ngHref, ngClick, ngView等。

如果你的修改影响了不止一个scope，你可以使用*代替。

> **subject**

subject是 commit 目的的简短描述，不超过50个字符。

其他注意事项：

以动词开头，使用第一人称现在时，比如change，而不是changed或changes
第一个字母小写
结尾不加句号（.）



## git log查看

``` shell
[imkindu@ubuntu dev01]$ git commit dev01.txt -m "feat text : dev02二次提交

简短描述

修改内容"
```

``` shell
[imkindu@ubuntu dev01]$ git log
commit efbe85c6c49859f2d1b6615880c3b341f828271d
Author: imkindu <imkindu@163.com>
Date:   Thu Nov 9 18:24:16 2017 +0800

    feat text : dev02二次提交
    
    简短描述
    
    修改内容
```

### git log搜索

``` shell
[imkindu@ubuntu dev01]$ git log dev01-02 --grep feat
commit efbe85c6c49859f2d1b6615880c3b341f828271d
Author: imkindu <imkindu@163.com>
Date:   Thu Nov 9 18:24:16 2017 +0800

    feat text : dev02二次提交
    
    简短描述
    
    修改内容
```