# day15 集成IDE使用



## 一、C++基础知识
1.**基本的输入输出指令**

- **使用结构**
```cpp

#include  <iostream>
//这个头文件在另一个自定义的头文件中
#include"struct.h"

using  namespace  std;

int  main()

{

  //在C++中以下两语句等价，其中定义在struct.h的头文件中

  //  struct  student  st  =  {"tom",23};

  student  st  =  {"tom",23};

  cout  <<  "name  =  "<<  st.name  <<"\nage  =  "<<  st.age  <<  endl;

  return  0;

}
```

**补充：另一个头文件**
```cpp
#ifndef  STRUCT_H
#define  STRUCT_H
struct  student
{
  char  name[10];
  int  age;
};
#endif  //  STRUCT_H
```
程序运行结果：

![基本输入输出]($resource/%E5%9F%BA%E6%9C%AC%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA.png)

- **使用类**

**main函数**
```cpp
#include  <iostream>

#include"class.h"

#include<string.h>

using  namespace  std;
int  main()

{

  struct  student  st;

  strcpy(st.name,"tom");

  st.age  =  23;

  cout  <<  "name  =  "<<  st.name  <<"\nage  =  "<<  st.age  <<  endl;

  return  0;

}
```

**class头文件**
```cpp
#ifndef  CLASS_H

#define  CLASS_H

class  student

{

public://关键字public指下面的变量是公用的，可以在类的外部访问

  char  name[10];

  int  age;

private://关键字private说明下面的变量是私有的，也是C++的默认类型

  int  score;

};

#endif  //  CLASS_H
```

程序运行的结果：



![使用类的结果]($resource/%E4%BD%BF%E7%94%A8%E7%B1%BB%E7%9A%84%E7%BB%93%E6%9E%9C.png)

**注：使用私有的score**

**main函数**
```cpp
#include  <iostream>

#include"class.h"

#include<string.h>

using  namespace  std;
int  main()

{

  struct  student  st;

  strcpy(st.name,"tom");

  st.age  =  23;
  st.set_score(98);
  cout  <<  "name  =  "<<  st.name  <<"\nage  =  "<<  st.age  <<  endl;

  cout << "score = " << st.get_score()  << end1;

  return  0;

}
```

**class头文件**
```cpp
#ifndef  CLASS_H

#define  CLASS_H

class  student

{

public://关键字public指下面的变量是公用的，可以在类的外部访问

  char  name[10];

  int  age;

private://关键字private说明下面的变量是私有的，也是C++的默认类型

  int  score;

public:
  void set_score(int n)
    {
      score = n;
    }

  int get_score()
    {
      return score;
    }
};

#endif  //  CLASS_H
```
```


