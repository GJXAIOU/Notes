# hashcode（）和equals（）的关系：

首先如果hashcode相等的话，这两个类也是不一定相等的，如果是反过来的话（通常情况下，如果两个对象的内容相同，两个对象的hashcode也是相同的）


**（1）如果不创建“类对应的散列表的话”（就是当我们不会把一个类放到在HashSet, Hashtable, HashMap这种底层实现是以hashcode来去定位存储位置的话），如果不是这种情况下的话，此时这个类的hashcode（）和equals（）是没有一点关系的**

**（2）如果恰好用到了上面所说的“创建了类对应的散列表的话”，那么也就是你把这个类作为key来去存储其他的value的话，这种情况下是可以进行比较的**

*   如果两个对象相等，那么它们的hashCode()值一定相同。这里的相等是指，通过equals()比较两个对象时返回true。
*   如果两个对象hashCode()相等，它们并不一定相等。
    因为在散列表中，hashCode()相等，即两个键值对的哈希值相等。然而哈希值相等，并不一定能得出键值对相等。补充说一句：“两个不同的键值对，哈希值相等”，这就是哈希冲突。**（若要判断两个对象是否相等，除了要覆盖equals()之外，也要覆盖hashCode()函数。否则，equals()无效。 ）**

```
public class test {

    public static void main(String[] args) {
        // 新建Person对象，
        Person p1 = new Person("eee", 100);
        Person p2 = new Person("eee", 100);
        Person p3 = new Person("aaa", 200);

        // 新建HashSet对象
        HashMap map = new HashMap();
        map.put(p1,"woshi---p1");
        map.put(p2,"woshi---p2");
        map.put(p3,"我是P3");

        System.out.println(map.get(p1)+"---------------------");
        System.out.println(map.get(p2)+"---------------------");
        System.out.println(map.get(p3));

        // 比较p1 和 p2， 并打印它们的hashCode()
        System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());

    }

    /**
     * @desc Person类。
     */
    private static class Person {
        int age;
        String name;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String toString() {
            return "("+name + ", " +age+")";
        }
    }
}

```

**输出结果：**
woshi—p1---------------------
woshi—p2---------------------
我是P3
p1.equals(p2) : false; p1(821270929) p2(1160460865)