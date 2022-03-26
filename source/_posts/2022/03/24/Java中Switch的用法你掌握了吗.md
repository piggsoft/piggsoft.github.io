---
title: Java中switch的用法你掌握了吗?
date: 2022-03-24 21:51:07
keyword:
   - Java
   - switch
tags: Java基础
categories: Java基础
---

Java中switch的用法你掌握了吗?我们一起来温故知新吧！

<!--more-->

# Java重复判断之 `switch`

`switch` `case` 语句是用于判断一个变量与一系列值中某个值是否相等来作相应的操作，其执行效率比`if`语句要高，但相应的也有一定的限制性，它是只做等式比较，不能做其他条件筛选的。  
`switch` `case` 语句语法格式如下：

```java
switch(expression) {
   case value:
       //语句
       break;//可选
    case value:
       //语句
       break;//可选
    default://可选
        //语句
}
```

`switch` `case` 语句有如下规则：

* `switch` 语句中的expression变量类型可以是： `byte`、`short`、`int` 或者 `char`。从 Java SE 7 开始，`switch` 支持字符串 `String` 类型了，同时 `case` 标签必须为字符串常量或字面量，不能出现变量。  
* `switch` 语句可以拥有多个 `case` 语句。每个 `case` 后面跟一个要比较的值和冒号。
* `case` 语句中的值的数据类型必须与变量的数据类型相同，而且只能是常量或者字面常量。
* 当变量的值与 `case` 语句的值相等时，那么 `case` 语句之后的语句开始执行，直到 `break` 语句出现才会跳出 `switch` 语句。
* 当遇到 `break` 语句时，`switch` 语句终止。程序跳转到 `switch` 语句后面的语句执行。`case` 语句不必须要包含 `break` 语句。如果没有 `break` 语句出现，程序会继续执行下一条 `case` 语句，直到出现 `break` 语句。
* `switch` 语句可以包含一个 `default` 分支，该分支一般是 `switch` 语句的最后一个分支（可以在任何位置，但建议在最后一个）。`default` 在没有 `case` 语句的值和变量值相等的时候执行。`default` 分支不需要 `break` 语句

```java
switch(i) {
     case 0:
        System.out.println("0");
     case 1:
        System.out.println("1");
     case 2:
        System.out.println("2");
     default:
        System.out.println("default");
      }
```

## 注意事项

### `case`穿透

如果想要终止程序，必须在`case`语句中加上`break`，
比如上例中若`i`的值为`1`，则会执行到

```java
   case 1: 
      System.out.println("1");
```

语句，同时由于`case 1` 分支中没有加入`break`语句，程序会继续执行下面的所有语句而不会继续做判断，直到遇到`break`语句，结果就会打印

```java
1
2
default
```

如果想要匹配到结果后中断程序，就需要在`case`分支中加入`break`语句，修改如下：

```java
switch(i) {
     case 0:
        System.out.println("0");
        break;
     case 1:
        System.out.println("1");
        break;
     case 2:
        System.out.println("2");
        break;
     default:
        System.out.println("default");
        break;
      }

```

### `case`中定义变量

`case`分支中如果想要定义变量，必须用“{}”将分支内容给包含进去，否则，不能定义变量。  
如：

```java
case 1 :
    {
        int num = 0 ;
        System.out.println(num);
        break;
    }  

```