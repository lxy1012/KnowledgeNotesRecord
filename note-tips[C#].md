# C#全面学习

## 类型系统

### C#是一种强类型语言，类型中可存储以下项信息：

- 类型变量所需的存储空间

- 可以表示的最大值和最小值

- 包含的成员（方法、字段、事件等）

- 继承的基类型

- 实现的接口

- 允许执行的运算种类

### 一般来说，类用于对更复杂的行为建模。类通常存储计划在创建类对象后进行修改的数据。结构最适用于小型数据结构，通常存储不打算在创建结构后修改的数据

### 记录类型可以是引用类型record class或值类型record struct。是具有附加编译器合成成员的数据结构。通常存储不打算在创建对象后修改的数据

## 打牢基础！！！并提出疑问

### 1.为什仫提供record类型

- 遇到的情况使用：
比如想要克隆一个新的实体而不是简单的引用传递
比如想要简单的比较属性值是否都一致
比如在输出，希望得到内部数据结构而不是简单地甩给我们一个类型名称

	- 说的有些类似结构体的一些特性，为什仫不使用？
结构体也有些不足：
结构体不支持继承
结构体是值传递的过程，因为，这意味着大量的结构体拥有着相同的数据，但是占用不同的内存
结构体内部相等判断使用ValueType.Equals方法，它使用反射来实现，因此性能不快

- 语法发展

	- C#9.0引入：使用record关键字声明一个记录类型，只能是引用类型

		- public record Animal；

	- C#10.0不仅有引用类型记录，还有结构体记录

		- //使用record class声明为引用类型记录，class关键词是可选的，当缺省时等价于C#9.0record用法：
public record Animal； == public record class Animal；

		- //使用record struct声明为结构体类型记录
public record struct Animal；
//只读结构体类型记录
public readonly record struct Animal；

- 与普通class、struct的区别：
记录虽然是引用类型，但是应该将记录按值类型一样去使用

	- 引用类型记录--是class用法的一个新用法，新的语法糖。setter是init，说明记录具有不可变性，记录一旦初始化完成，那么属性值将不可修改（可以通过反射修改）

		- 无构造参数，实例化时：
重载object相关函数、<Clone>$()--编译器生成

		- 有构造参数，实例化时：
将构造参数生成了属性、重载object相关函数、<Clone>$()--编译器生成、还生成了Deconstruct方法--具有解构能力

		- 因为记录可继承，那么如果父子记录的属性值一样，如果判定他们相同显然不合理，因此编译时额外生成了一个EqualityContract属性：
protected virtual Type EqualityContract => typeof(Person);

			- EqualityContract属性指向当前的记录类型(Type)，使用protected修饰

			- 如果记录没有从其他记录继承，那么EqualityContract属性会带有virtual修饰，否则override重写

			- 如果记录指定为sealed，即不可派生，那么EqualityContract属性会带有sealed修饰

	-  值相等性

		- 重写Object的Equal和GetHashCode方法

		- 重写运算符==和！=方法

		- 实现了IEquatable<T>接口

			- 为了让记录在泛型集合中使用Contains、IndexOf、Remove等方法是可以像string，int，bool等类型一样对待

	- 非破坏性变化：with

		- 因为记录是引用类型，而属性的setter的init，因此当克隆一个记录时就出现困难了。可以通过自定义属性来修改setter来实现，但这不是记录的初衷

		- 使用with关键词时先调用<Clone>$()方法来创建一个对象，然后对这个对象进行指定属性的初始化

	- 内置格式化

		- 实现PrintMembers，而不用重写ToString方法

### 2.DTO是什么

- Data Transfer Object数据传输对象。是一种设计模式，用于在不同层之间的传输数据，通常用于在应用程序的不同部分之间传输数据。主要目的是简化数据交互，减少因为数据传输而引入的不必要耦合

### 3.什么是缺省

- 默认选项，又称缺省值，是一种计算机术语。指在无决策者干预情况下，对于决策或应用软件、计算机程序的系统参数的自动选择

### 4.什么是浅拷贝和深拷贝

- 深浅拷贝在针对引用类型数据

- 浅拷贝：只复制某个指向对象的指针，而不复制对象本身。新旧对象还是共享同一块内存。

- 深拷贝：新对象跟原对象不共享内存，指针和指针对应的内存数据完全复制

### 5.物理内存块前的8字节数据是什么

- 好像是方法表指针

