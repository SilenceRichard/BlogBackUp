---
title: 《CSS世界》学习笔记（二）
date: 2020-01-24 09:28:55
tags:
  - CSS
  - 读书笔记
---

# 第三章

- [块级元素清除浮动示例](https://demo.cssworld.cn/3/1-1.php)

- [“外部尺寸”block元素的流动性示意实例](https://demo.cssworld.cn/3/2-3.php)

- min-width/max-width 和 min-height/max-height

- 内联元素

## 块级元素

### 概述

> 基本特征：一个水平流上只能单独显示一个元素

> 常见的块级元素有\<div>,\<li>,\<table>,
注意**块级元素和 display: block并不是一个概念**，例如\<li>的display默认值为list-item,\<table>的display默认值为table，它们均为块级元素

### 清除浮动

块级元素可以配合clear属性清除浮动，见 [块级元素清除浮动示例](https://demo.cssworld.cn/3/1-1.php)

```CSS
.clear:after {
  content: '';
  display: table;
  clear: both;
}
```

### list-item

在清除浮动的示例里，我们设置display: list-item, 会发现多出一个圆点

- 所有块级元素都有一个**主块级盒子**，list-item除此之外还有一个**附加盒子**，学名**标记盒子（marker box）**,专门用来存放圆点、数字

### 外在盒子与内在盒子

- 如 **inline-block**, 穿着inline的皮，裹着block的心

- 外在盒子**负责元素可以一行显示还是只能换行显示**

- 内在盒子（容器盒子）负责宽高，内容呈现

- 我们可以把display: block 脑补成 display: block-block, display: table 脑补成 display: block-table

## width/height作用的具体细节

### width: auto 属性

#### width:auto 的宽度表现
  - 充分利用可用空间(fill-available)
  > 默认100%于父级容器，如\<div>,\<p>
  - 收缩与包裹(shrink-to-fit)
  > 收缩到合适，“包裹性”，典型代表 **浮动、绝对定位、inline-block、table元素**
  - 收缩到最小(min-content)
  > 常出现在**table-layout为auto的表格中**,当每一列空间都不够时，文字能断就断，**中文随便断，英文单词不能断**，如[收缩到最小示例](https://demo.cssworld.cn/3/2-1.php)，第一列变成"一柱擎天"
  - 超出容器限制
  
  > 除非是有明确的width相关设置，否则上面3种情况尺寸都不会主动超过父级容器宽度，特殊情况： **内容很长的连续的英文和数字，或者内联元素被设置了white-space: nowrap**

  ### 格式化宽度

  - 出现在绝对定位模型（fixed,absolute）
  - 默认宽度由绝对定位元素的内部尺寸决定
  - 对于*非替换元素*，当left/top或top/bottom **对立方位**的属性值同时存在时，其元素的宽度表现为“格式化宽度”，宽度大小相对于最近的具有定位特性的祖先元素计算

  如：
  ```css
  div {
    position: absolute;
    left: 20px;
    right: 20px;
  }
  /*该div最近的具有定位特性的祖先元素的宽度是1000px, 则这个<div>元素的宽度是（1000 - 20 - 20）= 960px*/
  ```
  - 格式化宽度具有完全的流体性