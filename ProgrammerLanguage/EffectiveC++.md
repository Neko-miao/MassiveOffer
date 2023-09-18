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
- 为基类声明一个virtual析构函数，这样才能正确的在多态中释放掉派生类所占用的内存；  
- 需要注意的是，如果class在设计上不需要virtual那就不要添加virtual关键字，因为virtual会让类维护一张vptr（virtual table pointer）表，增加内存占用；
- 声明纯虚函数：
```C++
class AWOV{
public:
    virtual ~AWOV() = 0;
}

// 纯虚函数需要有个定义
AWOV::~AWOC() { }
```

### provision 08. Prevent exceptions from leaving destructors  
- 捕获析构时的异常或在析构异常时直接结束程序（std::abort()）； 
- 如果在析构函数的某些操作可能会导致异常，最后将这些操作抽象出来定义为某个函数，并捕获该函数可能发生的异常行为，因为析构函数异常往往比普通函数异常的危害要大一些，原因在于普通函数异常捕获之后，用户有机会去补救； 

### provision 09. Never call virtual functions during construction or destruction.  
- 一定不要在构造函数或者析构函数中调用虚函数，因为派生类在构造前或析构后，其父类调用virtual函数时不会调用符合预期的函数；
- 在派生类对象的基类构造期间，该派生类对象的类型其实是基类类型； 

### provision 10. Have assignment operator return a reference to *this.  
- 即：
```C++
public Widget{
public:
    // ...
    Widget& operator=(const Widget& rhs){
        // ...
        return *this;
    }
}
```
- 只是建议，并不强制要求，不过最好这样做...  

### provision 11. Handle assignment to self in operator=.  
- 自定义的赋值操作在进行自我赋值时需要额外注重对象和资源的释放，如：
```C++
class Widget{
public:
    Bitmap* pb;
    Widget& operator=(const Widget& rhs){
        del pb;
        pb = new Bitmap(rhs.pb);
        return *this;
    }
}

Widget w;
w = w;  // 引发异常

```
- 处理方法是增加“证同测试”和“异常捕获”功能；

### provision 12. Copy all parts of an object.  
- 不要在opeator=中调用copy构造函数，反过来也不行；


## 资源管理  
### provision 13. 以对象管理资源  
- 回顾智能指针内容，用“罐装式”思维去管理对象资源；  
- RAII(Resource Acquisition Is Initialization;)  资源获取的时机就是初始化时机；  
- RCSP(reference-counting smart pointer)；

### provision 14. 在资源管理类中小心coping行为  
- 看如下代码:
```C++
class Lock{
public:
    Lock(Mutex *mp)mutexPtr(mp){
        lock(mutePtr);
    }
    ~Lock(){
        unlock(mutePtr);
    }
private:
    Mutex *mutexPtr;
}

Mutex m;
Lock l1(&m);
Lock l2(l1); // 不合适
```  
- 解决解决上述代码问题可以从一下2点入手：  
    - 禁止复制，如继承一个private修饰copy构造函数的基类；  
    - 施行引用计数法来管理对象资源；  

### provision 15. 在资源管理类中提供对原始资源的访问  
- 资源访问时可以通过显示转换访问也可以通过隐式转换访问；
- 有时为了访问方便，可以定义类的转换构造函数和类型转换函数；

### provision 16. 成对使用new和delete时要采取相同形式  
- 即new的时候如果有[]那么delete也要带[]，如果new没带[]那么delete也不需要delete；
- 这点在typedef时尤为重要；  

### provision 17. 以独立语句将newd对象置于智能指针
- 看如下代码:
```C++
int priority();
void processWidget(shared_ptr<Widget> wid, int priority);

processWidget(shared_ptr<Widget>(new Widget), priority());
```

上述代码看似没有问题，在调用processWidet函数时，参数处理的过程可能如下：
1. new Widget
2. priority()
3. shared_prt<Widget>()  

不同于Java和C#，C++的编译器确实可能出现上述的情况。而如果编译器真的按照上述顺序初始化参数且priority()发生异常，new Widget的返回丢失，这部分内存就会泄露  

因此用如下方式调用会更好一些：
```C++
shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());
```

## 设计于声明  
### provision 18. 让接口容易被正确使用，不易被误用  
- 接口设计应该易于理解，不易被误用。
- 要做到易于理解可以：
    - 保证接口的一致性，如用size表示大小，而不是length，count混用；
    - 与内置类型行为兼容，如日期类型的operator+和int类型的operator+都可以达到用户预期；
- 要做到不易误用，可以：
    - 建立新类型，如用Month类型表示“月”而不是用int，用Day类型表示“天”而不是int；
    - 限制类型上的操作，防止一些潜在的非法操作出现；
    - 束缚对象值，如以下代码：
    ```C++
    class Month{
    public:
        static Month Jan() {return Month(1);}
        static Month Feb() {return Month(2);}
        //...
    private:
        explicit Month(int m);
    }
    ```
    注意，上述代码使用函数Jan()而不是变量Jan来表示一月，这是为了局部静态对象的正确初始化；
    - 消除用户的资源管理责任，如shared_ptr，用户不需要担心指针删除问题；而且shared_ptr有专属的删除器，不会出现在DLL1中new出的对象，在DLL2中delete(cross-DLL problem)；shared_ptr有额外的内存成本和多线程同步化开销，但是却能有效降低bug数量；


### provision 19. 设计class犹如设计type  
设计class时谨慎考虑以下问题：
- class的对象何时创建？何时销毁？
- 对象初始化和对象赋值有什么差别？
- 对象被passed by value如何处理？(拷贝构造函数实现)
- class的合法取值在什么范围？
- class需要配合哪个继承图系？
- class需要什么样的转换？（显式转换和隐式转换规范）
- 什么样的操作符和函数对class是合法的？  
- 什么样的标准函数应该被驳回？（即哪些应该是private函数）
- 谁该取用新class的成员？
- 什么是新class的“未声明接口”？
- 新class的抽象程度如何？
- 是否真的需要一个新class？

### provision 20. 传引用替代传值  
- 值传参的缺点：
    - 增加了额外的构造和析构成本
    - 造成对象割裂，即一个derived对象传值给一个based对象，因为值传参的缘故，该对象在函数内部的行为完全是一个based class的行为，而割裂了derived class部分的特性

### provision 21. 必须返回对象时，别妄想返回其reference
- 如果函数返回值声明为reference或指针，而返回结果确实一个local stack
对象或者heap-allocated对象或者local staic对象，这一定时非常糟糕的代码；

### provision 22. 成员变量声明为private  
- class的成员变量声明为private，用成员函数提供访问方式，这样更能类的封装性，也赋予客户访问数据的一致性、细粒度访问控制、约束条件获得保证，class的实现也会更具弹性；
- protected并不比public更具封装性，因为要永远假设对外暴露的变量一定会被修改；

### provision 23. 宁以non-member、non-friend替换member函数  
- 简单来说，就是用工具类（utility）来为类提供便利的接口，这样增加了封装弹性，同时减少了编译依赖；

### provision 24. 若所有参数皆需类型转换，请为此采用non-member函数
- 举个例子，对于有理数类（Rational）的operator*运算，采用non-member的方式才能实现所有参数（包括被this指针所指的那个隐喻参数）都要转换为Rational类的情况；

### provision 25. 考虑写出一个不抛异常的swap函数
- 最一般化的版本：
```C++
template<typename T>
void swap(T& a, T& b){
    T tmp(a);
    a = b;
    b = tmp;
}
// 代码中需要的copy构造函数和operator=操作都有缺省版本，因此不需要额外的工作  
```
- 但如果class包含一个指针，而这个指针指向的正是真正的数据，这样在最一般化的swap版本中，就拜拜浪费了copy和operator=的成本，因为真正需要交换的只是两个指针的值而已：
```C++
class MyClass{
public:
    MyClass(const MyClass& rhs);
    Widget& operator=(const MyClass& rhs){
        //...
    }
private:
    int* data;
}

namespace std{
    template<>
    void swap<Widget>(Widget& a, Widget& b){
        swap(a.data, b.data);  // 只需要换指针，注意data不能直接访问
    }
}
```
以上是swap的一个全特化版本，放在std命名空间下，为了能访问到被声明为private的data指针，可以修改上述代码为以下版本，这个版本可以保持STL的一致性：
```C++
class MyClass{
public:
    void swap(MyClass& other){
        using std::swap;
        swap(data, other.data)
    }
private:
    int* data;
}

namespace std{
    template<>
    void swap<MyClass>(MyClass& a, MyClass& b){
        a.swap(b);
    }
}
```

- 如果MyClass本身是一个template class，那以下操作其实是不合法的
```C++
namespace std{
    template<typename T>
    swap<MyClass<T>>(MyClass<T>& a, MyClass<T>& b){ // error!
        a.swap(b);
    }
}
```
这是因为，偏特化一个function template是不允许的，只能对class template进行偏特化；

- 如果要偏特化一个function template，一般是添加一个重载版本
```C++
namespace std{
    template<typename T>
    void swap(MyClass<T>& a, MyClass<T>& b){
        a.swap(b);
    }
    // 注意swap之后没有<>
}
```
上述代码也不合法，这是因为std命名空间下，不允许添加新的模板（包括模板函数、模板类或其他任何东西），所以需要换一个命名空间
```C++
namespace MyClassStuff{
    template<typename T>
    void swap(MyClass<T>& a, MyClass<T>& b){
        a.swap(b);
    }
}
```

## 实现
### provision 26. 尽可能延后变量定义式出现的时间  
- 即对象最好做到定义就使用，尽可能减少构造和析构成本；
- 先定义在赋值的成本高于直接通过构造函数定义：
```C++
string s1;
s1 = "string";

string s2("string") // 效率更高
```

### provision 27. 尽可能少做转型动作  
- C++的四种转型方式：
    - const_cast<T>: 常量转除，即将const转为非const
    - dynamic_cast<T>: 安全向下转型，效率差，因为很可能要进行许多class名字的strcmp操作
    - reinterpret_cast<T>: 低级转型，如pointer to int转型为int，可移植性差，不同编译器实现不同
    - static_cast<T>: 强迫隐式转换，如将non-const转换为const，或int转double，但是无法将const转non-const

### provision 28. 避免返回handles指向对象内部成分  


















