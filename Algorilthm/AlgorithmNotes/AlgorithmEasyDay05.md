# AlgorithmEasyDay05


## 一、哈希函数和哈希表
哈希函数的概念是：将任意大小的数据转换成特定大小的数据的函数，转换后的数据称为哈希值或哈希编码。下面是一幅示意图：

![](https://images2015.cnblogs.com/blog/1030776/201701/1030776-20170106142012816-1867044021.png)

可以明显的看到，原始数据经过哈希函数的映射后称为了一个个的哈希编码，数据得到压缩。哈希函数是实现哈希表和布隆过滤器的基础。
### （一）哈希函数
![输入图片描述](AlgorithmEasyDay05_md_files/%E5%93%88%E5%B8%8C%E5%87%BD%E6%95%B0_20191227095740.png?v=1&type=image&token=V1:uwkxbbjwqvE2w2E7-NbrpGnPqFfe_T1gg3yPdunSQU0)
- 哈希函数的特征
	- 输入域是无穷大的，输出域是有穷的；
	- 确定的输入值对应确定的输出值，即相同的输入值多次经过同一个哈希函数结果相同；
	- 不同的输入可能对应相同的输出（哈希碰撞）；因为无限大的输入域通过哈希函数映射到有限大的输出 S 域，必然存在多个输入映射到一个输出；
	- 哈希函数的离散型：对于一定量的输入，对应的输出值在 S 域是均匀分布的；
	如输入是 0 ~ 99（当然输入是无限的，这里只是示例），对应输出域为 0，1，2。则对应出现三个值的次数是**大致** 相同的。
	- 输入值是有规律的，不代表输出值是有规律的；
如输入：a1,a2,a3...但是对应的输出值可能差距很大；

**特征推广**
 如果输出值在 S 域上是均匀分布的，则 S 域上所有值 %m 在 0 ~ m-1上为均匀分布；


**哈希码**
输入值经过哈希函数之后得到哈希码，哈希码上每一位都是（`0~9，a~f`）之间的某个值，并且该值在这个范围内均匀分布，同时哈希码的每一位之间都是相互独立的。

![输入图片描述](AlgorithmEasyDay05_md_files/%E5%93%88%E5%B8%8C%E5%80%BC_20191227100917.png?v=1&type=image&token=V1:Kd_h-5I4y42A0XH4nRS5GWjRvuDw1Rk6EpkCW-SnQOY)

### （二）问：实现一千个相互独立的哈希函数
- 方案一：使用一个哈希函数分开作为两个种子
因为哈希值为 N 个 16 位数的一串值，可以将 16 位数分为前 8 和后 8 位，分别作为 h1 和 h2 种子，即得到两个不相关的哈希函数（前后两部分相互独立），然后得到其他剩余的哈希函数： h3  = h1 + 1 * h2， h4 = h1 + 2 * h2，。。。。
- 方案二：使用两个哈希函数作为两个种子
即使用两个哈希函数作为种子，然后不断使用上面的公式得到其他的哈希函数。

### （三）问：对于一个 10 T 大小的文件，每行为无序的字符串，打印里面所有重复的字符串
**解释**：首先文件中每行数据之间是无序的，然后重复的规则可能是多种。
**条件**：允许 1000 台机器（编号为 0 ~ 999），文件存放在分布式系统上；
**过程**：将每行读出来，然后计算每行字符串的哈希值，然后将该值 % 1000（结果值为 0 ~ 999 之一），对应的将这条字符串放入计算出来的编号位置上。所以如果字符串相同，则其哈希值相同，最终会落在同一个机器上。
虽然不同的字符串得到的哈希值可能相同，但是现在将所有的数据分布在 1000 台机器上进行独立处理。


## 二、哈希表
### （一）经典的哈希表结构
经典的哈希表是使用**数组 + 链表** 结构实现的，常用的方法有：`put(key,value)`， `get(key)`， `remove(key)`如图所示：
![输入图片描述](AlgorithmEasyDay05_md_files/%E7%BB%8F%E5%85%B8%E5%93%88%E5%B8%8C%E8%A1%A8%E7%BB%93%E6%9E%84_20191227102458.png?v=1&type=image&token=V1:BIuaD34iWguwf2iPCn014lMmHbOCIrp38zf6mjDwDUI)
首先会分配一个数组空间（这里示例长度为 17），然后使用 `put(key1,value1)`时候，首先计算 key1 的哈希值得到 Code1，然后将 Code1 % 17（得到 0 ~ 16 之间一个值），然后将这对 key-vlaue 值挂在计算得到的值对应的位置后面（例如计算得到 5，就把该键值对挂载到下标为 5 的数组后面），其他所有的键值对都使用类似的方法。因为不同或者相同的 key 计算可能得到相同的哈希值，从而 % 17 的时候得到相同的数组下标，所以挂载之前看看该位置上是否已挂载其它键值对，如果有，则比较挂载的 key 值有无和当前的 key 值是否相同，如果相同则将原来键值对的 value 值该为当前键值对的 value 值，如果两个 key 值不相同则直接将该键值对挂载在原来链表后面即可。

- 根据哈希函数的性质，**每个位置后面挂载的链表数目应该是大致相同的**，即大致各位置后链表数目增长速度相同。

### （二）哈希表扩容
==哈希表有很多优化方式，因此默认增删改查操作的时间复杂度为 O（1）==

- 扩容方式
	- 在线扩容：用户线程需要暂停
	- 离线扩容：扩容和用户操作同步进行

**在线扩容**
即原来数组长度为 17，数据量过大的时候后面挂载的链表数目就会很多，会影响操作效率，这里假设当后面链表节点数超过 5 即需要扩容至 104,。
扩容时候需要将原有哈希值重新经历哈希函数然后放置到新的数组对应位置上。

**离线扩容**
当后面节点数到 5 的时候启动离线扩容（当前结构仍然可以使用，仅仅是效率有点下降），这时候在后台同时创建一个长度为 104 的数组，当用户使用 get 方法获取值时从原来结构中获取，当使用 put 方法放入值得时候，该键值对同时放入原来结构和扩容后的结构，扩容完成之后只需要将用户的指向该为扩容之后的结构即可。

```java
package nowcoder.easy.day05;  
  
import java.util.ArrayList;  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map.Entry;  
  
public class Code_01_HashMap {  
  
   public static void main(String[] args) {  
      HashMap<String, String> map = new HashMap<>();  
      map.put("zhangsan", "20");  
  
      System.out.println(map.containsKey("zhangsan"));  
      System.out.println(map.containsKey("lisi"));  
      System.out.println("=========================");  
  
      System.out.println(map.get("zhangsan"));  
      System.out.println(map.get("lisi"));  
      System.out.println("=========================");  
  
      System.out.println(map.isEmpty());  
      System.out.println(map.size());  
      System.out.println("=========================");  
  
      System.out.println(map.remove("zhangsan"));  
      System.out.println(map.containsKey("zhangsan"));  
      System.out.println(map.get("zhangsan"));  
      System.out.println(map.isEmpty());  
      System.out.println(map.size());  
      System.out.println("=========================");  
  
      map.put("zhangsan", "31");  
      System.out.println(map.get("zhangsan"));  
      map.put("zhangsan", "32");  
      System.out.println(map.get("zhangsan"));  
      System.out.println("=========================");  
  
      map.put("zhangsan", "31");  
      map.put("lisi", "32");  
      map.put("wangwu", "33");  
  
      for (String key : map.keySet()) {  
         System.out.println(key);  
      }  
      System.out.println("=========================");  
  
      for (String values : map.values()) {  
         System.out.println(values);  
      }  
      System.out.println("=========================");  
  
      map.clear();  
      map.put("A", "1");  
      map.put("B", "2");  
      map.put("C", "3");  
      map.put("D", "1");  
      map.put("E", "2");  
      map.put("F", "3");  
      map.put("G", "1");  
      map.put("H", "2");  
      map.put("I", "3");  
      for (Entry<String, String> entry : map.entrySet()) {  
         String key = entry.getKey();  
         String value = entry.getValue();  
         System.out.println(key + "," + value);  
      }  
      System.out.println("=========================");  
  
      // you can not remove item in map when you use the iterator of map  
//     for(Entry<String,String> entry : map.entrySet()){  
//        if(!entry.getValue().equals("1")){  
//           map.remove(entry.getKey());  
//        }  
//     }  
  
 // if you want to remove items, collect them first, then remove them by // this way.  List<String> removeKeys = new ArrayList<String>();  
      for (Entry<String, String> entry : map.entrySet()) {  
         if (!entry.getValue().equals("1")) {  
            removeKeys.add(entry.getKey());  
         }  
      }  
      for (String removeKey : removeKeys) {  
         map.remove(removeKey);  
      }  
      for (Entry<String, String> entry : map.entrySet()) {  
         String key = entry.getKey();  
         String value = entry.getValue();  
         System.out.println(key + "," + value);  
      }  
      System.out.println("=========================");  
  
   }  
  
}
```
输出结果为：
```java
true
false
=========================
20
null
=========================
false
1
=========================
20
false
null
true
0
=========================
31
32
=========================
lisi
zhangsan
wangwu
=========================
32
31
33
=========================
A,1
B,2
C,3
D,1
E,2
F,3
G,1
H,2
I,3
=========================
A,1
D,1
G,1
=========================
```


### （三）问：设计 RandomPool 结构

【题目】 设计一种结构，在该结构中有如下三个功能：
- insert(key)：将某个key加入到该结构，做到不重复加入。
- delete(key)：将原本在结构中的某个key移除。 
- getRandom()：等概率随机返回结构中的任何一个key。

【要求】 Insert、delete和getRandom方法的时间复杂度都是O(1)

**思路**：因为如果使用一个哈希表则绝对不能保证 getRandom 方法中的严格等概率，因为输入值经过哈希函数之后的输出值仅仅是大致等概率均匀分布（均匀性只是在大样本情况下的近似均匀），因此需要使用两个哈希表；
两个哈希表结构为：

![输入图片描述](AlgorithmEasyDay05_md_files/RandomPool%20%E7%BB%93%E6%9E%84_20191227114346.png?v=1&type=image&token=V1:DbdqtAkHRk-HqcPZvb9GjaIqMAr5yHgSY34K2TeQutg)

针对 put 方法，针对第一个键值对（str0,0），数据放入哈希表之后（具体的存放位置不一定是图示的位置（因为经过哈希计算之后是离散的），正因为离散型，所以实际上放置的位置无关紧要，这里仅仅表示该哈希表有这个键值对即可，同时在另一个哈希表中存入对应的键值对（0，str0），每存入一个值 size 值 + 1。
**如果不考虑 remove 方法**，则使用下面代码即可保证 getRandom 方法等概率返回结构中任意一个 key。

```java
package nowcoder.easy.day05;  
  
import java.util.HashMap;  
  
public class RandomPoolWithoutRemove {  
    public HashMap<String, Integer> map1;  
    public HashMap<Integer, String> map2;  
    public int size;  
  
    public RandomPoolWithoutRemove() {  
        map1 = new HashMap<>();  
        map2 = new HashMap<>();  
        size = 0;  
    }  
  
    public void add(String str) {  
        // 这里只是巧合：value 值正好为 size 大小；  
  map1.put(str, size);  
        map2.put(size, str);  
        size++;  
    }  
  
    public String getRandom() {  
        if (size == 0) {  
            return null;  
        }  
        int index = (int) (Math.random() * size);  
        return map2.get(index);  
    }  
}
```
**考虑到 remove 方法**：因为使用 remove 方法，会在整个数组中形成空位置，所以如果使用 getRandom 方法可能会获取到空位置，当空位置很多时候就得不断的进行重新计算使得时间复杂度不是 O(1)。
默认删除方式：例如这里删除 key 为 str17 的记录，则首先在哈希表1 中找到 key 为 str17，删除该键值对，同时根据其对应的 value 值作为 key  值查询哈希表 2 中对应的键值对，然后删除该记录。 
**为了保证 getRandom 查询严格等概率并且时间复杂度为 O（1）**： **最后一个元素填洞**。
在删除哈希表中 str17 之前，将表中最后一个位置的 str999 放到 str17 位置，然后同样将哈希表 1 中该 key 对应的 value 对应的哈希表 2 中的 key = 17 的 value 值也设置为最后一个值 str999，然后删除两张表中的最后一条记录，同时将哈希表的长度 size 改为 999。
最后使用 Math.Random() * size 获取 0 ~ 998 之间的随机值，最后返回哈希表 2 中对应值的 value 值即可。
```java
package nowcoder.easy.day05;  
  
import java.util.HashMap;  
  
public class Code_02_RandomPool {  
  
    public static class Pool<K> {  
        private HashMap<K, Integer> keyIndexMap;  
        private HashMap<Integer, K> indexKeyMap;  
        private int size;  
  
        public Pool() {  
            this.keyIndexMap = new HashMap<K, Integer>();  
            this.indexKeyMap = new HashMap<Integer, K>();  
            this.size = 0;  
        }  
  
        public void insert(K key) {  
            if (!this.keyIndexMap.containsKey(key)) {  
                this.keyIndexMap.put(key, this.size);  
                this.indexKeyMap.put(this.size++, key);  
            }  
        }  
  
        public void delete(K key) {  
            if (this.keyIndexMap.containsKey(key)) {  
                int deleteIndex = this.keyIndexMap.get(key);  
                int lastIndex = --this.size;  
                K lastKey = this.indexKeyMap.get(lastIndex);  
                this.keyIndexMap.put(lastKey, deleteIndex);  
                this.indexKeyMap.put(deleteIndex, lastKey);  
                this.keyIndexMap.remove(key);  
                this.indexKeyMap.remove(lastIndex);  
            }  
        }  
  
        public K getRandom() {  
            if (this.size == 0) {  
                return null;  
            }  
            int randomIndex = (int) (Math.random() * this.size); // 0 ~ size -1  
  return this.indexKeyMap.get(randomIndex);  
        }  
    }  
  
    public static void main(String[] args) {  
        Pool<String> pool = new Pool<String>();  
        pool.insert("zhangsan");  
        pool.insert("lisi");  
        pool.insert("wangwu");  
        System.out.println(pool.getRandom());  
        System.out.println(pool.getRandom());  
        System.out.println(pool.getRandom());  
        System.out.println(pool.getRandom());  
        System.out.println(pool.getRandom());  
        System.out.println(pool.getRandom());  
    }  
}
```
程序运行结果：每次运行结果都是不一样的。
```java
zhangsan
wangwu
wangwu
wangwu
zhangsan
lisi
```

## 三、布隆过滤器
### （一）作用
- 一般用于判断某个值是否在集合中，如果在则一定返回 true，但是不在也可能返回 true，**有失误率**；判断==某样东西一定不存在或者可能存在==。
- 相比于传统的 List、Set、Map 等数据结构，它更高效、占用空间更少，但是缺点是其返回的结果是概率性的，而不是确切的；
- 判断某个元素是否存在可以使用 HashMap，时间复杂度为 O（1），但是存储容量占比高（占用内存高），同时对于本地输入，数据在远程服务器，同时数据集大到无法一次性读进内存构建 HashMap 时候也存在问题。
- 比较针对黑名单问题和爬虫去重问题；

### （二）示例
**问题**： 判断 100 亿条占 64 字节的 URL，是否为一个 URL 黑名单集合中元素，在则返回 true，反之返回 false；

**解答**：
- 方案一：使用 HashSet，但是至少需要使用 6400 亿个字节长度（不包括指针等等占用位置），约为 640 G空间。
- 方案二：使用上述提到的哈希分流。
- 方案三：使用布隆过滤器
布隆过滤器是一个数组，每个元素为一个 bit。
![输入图片描述](AlgorithmEasyDay05_md_files/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8_20191227144111.png?v=1&type=image&token=V1:sZ8sq-ISj71jUBQD1bvqtwEa83QZrcgr5ZcE143XCuQ)

**实现布隆过滤器代码**：
```java
package nowcoder.easy.day05;

/**
 * 使用基本数据结构实现布隆过滤器，如果想减少数组空间，可以使用 Long 类型或者使用矩阵
 */
public class BloomFilter {
    public static void main(String[] args) {
        // 使用 int 类型，因为一个 int 占 4 位即 32 bit，因此 1000 个 int 数据可以表示 32000bit；
        int[] arr = new int[1000];
        // 想要查询的 30000 位置对应哪一个下标；即将第 30000 位置描黑。
        int index = 30000;
        // 结果对应于上面数组中的 0 ~ 999 中间一个位置（桶）； intIndex = 937
        int intIndex = index / 32;
        // 对应于桶中的具体哪一个 bit 应该被描黑；bitIndex = 16
        int bitIndex = index % 32;
        // 1 << 16,即只有第 16 位为 1，其他均为0，同时 num | (1 << 16) 使得 num 的第 16 号位置变为 1，
        arr[intIndex] = (arr[intIndex] | (1 << bitIndex));
    }
}
```
**实现过程**
取其中一个 URL 分别经过 K 个相互独立的哈希函数（hash1，hash2，hash3，。。。hashk），分别得到哈希值 code1，code2，code3，。。。codeK，然后对所有的哈希值 %m，得到 0 ~ m-1 之间的某个值，将这个值的位置描黑（如果该位置已经描黑，则继续描黑）。将所有的 URL 都按照此步骤走一遍，如果某个 URL 对应的 K 个数组中位置都是描黑的，则该 URL 在黑名单中，如果有一个不为黑则不在 URL 中。

### 布隆过滤器添加元素

-   将要添加的元素给k个哈希函数
-   得到对应于位数组上的k个位置
-   将这k个位置设为1

### 布隆过滤器查询元素

-   将要查询的元素给k个哈希函数
-   得到对应于位数组上的k个位置
-   如果k个位置有一个为0，则肯定不在集合中
-   如果k个位置全部为1，则可能在集合中


**示例**：
布隆过滤器是一个 bit 向量或者说 bit 数组，长这样：
![输入图片描述](AlgorithmEasyDay05_md_files/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8%E7%BB%93%E6%9E%84_20191227160326.jpg?v=1&type=image&token=V1:cdVwBm4uHcL-lGBCevd4cgE6rYdxwkrkKsvGMCf8oJ8)


如果我们要映射一个值到布隆过滤器中，我们需要使用**多个不同的哈希函数**生成**多个哈希值，**并对每个生成的哈希值指向的 bit 位置 1，例如针对值 “baidu” 和三个不同的哈希函数分别生成了哈希值 1、4、7，则上图转变为：

![](AlgorithmEasyDay05.resource/v2-a0ee721daf43f29dd42b7d441b79d227_hd.jpg)

Ok，我们现在再存一个值 “tencent”，如果哈希函数返回 3、4、8 的话，图继续变为：

![](AlgorithmEasyDay05.resource/v2-c0c20d8e06308aae1578c16afdea3b6a_hd.jpg)

值得注意的是，4 这个 bit 位由于两个值的哈希函数都返回了这个 bit 位，因此它被覆盖了。现在我们如果想查询 “dianping” 这个值是否存在，哈希函数返回了 1、5、8三个值，结果我们发现 5 这个 bit 位上的值为 0，**说明没有任何一个值映射到这个 bit 位上**，因此我们可以很确定地说 “dianping” 这个值不存在。而当我们需要查询 “baidu” 这个值是否存在的话，那么哈希函数必然会返回 1、4、7，然后我们检查发现这三个 bit 位上的值均为 1，那么我们可以说 “baidu”  **存在了么？答案是不可以，只能是 “baidu” 这个值可能存在。**

这是为什么呢？答案跟简单，因为随着增加的值越来越多，被置为 1 的 bit 位也会越来越多，这样某个值 “taobao” 即使没有被存储过，但是万一哈希函数返回的三个 bit 位都被其他值置位了 1 ，那么程序还是会判断 “taobao” 这个值存在。

### **支持删除么**

目前我们知道布隆过滤器可以支持 add 和 isExist 操作，那么 delete 操作可以么，答案是不可以，例如上图中的 bit 位 4 被两个值共同覆盖的话，一旦你删除其中一个值例如 “tencent” 而将其置位 0，那么下次判断另一个值例如 “baidu” 是否存在的话，会直接返回 false，而实际上你并没有删除它。

如何解决这个问题，答案是计数删除。但是计数删除需要存储一个数值，而不是原先的 bit 位，会增大占用的内存大小。这样的话，增加一个值就是将对应索引槽上存储的值加一，删除则是减一，判断是否存在则是看值是否大于0。

### **如何选择哈希函数个数和布隆过滤器长度**

很显然，过小的布隆过滤器很快所有的 bit 位均为 1，那么查询任何值都会返回“可能存在”，起不到过滤的目的了。布隆过滤器的长度会直接影响误报率，布隆过滤器越长其误报率越小。

另外，哈希函数的个数也需要权衡，个数越多则布隆过滤器 bit 位置位 1 的速度越快，且布隆过滤器的效率越低；但是如果太少的话，那我们的误报率会变高。

![](https://pic4.zhimg.com/80/v2-05d4a17ec47911d9ff0e72dc788d5573_hd.jpg)

k 为哈希函数个数，m 为布隆过滤器长度，n 为插入的元素个数，p 为误报率

如何选择适合业务的 k 和 m 值呢，这里直接贴一个公式：

![](https://pic1.zhimg.com/80/v2-1ed5b79aa7ac2e9cd66c83690fdbfcf0_hd.jpg)

如何推导这个公式这里只是提一句，因为对于使用来说并没有太大的意义，你让一个高中生来推会推得很快。k 次哈希函数某一 bit 位未被置为 1 的概率为：

![[公式]](https://www.zhihu.com/equation?tex=%281-%5Cfrac%7B1%7D%7Bm%7D%29%5E%7Bk%7D)

插入n个元素后依旧为 0 的概率和为 1 的概率分别是：

![[公式]](https://www.zhihu.com/equation?tex=%5Cleft%28+1-%5Cfrac%7B1%7D%7Bm%7D+%5Cright%29%5E%7Bnk%7D)  ![[公式]](https://www.zhihu.com/equation?tex=1-+%5Cleft%28+1-%5Cfrac%7B1%7D%7Bm%7D+%5Cright%29%5E%7Bnk+%7D)

标明某个元素是否在集合中所需的 k 个位置都按照如上的方法设置为 1，但是该方法可能会使算法错误的认为某一原本不在集合中的元素却被检测为在该集合中（False Positives），该概率由以下公式确定

![[公式]](https://www.zhihu.com/equation?tex=%5Cleft%5B+1-+%5Cleft%28+1-%5Cfrac%7B1%7D%7Bm%7D+%5Cright%29%5E%7Bnk%7D+%5Cright%5D%5E%7Bk%7D%5Capprox%5Cleft%28+1-e%5E%7B-kn%2Fm%7D+%5Cright%29%5E%7Bk%7D)

## **最佳实践**

常见的适用常见有，利用布隆过滤器减少磁盘 IO 或者网络请求，因为一旦一个值必定不存在的话，我们可以不用进行后续昂贵的查询请求。

另外，既然你使用布隆过滤器来加速查找和判断是否存在，那么性能很低的哈希函数不是个好选择，推荐 MurmurHash、Fnv 这些。

**大Value拆分**

Redis 因其支持 setbit 和 getbit 操作，且纯内存性能高等特点，因此天然就可以作为布隆过滤器来使用。但是布隆过滤器的不当使用极易产生大 Value，增加 Redis 阻塞风险，因此生成环境中建议对体积庞大的布隆过滤器进行拆分。

拆分的形式方法多种多样，但是本质是不要将 Hash(Key) 之后的请求分散在多个节点的多个小 bitmap 上，而是应该拆分成多个小 bitmap 之后，对一个 Key 的所有哈希函数都落在这一个小 bitmap 上。

**优化**
- 所需数组的长度和 URL 的数目有关，和需求的失误率有关，和每条 URL 的长度（所含字节数）无关；
- 数组长度计算公式：$$m = - \frac{n * (ln^p)}{(ln^2)^2}$$，其中 n表示样本量， p 为预期失误率；
此题中： n = 100亿，p = 0.0001，则 m = 131571428572 bit ≈ 16G。
















