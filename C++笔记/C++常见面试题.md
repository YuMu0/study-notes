## C++常见面试题

#### 引用和指针有什么区别？

* 定义一个指针变量时，编译器会为它分配内存，而引用不占用任何内存
* 引用必须在定义时被初始化，指针不必
* 不存在指向空值的引用，但存在指向空值的指针

#### new/delete和malloc/free的区别

* new/new[]：完成了两件事，先底层调用malloc分配了内存，然后调用构造函数创建对象
* delete/delete[]：也完成了两件事，先调用析构函数清理资源，然后底层调用free释放空间
* new在申请时会自动计算所需字节数，而malloc则需要我们输入申请内存空间的字节数

#### 成员delete this合法吗？

合法，但

* 必须保证this对象是通过`new`分配的
* 必须保证调用`delete this`的成员函数是最后一个调用this的成员函数
* 必须保证`delete this`后没有人使用了

#### 智能指针

C++标准库中，头文件：`#include<memory>`

**C++11**

1. shared_ptr
2. unique_ptr
3. weak_ptr
4. auto_ptr（被C++11弃用）

* Class shared_ptr实现**共享式拥有**(shared ownership)概念。多个智能指针指向相同的对象，该对象和其资源会在“最后一个reference被销毁时被释放。为了在结构较复杂的场景中工作，标准库提供weak_ptr、bad_weak_ptr和enable_shared_from_this等辅助类
* Class unique_ptr实现**独占式拥有**(exclusive ownership)或**严格拥有**(strict ownership)概念，保证同一时间内只有一个智能指针可以指向该对象，你可以移交所有权。它对于**避免内存泄露**(resource leak)——如new后忘记delete——特别有用

**shared_ptr**

多个智能指针可以共享同一个对象，对象的最末一个拥有者有责任销毁对象，并清理与该对象相关的所有资源

* 支持**定制型删除器**(custom deleter)

**weak_ptr**

weak_ptr允许你共享但不拥有某对象，一旦最末一个拥有该对象的智能指针失去了所有权，任何weak_ptr都会自动成空。因此，在default和copy构造函数外，weak_ptr只提供“接受一个shared_ptr"的构造函数

* 可打**破环状引用**的问题（两个其实已经没有被使用的对象彼此互指，使之看似还在"被使用"的状态）

**unique_ptr**

unique_ptr是C++11才开始提供的类型，是一种在异常时可以帮助避免资源泄露的智能指针。采用独占式拥有，意味着可以确保一个对象和其相应的资源在同一时间只被一个pointer拥有。一旦拥有者被销毁或定义empty，或开始指向另一个对象，先前拥有的那个对象就会被销毁，其相应资源同时被释放

* unique_ptr用于**取代auto_ptr**

**注意：**

> 智能指针的拷贝是浅拷贝，如果我们直接改变了对象的值，那么指向它的指针都会指向新值

#### 智能指针shared_ptr的实现

```c++
/*智能指针类定义*/
template<class T>
class SmartPtr
{
private:
	T *ptr;
	int *use_count;
public:
	SmartPtr(T *p);
	~SmartPtr();
	SmartPtr(const SmartPtr<T> &orig);
	SmartPtr<T> & operator = (const SmartPtr<T> &orig);
};

/*构造函数*/
template<class T>
SmartPtr<T>::SmartPtr(T *p) : ptr(p)
{
	try 
	{
		use_count = new int(1);
	}
	catch (...)
	{
		delete ptr;
		ptr = nullptr;
		use_count = nullptr;
		cout << "Allocate memory for use_count fails" << endl;
		exit(1);
	}
	cout << "Constructor is called!" << endl;
}

/*析构函数*/
template<class T>
SmartPtr<T>::~SmartPtr()
{
	/*只有最后一个引用时才释放内存*/
	if (--(*use_count) == 0)
	{
		delete ptr;
		ptr = nullptr;
		delete use_count;
		use_count = nullptr;
		cout << "Destructor is called!" << endl;
	}
}

/*拷贝构造函数*/
template<class T>
SmartPtr<T>::SmartPtr(const SmartPtr<T>& orig)
{
	ptr = orig.ptr;
	use_count = orig.use_count;
	(*use_count)++;
	cout << "Copy constructor is called!" << endl;
}

/*赋值操作符重载*/
template<class T>
SmartPtr<T>& SmartPtr<T>::operator = (const SmartPtr<T> &orig)
{
    ++(*orig.use_count);		//先让原先的引用计数+1

	/*=左边的指针对象引用数-1，如果为0，要释放*/
    if (--(*this->use_count) == 0)
    {
        delete ptr;
        delete use_count;
        cout << "Left side object is deleted!" << endl;
    }

    this->ptr = orig.ptr;
    this->use_count = orig.use_count;
    
    cout << "Assignment operator overloaded is called!" << endl;
    return *this;
}
```



#### 强制类型转换运算符

旧式风格的类型转换

```
type(expr); // 函数形式的强制类型转换
(type)expr; // C语言风格的强制类型转换
```

现代C++风格的类型转换

```
cast-name<type>(expression)
```

**static_cast**

任何编写程序时能够明确的类型转换**都可以使用**static_cast（不能转换掉底层const、volatile和_unaligned属性）。其**不提供运行时的检查**，所以叫static_cast，因此需要在编写程序时**确认转换的安全性**

主要在以下几个场合中使用：

* 用于类层次结构中，父类和子类之间指针和引用的转换
  * 上行转换，把子类对象的指针/引用转换为父类的指针/引用，这种转换是安全的
  * 下行转换，把父类对象的指针/引用转换为子类的指针/引用，这种转换是不安全的，需要编写程序时进行确认

* 用于**基本数据类型之间**的转换，如int转char，int转enum等，需要编程时确认安全性，不能用于基本数据类型指针之间的转换
* 把void指针转换成目标类型的指针（极其不安全）

> 上行转换是一种隐式转换

注意：**C++基本类型的指针之间不含有隐式类型转换**

```c++
base* b;
child* c;
c = static_cast<child*> b;	//正确
c = b;		//编译错误
```

**dynamic_cast**

会在运行时检查类型转换是否合法，具有一定安全性，但是会额外消耗一些性能

使用场景与static相似。在类层次结构中使用时，上行转换与static_cast没有区别，都是安全的；下行转换时，会检查转换的类型，比static_cast更安全

* 仅适用于**类指针或类引用**，不能用于基本数据类型的指针或引用

**const_cast**

用于移除类型的const、volatile和_unaligned属性，常量指针/引用被转换成非常量指针/引用，并仍然指向原来的对象

**reinterpret_cast**

非常激进的指针类型转换，在编译期完成，可以**转换任何类型的指针**，所以**极不安全**，非极端情况不要使用

#### AVL树/红黑树/B树/B+树

**AVL树**

最早的平衡二叉树之一，应用相对其他数据结构较少，windows对进程地址空间的管理用到了AVL树

**红黑树**

红黑树相比于AVL树，**牺牲了部分的平衡性**，以换取删除、插入操作时**少量的旋转次数**，因此**插入效率更高**。但是由于牺牲了部分树的平衡性，**查找效率稍低**

平衡二叉树，广泛用在C++的STL中，如map和set都是用红黑树实现的

**B树/B+树**

N叉平衡树，每个节点可以有更多的子节点，新的值可以插在已有节点上，而不需要改变树的高度，从而大量减少重新平衡和数据迁移的次数。用在磁盘文件组织、数据索引和数据库索引

**Trie树（字典树）**

并不是平衡树，也不一定非要有序。主要用于前缀匹配，用在统计和排序大量字符串

#### static关键字

1. （面向对象的）静态成员变量

* 其属于类，而不属于某个对象，所有对象共享此变量
* 其**必须在类的外部初始化**
* 其内存空间既不是在声明类时分配，也不是在创建对象时分配，而是在类外初始化时分配（也就是说，没有在类外初始化的static成员变量不能使用）
* static成员变量**不占用对象的内存**，而是在对象之外开辟内存，与普通的static成员变量类似，都在内存分区的全局区分配内存

2. （面向对象的）静态成员函数

* 属于类本身，而不属于某对象，因而**不具有this指针**
* 没有this指针，不知道指向哪个对象，因此**只能访问static静态成员**

3. （面向过程的）静态全局变量

该变量在全局数据区分配内存，自动变量会随着函数的退出而释放空间，而全局数据区的数据不会因为函数的退出而释放内存；静态全局变量不能被其他文件所用，可以防止命名冲突

4. （面向过程的）静态局部变量

在全局数据区内分配内存。程序执行到声明处被首次初始化，之后再次调用函数时，不会再初始化；不能被其他文件所用，可以防止命名冲突

5. （面向过程的）静态函数

不能被其他文件所用，可以防止命名冲突

#### const关键字

总的来说：

1. 用于变量修饰，表示这个变量不能被修改

2. 用于指针修饰，表明指针的指向物不能被修改

3. 用于方法修饰，表明这个方法不会对对象造成改变

在类中使用时：

1. const成员变量

* 初始化const成员变量只有一种方法，就是通过**构造函数的初始列表**

2. const成员函数`T fun() const;`

* 可以使用类中的所有成员变量，但**不能修改他们的值**
* 声明和定义都要加**const**，`T fun()`和`T fun() const`是两种不同的函数原型

#### struct和class的区别

* C++对C语言的的struct做了扩展，让其拥有了很多class的特性，两者几乎可以划等号
* 细节方面的不同，class默认`private`权限，而struct默认`public`权限

使用时，最好遵守：

> 仅当只有数据时使用struct，其它一概使用class

#### define和const的区别

`#define`属于宏，是预处理器的一部分。预处理是在编译之前的一道程序，简单地用字符串进行替换

`const`见const关键字笔记部分

#### STL相关

##### vector

vector内部是用**动态数组**的方式实现的，当进行`insert()`或`push_back()`增加元素时，如果动态数组的内存不够用，就要动态的重新分配当前大小的1.5~2倍（**指数增加**）的新内存区，再把原数组的内容复制进去。所以，在一般情况下，其访问速度与一般数组相比，只有在重新分配时，性能才会下降

* 进行`pop_back()`时，`capacity`并不会变小
* 如果进行大量的`push_back()`，应当使用`reserve()`函数**提前设定其大小**，否则会出现很多次容量扩充操作，导致效率低下

`size()`返回容器中有多少元素，`capacity()`返回容器当前的总空间大小

#### 栈与堆的区别

1. 分配和管理方式不同

堆是动态分配的，其空间的分配和释放都由程序员控制；栈由编译器自动管理

2. 产生碎片不同

对堆来说，频繁的new/delete或者malloc/free可能会造成内存空间的不连续，造成大量的碎片，使程序效率低下；对栈而言，则不存在碎片问题，因为栈是先进后出的队列，永远不可能有一个内存块从栈中间弹出

3. 增长方向不同

堆由低地址向高地址增长；栈由高地址向低地址增长

#### 面向对象的三大特性

**封装**

对外只暴露最小完整可用接口，隐藏内部实现细节。封装解决了数据的安全性，体现了对象为自己负责

**继承**

继承最大的意义其实是为了实现多态。继承会带来高耦合，增加代码维护的难度，因此**少用继承多用组合**

**多态**

给多态下一个定义：

> 同一个操作，作用于不同的对象，会产生不同的结果

#### 设计模式

设计模式的核心就两个词：**组合**与**解耦**

**工厂模式**



**单例模式**

简单来说就是一个类只能构建一个对象的设计模式

其关键实现在于

* 构造函数是私有的，即禁止用户自己声明并定义实例
* 禁止赋值和拷贝

优点

* 单例模式会阻止其他对象实例化自己单例对象的副本，从而确保所有对象都访问唯一的实体

使用场景，例如在Windows下就只能打开一个任务管理器。如果不使用机制对窗口对象进行唯一化，将弹出多个窗口，如果这些窗口显示的内容完全一致，则是重复对象，浪费内存资源；如果不同窗口内容不一致，更会给用户带来误解