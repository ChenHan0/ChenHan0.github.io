---
title: "Mastering Qt 5 笔记"
categories:
    - Cpp  
tag:
    - Cpp  
    - Qt  
last_modified_at: 2018-02-04T14:15:00+08:00
---

### 1. If your class only uses pointers or references for a class type, you can avoid including the header by using forward declaration. That will drastically reduce compilation time.  
#### 如果你的类只使用一个类的指针或者引用，你可以通过使用前向声明（Forward Declaration）来避免包含头文件。这将显著减少编译时间。  
前向声明是指声明标识符(表示编程的实体，如数据类型、变量、函数)时还没有给出完整的定义。即声明一个类foo：  

<!-- more -->  

``` cpp  
class foo；  
```  

在声明完这个类之后，定义完这个类之前，foo为一个不完全类型(incomplete type)，即foo是一个类型，但这个类型的性质（比如包含的成员，具有的操作）都不知道。  

这个类的作用也有限：
1. 不能定义foo类的对象；  
2. 可以用于定义指向这个类型的指针或引用；
3. 用于声明(不是定义)使用该类型作为形参或者返回类型的函数。

利用前向声明和C++编译器的特性，大部分情况下可以省掉#include。  

    编译器做的工作主要是：
    1. 扫描符号
    2. 确定对象大小

如：  

1. 由于所有对象的引用所占用的空间都是一样大，故c++编译器能很容易的确定大小。  
这里只需要做一个string的前向声明而不需要 `#include <string>`   

    ``` cpp
    class string;
    class Sample {
        private:
            string &s;
    };
    ```

2. 所有对象的指针所占的空间也是一样大的，故与第一种情况类似。  
3. 在声明成员函数的形参或者返回类型的时候，也可以利用前向声明的特性。  

    ``` cpp
    class string;
    class foo;
    class Sample {
        public:
            foo foo_test(foo &);

        private:
            string &s;
            foo *f;
    };
    ```

### 2. signals/slots机制  
Qt框架通过signals, slots, 和connections三个概念，带来了一个灵活的消息交换机制：  
- signals是由对象发送的消息
- slot是一个当signal被触发时调用的函数
- connections函数指定哪一个signal链接到哪一个slot

为什么你会喜欢上signals/slots？
- 一个slot仍然是一个普通的函数，所以你仍可以自己调用他
- 单个signal可以链接到不同的slot
- 单个slot可以被不同的链接的signal所调用
- 一个signal和slot可以在不同的对象之间建立连接，甚至可以在不同的线程之间建立连接




