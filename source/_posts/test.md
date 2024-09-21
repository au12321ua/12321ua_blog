---
title: test
date: 2024-08-28 13:46:48
tags: [test]
excerpt : "测试帖子，用来测试博客的基本功能。"
categories: ["others"]
---

这是个测试帖子，用来测试博客的基本功能。

## 1.笔记功能

语法如下：

```
{% notel [颜色] [可选: 自定义图标] [标题] %}
内容
支持换行
{% endnotel %}
```

例如：

{% notel default fa-info 信息 %}
换行测试
换行测试
换行测试
{% endnotel %}

{% notel blue 提示 %}
换行测试
换行测试
换行测试
{% endnotel %}

## 2. 按钮功能

语法如下：

```
{% btn [颜色] [可选: 自定义图标] [标题] [链接] %}
# 不同板块除了btn，中间可用::分隔
```

{% btn regular::示例博客::https://12321ua.cn ::fa-solid fa-play-circle %}

## 3. Folding 折叠模块

语法如下：

```
{% folding [颜色]::[标题] %}
内容
支持换行
{% endfolding %}
```

例如：

{% folding 发疯，不要点开啊！！！！ %}
啊啊啊啊啊啊啊啊！！！！！！
{% endfolding %}

颜色列表：

```
yellow, blue, green, red, orange, pink, cyan, white, black, gray
```

## 4. 选项卡

语法如下：

```
{% tabs 页面内不重复的ID %}
<!-- tab 栏目1名称 -->
内容
<!-- endtab -->
<!-- tab 栏目2名称 -->
内容
<!-- endtab -->
{% endtabs %}
```

其中，页面内不重复的ID 为你为这个选项卡创建的唯一标识符，可以随便取。

每个栏目内容使用 <!-- tab 栏目名称 --> 和 <!-- endtab --> 来定义。

比如：

{% tabs First unique name %}
<!-- tab First Tab-->
**This is Tab 1.**
<!-- endtab -->
 
<!-- tab Second Tab-->
**This is Tab 2.**
 
This is Tab 2.
<!-- endtab -->
 
<!-- tab Third Tab-->
**This is Tab 3.**
 
This is Tab 3.
 
This is Tab 3.
<!-- endtab -->
{% endtabs %}