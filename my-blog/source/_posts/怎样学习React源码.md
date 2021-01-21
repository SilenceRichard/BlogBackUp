---
title: 怎样学习React源码
date: 2020-12-28 20:23:23
tags:
  - 读书笔记
  - react
---

## 你在第几层

- 掌握术语、基本实现思路
- 掌握整体工作流程、局部细节
- 掌握关键流程的细节
- 掌握思想
- ？

## 掌握思想

 推荐阅读[build your own react](https://pomb.us/build-your-own-react/)

 写出`mini react`

## 整体工作流程、局部细节

- 整体流程

  推荐阅读[react技术揭秘]()

    1. schedule 调度 scheduler
    2. render 协调  reconciler
    3. commit 渲染  renderer（改变视图）
  
- 局部细节

    1. diff算法
    2. hook

## 掌握关键流程的细节

  > 探索前端的边界

  1. scheduler 调度 小顶堆
  2. reconciler - fiber dfs
  3. renderer

## 掌握思想

  > 践行代数效应