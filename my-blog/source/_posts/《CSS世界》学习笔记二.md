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

- [实现凹凸效果的示例](https://demo.cssworld.cn/3/2-6.php)

- [最大宽度示例：平滑滚动](https://demo.cssworld.cn/3/2-7.php)

- [height: 100%示例, 绝对定位与非绝对定位的区别](https://demo.cssworld.cn/3/2-11.php)

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

  ### 内部尺寸

> 元素的尺寸由内部的元素决定，而非外部的容器决定，**一个元素里面内容，宽度为0，则其应用的是“内部尺寸”**

  - 包裹性

> inline-block属性是包裹性最好的例子，具体表现为：**内容越多宽度越宽（内部尺寸特性），内容足够多，在容器的宽度处换行(*自适应性，永远小于包裹块容器的尺寸*)**

  - 首选最小宽度

  1. 东亚文字最小宽度为每个字的宽度
  2. 西方文字最小宽度由特定的连续的英文字符单元决定
  3. 图片最小宽度为该元素内容本身的宽度

  [实现凹凸效果的示例](https://demo.cssworld.cn/3/2-6.php)

  - 最大宽度

  > 元素可以有的最大宽度，若内部无块级元素或块级元素没有设定宽度值，“最大宽度”是最大的连续内联盒子的宽度

  例： 
  ```html
  <div>
    "我是文本"
    <span>我是span</span>
    <button>按钮</button>
    <img src="123" />
    <br>
    我是下一行
  </div>
  ```
  在*最大宽度模式下*显示效果为：
  “我是文本”我是span<button>按钮</button><img src='123'>

  [最大宽度示例：平滑滚动](https://demo.cssworld.cn/3/2-7.php)

  ### width值作用的细节

  > **width值作用在内在盒子上**

  - content-box
  - padding-box
  - border-box
  - margin box无对应的CSS关键字名称

  **width 作用在content-box上**

  ### 宽度分离原则

  > CSS中的width属性不与影响宽度的padding/border 共存，
  
  也就是不能出现以下结构:
  ```CSS
    .box {
      width: 100px;
      border: 1px solid;
    }
  ```

  应写成以下结构
  ```CSS
  .father {
    width: 180px;
  }
  .son {
    margin: 0 20px;
    padding: 20px;
    border: 1px solid;
  }
  ```

  ### box-sizing属性

  > box-sizing属性改变width的作用细节

   - box-sizing: content-box 默认值
   - box-sizing: padding-box Firefox曾经支持
   - box-sizing: border-box 全线支持

   **box-sizing的发明初衷是解决替换元素宽度自适应的问题**

   ```css
   input, textarea, img, video, object {
     box-sizing: border-box;
   }
   ```

  ### Height: 100%

   - 在**非绝对定位**中相对于content-box计算
   - 在**绝对定位**中相对于padding-box计算

  [height: 100%示例](https://demo.cssworld.cn/3/2-11.php)

  ## min-width/min-height,max-width/max-height

   - 初始值均为"**none**"
   - 权重**超越important**

   ```html
    <img src="1.jpg" style="width:356px !important;">
   ```

   ```css
    img { max-width: 256px; }
   ```

   该img为256px

   - 超越最大（min > max)

   ```css
    .container {
      min-width: 1200px;
      max-width: 1000px;
    }
    /* 该类元素为 1200px */
   ```

   [利用max/min width/height实现的元素滑动效果](https://demo.cssworld.cn/3/3-2.php)

   ## 内联元素的“幽灵空白节点”

  ```html
    <div>
      <span></span>
    </div>
  ```
  选中span，高度18像素，可假象为内部有一“幽灵空白节点”
