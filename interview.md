[C++基础](https://github.com/huihut/interview)

以下的内容是基于上面的链接补充的一些比较细节的知识点，包括但不局限于C++

1. #### 拷贝构造函数参数为什么必须是引用？

     为了防止拷贝构造函数的无限递归，最终导致栈溢出

2. #### 虚函数表存在哪？

     虚函数表存放在全局数据区.被所有对象共享，对象的虚指针指向虚函数表，每个类有自己的虚函数表，如果重写了父类的虚函数，则子类虚函数表指向自己的虚函数地址，反之，指向父类的虚函数地址，虚函数表在编译期间生成.

3. ####  重载和重写（覆盖,override）的区别
   1. 首先是含义不同
      1. 方法重载是在同一个类中，声明多个同名方法，通过参数列表来区分不同的方法，与参数列表的数量、类型和顺序有关，与修饰符和返回值类型    以及抛出异常类型无关
      2. 方法重写的前提是发生在具有继承关系的两个类之间，方法重写有以下规则：
         a.参数列表必须保持一致
         b.返回值类型必须保持一致
         c.方法名必须保持一致
         d.重写方法的访问权限范围必须大于等于父类方法
         e.重写方法的抛出异常类型范围不能大于父类方法
      
    2. 方法的重载和重写的作用不同
         重载：在一个类中为一种行为提供多种实现方式并提高可读性
         重写：父类方法无法满足子类的要求，子类通过方法重写满足需求

4. #### 虚函数（virtual）可以是内联函数（inline）吗？

     虚函数可以是内联函数，内联是可以修饰虚函数的，但是当虚函数表现多态性的时候不能内联。
     内联是在编译器建议编译器内联，而虚函数的多态性在运行期，编译器无法知道运行期调用哪个代码，因此虚函数表现为多态性时（运行期）不可以内联。
     inline virtual 唯一可以内联的时候是：编译器知道所调用的对象是哪个类（如 Base::who()），这只有在编译器具有实际对象而不是对象的指针或引用时才会发生。
     虚函数内联使用

```c++
#include <iostream>  
using namespace std;
class Base
{
public:
	inline virtual void who()
	{
		cout << "I am Base\n";
	}
	virtual ~Base() {}
};
class Derived : public Base
{
public:
	inline void who()  // 不写inline时隐式内联
	{
		cout << "I am Derived\n";
	}
};

int main()
{
	// 此处的虚函数 who()，是通过类（Base）的具体对象（b）来调用的，编译期间就能确定了，所以它可以是内联的，但最终是否内联取决于编译器。 
	Base b;
	b.who();

    // 此处的虚函数是通过指针调用的，呈现多态性，需要在运行时期间才能确定，所以不能为内联。  
    Base *ptr = new Derived();
    ptr->who();

    // 因为Base有虚析构函数（virtual ~Base() {}），所以 delete 时，会先调用派生类（Derived）析构函数，再调用基类（Base）析构函数，防止内存泄漏。
    delete ptr;
    ptr = nullptr;

    system("pause");
    return 0;
} 
```

#### 5.菱形继承与虚继承、多继承

菱形继承

```c++
class A
{
public:
	A() { cout << "A()"; }
	virtual void fun(){}
};
class B : public A
{
public:
	B() { cout << "B()"; }
	void fun() {}

};
class C : public A
{
public:
	C() { cout << "C()"; }
	void fun(){}
};
class D :public B, public  C
{
public:
	D() { cout << "D()"; }
	void fun(){}
};

int main(int argc, char *argv[])
{
	A a; // A()
	int size = sizeof(a);//size = 4
	B b;// A() B()
	size = sizeof(b);//size = 4
	C c;// A() C()
	size = sizeof(c);//size = 4
	D d;//A() B() A() C() D()
	size = sizeof(d);//size = 8
    return 0;
}
```

虚继承

```c++
class A
{
public:
	A() { cout << "A()"; }
	virtual void fun(){}
};
class B : virtual public A
{
public:
	//这里不实现构造函数，是因为虚继承中如果同时满足以下两个条件，会产生vtordisp域，影响sizeof的大小
    //1. 派生类重写了虚基类的虚函数。
	//2. 派生类定义了构造函数或者析构函数。
	void fun() {}

};
class C : virtual public A
{
public:
	void fun(){}

};
class D :public B, public  C
{
public:
	void fun(){}
};

int main(int argc, char *argv[])
{
	A a; // A()
	int size = sizeof(a);//size = 4
	B b;
	size = sizeof(b);//size = 8 虚函数指针大小+虚基类指针大小
	C c;
	size = sizeof(c);//size = 8 虚函数指针大小+虚基类指针大小
	D d;//构造顺序是A() B() C() D() 与菱形继承的差别是少构造了一个A
	size = sizeof(d);//size = 12 sizeof(b) + sizeof(c) - sizeof(a),只有一个A对象，所以应该减去B/C中同时拥有的A的大小
    return 0;
}
```

多继承

```c++
class A
{
public:
	virtual void  fun(){}
};
class B 
{
public:
	virtual void  fun1(){}
};
class C :  public A,B
{
};

int main(int argc, char *argv[])
{
	C c;
	int size = sizeof(c);//size = 8 A虚函数指针大小+B虚函数指针大小
    return 0;
}
```

#### 6.类与对象的区别

类是对象的抽象，而对象是类的具体实例。类是抽象的，不占用内存，而对象是具体的，占用存储空间。类是用于创建对象的蓝图，它是一个定义包括在特定类型的对象中的方法和变量的软件模板。

#### 7.new与malloc的区别

1. new / new[]：完成两件事，先底层调用 malloc 分配了内存，然后调用构造函数（创建对象）。

2. delete/delete[]：也完成两件事，先调用析构函数（清理资源），然后底层调用 free 释放空间。

3. new 在申请内存时会自动计算所需字节数，而 malloc 则需我们自己输入申请内存空间的字节数。

4. new允许函数重载，malloc不允许

5. new是关键字，malloc是函数

#### 8.C与C++的区别

C++是在C语言的基础上开发的一种面向对象编程语言，应用广泛。C++支持多种编程范式 －－面向对象编程、泛型编程和过程化编程。 支持类：类、封装、重载、继承、多态等特性!

　　C++在C的基础上增添类，C是一个结构化语言，它的重点在于算法和数据结构。C程序的设计首要考虑的是如何通过一个过程，对输入（或环境条件）进行运算处理得到输出（或实现过程（事务）控制），而对于C++，首要考虑的是如何构造一个对象模型，让这个模型能够契合与之对应的问题域，这样就可以通过获取对象的状态信息得到输出或实现过程（事务）控制。

1. ​	在C语言中，不存在函数重载，原因为以函数名来唯一区分一个全局函数**。 **而在C++中 以函数名+参数列表来唯一区分函数。

2. C++中有引用，指针和引用都是地址的概念，指针指向一块儿内存，其内容为所指内存的地址；引用是某块儿内存的别名

3. C管理内存用malloc/free，C++还有new/delete

4. C中定义结构体必须加struct前缀，c++中不用

5. C++的类是C中没有的，C中的struct可以在C++中等同类来使用，struct和类的差别是，struct的成员默认访问修饰符是public，而类默认是private

#### 9.智能指针

智能指针的出现主要是为了解决代码在发生异常时的内存泄漏，以及在某些情况下，指针被多次delete的问题

```c++
void foo()
{
    try
    {  
        int *p = new int(0);  
        do_something(p);  
        delete p;
    }catch(...){}
}
```

如果do_something抛出异常，delete p是不会被执行的，从而造成内存泄露。使用智能指针的话，即使do_something发生异常，在栈上申请的局部变量依旧会被销毁，当指针指针被销毁时，它的析构函数会自动释放内存。

[智能指针的正确使用方式](https://www.cyhone.com/articles/right-way-to-use-cpp-smart-pointer/)

[智能指针的线程安全问题](https://www.jianshu.com/p/cb3e574eee5f)

（shared_ptr）的引用计数本身是安全且无锁的，但对象的读写则不是，因为 shared_ptr 有两个数据成员，读写操作不能原子化。

1. 同一个shared_ptr被多个线程读，是线程安全的；

2. 同一个shared_ptr被多个线程写，不是线程安全的；

3. 共享引用计数的不同的shared_ptr被多个线程写，是线程安全的。

4. weak_ptr最初的引入，是为了解决shared_ptr互相引用导致的内存无法释放的问题。weak_ptr不会增加引用计数，不能直接操作对象的内存（需要先调用[lock](https://links.jianshu.com/go?to=https%3A%2F%2Fen.cppreference.com%2Fw%2Fcpp%2Fmemory%2Fweak_ptr%2Flock)接口），需要和shared_ptr配套使用。

   同时，通过weak_ptr获得的shared_ptr可以安全使用，因为其[lock](https://links.jianshu.com/go?to=https%3A%2F%2Fen.cppreference.com%2Fw%2Fcpp%2Fmemory%2Fweak_ptr%2Flock)接口是原子性的，那么lock返回的是一个新的shared_ptr，不存在同一个shared_ptr的读写操作，除非后续又对这个新的shared_ptr又被其他线程同时读写。

#### 10.右值引用、移动语义、完美转发

[右值引用、移动语义和完美转发](https://www.jianshu.com/p/d19fc8447eaa)

#### 11.STL

map以自定义类做Key时，该类需要重载<操作符，否则无法插入，编译报错

vector和list的区别：

1. vector和数组类似，拥有一段连续的内存空间，并且起始地址不变。vector和数组类似，拥有一段连续的内存空间，并且起始地址不变。
   因此，它能够高效地进行**随机存取，时间复杂度是O(1)**。
   但是，因为其内存空间是连续的，所以在进行插入和删除操作时，**会造成内存块的拷贝，因此时间复杂度为O(n)**。
   另外，当数组内存空间不够时，会重新申请一块内动空间并进行内存拷贝。

2. list是由**双向链表**实现的，因此内存空间是不连续的。
   其只能通过指针访问数据，所以**list的随机存取效率很低，时间复杂度为O(n)**。
   不过由于链表自身的特点，**能够进行高效的插入和删除**。

vector:

- size()：返回vector中的元素个数

- capacity()：返回vector能存储元素的总数

- resize()操作：创建指定数量的的元素并指定vector的存储空间

- reserve()操作：指定vector的元素总数,并不创建元素

#### 12.c++11实现生产者消费者模式

```c++
#include<iostream> 
#include<mutex> 
#include<condition_variable> 
#include<queue> 
#include<thread> 
using namespace std; mutex g_mtx; 
condition_variable g_cv; 
queue<int> q; 
int maxsize=10; 
int data=0; 
void produce()
{ 
    while(true)
    { 	 
        this_thread::sleep_for(chrono::milliseconds(1000)); 		  	   	                     unique_lock<mutex> lc(g_mtx); //条件变量会先解锁  满足条件在加锁  	 
        g_cv.wait(lc,[](){ 	 	
            return q.size()<maxsize; 	 
        }); 	 
        q.push(++data); 	 
        cout<<"p id:"<<this_thread::get_id()<<"  data:: "<<data<<" q.szie:  "<<q.size()<<endl; 	 
        g_cv.notify_all();  
    } 
} 

void consume()
{ 
    while(true)
    { 	
        this_thread::sleep_for(chrono::milliseconds(500)); 	
        unique_lock<mutex> lc(g_mtx) ; 	 
        g_cv.wait(lc,[](){ 	 	
            return q.size()>0; 	 
        }); 	
        int c_data=q.front(); 	
        q.pop(); 	
        cout<<"     s id:"<<this_thread::get_id()<<"  data:: "<<c_data<<" q.szie:  "<<q.size()<<endl; 	
        g_cv.notify_all();  
    } 
} 

int main()
{ 
    thread t_p[3]; 
    thread t_c[3]; 
    for(auto &t:t_p)
    { 	
        t=thread(consume); 
    } 
    for(auto &t:t_c)
    { 	
        t=thread(produce); 
    } 
    for(auto &t:t_p)
    { 	
        t.join(); 
    }
    for(auto &t:t_c)
    { 	
        t.join(); 
    }
    return 0; 
}
```

