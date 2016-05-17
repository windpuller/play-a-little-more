title: RESTful note
tags: restful
date: 2016-05-17 18:14:51
---

## 1. RESTful
RESTful，即Representational State Transfer的缩写，字面含义是**表现层状态转化**。是某歪果仁博士提出的一种**网络应用软件架构**风格。  
根据[理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)的分析，RESTful的关键因素主要有三个：资源、表现层、状态转化。
### 资源
表现层依赖的实体，在网络中一般使用URI指定。可以是一张图片，也可以是一段文本。
### 表现层
资源的表现形式，是图片还是文本还是视频，应使用HTTP协议头部的Accept和Content-Type字段指定。
### 状态转化
client通过HTTP协议的GET、POST、PUT、DELETE等操作获取、创建、修改、删除服务器端的资源，并反映在client的展示中。
## 2. 优势
RESTful的优势主要体现在与传统C/S架构应用软件的对比上：
> * 浏览器即客户端，客户端的开发难度和成本降低
> * 仅在服务器端存在状态转化，有利于系统结构的拆分和扩展，有利于合理配置服务器资源
> * 系统更新升级方便，向后兼容性好