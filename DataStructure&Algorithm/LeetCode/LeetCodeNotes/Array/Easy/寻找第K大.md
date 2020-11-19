## 题目描述

有一个整数数组，请你根据快速排序的思路，找出数组中第K大的数。

给定一个整数数组a,同时给定它的大小n和要找的K(K在1到n之间)，请返回第K大的数，保证答案存在。

测试样例：

```
[1,3,5,2,2],5,3
返回：2
```

### 解答

```java
import java.util.*;

public class Finder {
    public int findKth(int[] a, int n, int K) {
        PriorityQueue<Integer> pq =  new PriorityQueue<Integer>();
        for (int i = 0; i < a.length; i++) {
            pq.add(a[i]);
            if (pq.size() > K){
                pq.poll();
            }
        }
       return pq.poll();
    }
}
```

