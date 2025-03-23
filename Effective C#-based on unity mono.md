# Effective C#-based on unity mono-22条

## 1.尽可能使用属性（property）,而不是可直接访问的数据成员

### 属性允许将数据成员作为共有接口的一部分暴露出去，同事仍旧提供面向对象环境下所需的封装。属性这个语言元素可以像访问数据成员一样使用，但其底层依旧是使用方法实现的

### 拥有方法所拥有的一切语言特性：
1.属性增加多线程的支持是非常方便的。可以加强get和set访问器的实现来提供数据访问的同步
2.属性可被定义为virtual
3.可以把属性拓展为abstract
4.可以使用泛型版本的属性类型
5.属性也可以定义为接口
6.因为实现访问的方法get与set是独立的两个方法没在C#2.0之后，可以定义不同的访问权限，来更好的控制类成员的可见性
7.为了和多维数组保持一致，可以创建多维索引器，在不同的维度上使用相同或不同的类型

## 2.偏向于使用运行时常量而不是编译时常量

### 对应常量，c#:运行时常量readonly 和编译时常量const
编译时常量的值会被目标代码中的值直接取代
运行时常量的值是在运行时求值。运用运行时生成的IL将引用到的readonly变量，而不是变量的值

- 差别就带来了如下规则：
const仅能用于数值和字符串
readonly可以为任意类型。运行时常量必须在构造函数或初始化器中初始化，因为在构造函数执行后不能再被修改。
可以用readonly值保存实例常量，为类的每个实例存放不同的值。而编译时常量就是静态的常量。
有时候需要让某个值在编译时才确定，就最好是使用运行时常量readonly
标记版本号的值就应该使用运行时常量，因为他的值会随着每个不同版本的发布而改变
const优于readonly的地方仅仅是性能，使用已知的常量值要比访问readonly值略高一点，不过这点效率提升，可算是微乎其微

### 应仅仅在那些性能异常敏感，且常量的值在各个版本之间绝对不会变化时，再使用编译时常量

## 3.推荐使用is 或 as操作符，而不是强制类型转换

### is：检查一个对象是否兼容于其他指定的类型，并返回一个bool值，永远不会抛出异常
as:作用与强制类型转换是一样，但是永远不会抛出异常，不成功则返回null。更加安全也更加高效

### as和is仅当运行时类型符合目标类型时才能转换成功，也不会在转换时创建新的对象。

### as运算符对值类型是无效，此时可以使用is，配合强制类型转换进行转换

### 仅当不能使用as进行转换时，才应该使用is操作符。否则is就是多余的

## 4.推荐使用条件属性而不是#if条件编译

### private string ChackMethod()
{
#if DEBUG
      Trace.WriteLine("...");
      string methodName = new StackTrace().GetFrame(1).GetMethod.Name;
      return methodName;
#endif
      return null;
}

--->

[Conditional[“DEBUG”]]
private void CheckMethod()
{
      Trace.WriteLine("...");
      string methodName = new StackTrace().GetFrame(1).GetMethod.Name;
}

### #if/#endif容易被滥用，其代码页进而难以理解或调试。
Conditional特性，用来为不同的环境编译不同的机器码。该特性适用于方法的层面，这将强制我们将条件代码拆分为独立的方法。
Conditional特性最常用的地方就是将一段代码变成调试语句。使用该特性的隔离策略比#if/#endif不容易出错

### Conditional特性只可以应用在整个方法上。
任何使用了Conditional特性的方法都只能返回void类型

## 5.理解几个等同性判断之间的关系

### WHY？
C#可以创建两种类型：值类型和引用类型。
如果两个引用类型的变量指向的是同一个对象，它们将被认为是“引用相等“。
如果两个值类型的变量类型相同，而且包含统一的内容，它们认为是”值相等“。这也是等同性判断需要如此多方法的原因。
一共提供4种不同的函数来判断两个对象是否”相等“

- public static bool ReferenceEquals(object left,object right);
判断两个不同变量的对象标识是否相等。无论比较的是引用类型还是值类型，该方法判断的依据都是对象标识，而不是对象内容

- public static bool Equals(object left,object right);
用于判断两个变量的运行时类型是否相等

- public virtual bool Equals(object right);用于重载

- public static bool operator ==(MyClass left,MyClass right);用于重载

### 对于值类型，我们应该总是覆写Object.Equals()实例方法和operator==(),以便为其提供效率较高的等同性判断。对于引用类型，仅当认为相同的含义并非是对象标识相等时，才需要覆写Object.Equals()实例方法。在覆写Equals()时也要实现IEquatable<T>

- public calss foo:IEquatable<foo>
{
    public override bool Equals(object right)
    { 
         //是否为null
         if(Object.ReferenceEquals(right,null))
               return false;
         //是否引用相等
         if(Object.ReferenceEquals(this,right))
               return true; 
         //可能是子类，所以需要精确的类型判断
         if(this.GetType() != right.GetType())
              return false;
         //调用实例的Equals方法
         return this.Equals(right as foo);
      }


     // IEquatable<foo>成员
     public bool Equals(foo other)
     { 
           ...
           return true;
     }
}

### 引用类型应尽量避免覆写operator==(); .NET希望所有引用类型都应用的operator==()都遵循引用语义，因为系统提供的默认版本时通过比较两个值类型实例的内容，并且是用反射来实现的

## 6.明白GetHashCode()的一些陷阱

### GetHashCode()只用在一个地方：定义基于HashKey集合，典型的有HashSet<T>或者Dictionary<K,V>容器。

### 在.NET中，每个对象都有一个散列码，其值由System.Object.GetHashCode()决定

### HOW? 遵循下述三条原则：
1.如果两个对象相同（由operator==定义），那么它们必须生成相同的散列码。否则，这样的散列码将无法用来查找容器中的对象
2.对于任何一个对象A，A.GetHashCode()必须保持不变
3.对于所有的输入，散列函数勇敢在所有整数中按随机分别生成散列码。这样的散列容器才能得到足够的效率提升

## 7.理解短小方法的优势

### 将C#代码翻译成可执行的机器码需要两个步骤
C#编译器将生成IL，并放在程序集中，随后JIT将根据需要逐一为方法（或是一组方法，如果涉及内联）生成机器码。
短小的方法让JIT编译器能够更好地平摊编译的代价，也更适合内联
除了短小之外，简化控制流程也很重要。控制分支越少，JIT编译器也会越容易地找到最适合放在寄存器中的变量
其优势，不仅体现在代码的可读性上，还关系到程序运行时的效率

## 8.选择变量初始化而不是赋值语句，类似C++

## 8.选择变量初始化而不是赋值语句，类似C++

### 初始化器将在所有构造函数执行之前执行

-     public class Dog
    {
        private int age;
        private string name;

        public Dog(int age)
        {
            Console.WriteLine("Hello from Dog's non-parameterless constructor");
            this.age = age;
        }

        public required string Name
        {
            get { return name; }

            set
            {
                Console.WriteLine("Hello from setter of Dog's required property 'Name'");
                name = value;
            }
        }
    }

	-  public class Person
    {
        private string firstName;
        private string lastName;
        private string city;

        public Person()
        {
            Console.WriteLine("Hello from Person's parameterless constructor");
        }

        public required string FirstName
        {
            get { return firstName; }

            set
            {
                Console.WriteLine("Hello from setter of Person's required property 'FirstName'");
                firstName = value;
            }
        }

        public string LastName
        {
            get { return lastName; }

            init
            {
                Console.WriteLine("Hello from setter of Person's init property 'LastName'");
                lastName = value;
            }
        }

        public string City
        {
            get { return city; }

            set
            {
                Console.WriteLine("Hello from setter of Person's property 'City'");
                city = value;
            }
        }
    }

		- public static void Main()
    {
        new Person { FirstName = "Paisley", LastName = "Smith", City = "Dallas" };
        new Dog(2) { Name = "Mike" };
    }

- public class BaseballTeam
    {
        private string[] players = new string[9];
        private readonly List<string> positionAbbreviations = new List<string>
        {
            "P", "C", "1B", "2B", "3B", "SS", "LF", "CF", "RF"
        };

        public string this[int position]
        {
            // Baseball positions are 1 - 9.
            get { return players[position-1]; }
            set { players[position-1] = value; }
        }
        public string this[string position]
        {
            get { return players[positionAbbreviations.IndexOf(position)]; }
            set { players[positionAbbreviations.IndexOf(position)] = value; }
        }
    }

	- public static void Main()
    {
        var team = new BaseballTeam
        {
            ["RF"] = "Mookie Betts",
            [4] = "Jose Altuve",
            ["CF"] = "Mike Trout"
        };

        Console.WriteLine(team["2B"]);
    }

- public static void Main()
    {
        // Declare a StudentName by using the constructor that has two parameters.
        StudentName student1 = new StudentName("Craig", "Playstead");

        // Make the same declaration by using an object initializer and sending
        // arguments for the first and last names. The parameterless constructor is
        // invoked in processing this declaration, not the constructor that has
        // two parameters.
        StudentName student2 = new StudentName
        {
            FirstName = "Craig",
            LastName = "Playstead"
        };

        // Declare a StudentName by using an object initializer and sending
        // an argument for only the ID property. No corresponding constructor is
        // necessary. Only the parameterless constructor is used to process object
        // initializers.
        StudentName student3 = new StudentName
        {
            ID = 183
        };

        // Declare a StudentName by using an object initializer and sending
        // arguments for all three properties. No corresponding constructor is
        // defined in the class.
        StudentName student4 = new StudentName
        {
            FirstName = "Craig",
            LastName = "Playstead",
            ID = 116
        };

        Console.WriteLine(student1.ToString());
        Console.WriteLine(student2.ToString());
        Console.WriteLine(student3.ToString());
        Console.WriteLine(student4.ToString());
    }

	- public class StudentName
    {
        // This constructor has no parameters. The parameterless constructor
        // is invoked in the processing of object initializers.
        // You can test this by changing the access modifier from public to
        // private. The declarations in Main that use object initializers will
        // fail.
        public StudentName() { }

        // The following constructor has parameters for two of the three
        // properties.
        public StudentName(string first, string last)
        {
            FirstName = first;
            LastName = last;
        }

        // Properties.
        public string? FirstName { get; set; }
        public string? LastName { get; set; }
        public int ID { get; set; }

        public override string ToString() => FirstName + "  " + ID;
    }

## 9.正确地初始化静态成员变量

### C#提供了有静态初始化器和静态构造函数来专门用于静态成员变量的初始化

### 静态构造函数是一个特殊的函数，用于初始化任何静态数据，或执行仅需执行一次的特定操作。 它会在创建第一个实例或引用任何静态成员之前自动调用。 静态构造函数最多调用一次

- class SimpleClass
{
    // Static variable that must be initialized at run time.
    static readonly long baseline;

    // Static constructor is called at most one time, before any
    // instance constructor is invoked or member is accessed.
    static SimpleClass()
    {
        baseline = DateTime.Now.Ticks;
    }
}

### 有多个操作在静态初始化时执行。 这些操作按以下顺序执行：

静态字段设置为 0。 通常由运行时执行此初始化。
静态字段初始值设定项运行。 派生程度最高类型的静态字段初始值设定项运行。
基类型静态字段初始值设定项运行。 以直接基开头从每个基类型到 System.Object 的静态字段初始值设定项。
所有静态构造函数运行。 任何静态构造函数都会运行，从 Object.Object 的最终基类到每一个基类，再到类型。 静态构造函数执行的顺序没有指定。 但是，在创建任何实例之前，层次结构中的所有静态构造函数都会运行。

### 静态构造函数在创建任何实例之前运行的规则有一个重要例外。 如果静态字段初始值设定项创建了该类型的实例，那么该初始值设定项（包括对实例构造函数的任何调用）将在静态构造函数运行之前运行。 这在单一实例模式中最为常见，如以下示例所示：

- public class Singleton
{
    // Static field initializer calls instance constructor.
    private static Singleton instance = new Singleton();

    private Singleton()
    { 
        Console.WriteLine("Executes before static constructor.");
    }

    static Singleton()
    { 
        Console.WriteLine("Executes after instance constructor.");
    }

    public static Singleton Instance => instance;
}

### 使用静态构造函数而不是静态初始化器最常见的理由就是处理异常。在使用静态初始化器时，我们无法自己捕获异常。而在静态构造函数中却可以做到

## 10.使用构造函数链（减少重复的初始化逻辑）

## 11.实现标准的销毁模式

### 这里有一些规则，可以帮你尽量降低GC的工作量：
1）若某个引用类型（值类型无所谓）的局部变量用于被频繁调用的例程中，那么应该将其提升为成员变量。
2）为常用的类型实例提供静态对象。
3）创建不可变类型的最终值。比如string类的+=操作符会创建一个新的字符串对象并返回，多次使用会产生大量垃圾，不推荐使用。对于简单的字符串操作，推荐使用string.Format。对于复杂的字符串操作，推荐使用StringBuilder类。

## 12.区分值类型和引用类型

### C#中，class对应引用类型，struct对应值类型。
C#不是C++，不能将所有类型定义成值类型并在需要时对其创建引用。C#也不是Java，不像Java中那样所有的东西都是引用类型。你必须在创建时就决定类型的表现行为，这相当重要，因为稍后的更改可能带来很多灾难性的问题。

### 值类型无法实现多态，因此其最佳用途就是存放数据。引用类型支持多态，因此用来定义应用程序的行为。

### 应该创建struct值类型：
1）该类型主要职责在于数据存储吗？
2）该类型的公有接口都是由访问其数据成员的属性定义的吗？
3）你确定该类型绝不会有派生类型吗？
4）你确定该类型永远都不需要多态支持吗？

## 13.保证0为值类型的有效状态

### 在创建自定义枚举值时，请确保0是一个有效的选项。若你定义的是标志(flag)，那么可以将0定义为没有选中任何状态的标志（比如None）。即作为标记使用的枚举值（即添加了Flags特性）应该总是将None设置为0。

## 14.保证值类型的常量性和原子性

### 常量性的类型使得我们的代码更加易于维护。不要盲目地为类型中的每一个属性都创建get和set访问器。对于那些目的是存储数据的类型，应该尽可能地保证其常量性和原子性。

### 三种常量定义方式罗列一下，且越往后的方式越使得常量的原子性更强
public   struct  Animal
{
       public   string  Name
       {
               get ;
               set ;
          }           
       public   string  Age
        {
               get ;
               set ;
          }
      }

- 单线程情况下的实现形式：private保证了在初始化后，常量内部成员不被修改，但不能保证多线程同时初始化而产生的数据冲突。
public struct Animal2

      {
           public   string  Name
          {
               get ;
               private   set ;
          }
 
           public   string  Age
          {
               get ;
               private   set ;
          }
 
           public  Animal2( string  name,  string  age)
              :  this ()
          {
              Name  =  name;
              Age  =  age;
          }
      }

	- 多线程情况下的形式：readonly保证了只能在构造函数初始化的时候初始化一次，进而保证多线程不多次初始化，并且初始化之后不能被修改。
public struct Animal3
     {
         private readonly string _name;
         private readonly string _age;

         public string Name
         {
             get
 {
                 return _name;
             }
         }

         public string Age
         {
             get
             {
                 return _age;
             }
         }

         public Animal3( string name, string age)
             : this ()
         {
             this ._name = name;
             this ._age = age;
         }
     }

## 15.限制类型的可见性

### 在保证类型可以完成其工作的前提下。你应该尽可能地给类型分配最小的可见性。也就是，仅仅暴露那些需要暴露的。尽量使用较低可见性的类来实现公有接口。可见性越低，能访问你功能的代码越少，以后可能出现的修改也就越少。

## 16.通过定义并实现接口替代继承

### 理解抽象基类（abstract class）和接口（interface）的区别：

- 接口是一种契约式的设计方式，一个实现某个接口的类型，必须实现接口中约定的方法。抽象基类则为一组相关的类型提供了一个共同的抽象。也就是说抽象基类描述了对象是什么，而接口描述了对象将如何表现其行为。

- 接口不能包含实现，也不能包含任何具体的数据成员。而抽象基类可以为派生类提供一些具体的实现。

- 基类描述并实现了一组相关类型间共用的行为。接口则定义了一组具有原子性的功能，供其他不相关的具体类型来实现。

### 使用类层次来定义相关的类型。用接口暴露功能，并让不同的类型实现这些接口。

## 17.理解接口方法和虚方法的区别

### 实现接口和覆写虚方法之间的差别很大：
接口中声明的成员方法默认情况下并非虚方法，所以，派生类不能覆写基类中实现的非虚接口成员。若要覆写的话，将接口方法声明为virtual即可。
基类可以为接口中的方法提供默认的实现，随后，派生类也可以声明其实现了该接口，并从基类中继承该实现。
实现接口拥有的选择要比创建和覆写虚方法多。我们可以为类层次创建密封（sealed）的实现，虚实现或者抽象的契约。还可以创建密封的实现，并在实现接口的方法中提供虚方法进行调用。

### 总的来说实现接口要比重载virtual方法更具有灵活性。你可以sealed、virtual实现的方法或用 abstract来定义方法契约，甚至可以在实现接口的方法中调用virtual方法来获取子类对父类方法重载的多态控制，和父类对自己实现的统一控制。 父类的虚方法可以提供所有子类对该方法的默认实现，而接口提供了多种方法定义组合（因为可以被多继承），对外提供了更加灵活的暴露契约的方式。

## 18.用委托实现回调

### 在C#中，回调是用委托来实现的
当类之间有通信的需要，并且我们期望一种比接口所提供的更为松散的耦合机制时，委托便是最佳的选择。

### 由于回调和委托在C#中非常常用，以至于C#特地以lambda表达式的形式为其提供了精简语法

### 委托允许我们在运行时配置目标并通知多个客户对象。委托对象中包含一个方法的应用，该方法可以是静态方法，也可以是实例方法。也就是说，使用委托，我们可以和一个或多个在运行时联系起来的客户对象进行通信。

### 由于一些历史原因，.NET中的委托都是多播委托（multicast delegate）。多播委托调用过程中，每个目标会被依次调用。委托对象本身不会捕捉任何异常。因此，任何目标抛出的异常都会结束委托链的调用。

## 19.用事件模式实现通知

### 事件提供了一种标准的机制来通知监听者，而C#中的事件其实就是观察者模式的一个语法上的快捷实现。

### 事件是一种内建的委托，用来为事件处理函数提供类型安全的方法签名。任意数量的客户对象都可以将自己的处理函数注册到事件上，然后处理这些事件，这些客户对象无需在编译器就给出，事件也不必非要有订阅者才能正常工作

### 在C#中使用事件可以降低发送者和可能的通知接受者之间的耦合，发送者可以完全独立于接受者进行开发。

### 对于委托 delegate ，C# 衍生出了很多版本：deleage , Action , Func<> , Predicate<> 和 event 。

delegate: 就是原始的委托，可以理解为方法指针或方法的签名

Action: 是没有返回值的泛型委托

Func:有返回值的泛型委托

Predicate<>:返回值为 bool 类型的谓词泛型委托

event:对 delegate 的封装

至于，平常说的匿名函数和 Lambda 表达式跟具体函数一样都是委托的实现方式。

- public class Example
{
   public static void Main()
   {
      // Create an array of Point structures.
      Point[] points = { new Point(100, 200),
                         new Point(150, 250), new Point(250, 375),
                         new Point(275, 395), new Point(295, 450) };

      // Define the Predicate<T> delegate.
      Predicate<Point> predicate = FindPoints;

      // Find the first Point structure for which X times Y
      // is greater than 100000.
      Point first = Array.Find(points, predicate);

      // Display the first structure found.
      Console.WriteLine("Found: X = {0}, Y = {1}", first.X, first.Y);
   }

   private static bool FindPoints(Point obj)
   {
      return obj.X * obj.Y > 100000;
   }
}
// The example displays the following output:
//        Found: X = 275, Y = 395

	- public class Example
{
   public static void Main()
   {
      // Create an array of Point structures.
      Point[] points = { new Point(100, 200),
                         new Point(150, 250), new Point(250, 375),
                         new Point(275, 395), new Point(295, 450) };

      // Find the first Point structure for which X times Y
      // is greater than 100000.
      Point first = Array.Find(points, x => x.X * x.Y > 100000 );

      // Display the first structure found.
      Console.WriteLine("Found: X = {0}, Y = {1}", first.X, first.Y);
   }
}
// The example displays the following output:
//        Found: X = 275, Y = 395

## 20.避免返回对内部类对象的引用

### 若将引用类型通过公有接口暴露给外界，那么对象的使用者即可绕过我们定义的方法和属性来更改对象的内部结构，这会导致常见的错误。

### 共有四种不同的策略可以防止类型内部的数据结构遭到有意或无意的修改：
1）值类型。当客户代码通过属性来访问值类型成员时，实际返回的是值类型的对象副本。
2）常量类型。如System.String。
3）定义接口。将客户对内部数据成员的访问限制在一部分功能中。
4）包装器（wrapper）。提供一个包装器，仅暴露该包装器，从而限制对其中对象的访问。

## 21.仅用new修饰符处理基类更新

### 使用new操作符修饰类成员可以重新定义继承自基类的非虚成员。
new修饰符只是用来解决升级基类所造成的基类方法和派生类方法冲突的问题。
new操作符必须小心使用。若随心所欲的滥用，会造成对象调用方法的二义性

