# C++ new关键字深入理解

“new”是C++的一个关键字，同时也是操作符。关于new的话题非常多，因为它确实比较复杂，也非常神秘，下面我将把我了解到的与new有关的内容做一个总结。

## new的过程

当我们使用关键字new在堆上动态创建一个对象时，它实际上做了三件事：获得一块内存空间、调用构造函数、返回正确的指针。当然，如果我们创建的是简单类型的变量，那么第二步会被省略。假如我们定义了如下一个类A：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **class** A 
2. { 
3.   **int** i; 
4. **public**: 
5.   A(**int** _i) :i(_i*_i) {} 
6.   **void** Say() { printf("i=%d\n", i); } 
7. }; 
8. //调用new： 
9. A* pa = **new** A(3); 



那么上述动态创建一个对象的过程大致相当于以下三句话（只是大致上）：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. A* pa = (A*)malloc(**sizeof**(A)); 
2. pa->A::A(3); 
3. **return** pa; 



虽然从效果上看，这三句话也得到了一个有效的指向堆上的A对象的指针pa，但区别在于，当malloc失败时，它不会调用分配内存失败处理程序new_handler，而使用new的话会的。因此我们还是要尽可能的使用new，除非有一些特殊的需求。

## new 的三种形态

到目前为止，本文所提到的new都是指的“new operator”或称为“new expression”，但事实上在C++中一提到new，至少可能代表以下三种含义：new operator、operator new、placement new。

new operator就是我们平时所使用的new，其行为就是前面所说的三个步骤，我们不能更改它。但具体到某一步骤中的行为，如果它不满足我们的具体要求时，我们是有可能更改它的。三个步骤中最后一步只是简单的做一个指针的类型转换，没什么可说的，并且在编译出的代码中也并不需要这种转换，只是人为的认识罢了。但前两步就有些内容了。

new operator的第一步分配内存实际上是通过调用operator new来完成的，这里的new实际上是像加减乘除一样的操作符，因此也是可以重载的。operator new默认情况下首先调用分配内存的代码，尝试得到一段堆上的空间，如果成功就返回，如果失败，则转而去调用一个new_hander，然后继续重复前面过程。如果我们对这个过程不满意，就可以重载operator new，来设置我们希望的行为。例如：

**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **class** A 
2. { 
3. **public**: 
4.   **void*** operator **new**(**size_t** size) 
5.   { 
6. ​    printf("operator new called\n"); 
7. ​    **return** ::operator **new**(size); 
8.   } 
9. }; 
10.  
11. A* a = **new** A(); 

这里通过::operator new调用了原有的全局的new，实现了在分配内存之前输出一句话。全局的operator new也是可以重载的，但这样一来就不能再递归的使用new来分配内存，而只能使用malloc了：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **void*** operator **new**(**size_t** size) 
2. { 
3.   printf("global new\n"); 
4.   **return** malloc(size); 
5. } 



相应的，delete也有delete operator和operator delete之分，后者也是可以重载的。并且，如果重载了operator new，就应该也相应的重载operator delete，这是良好的编程习惯。

new的第三种形态——placement new是用来实现定位构造的，因此可以实现new operator三步操作中的第二步，也就是在取得了一块可以容纳指定类型对象的内存后，在这块内存上构造一个对象，这有点类似于前面代码中的“p->A::A(3);”这句话，但这并不是一个标准的写法，正确的写法是使用placement new：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. \#include <new.h> 
2.  
3. **void** main() 
4. { 
5.   **char** s[**sizeof**(A)]; 
6.   A* p = (A*)s; 
7.   **new**(p) A(3); //p->A::A(3); 
8.   p->Say(); 
9. } 



对头文件<new>或<new.h>的引用是必须的，这样才可以使用placement new。这里“new(p) A(3)”这种奇怪的写法便是placement new了，它实现了在指定内存地址上用指定类型的构造函数来构造一个对象的功能，后面A(3)就是对构造函数的显式调用。这里不难发现，这块指定的地址既可以是栈，又可以是堆，placement对此不加区分。但是，除非特别必要，不要直接使用placement new ，这毕竟不是用来构造对象的正式写法，只不过是new operator的一个步骤而已。使用new operator地编译器会自动生成对placement new的调用的代码，因此也会相应的生成使用delete时调用析构函数的代码。如果是像上面那样在栈上使用了placement new，则必须手工调用析构函数，这也是显式调用析构函数的唯一情况：

```cpp
p->~A();
```

当我们觉得默认的new operator对内存的管理不能满足我们的需要，而希望自己手工的管理内存时，placement new就有用了。STL中的allocator就使用了这种方式，借助placement new来实现更灵活有效的内存管理。

## 处理内存分配异常

正如前面所说，operator new的默认行为是请求分配内存，如果成功则返回此内存地址，如果失败则调用一个new_handler，然后再重复此过程。于是，想要从operator new的执行过程中返回，则必然需要满足下列条件之一：

- 分配内存成功
- new_handler中抛出bad_alloc异常
- new_handler中调用exit()或类似的函数，使程序结束

于是，我们可以假设默认情况下operator new的行为是这样的：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **void*** operator **new**(**size_t** size) 
2. { 
3.   **void*** p = null 
4.   **while**(!(p = malloc(size))) 
5.   { 
6. ​    **if**(null == new_handler) 
7. ​     **throw** bad_alloc(); 
8. ​    **try** 
9. ​    { 
10. ​     new_handler(); 
11. ​    } 
12. ​    **catch**(bad_alloc e) 
13. ​    { 
14. ​     **throw** e; 
15. ​    } 
16. ​    **catch**(…) 
17. ​    {} 
18.   } 
19.   **return** p; 
20. } 



在默认情况下，new_handler的行为是抛出一个bad_alloc异常，因此上述循环只会执行一次。但如果我们不希望使用默认行为，可以自定义一个new_handler，并使用std::set_new_handler函数使其生效。在自定义的new_handler中，我们可以抛出异常，可以结束程序，也可以运行一些代码使得有可能有内存被空闲出来，从而下一次分配时也许会成功，也可以通过set_new_handler来安装另一个可能更有效的new_handler。例如：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **void** MyNewHandler() 
2. { 
3.   printf(“New handler called!\n”); 
4.   **throw** std::bad_alloc(); 
5. } 
6.  
7. std::set_new_handler(MyNewHandler); 



这里new_handler程序在抛出异常之前会输出一句话。应该注意，在new_handler的代码里应该注意避免再嵌套有对new的调用，因为如果这里调用new再失败的话，可能会再导致对new_handler的调用，从而导致无限递归调用。——这是我猜的，并没有尝试过。
在编程时我们应该注意到对new的调用是有可能有异常被抛出的，因此在new的代码周围应该注意保持其事务性，即不能因为调用new失败抛出异常来导致不正确的程序逻辑或数据结构的出现。例如：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **class** SomeClass 
2. { 
3.   **static** **int** count; 
4.   SomeClass() {} 
5. **public**: 
6.   **static** SomeClass* GetNewInstance() 
7.   { 
8. ​    count++; 
9. ​    **return** **new** SomeClass(); 
10.   } 
11. }; 



静态变量count用于记录此类型生成的实例的个数，在上述代码中，如果因new分配内存失败而抛出异常，那么其实例个数并没有增加，但count变量的值却已经多了一个，从而数据结构被破坏。正确的写法是：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **static** SomeClass* GetNewInstance() 
2. { 
3.   SomeClass* p = **new** SomeClass(); 
4.   count++; 
5.   **return** p; 
6. } 



这样一来，如果new失败则直接抛出异常，count的值不会增加。类似的，在处理线程同步时，也要注意类似的问题：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **void** SomeFunc() 
2. { 
3.   lock(someMutex); //加一个锁 
4.   **delete** p; 
5.   p = **new** SomeClass(); 
6.   unlock(someMutex); 
7. } 



此时，如果new失败，unlock将不会被执行，于是不仅造成了一个指向不正确地址的指针p的存在，还将导致someMutex永远不会被解锁。这种情况是要注意避免的。（参考：[C++箴言：争取异常安全的代码](http://dev.yesky.com/490/2087990.shtml)）

## STL 的内存分配与 traits 技巧

在《STL原码剖析》一书中详细分析了SGI STL的内存分配器的行为。与直接使用new operator不同的是，SGI STL并不依赖C++默认的内存分配方式，而是使用一套自行实现的方案。首先SGI STL将可用内存整块的分配，使之成为当前进程可用的内存，当程序中确实需要分配内存时，先从这些已请求好的大内存块中尝试取得内存，如果失败的话再尝试整块的分配大内存。这种做法有效的避免了大量内存碎片的出现，提高了内存管理效率。

为了实现这种方式，STL使用了placement new，通过在自己管理的内存空间上使用placement new来构造对象，以达到原有new operator所具有的功能。



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **template** <**class** T1, **class** T2> 
2. **inline** **void** construct(T1* p, **const** T2& value) 
3. { 
4.   **new**(p) T1(value); 
5. } 



此函数接收一个已构造的对象，通过拷贝构造的方式在给定的内存地址p上构造一个新对象，代码中后半截T1(value)便是placement new语法中调用构造函数的写法，如果传入的对象value正是所要求的类型T1，那么这里就相当于调用拷贝构造函数。类似的，因使用了placement new，编译器不会自动产生调用析构函数的代码，需要手工的实现：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **template** <**class** T> 
2. **inline** **void** destory(T* pointer) 
3. { 
4.   pointer->~T(); 
5. } 



与此同时，STL中还有一个接收两个迭代器的destory版本，可将某容器上指定范围内的对象全部销毁。典型的实现方式就是通过一个循环来对此范围内的对象逐一调用析构函数。如果所传入的对象是非简单类型，这样做是必要的，但如果传入的是简单类型，或者根本没有必要调用析构函数的自定义类型（例如只包含数个int成员的结构体），那么再逐一调用析构函数是没有必要的，也浪费了时间。为此，STL使用了一种称为“type traits”的技巧，在编译器就判断出所传入的类型是否需要调用析构函数：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **template** <**class** ForwardIterator> 
2. **inline** **void** destory(ForwardIterator first, ForwardIterator last) 
3. { 
4.   __destory(first, last, value_type(first)); 
5. } 



其中value_type()用于取出迭代器所指向的对象的类型信息，于是：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **template**<**class** ForwardIterator, **class** T> 
2. **inline** **void** __destory(ForwardIterator first, ForwardIterator last, T*) 
3. { 
4.   **typedef** **typename** __type_traits<T>::has_trivial_destructor trivial_destructor; 
5.   __destory_aux(first, last, trivial_destructor()); 
6. } 
7. //如果需要调用析构函数： 
8. **template**<**class** ForwardIterator> 
9. **inline** **void** __destory_aux(ForwardIterator first, ForwardIterator last, __false_type) 
10. { 
11.   **for**(; first < last; ++first) 
12. ​    destory(&*first); //因first是迭代器，*first取出其真正内容，然后再用&取地址 
13. } 
14. //如果不需要，就什么也不做： 
15. tempalte<**class** ForwardIterator> 
16. **inline** **void** __destory_aux(ForwardIterator first, ForwardIterator last, __true_type) 
17. {} 



因上述函数全都是inline的，所以多层的函数调用并不会对性能造成影响，最终编译的结果根据具体的类型就只是一个for循环或者什么都没有。这里的关键在于__type_traits这个模板类上，它根据不同的T类型定义出不同的has_trivial_destructor的结果，如果T是简单类型，就定义为__true_type类型，否则就定义为__false_type类型。其中__true_type、__false_type只不过是两个没有任何内容的类，对程序的执行结果没有什么意义，但在编译器看来它对模板如何特化就具有非常重要的指导意义了，正如上面代码所示的那样。__type_traits也是特化了的一系列模板类：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **struct** __true_type {}; 
2. **struct** __false_type {}; 
3. **template** <**class** T> 
4. **struct** __type_traits 
5. { 
6. **public**: 
7.   **typedef** __false _type has_trivial_destructor; 
8.   …… 
9. }; 
10. **template**<> //模板特化 
11. **struct** __type_traits<**int**>  //int的特化版本 
12. { 
13. **public**: 
14.   **typedef** __true_type has_trivial_destructor; 
15.   …… 
16. }; 
17. …… //其他简单类型的特化版本 





如果要把一个自定义的类型MyClass也定义为不调用析构函数，只需要相应的定义__type_traits的一个特化版本即可：

```cpp
template<>



struct __type_traits



{



public:



   typedef __true_type has_trivial_destructor;



   ……



};
```





模板是比较高级的C++编程技巧，模板特化、模板偏特化就更是技巧性很强的东西，STL中的type_traits充分借助模板特化的功能，实现了在程序编译期通过编译器来决定为每一处调用使用哪个特化版本，于是在不增加编程复杂性的前提下大大提高了程序的运行效率。更详细的内容可参考《STL源码剖析》第二、三章中的相关内容。带有“[]”的new和delete

我们经常会通过new来动态创建一个数组，例如：

```cpp
char* s = new char[100];



……



delete s;
```





严格的说，上述代码是不正确的，因为我们在分配内存时使用的是new[]，而并不是简单的new，但释放内存时却用的是delete。正确的写法是使用delete[]：

```cpp
delete[] s;
```





但是，上述错误的代码似乎也能编译执行，并不会带来什么错误。事实上，new与new[]、delete与delete[]是有区别的，特别是当用来操作复杂类型时。假如针对一个我们自定义的类MyClass使用new[]：

```cpp
MyClass* p = new MyClass[10];
```





上述代码的结果是在堆上分配了10个连续的MyClass实例，并且已经对它们依次调用了构造函数，于是我们得到了10个可用的对象，这一点与Java、C#有区别的，Java、C#中这样的结果只是得到了10个null。换句话说，使用这种写法时MyClass必须拥有不带参数的构造函数，否则会发现编译期错误，因为编译器无法调用有参数的构造函数。

当这样构造成功后，我们可以再将其释放，释放时使用delete[]：

```cpp
delete[] p;
```



当我们对动态分配的数组调用delete[]时，其行为根据所申请的变量类型会有所不同。如果p指向简单类型，如int、char等，其结果只不过是这块内存被回收，此时使用delete[]与delete没有区别，但如果p指向的是复杂类型，delete[]会针对动态分配得到的每个对象调用析构函数，然后再释放内存。因此，如果我们对上述分配得到的p指针直接使用delete来回收，虽然编译期不报什么错误（因为编译器根本看不出来这个指针p是如何分配的），但在运行时（DEBUG情况下）会给出一个Debug assertion failed提示。

到这里，我们很容易提出一个问题——delete[]是如何知道要为多少个对象调用析构函数的？要回答这个问题，我们可以首先看一看new[]的重载。



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **class** MyClass 
2. { 
3.   **int** a; 
4. **public**: 
5.   MyClass() { printf("ctor\n"); } 
6.   ~MyClass() { printf("dtor\n"); } 
7. }; 
8.  
9. **void*** operator **new**[](**size_t** size) 
10. { 
11.   **void*** p = operator **new**(size); 
12.   printf("calling new[] with size=%d address=%p\n", size, p); 
13.   **return** p; 
14. } 
15.  
16. // 主函数 
17. MyClass* mc = **new** MyClass[3]; 
18. printf("address of mc=%p\n", mc); 
19. **delete**[] mc; 



运行此段代码，得到的结果为：（VC2005）

```cpp
calling new[] with size=16 address=003A5A58



ctor



ctor



ctor



address of mc=003A5A5C



dtor



dtor



dtor
```





虽然对构造函数和析构函数的调用结果都在预料之中，但所申请的内存空间大小以及地址的数值却出现了问题。我们的类MyClass的大小显然是4个字节，并且申请的数组中有3个元素，那么应该一共申请12个字节才对，但事实上系统却为我们申请了16字节，并且在operator new[]返后我们得到的内存地址是实际申请得到的内存地址值加4的结果。也就是说，当为复杂类型动态分配数组时，系统自动在最终得到的内存地址前空出了4个字节，我们有理由相信这4个字节的内容与动态分配数组的长度有关。通过单步跟踪，很容易发现这4个字节对应的int值为0×00000003，也就是说记录的是我们分配的对象的个数。改变一下分配的个数然后再次观察的结果证实了我的想法。于是，我们也有理由认为new[] operator的行为相当于下面的伪代码：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **template** <**class** T> 
2. T* New[](**int** count) 
3. { 
4.   **int** size = **sizeof**(T) * count + 4; 
5.   **void*** p = T::operator **new**[](size); 
6.   *(**int***)p = count; 
7.   T* pt = (T*)((**int**)p + 4); 
8.   **for**(**int** i = 0; i < count; i++) 
9. ​    **new**(&pt[i]) T(); 
10.   **return** pt; 
11. } 



上述示意性的代码省略了异常处理的部分，只是展示当我们对一个复杂类型使用new[]来动态分配数组时其真正的行为是什么，从中可以看到它分配了比预期多4个字节的内存并用它来保存对象的个数，然后对于后面每一块空间使用placement new来调用无参构造函数，这也就解释了为什么这种情况下类必须有无参构造函数，最后再将首地址返回。类似的，我们很容易写出相应的delete[]的实现代码：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **template** <**class** T> 
2. **void** Delete[](T* pt) 
3. { 
4.   **int** count = ((**int***)pt)[-1]; 
5.   **for**(**int** i = 0; i < count; i++) 
6. ​    pt[i].~T(); 
7.   **void*** p = (**void***)((**int**)pt – 4); 
8.   T::operator **delete**[](p); 
9. } 



由此可见，在默认情况下operator new[]与operator new的行为是相同的，operator delete[]与operator delete也是，不同的是new operator与new[] operator、delete operator与delete[] operator。当然，我们可以根据不同的需要来选择重载带有和不带有“[]”的operator new和delete，以满足不同的具体需求。

把前面类MyClass的代码稍做修改——注释掉析构函数，然后再来看看程序的输出：

```cpp
calling new[] with size=12 address=003A5A58



ctor



ctor



ctor



address of mc=003A5A58
```





这一次，new[]老老实实的申请了12个字节的内存，并且申请的结果与new[] operator返回的结果也是相同的，看来，是否在前面添加4个字节，只取决于这个类有没有析构函数，当然，这么说并不确切，正确的说法是这个类是否需要调用构造函数，因为如下两种情况下虽然这个类没声明析构函数，但还是多申请了4个字节：一是这个类中拥有需要调用析构函数的成员，二是这个类继承自需要调用析构函数的类。于是，我们可以递归的定义“需要调用析构函数的类”为以下三种情况之一：

1. 显式的声明了析构函数的
2. 拥有需要调用析构函数的类的成员的
3. 继承自需要调用析构函数的类的

类似的，动态申请简单类型的数组时，也不会多申请4个字节。于是在这两种情况下，释放内存时使用delete或delete[]都可以，但为养成良好的习惯，我们还是应该注意只要是动态分配的数组，释放时就使用delete[]。

**释放内存时如何知道长度**

但这同时又带来了新问题，既然申请无需调用析构函数的类或简单类型的数组时并没有记录个数信息，那么operator delete，或更直接的说free()是如何来回收这块内存的呢？这就要研究malloc()返回的内存的结构了。与new[]类似的是，实际上在malloc()申请内存时也多申请了数个字节的内容，只不过这与所申请的变量的类型没有任何关系，我们从调用malloc时所传入的参数也可以理解这一点——它只接收了要申请的内存的长度，并不关系这块内存用来保存什么类型。下面运行这样一段代码做个实验：



**[cpp]** [view plain](http://blog.csdn.net/bbs375/article/details/53202079#) [copy](http://blog.csdn.net/bbs375/article/details/53202079#)



1. **char** *p = 0; 
2. **for**(**int** i = 0; i < 40; i += 4) 
3. { 
4.   **char*** s = **new** **char**[i]; 
5.   printf("alloc %2d bytes, address=%p distance=%d\n", i, s, s - p); 
6.   p = s; 
7. } 



我们直接来看VC2005下Release版本的运行结果，DEBUG版因包含了较多的调试信息，这里就不分析了：

```cpp
alloc 0 bytes, address=003A36F0 distance=3815152



alloc 4 bytes, address=003A3700 distance=16



alloc 8 bytes, address=003A3710 distance=16



alloc 12 bytes, address=003A3720 distance=16



alloc 16 bytes, address=003A3738 distance=24



alloc 20 bytes, address=003A84C0 distance=19848



alloc 24 bytes, address=003A84E0 distance=32



alloc 28 bytes, address=003A8500 distance=32



alloc 32 bytes, address=003A8528 distance=40



alloc 36 bytes, address=003A8550 distance=40
```



每一次分配的字节数都比上一次多4，distance值记录着与上一次分配的差值，第一个差值没有实际意义，中间有一个较大的差值，可能是这块内存已经被分配了，于是也忽略它。结果中最小的差值为16字节，直到我们申请16字节时，这个差值变成了24，后面也有类似的规律，那么我们可以认为申请所得的内存结构是如下这样的：

![img](http://static.codeceo.com/images/2016/10/06a567c43a9ef88efea8059ef439618b1.jpeg)

从图中不难看出，当我们要分配一段内存时，所得的内存地址和上一次的尾地址至少要相距8个字节（在DEBUG版中还要更多），那么我们可以猜想，这8个字节中应该记录着与这段所分配的内存有关的信息。观察这8个节内的内容，得到结果如下：

![img](http://static.codeceo.com/images/2016/10/09ce4d85b177141472551b67e5821ae11.jpeg)

图中右边为每次分配所得的地址之前8个字节的内容的16进制表示，从图中红线所表示可以看到，这8个字节中的第一个字节乘以8即得到相临两次分配时的距离，经过试验一次性分配更大的长度可知，第二个字节也是这个意义，并且代表高8位，也就说前面空的这8个字节中的前两个字节记录了一次分配内存的长度信息，后面的六个字节可能与空闲内存链表的信息有关，在回收内存时用来提供必要的信息。这就解答了前面提出的问题，原来C/C++在分配内存时已经记录了足够充分的信息用于回收内存，只不过我们平常不关心它罢了。