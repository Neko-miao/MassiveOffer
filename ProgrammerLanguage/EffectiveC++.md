# Effective C++
## 一些习惯  
### provision 02. Prefer const, enum, inline to \#define.   
- 原因：宏不会进符号表，难以调试和错误定位；宏不能封装； 
- enum hack， 例如用 enum {NumTurns = 5}; 来替换const int NumTurns = 5; 功能类似宏，但是常量会进符号表；
- 宏函数参数要带括号，或用模板函数和inline来代替；


### provision 03. Use const whenever possible.  
- 常量指针和指针常量，主要理解点在于常量对象不能被改变，所以要搞明白是指针本身是常量还是指针所指内容是常量； 
- 函数返回往往是一个常量，可以降低客户端的意外出错； 
- const成员函数是可以重载的；
- const成员函数是不可以更改成员变量的，但如果允许更改，可以在给成员变量声明mutable；  
- 如果一个非const成员函数重载了一个const成员函数且两者实现是等价时，可以设计让非const函数调用const函数，避免代码重复，也提高了代码的扩展性（可以利用static_cast和const_cast）；  


### provision 04. Make sure that objects are initialized before they're used.  
- 内置类型，如int，有些编译器会初始化，有些不会；
- 构造函数参数初始化的顺序由成员变量定义的顺序决定，这就要清楚成员变量的初始化是否有依赖关系；
- 构造函数参数初始化的效率往往更高，但从性能上说，内置类型在构造参数初始化时初始化和构造函数体内赋值初始化是一样的； 
- 不同编译单元内定义的非局部静态对象（non-local static objects），局部对象是指函数内部定义的对象，局部静态对象在函数调用时开始初始化，这就给定了一个初始化时刻，方便用户在适当时候初始化静态对象，常见的例子如单例模式的GetInstance()函数。


## 构造、析构、赋值  
### provision 04. Know what funcionts C++ silently writes and calls.  
- 默认情况下，编译器会自动为class声明copy构造函数、copy赋值操作和析构函数，如果class没有定义任何构造函数，那编译器还会为为这个class声明一个default构造函数。 所有这些函数都是public且inline的。  
```C++
// 用户定义一个类：
class Empty {};

// 该类等价于：
class Empty{
public:
    Empty() { ... }
    Empty(const Empty& rhs) { ... }
    ~Empty() { ... }

    Empty& operator=(const Empty& rhs) { ... }
};
```

- 如果class内有个引用类成员变量，则编译器会拒绝生成operator=的定义，因为reference本质是指针常量，初始化后其值不会更改，所以赋值操作可能会引起错误。这时候需要用户自定义operator=。
- 如果基类的operator=是private，编译器也会拒绝生成它派生类的operator=

### provision 06. Explicitly disaalow the use of compiler-generated functions you do not want. 
- 如果想拒绝编译器自动生成的代码，可将相应的成员函数声明为private并且不予实现，或者将基类的相应函数声明为private，在继承这个基类。  

### provision 07. Declare destructors virtual in polymorphic base classes. 











