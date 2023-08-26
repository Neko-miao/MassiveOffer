# C++ issues  
## 保留字
### const  

### external  

## 引用
### 左值引用
### 右值引用

## 智能指针
理解：构造对象在生命周期结束之后会调用析构函数，而指针对象在生命周期结束之后不会主动调用析构函数，需要手动delete；
### auto_ptr
- auto_ptr<T> : 简单封装一个T类型的指针，重载了*和->运算符；
- 常用操作：
    - get(); 
    - release();
    - reset(); 
- 缺点：
1.不能共享所有权，即赋值和复制都是危险操作；
2.由于不能共享所有权，因为在STL中使用风险很大；
3.不支持对象数组内存管理，即auto_ptr<T[]> 是非法的；

- C++11后建议不要再使用auto_ptr，而使用unique_ptr替代  
### unique_ptr
- 理解：类似auto_ptr，但是存在某些特殊约束：
1. 无法进行左值复制或赋值操作，但允许临时的右值复制构造和赋值（使用std::move）；
2. 允许在STL中使用，并禁止直接赋值，当然可以使用std::move来赋值；
3. 支持对象数组的内存管理， unique_ptr<T[]> 是合法的，会自动调用delete[]去释放内存；
4. 除了以上约束，unique_ptr还支持自定义析构，如 unique_ptr<T, D>，其中D是一个自定义的析构器；  

- 注意，unique_ptr只是auto_ptr的扩展，并在语义上禁止了auto_ptr可能的危险操作，但它仍是排他性指针，不能共享所有权；  
### shared_ptr  
- 理解：通过引用计数来共享所有权，即赋值或拷贝时，被引用的内存对象引用计数加1，而智能指针析构时，引用计数减1，当引用计数为0时，就可以释放内存了  
- use_count(); 获取指针指向内存的引用个数；
- make_shard<T>(); 推荐用来构造一个shared_ptr指针，内存分配效率更高；

- 缺点：循环引用导致会导致shared_ptr造成内存泄漏；

### weak_ptr  
- 理解：配合shared_ptr使用，可以从一个shared_ptr或者weak_ptr对象构造，但是它的构造和析构不会改变引用计数；
- lock(); 获取一个shared_ptr对象，注意weak_ptr没有重载*和->运算符，因此不能直接访问引用对象；
- expired(); weak_ptr中也会维护use_count，当weak_ptr托管对象时，返回false，否则返回true；



