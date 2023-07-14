---
tags : 
- java基础

flag: blue
---




# JavaDay 08 数组的自增和面向对象

## 一、数组的自增

1. 数组的增长是元素个数的增长，源数据不能破坏
2. 自增是当数组添加元素的时候自动调用的

  public static void grow(int[] array)
    
## 二、面向对象

## （一）类和对象

类 |对象 
---|--- 
人      |   马云，王宝强，马化腾，乔布斯 Jobs，雷布斯
电脑     |   樊晓晨的T400 刘晓磊的MacBook Pro 江总的Alienware
汽车     |   海洋的君越，海马M6，磊哥的五菱之光S 
手机     |   王石的8848 晓晨华为Mate9 磊哥的MOTO E6

-  类：
      是某些事物的统称，会包含这些事物的属性和行为
- 对象：
      唯一，独立，特殊的个体

## （二）在 Java 中用代码实现类和对象

 在Java中定义类的格式：
自定义类：

```java
class 类名 {         
    事物共有属性的描述; 【成员变量】          
    事物共有行为的描述; 【成员方法】          
}
```

【注意事项】
1. 类名要求是大驼峰命名法，要符合标识符规则
2. 类只是描述了同一类事物的共有属性和行为，但是不能确定属性的具体内容
3. 属性的描述称之为【成员变量】，行为的描述称之为【成员方法】

以下是自定义类的案例
```java
class Person {
    // 以下是【属性描述】
    // 名字的描述
    String name; //String 字符串类型

    // 年龄的描述
    int age; 

    // 体重描述
    float weight;

    // 以下是【行为描述】
    // 吃饭的行为描述
    public void eat() {
        System.out.println("吃饭~~~");
    }

    // 睡觉的行为描述
    public void sleep() {
        System.out.println("睡觉~~~");
    }

    // 读书的行为描述
    public void dadoudou() {
        System.out.println("读书");
    }
}

```
