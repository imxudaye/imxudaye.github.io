---
title: 一次关于Java8中的Stream测验
date: 2019-03-19 15:27:56
categories:
- 后端
tags:
- Stream
- Java
- 流式迭代
---

## 一次关于Java8中的Stream测验

### 	前言：

> ​	在业务代码当中，常常会有多个List或者Map之间的交集差集并集需求，在Java8流式stream出现之前，往往都是以嵌套循环达到目的。此文记录测试在不同写法之间，在串行流与并行流操作之间的性能差异

**先上代码：**

```java
// 定义对象
class Student {
    private String name;
    private String sex;
}
```

```java
// 初始化两个size为10万的对象集合
int num = 100000;
//男士集合
List<Student> manList = new ArrayList<Student>(){{
  for (int i = 0; i < num; i++) {
    add(new Student("旺财"+ ((int) (Math.random() * num)), "男"+i));
  }
}};
//女士集合
List<Student> womanList = new ArrayList<Student>(){{
  for (int i = 0; i < num; i++) {
    add(new Student("旺财"+ ((int) (Math.random() * num)), "女"+i));
  }
}};
```

### 测试需求

> ##### 查找出有与男同学同名的所有女同学

#### 	-- 非嵌套分步写法

```java

// 第一步 把男士集合名字提取出来
List<String> manNameList=manList.stream().map(Dog::getName).collect(Collectors.toList());
// 第二步 采用filter过滤名称重复
List<Student> result 
 			 = womanList.stream().filter(s -> manNameList.contains(s.getName())).collect(Collectors.toList());

```

#### -- 错误嵌套写法

```java
List<Student> result = womanList.stream().filter( s -> !(manList.stream().map(Student::getName).collect(Collectors.toList())).contains(s.getName())).collect(Collectors.toList());
```



#### 	-- 正确嵌套写法

```java
List<Student> result = womanList.stream().filter(s -> manList.stream().filter(f -> 	f.getName().equals(s.getName())).findAny().isPresent()).collect(Collectors.toList());
```

### 测试代码

```java
 public static void main(String[] args) {
        // 初始化两个size为10万的对象集合
         int num = 100000;
        //男士集合
         List<Student> manList = new ArrayList<Student>(){{
             for (int i = 0; i < num; i++) {
                 add(new Student("旺财"+((int) (Math.random() * num) "男"+i));
             }
         }};
        //女士集合
         List<Student> womanList = new ArrayList<Student>(){{
             for (int i = 0; i < num; i++) {
                 add(new Student("旺财"+ ((int) (Math.random() * num)), "女"+i));
             }
         }};

         //非嵌套流式分步写法
         long l = System.currentTimeMillis();
         // 第一步 把男士集合名字提取出来
         List<String> manNameList=manList.stream().map(Student::getName).collect(Collectors.toList());
        // 第二步 采用filter过滤名称重复
         List<Student> result
                 = womanList.stream().filter(s -> manNameList.contains(s.getName())).collect(Collectors.toList());
         long l2 = System.currentTimeMillis();
         System.out.println(String.format("非嵌套流式分步写法 结果：%s  完成time : %s " , result.size(),  (l2-l)));

        //错误嵌套流式迭代
         List<Student> stuList11 = womanList
                 .stream()
                    .filter( s -> !(manList.stream().map(Student::getName).collect(Collectors.toList())).contains(s.getName()))
                        .collect(Collectors.toList());
         long l3 = System.currentTimeMillis();
         System.out.println(String.format("错误嵌套流式迭代 结果：%s  完成time : %s " , stuList11.size(),  (l3-l2)));


         //正确嵌套流式迭代
         List<Student> stuList2 = womanList.stream().filter( s -> !manList.stream().filter(f -> f.getName().equals(s.getName())).findAny().isPresent()).collect(Collectors.toList());
         long l4 = System.currentTimeMillis();
         System.out.println(String.format("正确嵌套流式迭代 结果：%s  完成time : %s " , stuList2.size(),  (l4-l3)));

    }
```

### 测试结果

#### 串行流执行结果

|   执行方式   | 执行结果（数量） | 执行时间 （毫秒） |
| :----------: | :--------------: | :---------------: |
| 分步拆分写法 |      36726       |       56865       |
| 错误嵌套写法 |      36726       |      175816       |
| 正确嵌套写法 |      36726       |       93490       |



#### 并行流执行结果



|   执行方式   | 执行结果（数量） | 执行时间 （毫秒） |
| :----------: | :--------------: | :---------------: |
| 分步拆分写法 |      37041       |       16105       |
| 错误嵌套写法 |      37041       |       66748       |
| 正确嵌套写法 |      37041       |       35042       |



结果综述：

​	可以看到

​		执行性能上

1. 非嵌套的流式是要明显好于嵌套写法的
2. 盲目的嵌套写法带来的额外性能消耗是巨大的，代码习惯需注意  ⚠️
3. 并行流操作在性能上是远远大于串行流操作的
   

注：由于并行流使用多线程，则一切线程安全问题都应该是需要考虑的问题，如：资源竞争、死锁、事务、可见性等等