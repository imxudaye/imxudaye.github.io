---
title: ES 操作简单总结
date: 2020-05-26 14:12:57
categories:
    - 后端
tags:
    - elasticsearch
---

# ES 操作简单总结

## 一 简述

> 此篇总结基于在当前最新版本(7.15.2)
>
> ES的result API 分为不同的请求类型对应不同操作

### PUT 

​	/index_name 创建/修改索引

​	/index_name/_doc/{id} 创建文档  —创建不指定id,则默认随机

POST

​	/index_name/_doc/{id} 创建/修改文档

DELETE 

​	删除索引或者文档

GET

​	查询/获取文档(指定id)