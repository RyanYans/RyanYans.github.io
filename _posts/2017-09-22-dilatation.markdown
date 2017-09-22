---
layout:     post
title:      " 谈谈ArrayList、HashSet、HashMap的扩容机制 "
subtitle:   " \" 无独有偶，谈谈ArrayList、HashSet、HashMap的扩容机制...\""
date:       2017-09-22 00:30:00
author:     "RyanYans"
header-img: "img/post-bg-dilation.png"
catalog: true
tags:
    - Java
    - Collection
    - 扩容
---


本文谈谈ArrayList、HashSet、HashMap的扩容机制理解。


## 基本概念

初始大小:调用无参构造函数时默认的容量
加载因子:超过 (当前容量*加载因子) 时会进行扩容
扩容因子:扩容时增加的容量为 (当前容量*扩容因子)


## 扩容参数

容器	 		初始容量		加载因子		扩容因子
ArrayList:   		10				     1			   0.5
HashMap:		16				    0.75			    1
HashSet:		16				    0.75			    1


----


一般而言, 以哈希表 (链表+数组) 作为底层数据结构的容器, 当元素超过加载因子时会进行扩容,如HashMap,HashSet
以数组作为底层数据结构的容器, 当元素超过数组大小时会进行扩容,如ArrayList
以链表作为底层数据结构的容器, 容量没有限制, 不会进行扩容,如LinkedList,TreeMap


---

扩容比较消耗性能,在能预料到数据量大小时, 应给定稍大于预期数据量的初始容量参数.
ArrayList可接受一个int参数作为初始容量.
HashMap可接受一个int参数作为初始容量和一个float参数作为扩容参数.
HashSet同HashMap



----------  

##### 学习笔记，如有谬误，敬请指正。