## 防卫式声明
防卫式声明防止同一个头文件被包含多次，导致**重复定义**。
防卫式声明写法，
```cpp
#ifdef symbol
#define symbol
...
#endif

#pragma once //pragma预处理指令，还可能的值是pack，warning
```

## 前置声明
最大的好处是可以**节约编译时间**。
前置声明的类，是一个**不完整类型**，它是一个符号。可以用来定义指针、引用，或者函数行参，返回值类型等。
使用include，一个头文件的更改，导致所有包含此头文件的所有源文件都需要重新编译。
前置声明的类成员和方法是未知的，所以另一个好处是**隐藏实现细节**。
前置声明的缺点也很明显：**隐藏依赖关系**。如被前置声明的类更名时，所有前置声明处都需要更改。
**前置声明无法得知类的继承关系**，所以**函数重载决议**时，也可能出现问题。

## 函数调用
在x86中，调用函数是call指令。
call之前，参数从右到左，依次入栈
call时，将函数的返回地址压栈，跳转到函数入口
执行子函数时，首先保存上一个栈帧，将ebp压入栈中
接着，将创建新栈帧，将esp更新到ebp中
执行函数体中的指令，并将结果放到eax中
将前一个栈帧恢复，将ebp恢复
最后，执行ret，根据栈中返回地址，跳转

## 堆与栈
* 栈区由编译器自动分配和释放
* 堆区由程序员管理分配和释放。不释放，程序结束时统一由OS释放
* 栈一般存放临时局部变量
* 相对而言，栈空间有限，几个MB大小；堆区相对较大。
* 在分配效率上：栈分配更块；堆分配malloc和free时可能涉及到系统调用（mmap，brk等）。
* 生长方向上：堆是向上（由低地址向高地址生长），栈是向下（高地址向地址生长）

## 关键字
### static
static作用于函数体外的变量和函数，具有**文件内的作用域**，可以解决**名字冲突**问题。静态全局变量覆盖程序运行的**整个生命周期**。

static作用于函数体内的变量，属于**静态局部变量**，也是程序运行的整个生命周期。内存分配不在栈区，在**全局数据区**。

static作用于类成员变量时，该类的所有实例对象通向**同一份副本**。

static作用于类成员方法时，**不会隐式加入this指针参数，所以只能访问静态成员变量和方法**。调用静态类成员方法时可以使用**类名**。

### const
const作用于普通对象或形参时，表示**不可修改**属性，只读。

const作用于指针时，int const *p表示**指向的内存区不可修改**，int * const p表示**指针本身是常量**，不可修改。两个位置可同时加const。

const作用于成员方法，表示该方法**不会修改类成员变量**。 本质上，是在成员方法**this指针加上了const属性**。所以const对象只能调用const成员方法，同时const对象也当然不能修改类成员了。（而非const对象既可以调用const方法，也可以调用非const方法。）
**（注意：const和static不能同时使用，const是在this指针添加const，static则没有this指针）**

### define
宏定义，宏函数
有副作用
```cpp
#define max(a, b, c)  ( (a) > (b) ? ((a) > (c) ? (a)) : ((b) > (c) ? (b) : (c)) )
```

### inline
inline关键字用于申明函数为**内联**。在调用inline函数的地方，**函数将被展开**，避免了调用函数**入栈和出栈的开销**。

这是一个inline是一个**建议性的标志**，是否真正内联**由编译期决定**。内联函数应该**尽量短小**，通常在10行以内，**不能有循环和递归**。

### volatile
和const一样，属于**类型修饰符**。它修饰的变量，**编译器不会在执行期将变量寄存于寄存器中**，每次都在内存中获取。该关键字常被用于**寄存器编程**中，如串口数据，硬件状态位等，被映射的内存区可能已经被修改，如果此时使用缓存寄存器的值，是感知不到这个变化的。

### new
new分为new operator和operator new，前者是C++**内建**的，无法改变其行为，后者可以**被重载**。

new operator的行为是，
* 调用operator new分配内存；
* 调用类的构造函数；
* 进行指针转换并返回一个类指针。

new operator分配内存**默认的行为是抛出bad_alloc异常**，也有**nothrow版本**，可以返回值是否为NULL，判断分配成功与否。

```c++
try {
  int *p = new int[2];
} catch(const bad_alloc &e) {
  return -1;
}

int *p = new (std::nothrow) int;
if(!p) return -1;
```

当new分配内存失败，**即将抛出异常前**，会调用出错处理函数**new-handler**，可以使用**set_new_handler来安装oom处理函数**。

```c++
typedef void (*new_handler)();
new_handler set_new_handler(new_handler p) throw(); //throw()表示不抛出任何异常
//void func() throw(int); 只抛出int型异常
//void func() throw(...); 可能抛出任何类型异常
```

一般而言，**operaotr new/delete会调用malloc/free函数，用来分配释放内存。**

此外，还有operator new []，operator delete []和placement new，前者用于数组的分配和释放，一定要配套使用，否则会导致内存泄漏；后者用于**在已分配的内存块调用构造函数。**

```c++
class A{};
char *p = malloc(10);
A *pA = new(p)A();
```

set_new_handler原理是，设置了一个全局的oom（out of memory）处理方法，被封装在默认的operator new，operator new []中。当调用malloc失败后，即将抛出bad_alloc异常前，检查是否有设置new-handler方法。如果有，就调用这个方法，然后再次malloc。如果没有，直接抛出异常。

### mutable
mutable是为了**突破const的限制**而设置的，这样const成员方法就可以修改类成员变量了。

### explicit
用在**单参数构造函数中，避免被隐式转换**。多参数构造总是显式的。这在google c++和effective c++都推崇的。**禁止非预期的隐式转换。**

```c++
class A{
  explicit A(int);
  A(char *);
};
A a1(1); //ok
A a2 = 1;//error
A a3("duck");	//ok
A a4 = "apple";	//ok
```

### override
override用来显式说明该函数是**覆盖（重写）父类的虚函数**。需要**函数签名**完全一致。作用是如果这个函数签名不一致，编译期会报错。

函数签名包括：函数名，参数列表，const，volatile修饰符。

### final
final用于**禁止从类派生和覆盖虚方法**。类名后加final不能被继承，函数名后加final不能被覆盖。

### friend
以friend修饰的函数或类称为**友元函数**和**友元类**。

友元函数可以直接访问私有成员的**非成员函数**。友元函数必需是普通函数，非成员函数。

友元类的所有成员函数都是另一个类的友元函数，都可以访问私有成员。

友元一定程度上**破坏了封装性**。友元关系**不能被继承**，具有**单向性**，且**不具备传递性**。

## 野指针
**指向不可用的内存区**的指针。对这类指针操作，将使程序发生**未知行为**，可能导致程序崩溃。

导致野指针的三种常见原因：
* 指针变量**未初始化**，直接使用；
* 指针内存**被释放**，但指针未被设置为NULL；
* 指针**超过了变量作用域**。常见于返回局部指针。

一个好的编程风格是尽量少使用原始指针，改用智能指针，避免因为double free或未free导致的内存泄漏，或者内存释放后导致**指针悬空**，出现野指针。

## 引用和指针
* 指针是一个**实体**，引用是**别名**；
* 指针定义时，可以不初始化，而引用必须在定义时初始化，且后续不能改变**绑定关系**；
* 有**多级指针**，但没有多级引用；
* 对指针sizeof得到指针本身的大小，对引用sizeof得到被引用对象的大小；
* 指针需要解引用才能获取到指向的对象，引用不需要；

本质上，**引用也是指针**，引用也是记录被引用对象的地址。底层上也是占一个指针的大小。两者的区别是，指针所分配的内存可以是非const的，值可以改变，可以指向任意对象。但引用被初始化后，存储的是初始化对象的地址，且不能改变。

## strlen和sizeof
**strlen是库函数，而sizeof是关键字。**

strlen求得以'\0'结尾的字符串长度；

* 对类类型，对象或数组取sizeof，获得是真实占有的内存大小。
* 对于指向字符串指针取sizeof，获得指针的字节数，32bit占用4字节，64bit机占用8字节。
* 对于C风格数组作为**行参**，取sizeof，将被转换为指针，所以结果是指针占用字节数。（数组退化为指针）
* 对于C++风格数组作为行参，取sizeof，将获得数组真实的内存大小。

```c++
void print(int (&a)[2]){
  cout << a[0] << "," << a[1] << endl; //扩展性差
}

template <typename T, int N> //函数模版
void print(T (&a[N])){
  for(int i=0; i<N; ++i) cout << a[i] << ',';
}
```

## 字节对齐
对于**复合型结构**，为了**加速CPU访问速度**，调整各变量的起始地址。

如，32位机，一般按4字节对齐，如果变量的起始地址非4字节对齐，则需要**多次访存**，然后**组装**。如地址0x2，第一次读出0x0-0x3的值；第二次读出0x4-0x7的值，然后组装得到一个4字节变量。

在某些架构，访存地址必须对齐，否则发生**硬件错误**。

若要调整字节对齐数，有以下办法，

```c++
#pragma pack(x)
char buffer1[32] __attribute__((aligned(64)));      //指定对齐属性
//char buffer2[32] __attribute__((section(".data"))); //指定段属性
//char variable __attribute__((used)); //标记被使用，去掉警告
```

对齐规则
* struct结构中各个成员的起始地址一定是各成员所占字节的整数倍
* 整个struct结构的大小一定要为结构成员中最大成员所占字节数的整数倍。

## 大小端序
大端：高字节存低地址。
小端：高字节存高地址。

网络序：使用大端序。
主机序：和硬件架构相关。

默认情况下，ARM和x86都是小端序，更易使用。

## 强制类型转换
C语言中类型转换分为显示和隐式，显式可以用小括号，隐式可能发生在整型赋值给浮点型，字符型赋值给long型等。
C++中更推荐使用4中强制类型转换关键字：**static_cast，const_cast，reinterpret_cast，dynamic_cast。**

* static_cast
  - 用于类层结构的**基类和派生类**之间的指针或引用的转换；
    - 上行转换，将派生类指针或引用转为基类是安全的；
    - 下行转换，基类指针或引用转为派生类，没有动态类型检查RTTI，不安全
  - **基本数据类型转换**，整型与整型互转，整型与浮点互转，整型与枚举互转；枚举间互转；
  - **空指针**void*转为其他类型指针；
  - 左值转换到右值；
  - 注意，static_cast不能转换掉const、volatile、aligned等修饰符信息。
* const_cast
  - **去掉类型修饰符**const
* reinterpret_cast
  - **整型与指针或引用的互转**
  - **内置指针**、函数指针类型互转（非相关指针的相互转换）
* dynamic_cast
  - 与其他三类cast不同，dynamic_cast是**运行时检查类型转换**
  - dynamic_cast转换**指针失败时，返回nullptr**，转换为**引用失败时，抛异常bad_cast**
  - 要求源类型和目标类型为引用或指针，且必须是**非内置类型的类类型**，里面必须包含**虚方法**。
  - 主要用于安全的将基类指针或引用转为派生类的指针或引用。（**安全下行转换**）

**C++为什么需要引入cast运算符？**
* 编译期和运行期检查类型转换合法性，如static_cast，const_cast和reinterpret_cast是编译期检查，dynamic_cast是运行期检查。
* 方便后期**重构**时，定位强制转换

```cpp
//1. 初始化转换
int n = static_cast<int>(3.14);
vector<int> v = static_cast<vector<int>>(10);
//2. 静态向下转换
struct B{}; 
struct D : public B{};
D d;
B &br = d;
D &ad = static_cast<D&>(br);
//3. 向上转型
D a[10];
B *dp = static_cast<B*>(a);
//4. 左值到右值
vector<int> v2 = static_cast<vector<int> &&>(v);
//5. 舍弃表达式
static_cast<void>(v2.size());
//6. 显式转换
void *nv = &n;
int *ni = static_cast<int *>(nv);
//7. 枚举到整性或浮点数
E e = E:ONE;
int one = static_cast<int>(e);
//8. 枚举间互转，整性转枚举
E e2 = static_cast<E>(one);
//9. void *到任何类型转换
void *voidp = &e;
vector<int> *p = static_cast<vector<int> *>(voidp);
```

```cpp
int i = 7;
int *v1 = reinterpret_cast<int *>(7); //整性转指针
unsigned int *p1 = reinterpret_cast<unsigned int *>(v1); //指针互转
int f();
void (*fp1)() = reinterpret_cast<void(*)()>(f); //函数指针转换，类型不同
reinterpret_cast<unsigned int &>(i) = 42; //创建一个临时引用来赋值
```

```cpp
struct V {
  virtual void f(){}
}
struct A : virtual V{};
struct B : virtual V{};
struct D : A, B{};
D d;
A &a = d; //向上转型，可用static_cast或dynamic_cast
D &new_d = dynamic_cast<D&>(a); //向下转型
B &new_b = dynamic_cast<B&>(a); //侧向转型
```

## RTTI
RTTI是Runtime Type Identification的缩写，意为**运行时类型识别**。
C++主要为下列两个操作，提供RTTI，
* typeid运算符，返回表达式或类型名的**实际类型**，即const type_info struct。
    * **静态类型**，在**编译期推导出类型信息**
        * 类型名
        * 内置类型变量，如int，char等
        * 实例化对象
        * 指向**不含virtual的类对象指针的解引用**，或**不含virtual类对象的引用**
    * **动态类型**
        * 指向含有virtual的类对象的指针的解引用，或含有virtual类对象的引用
* dynamic_cast运算符，用于**多态指针或引用**安全的向下类型转换。
    * 使用虚表vtable中type_info指针，找到类类型信息
    * 在多重继承或虚拟继承中，类对象包含多个虚表指针，分别指向基类和子类的虚函数入口，但他们对应的type_info指针是相同的。这保证了该类的任意一个基类指针或引用指向派生类时，type_info是相同的。因此向下转型只需比较两者的type_info信息是否相同（operator==或operator!=）。
```cpp
class type_info{
public:
    virtual ~type_info();
    bool operator==(const type_info&) const;//type_info比较
    bool operator!=(const type_info&) const;
    bool before(const type_info&) const;
    const char* name() const;
private://non-copyable
    type_info(const type_info&);
    type_info operator=(const type_info&);
    //data members
};
```

```cpp
class X {
public:
    virtual ~X(){};
};
class XX : public X {};
class Y{};
int n = 0;
XX xx;
Y y;
Y *py = &y;
cout << typeid(int).name() << endl; //i
cout << typeid(XX).name() << endl;  //2XX
cout << typeid(n).name() << endl;   //i
cout << typeid(xx).name() << endl;  //2XX
cout << typeid(py).name() << endl;  //P1Y
cout << typeid(*py).name() << endl; //1Y，内部符号，不易读
```

```cpp
X *px = new XX();
cout << typeid(px).name() << endl;  //P1X
cout << typeid(*px).name() << endl; //2XX
 ```

## 智能指针

## 构造函数与析构函数
* 构造函数和析构函数为什么不能调用虚函数？
* 基类析构函数为什么声明为virtual？
## 虚函数与虚表
## 多态
## 重载
## 重写
## class与struct
## 右值与右值引用
## 序列式容器
## 关联式容器
## 无序容器
## 适配器

