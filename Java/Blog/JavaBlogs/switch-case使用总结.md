# switch-case使用总结


## 1.switch-case注意事项：

```java
 switch(A){
    case B:
        // 处理方式B
        break;
    case 1:
        // 处理方式C
        break;
    ...
    
    default:
        // 默认处理方式
        break;
    }
```

- switch(A),括号中**A的取值只能是整型或者可以转换为整型的数值类型**，比如 byte、short、int、char、还有枚举；需要强调的是：**long和String类型是不能作用在switch语句上的**。

- case B：；case 是常量表达式，也就是说**B的取值只能是常量**（需要定义一个 final 型的常量,后面会详细介绍原因）或者 int、byte、short、char（比如 1、2、3、200000000000（注意了这是整型）），如果你需要在此处写**一个表达式或者变量，那么就要加上单引号**； 
- 处理方式 B： case 后的语句可以不用大括号，就是处理方式语句不需要用大括号包裹着；

- default 就是如果没有符合的 case 就执行它,default 并不是必须的；


## 2.案例分析：

1.标准型(case 后面都有 break 语句，case 后的值都是整数)

```
int i=3; 
switch(i) { 
    case 1: 
        System.out.println(1); 
        break;  
    case 2: 
        System.out.println(2);          
        break;   
    default: 
        System.out.println("default"); 
        break;  
} 
```

2.常量型(case 后面都有 break 语句，case 后的值都是常量)

```
// 下面两个值必须声明为final类型
final int NUM1=1;
final int NUM2=2;
int i=3;
switch(i){
    case NUM1:
        System.out.println(1);
        break;
    case NUM2:
        System.out.println(2);
        break;
    default:
        System.out.println("default");
        break;
}
```

3.表达式型(case 后面都有 break 语句，case 后的值都是表达式)

```
int i = 3;
int b = 2;
switch (i) {
    case '类名.getId()':
        System.out.println(1);
        break;
    case 'b':
        System.out.println(2);
        break;
    default:
        System.out.println("default");
        break;

}
```

