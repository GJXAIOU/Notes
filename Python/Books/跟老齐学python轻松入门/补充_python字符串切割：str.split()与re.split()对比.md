# 补充_python字符串切割：str.split()与re.split()对比


## 1、str.split  不支持正则及多个切割符号，不感知空格的数量，比如用空格切割，会出现下面情况。
```python
>>> s1="aa bb  cc"
>>> s1.split(' ')
﻿['aa', 'bb', '', 'cc']
```

因此split只适合简单的字符分割
## 2、re.split  :支持正则及多个字符切割
```python
>>> print line
abc aa;bb,cc | dd(xx).xxx 12.12'	xxxx
按空格切
>>> re.split(r' ',line)
['abc', 'aa;bb,cc', '|', 'dd(xx).xxx', "12.12'\txxxx"]
加将空格放可选框内[]内
>>> re.split(r'[ ]',line)
['abc', 'aa;bb,cc', '|', 'dd(xx).xxx', "12.12'\txxxx"]
按所有空白字符来切割：\s（[\t\n\r\f\v]）\S（任意非空白字符[^\t\n\r\f\v]
>>> re.split(r'[\s]',line)
['abc', 'aa;bb,cc', '|', 'dd(xx).xxx', "12.12'", 'xxxx']
多字符匹配
>>> re.split(r'[;,]',line)
['abc aa', 'bb', "cc | dd(xx).xxx 12.12'\txxxx"]
>>> re.split(r'[;,\s]',line)
['abc', 'aa', 'bb', 'cc', '|', 'dd(xx).xxx', "12.12'", 'xxxx']
使用括号捕获分组的适合，默认保留分割符
re.split('([;])',line)
['abc aa', ';', "bb,cc | dd(xx).xxx 12.12'\txxxx"]
去掉分隔符，加?:
>>> re.split(r'(?:;)',line)
['abc aa', "bb,cc | dd(xx).xxx 12.12'\txxxx"]
```







