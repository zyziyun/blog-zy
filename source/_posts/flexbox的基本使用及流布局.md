---
title: flexbox的基本使用及流布局
date: 2017-03-02 20:51:15
tags: 
    - 前端
categories: CSS
---

## 基础用法
- display: flex | inline-flex; (适用于父类容器元素上)

## 父类元素属性

### 概览
1. flex-direction 
2. flex-wrap
3. flex-flow
4. flex-flow
5. justify-content
6. align-items
7. align-content

### 详细
- flex-direction (适用于父类容器的元素上)
- flex-direction: row | row-reverse | column | column-reverse
- flex-wrap (适用于父类容器上)
- flex-wrap: nowrap | wrap | wrap-reverse
- flex-flow (适用于父类容器上)复合属性。
- flex-flow: &lt;‘flex-direction’&gt; || &lt;‘flex-wrap’&gt;
- justify-content (适用于父类容器上)
- justify-content: flex-start | flex-end | center | space-between | space-around
- 设置或检索弹性盒子元素在主轴（横轴）方向上的对齐方式
- align-items (适用于父类容器上)
- align-items: flex-start | flex-end | center | baseline | stretch
- 设置或检索弹性盒子元素在侧轴（纵轴）方向上的对齐方式。
- align-content (适用于父类容器上)
- align-content: flex-start | flex-end | center | space-between | space-around | stretch
- 设置或检索弹性盒堆叠伸缩行的对齐方式。

## 子类元素属性

### 概述
1. order - 排序
2. flex-grow
3. flex-shrink
4. flex-basis
5. flex
6. align-self

### 详解
- order (适用于弹性盒模型容器子元素)
- order: &lt;integer&gt;数值小的排在前面。可以为负值。
- flex-grow (适用于弹性盒模型容器子元素)
- 设置或检索弹性盒的扩展比率。
- flex-grow: &lt;number&gt; (default 0)
- flex-shrink (适用于弹性盒模型容器子元素)
- 设置或检索弹性盒的收缩比率（根据弹性盒子元素所设置的收缩因子作为比率来收缩空间。）
- flex-shrink: &lt;number&gt; (default 1)
- flex-basis (适用于弹性盒模型容器子元素)
- 设置或检索弹性盒伸缩基准值。
- flex-basis: &lt;length&gt; | auto (default auto)
- flex (适用于弹性盒模型子元素)
- 复合属性。设置或检索伸缩盒对象的子元素如何分配空间。如果缩写flex:1, 则其计算值为：1 1 0
- flex：none | [ flex-grow ] || [ flex-shrink ] || [ flex-basis ]
- align-self (适用于弹性盒模型子元素)
- 设置或检索弹性盒子元素自身在侧轴（纵轴）方向上的对齐方式。
- align-self: auto | flex-start | flex-end | center | baseline | stretch

## 例子
**demo github：** https://github.com/zyziyun/flexbox-demo

{% asset_img 1.jpeg %}

{% asset_img 3.jpeg %}

{% asset_img 2.jpeg %}