# ★ 条款-WHAT WHY HOW

## 1.视C++为一个语言联邦

### WHY

- 以C语言为基础，包含但不限于区块blocks，语句statements,预处理器preprocessor,内置数据类型built-in data types,数组arrays,指针pointers

	- C语言局限：no 模板templates,no 异常exceptions,no 重载overloading

	- 不保证发生初始化

- Object-Oriented C++，包含但不限于classes(包括构造函数和析构函数)，封装encapsulation,继承inheritance,多态polymorphism,virtual函数（动态绑定）

	- 面向对象设计之古典守则在C++上的最直接实施

- Template C++,C++泛型编程generic programming部分---->新的编程范型TMP（template meteprogramming）模板元编程

- STL--template程序库，包含容器containers，迭代器iterators，算法algorithms以及函数对象function objects

	- 保证发生初始化

### HOW

- 高效编程守则需根据次语言改变策略,如：

	- pass-by-value > pass-by-reference

		- 针对built-in datas types

		- stl的迭代器和函数对象 都是在C指针之上塑造出来的

	- pass-by-reference-to-const > pass-by-value

		- object-oriented c++存在构造和析构函数

		- template c++ 甚至不知道处理的对象的类型

## 2.尽量以const,enum,inline替换#define

### WHY

- #define是在预处理器上处理，其余则是在编译器上处理，导致编译器在处理源码时，此源码部分已经被预处理器移走，不存在记号表（symbol table）中，导致浪费时间追踪问题

- #define不重视作用域，一旦宏被定义，其后的编译过程中一直有效，故不可用来定义class专属常量，还不能提供任何封装性

### HOW

- 用常量替换#defines

	- 定义常量指针，通常被放在头文件内（以便被不同源码含入）

	- class专属常量,如：

		- class GamePlayer{
private: 
     static const int NumTurns = 5; 
     int scores[NumTurns];
     ...
};

C++通常要求使用任何东西提供一个定义式，但如果是个class专属常量又是static且为整数类（例如ints,chars,bools）,需特殊处理。
只要不取它们的地址，可以声明并使用他们而无无须提供定义式。
若需取某个class专属常量的地址，需在实现文件添加定义式，如：const int GamePlayer::NumTurns;(已在声明时获得初值，所以在定义时不可以再设初值)

		- "the enum hack"补偿做法：
class GamePlayer{
private: 
     enum {NumTurns = 5};
     int scores[NumTurns];
     ...
};

enum hack行为某方面更像#define而非const。
不够优秀的编译器可能为“整数型const对象”设定另外的存储空间，而Enums和#defines不会导致非必要的内存分配

- 形似函数的宏，用inline替换#defines,如

	- #define CALL_WITH_MAX(a,b) f(a > b?a:b)

	- template<typename T>
inline void callWithMax(const T& a,const T& b)
{ 
       f(a > b?a:b);
}

## 3.尽可能使用const

### WHY

- 允许你指定一个"不该被改动"的对象，若明确定义，可以让编译器强制实施该项约束，确保该条约束不被违反

### HOW

- 可被施加于任何作用域得对象、函数参数、函数返回类型、成员函数本体

- const  *左边，则表示被指物是常量； *右边，则表示指针自身是常量 ，如：

	- const char* p = object; 或 char const * p = object; //const data
char* const p=object; //const pointer
const char*  const p = object; //both const

- const成员函数

	- 编译器强制实施bitwise constness:const成员函数不可以更改对象内任何non-static成员变量

	- logical constness:一个const成员函数可以修改它所处理的对象内的某些bits,但只有在客户端检测不出的情况喜爱才得如此

		- mutable(可变的)，释放掉non-static成员变量得bitwise constness约束

	- 当const和non-const成员函数有着实质等价得实现时，令non-const版本调用const版本可避免代码重复，但会导致两次转型操作

## 4.确定对象被使用前已先被初始化

### WHY

- 读取未初始化的值会导致不明确的行为

### WHAT

- 确保每一个构造函数都将对象的每一个成员初始化

- 区分“赋值”和“初始化”：C++规定，对象的成员变量的初始化发生在进入构造函数本体之前。
用member initialization list(成员初值列)替换赋值动作

### HOW

- 1.“内置型成员变量”明确地加以初始化
2.构造函数运用“成员初值列”初始化base classes和成员变量
3."不同编译单元内定义的non-local static的对象"初始化次序

- 内置类型，必须手动初始化；其他初始化则在构造函数

- non-local static对象替换成local static对象。因为C++保证，函数内的local static对象会在“该函数被调用期间”“首次遇上该对象之定义式”时被初始化

  多个编译单元内的non-local static对象经由“模板隐式具现化，implicit template instantations”形成，不但不可能决定正确的初始化次序，甚至往往不值得寻找“可决定正确次序”。
  
	- class FileSystem { 
public:
       ...
       std::size_t numDisks()  const;
       ...
};
FileSystem& tfs()                                      
{
      static FileSystem fs;
      return fs;
}
class Directory { ... };
Directory ::Directory (params)
{
       ...
       std::size_t disks = tfs().numDisks();
       ...
}
Directory& tempDir()
{ 
       static Directory td;
       return td;
}

## 5.了解C++默默编写并调用哪些函数

### WHY

- 每一个class都会有一或多个构造函数，一个析构函数，一个copy构造函数，一个copy assignment操作符。若自己不声明，编译器会声明default

- 默认copy assignment操作符存在以下三种情况导致拒绝编译赋值操作：

	- 对“内含reference成员”的class内赋值操作，因为C++不允许“让reference改指向不同对象”

	- 对“内含const成员”的class赋值操作，因为更改const成员是不合法的

	- base class将copy assignment操作符声明为private，编译器会拒绝为其derived classes生成该操作符，因为处理不了derived class中的base class成分

## 6.明确拒绝使用编译器自动生成的函数

### HOW

- 可将相应的成员函数声明为private并且不予实现。
也可以使用base class做法：
class Uncopyable{
protected:
     Uncopyable(){ }                                //允许构造和析构
     ~Uncopyable(){ }
private:
     Uncopyable (const Uncopyable&);  //但阻止copying
     Uncopyable& operator=(const Uncopyable&);
};

class HomeForSale:private Uncopyable{
      ...     //class不再声明copy构造函数和copy assign.操作符
};

## 7.为多态基类声明virtual析构函数

### WHY

- 对于多态类来说，基类不使用virtual析构函数，可能会导致“局部销毁”的问题，形成资源泄露，败坏数据结构，在调试器上浪费时间等问题

- virtual函数的目的是允许derived class实现客制化。通过base class接口处理derived class对象
每一个带有virtual函数的class都有一个由函数指针构成的数组的表vtbl,每一个virtual函数都有一个vptr指向该vtal。所以会导致对象所占内存增大

- 如果class不含virtual函数，通常表示它并不意图被用做一个base class

- pure virtual函数导致abstract classes --不能被实体化。

class AMOV {   
public:
     virtual ~AMOV () = 0;  //声明pure virtual 析构函数
};

AMOV::~AMOV() { }   //定义

## 8.别让异常逃离析构函数

### WHY

- C++不鼓励析构函数吐出异常。若是在析构函数中导致异常，则会传播该异常，即允许它离开该析构函数，此时会造成不可明确的问题

### HOW

- 绝对不要吐出异常。如果可能抛出异常，析构函数应该捕捉任何异常，然后吞下他们（不传播）【try{...}catch(...){制作运转记录，记下失败调用}】或者结束程序【std::abort()】

- 如果需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数，而非在析构函数中执行

## 9.绝不在构造和析构过程中调用virtual函数

### WHY

- 在构造过程中，基类早于派生；在基类构造过程中，virtual函数不是virtual函数；甚至当前运行期类型信息都是base classes类型

### HOW

- 在base class将函数改为non-virtual,然后要求derived class构造函数传递必要信息给base class构造函数

## 10.令operator= 返回一个reference to *this -->连锁赋值

## 11.在operator= 中处理“自我赋值”

### WHY

- 不具备“自我赋值安全性”，也不具备“异常安全性”

### HOW

- 对象自我赋值时有良好的行为，技术包括比较“来源对象”和“目标对象”的地址【if(this == &temp) return *this;】，精心周到的语句顺序，以及copy and swap技术

- 确定任何函数操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确

## 12.复制对象时勿忘其每一个成分

### HOW

- Copying函数应该确保复制“对象内的所有成员变量”以及”所有base class成分“。

- 不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并由两个copying函数共同调用

## 13.以对象管理资源

### HOW

- 为防止资源泄露，使用RAII对象，它们在给构造函数中获得资源，并在析构函数释放资源

- 两个常被使用的RAII classes分别是tr1::shared_ptr和auto_ptr。auto_ptr，复制动作会使被复制物指向null，而前者不会。

## 14.在资源管理类中小心copying行为

### HOW

- 复制RAII对象必须一并复制它所管理的资源（深度拷贝），所以资源的copying行为决定了RAII对象的copying行为

- 普遍且常见的RAII class copying行为是：抑制copying、施行引用计数法。

## 15.在资源管理类中提供对原始数据的访问

### HOW

- APIs往往要求访问原始数据，所以每个RAII class应该提供一个”取得其所管理之资源“的方法

- 对原始数据的访问可能经由显式转换或隐式转换。一般而言显示转换比较安全，但隐式转换更方便。

## 16.成对使用new和delete时要采用相同形式：new和delete;new[]和delete[] 

## 17.以独立语句将newed对象置入智能指针

### HOW

- 保证编译次序
std::tr1::shared_ptr<Widget> pw(new Widget);
processWidget(pw,priority());

## 18.让接口容易被正确使用，不易被误用

### HOW

- ”促进正确使用“的办法包括接口的一致性，以及与内置类型的行为兼容

- ”阻止误用“的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除资源管理责任

- tr1::shard_ptr支持定制型删除器。可防范DLL问题，可被用来自动解除互斥锁等问题

## 19.设计class犹如设计type

### HOW（思考以下问题：）

- 1.新type的对象应该如何被创建和销毁

	- 影响class的构造函数和析构函数以及内存分配函数和释放函数（operatornew/[] ,operatordelete/[]）的设计

- 2.对象的初始化和对象的赋值该有什么样的差别

	- 决定构造函数和赋值操作符的行为

- 3.新type的对象如果被passed by value,意味这什么？

	- copy构造函数如何实现

- 4.什么是新type的”合法值“

	- 决定了class的数值集必须维护的约束条件。这意味成员函数（特别是构造函数，赋值操作符和所谓的”setter“函数）必须进行错误检查工作

- 5.你的新type需要配合某个继承图系吗

	- 继承既有classes，就受到那些classes的设计的束缚，特别是它们的函数virtual or non-virtual的影响

- 6.你的新type需要什么样的转换

	- 考虑显/隐式转换

- 7.什么样的操作符和函数对此 新type而言是合理的

	- 决定class声明哪些函数，其中某些是member函数，哪些是non-member函数

- 8.什么样的标准函数应该驳回

	- 必须声明为private

- 9.谁该取用新type的成员

	- 决定哪个成员public/protected/private;也帮助决定哪个class，function应该是friends,以及嵌套是否合理

- 10.什么是新type的”未声明接口“

	- 对效率、异常安全性以及资源运用提供何种保证

- 11.你的新type有多么一般化

	- 若是定义一整个types家族，则考虑 class template方式 

- 12.你真的需要一个新type吗

	- 或许单纯定义一或多个non-member函数或templates，就可以实现需求

## 20.宁以pass-by-reference-to-const替换pass-by-value

### WHY

- 减少构造函数或析构函数的调用，来提高效率

- by-reference通常意味真正传递的是指针

- by-value存在slicing（对象切割）问题

- 内置类型以及stl的迭代器和函数对象，by-value效率更高

## 21.必须返回对象时，别妄想返回其reference

### HOW

- 绝不要返回pointer或reference指向一个local stack对象，或返回reference执行一个heap-allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象

## 22.将成员变量声明为private

### WHY（封装性）

- 可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性

- protected并不比public更具封装性

## 23.宁以non-member、non-friend替换member函数

### WHY

- 可以增加封装性、包裹弹性和机能扩充性

### HOW

- C++中比较自然的做法是，让含有non-member函数的对象位于目标对象同一个namespace中处理

## 24.若某个函数的所有参数（包括被this指针所指的那个隐喻参数）都需类型转换，必须采用non-member函数

## 25.考虑写出一个不抛异常的swap函数---copy and swap

### WHAT

- 置换两对象值
stl的一部分；异常安全编程的脊柱；处理自我赋值可能性的常见机制

- pimpl(pointer to implementation)手法：
置换对象中的pImpl指针

- default swap、member swaps、non-member swaps、std::swap特化版本、以及对swap的调用

### HOW

- swap缺省实现版若效率不足（几乎意味着你的class或template使用了 某种pimpl手法）：
1.当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常
2.如果提供一个member swap,也该提供一个non-member swap用来调用前者。对于classes（非templates），也请特化std::swap
3.调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何”命名空间修饰符“
4.为"用户定义类型"进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西

	- 1.提供一个public swap成员函数，让它高效置换两对象值。且不抛出异常

	- 2.在class或template所在命名空间内提供一个non-member swap，并令它调用上述swap成员函数,如：
namespace WidgetStuff {
     ...                         //模板化的WidgetImpl等等
     
    template<typename T>
     class Widget{
     public:
         ...
         void swap(Widget& other)
         {
              using std::swap;
              swap(pImpl,other.pImpl);
          }
          ...
      };

      ... 
     
     //non-memeber swap函数
     template<typename T>     
     void swap(Widget<T>& a,Widget<T>& b)
     {
           a.swap(b);    
      }
}

	- 3.若正编写class(而非class template),为此class特化std::swap.并令它调用你的swap成员函数,如：
 class Widget{
     public:
         ...
         void swap(Widget& other)
         {
              using std::swap;
              swap(pImpl,other.pImpl);
          }
          ...
      };

namespace std{
     template<>            //std::swap特化版本 
     void swap<Widget>(Widget& a,Widget& b)
      {
           a.swap(b);    
      }
}

	- 4.如果调用swap，请确定包含一个using声明式，以便让std::swap在你的函数内曝光可见，然后不加任何namespace修饰符，赤裸裸地使用swap,如：
template<typename T>     
void dosomething(T& obj1,T& obj2)
 {    
      using std::swap;
      ...
       swap(obj1,obj2); 
       ...   
 }

## 26.尽可能延后变量定义式的出现时间--直到确实需要它

### WHY

- 构造和析构成本

## 27.尽量少做转型动作

### HOW

- 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_casts。

- 如果转型是必要的，试着将它隐藏在某个函数背后。这样用户可以直接调用该函数，而不需要转型参数调用函数

- 尽量使用C++-style新式转型

## 28.避免返回handles 指向对象内部成分

### HOW

- 避免返回handles(包括references、指针、迭代器)指向对象内部。
可增加封装性，帮助const成员函数的行为像个const，并将发生”虚吊号码牌dangling handles“的可能性降至最低

## 29.为”异常安全“而努力是值得的

### WHAT

- 异常安全函数即使发生异常也满足异常安全性。

- 强烈保证-往往能够以copy-and-swap实现出来，但并非对所有函数都可实现或者具备现实意义

- 保证最高等级是根据调用的各个函数的最低等级来决定

## 30.透彻了解inlining的里里外外

### WHAT

- inline只是对编译器的一个申请，不是强制命令。这项申请可以隐喻提出，也可以明确提出

	- 隐喻方式--将函数定义于class定义式内：
class Person{
public:
     ...
     int age() const { return theAge; }
     ...
private:
     int theAge;
};

	- 明确声明--定义式前加上关键字inline:
inline FunctionName(params)
{  ...  }

- 一个表面上看似inline的函数是否真的实现inline,取决于当前建置环境，主要取决于编译器-“申请”通不通过

- 编译器通常不对“通过函数指针而进行的调用”实施inlining:
inline void f() { ... }  //假设编译器有意愿inline“对f的调用”
void (* pf) () = f; //pf指向f
...
f();     //这个调用将被inlined
pf();   //这个调用或许不被inlined,因为它通过函数指针达成

- inline函数无法随着程序库的升级而升级，一旦改变，必须要重新编译，将修改后的函数本体编进程序中

### HOW

- 将大多数inlinling限制在小型且被频繁调用的函数上。这可使日后的调式过程和二进制升级 更容易，也可使潜在的代码膨胀问题最小化，将程序的速度提升机会最大化

## 31.将文件间的编译依存关系降至最低

### WHAT

- #include <string>
#include "date.h"
#include "address.h"

class Person{
public:
    Person(const std::string& name,const Date& birthday,const Address& addr);
     std::string name() const;
     std::string birthDate() const;
     std::string address() const;
     ...
private:
     std::string theName;   //实现细目
     Date theBirthDate;      //实现细目
     Address theAddress;    //实现细目

“接口与实现分离”，调整如下：
#include <string>    //标准程序库组件不该被前置声明
#include <memory>

class PersonImpl;    //Person实现类的前置声明
class Date;              //Person接口用到的class的前置声明
class Address;

class Person{
public:
    Person(const std::string& name,const Date& birthday,const Address& addr);
     std::string name() const;
     std::string birthDate() const;
     std::string address() const;
     ...
private:
     std::tr1::shared_ptr<PersonImpl> pImpl;  //指针，指向实现物

完全与Dates.Addresses以及Persons的实现细目分离了。那些classes的任何实现 修改都不需要Person客户端重新编译，且客户 无法看到Person的实现细目。

- Handle classes：像Person这样使用pimpl idiom的classes,往往被称为该名称。将它们的所有函数转交给相应的实现类并由后者完成实际工作

	- 例子Person的实现类：
#include "Person.h"    //添加person的定义式
#include "PersonImpl.h"  

Person::Person(const std::string& name,const Date& birthday,const Address& addr)
:pImpl(new PersonImpl(name,birthday,addr))
{ }

std::string Person::name() const
{
      return pImpl -> name;
}

- Interface classes:令Person成为一种特殊的abstract base class。目的是详细一一描述derived classes的接口，因此他通常 不带成员变量，也没有构造函数，只有一个virtual析构函数以及一组pure virtual函数，用来叙述整个接口 

	- 针对Person的Interface class:
class Person{
public:
     virtual ~Person();
     virtual  std::string name() const = 0;
     virtual  std::string birthDate() const = 0;
     virtual  std::string address() const = 0;
      ...
      static std::tr1::shared_ptr<Person> create(const    std::string& name,const Date& birthday,const Address& addr);
     ...
};

class RealPerson : public Person{
public:
    RealPerson(const std::string& name,const Date& birthday,const Address& addr)
:theName(name),theBirthDate(birthday),theAddress(addr)
{ }
     virtual ~RealPerson();
     std::string name() const;
     std::string birthDate() const;
     std::string address() const;
     ...
private:
     std::string theName;   
     Date theBirthDate;     
     Address theAddress;  

     std::tr1::shared_ptr<Person> Person::create(const std::string&    name,const Date& birthday,const Address& addr)
    {  
           return std::tr1::shared_ptr<Person>(new    RealPerson(name,birthday,addr));
    }
};

### HOW

- 例子所举实现的分离关键在于以“声明的依存性”替换“定义的依存性”，正是编译依存性最小化的本质：现实中让头文件尽可能自我满足，万一做不到，则让它与其他文件的声明式（而非定义式）相依。

	- 如果使用object references 或object pointers可以完成任务，就不要用objects

	- 如果能够，尽量以 class声明式替换 class定义式 

	- 为声明式和定义式提供不同的头文件

- 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是Handle classes和Interface classes

- 程序库头文件应该以“完全且仅有声明式”的形式存在

## 32.确定你的public继承塑模出is-a关系

### WHAT

- 以C++进行面向对象编程，最重要的一个规则是：公开继承public inheritance意味”is-a“的关系
能够施行于base class对象身上的【每件事情】，也可以施行于derived class对象身上。

- classes之间的关系：”is-a“、”has-a“、”is-implemented-in-terms-of“根据某物实现出

## 33.避免遮掩继承而来的名称

### WHAT

- 同名称变量优先级：函数内>derived class作用域内>base class作用域>base class所在的namespace作用域>global作用域

### HOW

- 可以使用using声明式或者转交函数使得被遮掩的名称在当前作用域可见

## 34.区分接口继承和实现继承

### WHAT

- pure virtual函数目的是为了让derived classes只继承函数接口

- impure virtual函数是让derived classes继承该数据接口和缺省实现

- non-virtual函数目的是为了令derived class继承函数的接口及一份强制性实现

## 35.考虑virtual函数以外的其他选择

### WHAT

- class GameCharacter {
public:
    virtual int healthValue() const;
    ...
};

### HOW

- 藉由non-virtual interface（NVI）手法是实现Template Method模式：令客户通过public non-virtual成员函数间接调用private virtual函数的设计。
如果某些class继承体系要求derived class在virtual函数的实现内必须调用其base class的对应兄弟，而为了调用合法，virtual函数必须是protected，甚至一定是public，这么一来就不能实施NVI手法了。

	- class GameCharacter {
public:
    int healthValue() const
    {
         ...
         int retVal = doHealthValue();
         ...
         return retVal;
     }
    ...
private:
    virtual int doHealthValue() const
    {   ...  }
};

- 藉由Function Pointers实现Strategy模式
virtual函数替换为”函数指针成员变量“

	- class GameCharacter；   
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter{
public:
     typedef int (*HealthCalcFunc) (const GameCharacter&);

     explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
     :healthFunc(hcf)
     {}

      int healthValue() const
      { return healthFunc(*this); }
       ...
private:
     HealthCalcFunc  healthFunc;
};

	- 优点：
·同一人物类型之不同实体可以有不同的健康计算函数，如下：
class EvilBadGuy: public GameCharacter {
public:
    explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc)
    :GameCharacter(hcf)
    { ... }
     ...
};

int loseHealthQuickly(const GameCharacter&);
int loseHealthSlowly(const GameCharacter&);

EvilBadGuy ebg1(loseHealthQuickly);
EvilBadGuy ebg2(loseHealthSlowly);

·还可在运行期变更ebg1/ebg2健康指数计算函数

缺点：
可能必须降低GameCharacter封装性
...

- 藉由tr1::function完成Strategy模式
以tr1::function成员变量替换virtual函数，因而 允许使用 任何可调用物搭配一个兼容于需求的签名式

	- class GameCharacter；   
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter{
public:
    typedef std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;

     explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
     :healthFunc(hcf)
     {}

      int healthValue() const
      { return healthFunc(*this); }
       ...
private:
     HealthCalcFunc  healthFunc;
};

	- short calcHealth(const GameCharacter&);
struct HealthCalculator{
    int operator() (const GameCharacter&) const
    { ... }
};

class GameLevel {
public:
     float health(const GameCharacter&) const;
     ...
};

class EvilBadGuy:public GameCharacter{
   ...
};

class EyeCandyCharacter:public GameCharacter{
   ...
};

使用：
EvilBadGuy ebg1(calcHealth);    //函数计算健康指数

EyeCandyCharacter ecc1(HealthCalculator()); //函数对象计算健康指数

GameLevel currentLevel;     //成员函数计算健康指数
EvilBadGuy ebg2(std::tr1::bind(&GameLevel::health,currentLevel,_1));

- 古典的Strategy模式
将继承体系内的virtual函数替换 为 另一个 继承 体系 内的virtual函数

	- class GameCharacter;
class HealthCalcFunc {
public:
    ...
    virtual int calc(const GameCharacter& gc) const
    { ... }
    ...
};

HealthCalcFunc defaultHealthCalc;
class GameCharacter {
public:
    explicit GameCharacter(HealthCalcFunc * phcf = &defaultHealthCalc)
    :pHealthCalc(phcf)
    { }
     
    int healthValut const
    { return pHealthCalc->calc(*this); }
    ...
private:
     HealthCalcFunc*  pHealthCalc;
};

## 36.绝不重新定义继承而来的non-virtual函数

## 37.绝不重新定义继承而来的缺省参数值

## 38.通过复合塑模出has-a或”根据某物实现出“

### WHAT

- 符合是类型之间的一种关系

- 当复合发生于应用域内的对象之间，表现出has-a的关系

- 当它发生于实现域内则是表现is-implemented-in-terms-of的关系

## 39.明智而审慎地使用private对象

### WHY

- 使用private继承的两条规则

	- 如果classes之间 的继承关系是private,编译器不会自动将一个derived class对象转换为一个base class对象

	- 由private base class继承而来的所有成员，在derived class中都会变成private属性，纵使它们在base class中原本是protected或public属性

### WHAT

- private继承意味implement-in-terms-of
主要用于”当一个意欲成为derived class者想访问一个意欲成为base class者的protected成分，或为了重新定义一或多个virtual函数“

- 尽可能使用复合，必要时才使用private继承。
必要：主要是当protected成员或virtual函数牵扯进来的时候

## 40.明智而审慎地使用多重继承(MI   multiple inheritance)

### WHAT

- C++标准程序库的一个多重继承体系，名称分别是basic_ios,basic_istream,basic_ostream和basic_iostream

### HOW

- virtual继承会增加大小、速度、初始化复杂度等成本。如果virtual base classes不带任何数据，将是最具实用价值的情况（derived class可以不考虑base class部分数值初始化的问题）

- 多重继承的确有正当用途。其中一个情节涉及”public继承某个Interface class"和“private继承某个协助实现的class”的两相组合

## 41.了解隐式接口和编译器多态

### WHAT

- classes和templates都支持接口和多态

- 对classes而言接口是显式的，以函数签名为中心。多态则是通过virtual函数发生于运行期

- 对templates而言，接口是隐式的，奠基于有效表达式。多态则是通过template具现化和函数重载解析发生于编译期

## 42.了解typename的双重意义

### WHAT

- 声明template参数时，前缀关键字class和typename可互换

- 请使用关键字typename标识嵌套从属类型名称；但不得在base class lists或member initialization list内以它作为base class修饰符

	- template<typename T>
class Derived:public Base<T>::Nested {
public:
     explicit Derived(int x)
     :Base<T>::Nested(x)
     {
         typename Base<T>::Nested temp;
         typename std::iterator_traits<IterT>::value_type vtemp(*iter);
          ...
      }
     ...
};

## 43.学习处理模板化基类内的名称

### HOW

- 可在derived class templates内通过“this -> "指涉base class templates内的成员名称，或藉由一个明白写出的”base class资格修饰符“完成

## 44.将与参数无关的代码抽离templates

### HOW

- Templates生成多个classes和多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系

- 因非类型模板参数而造成的代码膨胀，往往可以消除，做法是以函数参数或class成员变量替换template参数

- 因类型参数而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述的具现类型共享实现码

## 45.运用成员函数模板接受所有兼容类型

### WHAT

- 同一个template的不同具现体是完全不同的classes

- 成员函数模板 member function templates/member templates：为class生成函数，如泛化copy构造函数。

### HOW

- 成员函数模板可以生成”可接受所有兼容类型“的函数

- 如果声明member templates用于”泛化copy构造“或”泛化assignment操作“，还是需要声明正常的copy构造函数和copy assignment操作符

## 46.需要类型转换时请为模板定义非成员函数

## 47.请使用traits classes表现类型信息

### WHAT

- 类型的traits信息必须位于类型自身之外。标准技术是把它放进一个template及其一或多个特化版本中。这样的templates在标准程序库中有若干个，其中针对迭代器者被名为iterator_traits;
习惯上traits总是被实现为structs,但它们却往往被成为traits classes

- iterator_traits运作方式是，针对每个IterT，在struct iterator_traits<IterT>内一定声明某个typedef名为iterator_category.这个typedef用来 确认IterT的迭代器分类
template< ... >
class deque{
public:
     class iterator {
     public:
           typedef random_access_iterator_tag iterator_category;
           ...
     };
     ...
};

template<typename IterT>
struct iterator_traits {
    typedef typename IterT:iterator_category iterator_category;     
     ...
};

- 因为指针不可能嵌套typedef，所以为了支持指针迭代器，特别提供一个偏特化版本
template<typename IterT>
struct iterator_traits<IterT*>
 {
    typedef random_access_iterator_tag iterator_category;   
     ...
};

### HOW

- 使用一个traits class

	- 建立一组重载函数或函数模板，彼此间的差异只在于各自的traits参数。令每个函数实现码与其接受之traits信息相应和

	- 建立一个控制函数或函数模板，它调用上述函数并传递traits class所提供的信息

- 整合重载技术后，traits classes有可能在编译器对类型执行if...else测试

## 48.认识template元编程

### TMP：编写template-based C++程序并执行于编译期的过程 ，因而得以实现早期错误侦测和更高的执行效率

### TMP可被用来生成”基于政策选择组合“‘的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码

### TMP的优点举例：
确保度量单位正确--time^(1/2) = time^(4/8)
优化矩阵计算--BigMatrix result=m1*m2*m3*m4;
可以生成客户定制之设计模式实现品

## 49.了解new-handler的行为

### WHAT

- 旧式编译器可能会在operator new时抛出异常，返回null指针；
现在多数当operator new抛出异常前,会先调用客户指定的错误处理函数【new-handler】。为了指定这个函数，客户必须通过调用set_new_handler函数设置，声明在<new>:
namespace std{
    typedef void (*new_handler) ( );
    new_handler set_new_handler(new_handler p) throw();

### HOW(设计良好的new-handler函数)

- 让更多内存可被使用：程序执行前分配内存，当new-handler第一次调用时 ，释还给程序

- 安装另一个new-handler：让new-handler可以修改自己的行为，当它下次被调用时，会根据不同情况做出不同事，做法之一就是令new-handler修改”会影响new-handler行为“的static数据、namespace数据或global数据

- 卸除new-handler,也就是将 null指针传给set_new_handler.若没有安装new-handler,当分配不成功时就抛出异常

- 抛出 bad-alloc（或 派生自bad-alloc）的异常。这样的异常不会被operator new捕捉，因此

- 不返回：通常调用abort或exit

## 50.了解new和delete的合理替换时机

### WHY

- 用来检测运用上的错误

- 为了强化效能

- 为了收集使用上的统计数据

- 为了增加分配和归还的速度

- 为了降低缺省内存管理器带来的空间额外开销

- 为了弥补缺省分配器中的非最佳齐位

- 为了将相关对象成簇集中

- 为了获得非传统的行为

### WHAT

- 编写自定义operator new/delete时，需要考虑齐位alignment：某些计算机体系结构要求特定的类型必须放在特定的内存地址上。例如可能会要求指数的地址必须是4倍数或者8倍数，不然可能会导致运行期硬件异常
C++要求所有operator news返回的指针都有适当的对齐（取决于数据类型）

## 51.编写new和delete时需固守常规

### WHAT

- operator new内含一个无穷循环，推出此循环的唯一办法是：内存被成功分配或new-handlering函数执行了相关行为;也应该有能力处理0 bytes申请。Class专属版本则还应该处理”比正确大小更大的是（错误）申请“。

- operator delete应该在收到null指针时不做任何事。Class专属版本则还应该处理”比正确大小更大的是（错误）申请“。

## 52.写了placement new也要写placement delete

### WHAT

- 建立一个base class，内涵所有正常形式的new和delete:

class StandardNewDeleteForms {
public: 
     //normal new/delete  --不建议遮掩正常版本
     static void* operator new(std::size_t size) throw(std::bad_alloc)
     { return ::operator new(size); }
     static void operator delete(void* pMemory) throw()
     { return ::operator delete(pMemory);}

      //placement new/delete
      static void* operator new(std::size_t size,void* ptr) throw()
     { return ::operator new(size,ptr); }
     static void operator delete(void* pMemory,void* ptr) throw()
     { return ::operator delete(pMemory,ptr);}
      
     //nothrow new/delete
     static void* operator new(std::size_t size,const std::nothrow_t& nt) throw(std::bad_alloc)
     { return ::operator new(size,nt); }
     static void operator delete(void* pMemory,const std::nothrow_t& nt) throw()
     { return ::operator delete(pMemory);}

- 若要扩充，可利用继承机制及using声明式取得标准形式：
class Widget : public StandardNewDeleteForms {
public:
    using StandardNewDeleteForms::operator new;
    using StandardNewDeleteForms::operator delete;
    
     static void* operator new(std::size_t size,std::ostream& logstream) throw(std::bad_alloc);
     static void operator delete(void* pMemory,std::ostream& logstream) throw();
    ...
};

## 53.不要轻忽编译器的警告

### WHAT

- 严肃对待编译器发出的警告信息。

- 不要过度依赖编译器的报警能力，因为不同编译器对待事情的态度并不相同

## 54.让自己熟悉包括TR1在内的标准程序库

### WHAT

- C++98列入的C++标准程序库主要成分

	- STL标准模板库，覆盖容器containers如vector,string,map、迭代器、算法algorithms如find，sort,transform、函数对象function objects如less,greater、各种容器适配器container adapters如stack,priority_queue和函数对象适配器function object adapters如mem_fun,not1

	- Iostreams,覆盖用户自定缓冲功能、国际化I/O，以及预先定义好的对象cin,cout,cerr和clog

	- 国际化支持，包括多区域能力。

	- 数值处理，包括复数模板complex和纯数值数据valarray

	- 异常阶层体系，包括base class excecption及其derived classes logic_error和runtime_error，以及更深继承的各个classes

	- C89标准库。1989C标准程序库

- TR1详细叙述了14个新组件components,也就是程序库机能单位

	- 智能指针 tr1::shared_ptr tr1::weak_ptr

	- tr1::function  只要其签名符合目标，可以表示任何函数或函数对象

	- tr1::bind 可以和const及non-const成员函数协同运作

	- Hash tables

	- 正则表达式 regular expressions

	- Tuples 变量组

	- tr1::array

	- tr1::mem_fn 扩充C++98的bind1st和bind2nd的能力

	- tr1::reference_wrapper 让 ”reference的行为更像对象“

	- 随机数

	- 数学特殊函数

	- C99兼容扩充

	- Type traits

	- tr1::result_of:用来推到函数调用的返回类型

## 55.让自己熟悉Boost http://boost.org

### WHAT

- 高质量、源码开放、平台独立、编译器独立的程序库

## 自由主题

## 自由主题

