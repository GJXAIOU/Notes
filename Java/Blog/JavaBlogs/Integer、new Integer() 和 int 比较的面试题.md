---
style : summer
---

# Integer、new Integer() 和 int 比较的面试题


##  一、基本概念的区分：

1、Integer 是 int 的包装类，int 则是 java 的一种基本数据类型 
2、Integer 变量必须实例化后才能使用，而int变量不需要 
3、Integer 实际是对象的引用，当new一个 Integer时，实际上是生成一个指针指向此对象；而 int 则是直接存储数据值 
4、Integer的默认值是null，int的默认值是0;

## 二、Integer、new Integer() 和 int 的比较

### （一）两个 new Integer() 变量比较 ，永远是 false
因为new生成的是两个对象，其内存地址不同
```java
Integer i = new  Integer(100);
Integer j = new  Integer(100);
System.out.print(i == j); //false
```

### （二）Integer变量 和 new Integer() 变量比较 ，永远为 false
因为 Integer变量 指向的是 java **常量池** 中的对象， 而 new Integer() 的变量指向**堆中** 新建的对象，两者在内存中的地址不同。
```java
Integer i = new  Integer(100);
Integer j = 100;
System.out.print(i == j); //false
```

### （三）两个Integer 变量比较，如果两个变量的值在区间 -128 到 127 之间，则比较结果为 true，如果两个变量的值不在此区间，则比较结果为 false。
```java
Integer i = 100;
Integer j = 100;
System.out.print(i == j); //true
Integer i = 128;
Integer j = 128;
System.out.print(i == j); //false
```

分析：**Integer i = 100 在编译时，会翻译成为 Integer i = Integer.valueOf(100)**，而 java 对 Integer类型的 valueOf 的定义如下：
```java
public  static  Integer valueOf(int i){
    assert  IntegerCache.high >= 127;

    if (i >= IntegerCache.low && i <= IntegerCache.high){
        return  IntegerCache.cache[i + (-IntegerCache.low)];
}
    return  new  Integer(i);
}
```
java对于-128到127之间的数，会进行缓存。所以 Integer i = 127 时，会将127进行缓存，下次再写Integer j = 127时，就会直接从缓存中取，就不会new了。

### （四）int 变量 与 Integer、 new Integer() 比较时，只要两个的值是相等，则为true 
因为包装类Integer 和 基本数据类型int 比较时，java会自动拆包装为int ，然后进行比较，实际上就变为两个int变量的比较。
```java
//自动拆箱为 int i=100; 此时，相当于两个int的比较
Integer i = new  Integer(100); 
int j = 100；
System.out.print(i == j); //true
```
示例1：
```java
public  class  IntegerDemo {
    public  static  void main(String[] args) {
        int i = 128;
        Integer i2 = 128;
        Integer i3 = new  Integer(128);
        // Integer会自动拆箱为int，所以为true
        System.out.println("i == i2 = " + (i == i2)); 
        System.out.println("i == i3 = " + (i == i3)); // true，理由同上
        
        // 编译时被翻译成：Integer i4 = Integer.valueOf(127);
        Integer i4 = 127;
        Integer i5 = 127;
        System.out.println("i4 == i5 = " + (i4 == i5));// true

        Integer i6 = 128;
        Integer i7 = 128;
        System.out.println("i6 == i7 = " + (i6 == i7));// false

        Integer i8 = new  Integer(127);
        System.out.println("i5 == i8 = " + (i5 == i8)); // false

        Integer i9 = new  Integer(128);
        Integer i10 = new  Integer(128);
        System.out.println("i9 == i10 = " + (i9 == i10)); // false
    }
}
```
答案是
```java
i == i2 = true
i == i3 = true
i4 == i5 = true
i6 == i7 = false
i5 == i8 = false
i9 == i10 = false
```
示例2：
```java
package demo1.demo1;
 public  class  Campare {
    public  static  void main(String[] args) {
        Integer a = new  Integer(127), b = new  Integer(128);
        int c = 127, d = 128, dd = 128;
        Integer e = 127, ee = 127, f = 128, ff = 128;
        // false 因为a,b都是new出来的对象，地址不同所以为false
        System.out.println(a == b); 
        // true a会自动拆箱为int类型
        System.out.println(a == c); 
        // false 指向的地址不同a是new出来的
        System.out.println(a == e); 
         // true e会自动拆箱为int类型
        System.out.println(e == c);
        // true Integer对 处于-128到127范围之间，指向了同一片地址区域
        System.out.println(e == ee);
         // false 指向的地址不同b是new出来的
        System.out.println(b == f);
        // true f自动拆箱为int类型
        System.out.println(f == d); 
        System.out.println(f == ff); 
/*
false 指向的不是同一片地址区域。
* 在Integer类型中，-128到127存放的是同一片区域地址，
* 之外的数是另外开辟空间来进行 存储的`
*/
  System.out.println(d == dd); // true 不解释
    }
}
```
示例3：
```java
Integer i01 = 59;
int i02 = 59;
Integer i03 =Integer.valueOf(59);
Integer i04 = new  Integer(59);
```
以下输出结果为false的是：
```java
System.out.println(i01== i02);
System.out.println(i01== i03);
System.out.println(i03== i04);
System.out.println(i02== i04);
```
解析：
*   i01 == i02 。i01.intValue()i02 两个值的比较5959 -->true;
*   i01 == i03 。由于 59在-128到127之间，所以，i01和i03的赋值操作返回的是同一个对象。都是从chche中返回的同一个对象，对象地址相同 true;
*   i03 == i04。i03是来自缓存值，i04是新new的对象 ，二者不是同一个对象，所以false。
*   i02 == i04。和第一个类似，true。
答案是 C 。

示例4：与示例3的唯一不同，就是将值全部改成128。
```java
Integer i01 = 128;
int i02 = 128;
Integer i03 = Integer.valueOf(128);
Integer i04 = new  Integer(128);
```
以下输出结果为false的是：
```java
System.out.println(i01 == i02);
System.out.println(i01 == i03);
System.out.println(i03 == i04);
System.out.println(i02 == i04);
```
答案：
```java
true
false
false
true
```
