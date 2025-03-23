# C++与C#区别整理

## C++存在“不明确（未定义）行为”

## Java和.Net语言都提供Interfaces为语言元素，但C++没有

## C++对线程（threads）没有任何意念

### 线程安全性

## 转型动作---C，Java,C#中转型casting比较必要且无法避免，也相对不危险，但C++不是

## C++可发生单一对象可能拥有一个以上的地址，但在C，Java，C#都不可能发生这种事

## Interface classes类似 Java和.NET的Interfaces,但C++的 并不需要负担Java和.NET的所需负担的责任。

### Java和.NET都不允许在接口类中实现 成员变量或成员函数，但C++并不禁止

## Java和.NET内置”垃圾回收能力“，C++纯手工管理内存

### 两个主角：分配例程operator new 和归还例程 operator delete
配角：new-handler，当operator new无法满足客户的内存需求时所调用的函数

