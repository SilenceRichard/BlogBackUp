---
title: 《CSS世界》读书笔记三
date: 2020-01-28 10:00:06
tags:
  - CSS
  - 读书笔记
---

> 本文主要记录盒尺寸中的content

# content与替换元素

## 什么是替换元素
> 内容可以被替换的元素，如 \<img src='1.jpg'>,将src属性替换，内容替换,**通过修改某个属性值，呈现的内容可以被替换的元素，称为“替换元素”**
- 替换元素的特性

  - 替换元素的内容外观不受页面上的css影响
  - 有自己的尺寸（很多替换元素默认尺寸为300 * 150 px）,也有少部分默认为0px, **如img**, **表单元素在不同浏览器上的尺寸不同**
  - 在很多css属性上有自己的表现规则，如*vertical-align*,默认值baseline,为字符x的下边缘，在替换元素中定义为元素的下边缘

## 替换元素的display值

> 在 IE，Chrome下，\<input>, \<textarea> 表现形式为 inline-block, 在火狐下表现为inline

## 替换元素的尺寸计算规则

### 固有尺寸/HTML尺寸/CSS尺寸

#### 定义

- 固有尺寸指替换内容原本的尺寸
- HTML尺寸， 指通过HTML的原生属性改变的尺寸
```html
<img width="300" height="300">
```
- CSS尺寸， 通过CSS的width/height,max-width/min-width或max-height/min-height设置的尺寸

#### 注意

- 最终尺寸由CSS属性决定
- 如果固有尺寸含宽高比例，HTML/CSS只设置了width/height,则元素按固有比例缩放
- 默认 300 * 150 px
- 内联和块级替换元素使用同一计算规则

```css
img { display: block }
```
```html
<img src="1.jpg">
```
该 img block, 宽度没有100%容器

### 替换元素和非替换元素的区别

- src属性
> \<img> src属性缺省，在IE下，有一个默认28 * 30 px的占位内容，在Firefox下，与一般的行内元素<span>表现相同，在chrome下为0 * 0的图片
- content属性，[利用content属性实现图片变化效果实例](https://demo.cssworld.cn/4/1-4.php)
> **content生成的内容为替换元素**
    
  
  1. content生成的文本无法选中， 无法SEO
  2. 不能左右:empty伪类，（即使content有内容，标签内为空，也会为empty）

  ## content内容生成