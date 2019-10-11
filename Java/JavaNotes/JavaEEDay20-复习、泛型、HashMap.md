# JavaEEDay20-复习、泛型、HashMap

## 一、复习：异常处理
  Java里面特别人性化的报错，报异常机制
  
  - Throwable 类 ，所有异常和错误的超类
     - Exception 异常，可以出来
     - Error  错误 没办法处理，只能避免
  
  getMessage()  toString()，展现信息；  printStackTrack()
  
  - 处理异常的方式：
      1. 捕获异常
      2. 抛出异常
      
  - 捕获异常：
      try - catch  
      try - catch - finally 
      
  - 抛出异常：
      throw  throws
      
  - 自定义异常
      class  自定义异常类 extends Exception {
          自定义异常类的有参数构造方法，参数类型是String message
      }
        
## 集合
 - Collection 集合接口 总接口
   - List 有序 可重复
      -  ArrayList    底层维护一个Object类型的数组，如果使用无参构造方法，创建ArrayList对象，Object数组默认元素个数为10；
          - 特征：
                查询快，增删慢
                ensureCapacity(int minCapacity);
                trimToSize();
      - LinkedList   底层维护的是一个链表
         - 特征：
                          查询慢，增删快
      -  Vector  线程安全的ArrayList，不经常用
    - Set 无序 不可重复
      - HashSet
                  底层维护的是一个哈希表
                  HashSet存储原理
                      hashCode equals
      - TreeSet
              树形结构 放入的数据要不有自然顺序，要不存在比较规则~~
              自定义的类对象放入TreeSet集合
            1. 遵从Comparable<T> 接口 实现compareTo(T o)
            2. 实现自定义比较器 遵从 Comparator<T> 接口 实现compare(T o1, T o2)
                  使用了匿名内部类
                   
   
Collection 方法：
    `add(E e) addAll(Collection<? extends E> c) remove(Object o) clear() 
    removeAll(Collection c) size() toArray() isEmpty() contains(Object o)
    containsAll(Collection c) retainAll(Collection c) removeAll(Collection c)
    iterator() equals() hashCode()`
      
迭代器方法：
     `hasNext() next() remove()`
  
List: 
    `add(int index, E e) addAll(int index, List<? extends > list)
    indexOf(Object o) lashIndexOf(Object o) get(int index)
    set(int index , E e)
    subList(int fromIndex, int toIndex)
    ListIterator() `  
    
   ListIterator()特有方法：
       `add(E e) set(E e)`
   
   
Set:
      没有特有方法

## 泛型
解决的问题是：
1. 数据类型一致化问题
2. 避免无意义的强制类型转换

泛型格式：
一. 函数/方法中使用泛型
```java
权限修饰符 <自定义泛型占位符> 返回值类型(可以使用自定义泛型)  方法名 (形式参数列表“也可以使用泛型”) {
    在函数体内，同样可以使用泛型
}
```

二. 类中使用泛型
```java
class 类名<自定义泛型占位符> {    
    成员变量    
    非静态成员方法可以使用类自己的泛型    
    静态成员方法不能使用类内的泛型，只能在方法中自定义声明泛型
}
```

三. 接口中使用泛型
```java
interface A<自定义泛型占位符> {
    成员变量; public static final 定义时必须初始化
    成员方法; abstract
}
```
使用的方式：
1>
```java
class 类名<同接口一致的泛型> implements A<自定义泛型> {        
    }
```
    在创建当前类对象时确定具体数据类型：
        例如：ArrayList<E> HashMap<K, V>


2> 
```java
 class 类名 implements A<确定数据类型> {       
    }
```
    泛型的具体数据类型，在创建定义当前类的时候，就确定了
        例如：Comparable接口，实现compareto方法
        
泛型的上下限
<? super E>
<? extends E>
    
## Map 双边队列 双列集合
    Map 
    ---| HashMap
    ---| TreeMap
    方法：
    put(K key, V value);
    putAll(Map<? extends K, ? entends V> map);
    
    clear();
    remove(Object key); //根据 Key 删去对应的键值对
    
    size();
    get(Object key)
    containsKey(Object Key);
    containsValue(Object Value);
    keySet();
    values();
    
    entrySet()
    
    entrySet的使用 【重点】  
```java
import java.util.Map.Entry;
        
    HashMap<String, Integer> map = new HashMap<String, Integer>();
    
    Set<Entry<String, Integer>> set = map.entrySet();
    
    Iterator<Entry<String, Integer>> it = set.iterator();
    
    while (it.hashNext()) {
        System.out.println(it.next());
    }

```
   
## 文件操作
    File 类对象
    File(String pathName);
    File(String parent, String child);
    File(File parent, String child);
    
    createNewFile(); boolean 
    mkdir();
    mkdirs();
    renameTo(File dest);
    
    delete();
    deleteOnExit();
    
    getPath();
    getName();
    getAbsolute();
    getParent();
    
    lastModify();
    length(); 
    
    exists();
    isFile();
    isDirectory();
    isHidden();
    isAbsolute();
    
    static File[] listRoots(); 
    String[] list();
    File[] listFile();
    
    FileNameFilter();
    accpet(File dir, String name);



