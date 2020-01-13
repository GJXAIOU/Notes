# numpy


- 用于数据分析，使用C语言编写完成


## 基础使用语法
```python
import numpy as np

```


```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
#@Time :2019/3/18 13:01
#Author:GJXAIOU


import numpy as np

#创建一个矩阵
array = np.array([1, 2, 3],[4, 5, 6])

#创建一个3*4全00矩阵
array1 = np.zeros((3,4))

#创建一个3*4全1矩阵
array2 = np.ones((3,4))

#将一系列随机数分为3*4矩阵
array3 = np.arange(1, 12, 1).reshape(3, 4)

#设置线的开头结尾，并且分为多段
array4 = np.linspace(1, 10, 5)

#使用三角函数
a = [1, 2, 3, 4, 5]
array5 = np.sin(a)

#矩阵的乘方：
##挨个对应元素相乘：a*b
##矩阵乘法：np.dot(a,b)或者a.dot(b)

#随机产生0-1之间的2行4列元素矩阵
array6 = np.random.random((2, 4))

#求矩阵中的最大值、最小值、和、平均值、中位数
array7 = np.max(array6)
array8 = np.min(array6)
array9 = np.sum(array6)
array10 = np.mean(array6) ## 求的是平均值
array11 = np.average(array6) ##求得是加权平均值
array12 = np.median(array6) #求中位数

#对元素之间进行操作：累加和累差
array13 = np.cumaum(array6) #逐渐累加，结果仍然为一个列表
array14 = np.diff(array6) #后面一个元素减去前面一个元素值，结果仍为列表
 
#对行或者列进行单独操作
array15 = np.sum(array6, axis=1) ##axis值为1表示按行求和；值为0表示按列求和

#对元素进行排序
array16 = np.sort(array6)

#对矩阵进行转置
array17 = np.transpose(array6)
##或者采用：array6.T

#对矩阵进行分片：clip(A, begin, over) 小于begin全部置为begin，大于over的全部置为over，中间的不变
array18 = np.clip(array6, 0.1, 0.4) 

#值索引
array19 = np.argmin(array6) #求出最小元素的索引，即第几个：从0开始

#元素索引
array6[2] #单行表示第三个元素，多行表示第三行
array6[1,2] #第二行第三个元素：等价于：array[1][2]
array6[:,2] #所有行的第三列

```
