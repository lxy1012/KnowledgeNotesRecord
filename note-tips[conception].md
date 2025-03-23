# 概念-谨记

## 声明式:用来告诉编译器变量的名称和类型，而不分配内存

声明式揭示其签名式（siganture）,也就是参数和返回类型；


### 对象（object）声明式   

- extern int x;

### 函数（function）声明式 

- std::size_t numDigits(int num);

  size_t 只是typedef,是C++计算个数时用的某种不带正负号（unsigned）类型；也是vector,deque和string内的operator[]函数接收的参数类型
  
### 类（class）声明式 

- class Widget;

### 模板（template）声明式    

- template<typename T>
class GraphNode;

## 定义式:为了给变量分配内存，可以为变量赋初值

定义（definition）式的任务是提供编译器一些声明式所遗漏的细节。对对象而言，定义式是编译器为此对象拨发内存的地点。
对function或function template而言，定义式提供了代码本体。
对class或class template而言，定义式列出它们的成员


### 对象定义式 

- int x; / extern int x = 10;

	- 即使是 extern ，如果给变量赋值了，就是定义了

### 函数定义式

- std::size_t numDigits(int num)
{
       std::size_t digitsSoFar = 1;
       ...
       return digitsSoFar ;
}

### class定义式

- class Widget{
   public:
       Widget();
        ~Widget();
        ...
};

### template定义式

- template<typename T>
class GraphNode{
   public:
       GraphNode();
        ~GraphNode();
        ...
};

## 声明/定义

### C/C++中，通常变量的定义和声明是同时发生的
int value；         //声明 + 定义
struct Node {    // 声明 + 定义
    int left;
    int right;
}; 

### 变量/函数可以声明多次，但是定义只能一次

### 声明是告诉编译器变量或函数的类型和名称等，定义是告诉编译器变量的值，函数具体干了什么

## 初始化（Initialization）：“给予对象初始”的过程

对“用户自定义类型”的对象而言，初始化由构造函数执行

### 构造函数

- default构造函数:无参数或每个参数都有缺省值

	- class A{
public:
    A(); 
}

	- class A{
public:
       explicit A(int x = 0,bool b = true);
}

	  explicit :阻止被用来执行隐式类型转换
	  
- 非构造函数

	- class A{
public:
    explicit A(); 
}

- copying函数

	- copy构造函数：“以同型对象初始化自我对象”
定义一个对象如何以值传递（passed by value）

		- class A{
public:
    A(const A& temp);
}

		- A a1;
A a2(a1);

	- copy assignment操作符： “从另一个同型对象中拷贝其value到自我对象”

		- class A{
public:
    A& operator=(const A& temp); 
}

		- A a1,a2;

a1=a2;

	- "="也可调用copy构造函数;
区分：有新对象被定义必有一个构造函数被调用，不执行赋值操作

		- A a1;
A a2=a1;

## TR1 “Technical Report 1”一份规范，描述加入C++标准库的诸多新机能。所有组件获取：std::tr1::

## Boost 一个组织 http://boost.org,提供可移植、同僚复审、源码开放的C++程序库。大多数tr1机能以boost的工作为基础

## C++"成员初始化次序"总是相同，根据声明次序

## C++不允许“让reference改指向不同对象”

## 标准std::string有一个non-virtual析构函数，不建议把它作为基类；包括所有STL容器中的；

## C++没有提供类似Java的final classes或C# 的sealed classed那样“禁止派生”的机制

## 析构函数运作方式是，最深层派生的那个class其析构函数最先被调用，然后是每一个base classes的析构函数被调用

## C++不希望再构造和析构期间调用virtual函数，而Java或C#可以支持

## RAII Resource Acquisition Is Initialization资源获取即初始化

## 资源管理原则：当不再使用任何一种资源时，必须把它还给系统。
常见资源有文件描述器（file descriptors）,互斥锁（mutex locks）,图形界面中的字型和笔刷，数据库连接，以及网络sockets.最常见的资源就是动态分配内存

## 受auto_ptrs管理的资源必须绝对没有一个以上的auto_ptr同时指向它。

## RCSP  reference-counting smart pointer；tr1::shared_ptr就是其中一个rcsp

### 持续追踪共有多少对象指向某笔资源，并在无人指向它时自动删除该资源。

### RCSPs提供的行为类似垃圾回收garbage collection,但无法打破环状引用，如两个已经没用的对象互指

## cross-DLL problem 

### 这个问题发生在于”对象在动态连接程序库（DLL）中被new创建，却在另一个DLL内被delete销毁“。可以用tr1::shared_ptr避免此类”跨DLL之new/delete成对运用“导致的运行期错误，因为它缺省的删除器是来自诞生该ptr所在的DLL的delete。

## tr1::shared_ptr

### 最常见的实现品来自Boost,为原始指针的两倍大,而且使用辅助动态内存。以动态分配内存作为簿记用途和”删除器之专属数据“，以virtual形式调用删除器，并在多线程程序修改引用次数时蒙受线程同步化的额外开销。（只要定义一个预处理器符号就可以关闭多线程支持）

## 能够访问private成员变量的函数只有class的member函数加上friend函数而已

## namespace可以跨域多个源码文件，而class不可以

## C++只允许对class templates偏特化，在function templates偏特化行不通

## 转型

### 旧式：
（T）a
T(a)

- 当要调用一个explicit构造函数将一个对象传递给一个函数时，如Widget(10)

### 新式：
const_cast<T>(a)
dynamic_cast<T>(a)
reinterpret_cast<T>(a)
static_cast<T>(a)

- const_cast:唯一有能力将对象的常量性转除

- dunamic_const:用来决定某对象是否归属继承体系中的某个类型。唯一无法由旧式语法执行的动作，和唯一可能耗费重大运行成本的转型动作

- reinterpret_cast:意图执行低级转型，实际动作以及结果可能取决于编译器，不可移植。

- static_cast:用来强迫隐式转换。如non-const ->const;int -> double,void*指针 -> type指针，pointer-to-base -> pointer-to-dervied等

## dangling handles 空悬的号码牌：这种handles所指东西（的所属对象）不复存在。最常见来源是函数返回值

## 异常安全

### 两个满足条件

- 不泄露任何资源

- 不允许数据败坏

### 异常安全函数三个保证层级

- 强烈保证：如果异常被抛出，程序状态不改变。函数成功就是完全成功，函数失败程序会回复到”调用函数之前“的状态

- 基本承诺：如果异常被抛出，程序内的任何事物仍然保持在有效状态下。没有任何对象或数据结构会因此而败坏，所有对象都处于一种内部前后一致的状态。然而程序的现实状态不可预料。

- 不抛掷保证：承诺绝不抛出异常，因为它们总是能够完成原先承诺的功能。作用于内置类型身上的所有操作都提供nothrow保证。这是异常安全码中必不可少的关键基础材料

### 异常安全码必须提供上述三种保证之一，不然就不具备异常安全性

## inline

### 使用内联函数可以加速程序运行，因为它们消除了与函数调用关联的开销。 调用函数需要将返回地址推送到堆栈、将参数推送到堆栈、跳转到函数体，然后在函数完成时执行返回指令。 通过内联函数可以消除此过程。

### inline意味“执行前，先将调用动作替换为被调用函数的本体”；virtual意味“等待，直到运行期才确定调用哪个函数”

### inline函数和templates通常都被定义于头文件内

## 在Handle class身上，成员函数必须通过Impl pointer取得对象数据。那会为每一次访问增加一层间接性。而每一个对象消耗的内存数量必须增加pImpl的大小。最后pImpl必须初始化（在Handle class构造函数内），指向一个动态分配得来的impl objectm,所以你将蒙受因动态内存分配/释放所带来的额外开销，以及遭遇bad_alloc异常（内存不足）的可能性

## Inteface class，由于每个函数都是virtual，必须为每次函数调用付出一个间接跳跃的成本。此外interface class派生的对象必须内涵一个vptr,这个指针可能会增加存放对象所需的内存数量--实际取决于该对象除了interface class之外是否还有其他virtual函数来源

## virtual函数是动态绑定，virtual函数的缺省参数值是静态绑定的，non-virtual函数也是静态绑定的

### 如果缺省值是动态绑定的，编译器就必须有某种办法再运行期为virtual函数决定适当的参数缺省值。这比目前实行的”在编译器决定“的机制更慢而且更复杂。为了程序的执行速度和编译器实现上的简易度，C++做了这样的取舍，来提高运行期效率

## set的实现往往招致”每个元素耗用三个指针“的额外开销。因为sets通常以平衡查找树实现而成，使它们在查找、安插、移除元素时保证拥有对数时间效率。优先级：速度>空间 

## 应用域部分：程序中的对象，相当于所塑造的世界的某些事物，例如人、汽车、一张张视频画面等

## 实现域部分：纯粹是实现细节上的人工制品 ，像是缓冲区、互斥器、查找树等

## EBO empty base optimization:空白基类最优化

## C++ Template

### 最初发展动机：建立“类型安全”的容器如vector,list和map

### 后来发现泛型编程generic programming--写出的代码和其所处理的对象类型bicultural独立更好。的如STL算法的for_each,find和merge

### 最终发现其机制自身是一部完整的图灵机：可以被用来计算任何可计算的值。于是导出了模板元变成template programming，创造出“在C++编译器内 执行并于编译完成时停止执行”的程序

## 面向对象编程世界总是以显式接口和运行期多态解决问题；Template及泛型编程的世界中，优先级隐式接口和编译器多态反而高于显式接口和运行期多态

### template<typename T>
void doProcessing(T& w)
{
      if(w.size() > 10 && w != someNastyWidget)
      {
           T temp(w);
           temp.normalize();
           temp.swap();
      }
}

### 隐式接口：w必须支持哪一种接口，系由template种执行于w身上的操作来决定

### 编译器多态：“以不同的template参数具现化”会导致调用不同的函数

## template<>  : 模板全特化，一旦类型参数被定义为当前 定义的类，再没有其他template参数可供变化

## STL共有5种迭代器分类

### input迭代器  istream_iterators：只能向前移动，一次一步，客户只可读取，不能涂写它们所指的东西，而且只能读取一次。模仿指向输入文件的阅读指针read pointer

### output迭代器 ostream_iterators :一切只为输出；只向前移动，一次一步，客户中可涂写它们做指的东西，而且只能涂写一次。模仿指向输出文件的涂写指针write pointer

### forward迭代器：做前述两种的每一件事，而且可以读写所指物一次以上。

### bidirectional迭代器【list,set,multiset,map,multimap】:除了可以向前移动，还可以向后移动

### random access迭代器【vector,deque，string】:还可以执行”迭代器算术“，也就是它可以在常量时间内向前或向后跳跃任意距离。以内置指针为榜样，内置指针某种程度也可以当作该类迭代器

## Traits广泛用于标准程序库。其中除了iterator_traits，供应iterator_category和四类迭代器外；还有char_traits用来保存字符类型的相关信息，以及numeric_limits用来保存数值类型的相关信息，例如某数值类型可表现之最大最小值等

### TR1导入许多新的traits classes用以提供类型信息，总计为标准C++添加了50+的traits classes，包括is_fundamental<T>判断T是否为内置类型，is_arrqy<T>判断T是否为数组类型，以及is_base_of<T1,T2>判断T1和T2相同，一伙T1是T2的base class

