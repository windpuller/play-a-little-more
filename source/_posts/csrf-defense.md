title: csrf defense
date: 2015-11-22 14:39:32
tags: csrf
---
*本文仅用来自己备忘，更详细内容请[阅读原文](https://www.ibm.com/developerworks/cn/web/1102_niugang_csrf/)。*

#### 防止csrf攻击的常用方法
##### 1. HTTP Reference
**原理**：检查HTTP Reference的地址，判定请求的来源。
**缺点**：低版本浏览器（IE6等）可以篡改HTTP Reference，达不到防御效果；较新的浏览器  
允许用户关闭HTTP Reference的使用，致使用户的正常请求也不能通过。

##### 2. url中添加token参数
**原理**：检查请求url中的token值是否是合法的。
**缺点**：所有请求地址中添加token字段，实现起来较复杂；url中的token很容易被攻击者获取到。

##### 3. HTTP协议头部添加自定义属性
**原理**：检查HTTP头部的csrfToken字段值是否合法。
**缺点**：要使用XMLHttpRequest一次性给该类的所有请求添加csrfToken字段，而XMLHttpRequest
的使用局限性很大，并非所有的请求都适合由XMLHttpRequest发起，使用XMLHttpRequest请求到
的页面不能被浏览器记录，给用户带来不便。

#### 推荐的防御方式
2+3, 对于安全性要求没那么高的请求，可以采用在url中添加token参数的方法防御  
csrf，对于安全性要求比较高的请求，则使用在HTTP协议头部添加csrfToken的方式。