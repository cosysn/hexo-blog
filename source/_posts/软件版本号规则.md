---
title: 软件版本号规则
date: 2016-11-12 02:39:11
tags:
---

<!--
  @author: cosysn    
  @time: 2014/11/18     
  @license: GPLv2
  @contact: cosysn@163.com

  This program is free software; you can redistribute it and/or
  modify it under the terms of the GNU General Public License
  as published by the Free Software Foundation; either version 3
  of the License, or (at your option) any later version.     

  This program is distributed in the hope that it will be useful,
  but WITHOUT ANY WARRANTY; without even the implied warranty of
  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  GNU General Public License for more details.

  You should have received a copy of the GNU General Public License
  along with GNU Emacs; see the file COPYING.  If not, write to the
  Free Software Foundation, Inc., 51 Franklin Street, Fifth Floor,
  Boston, MA 02110-1301, USA.

  Copyright (c) 2014 cosysn, Colibri Team, All Rights Reserved.
end!
-->
<!-- more -->
## 版本号命名格式
主版本号.次版本号[修正版本号[.编译版本号]]

`Marjor_version_Number.Minor.Version.Number[.Resvision_Number[.Build Number]]`

版本号由二至四个部分组成：主版本号、次版本号、内部版本号和修订号。主版本号和次版本号是必选的；内部版本号和修订号是可选的，但是如果定义了修订号部分，则内部版本号就是必选的。所有定义的部分都必须是大于或等于 `0` 的整数。 应根据下面的约定使用这些部分： 

- `Major`：具有相同名称但不同主版本号的程序集不可互换。例如，这适用于对产品的大量重写，这些重写使得无法实现向后兼容性。 

- `Minor`：如果两个程序集的名称和主版本号相同，而次版本号不同，这指示显著增强，但照顾到了向后兼容性。例如，这适用于产品的修正版或完全向后兼容的新版本。 

- `Build`：内部版本号的不同表示对相同源所作的重新编译。这适合于更改处理器、平台或编译器的情况。

- `Revision`：名称、主版本号和次版本号都相同但修订号不同的程序集应是完全可互换的。这适用于修复以前发布的程序集中的安全漏洞。程序集的只有内部版本号或修订号不同的后续版本被认为是先前版本的修补程序 `(Hotfix)` 更新。 



## 版本号管理策略
1. 项目出版本，版本号为`0.1`；
2. 当项目进行了局部修正或`BUG`修复时，主版本和子版本号都不变，修正版本号递增`1`
3. 当项目增加了部分功能,主版本号不变，子版本号递增`1`，同时将修正版本置`0`。 
4. 当项目进行了重大修改或局部修改积累较多时，将导致项目的全局变化时，主版本号递增`1`；
5. 编译版本号一版由编译器在编译过程中自动生成，我们只定义其格式，并不进行控制；
6. 当修正版本号为`0`时，可以省略不写。如初始版本应该是`0.1.0`，可简写成`0.1`；
7. 还可以在版本号的后面加入 `alpha`、`beta`、`gamma`、`rc (release candidate)`、`release`、`stable` 等后缀，在这些后缀后面还可以加数字的版本号。