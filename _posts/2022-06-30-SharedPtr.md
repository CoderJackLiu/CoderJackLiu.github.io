---
title: 【UE C++】SharedPtr
author: CoderOldSix
date: 2022-06-30 23:32:00 +0800
categories: [Blogging, UEC++]
tags: [UE4 ,UEC++,TSharedPtr]
pin: true
---

# UE C++智能指针

虚幻智能指针库 是共享引用(TSharedRef)、共享指针 (TSharedPtr)、弱指针(TWeakPtr)及其他相关辅助函数及类的自定义实现。 该实现是根据 C++0x 标准库的 shared_ptr 和 Boost 的智能指针建模的。

### 类型	描述

共享指针 (TSharedPtr)	引用计数的非侵入式的权威智能指针。

共享引用 (TSharedRef)	不能设置为null值的、引用计数的、非侵入式权威智能指针。

弱指针 (TWeakPtr)	引用计数的、非侵入式弱指针引用。

共享引用和共享指针的优点

### 优点	描述

简洁的语法	您可以像操作常规的C++指针那样来复制、解引用及比较共享指针。

防止内存泄露	当没有共享引用时资源自动销毁。

弱引用	弱指针允许您安全地检查一个对象是否已经被销毁。

线程安全	包含了可以通过多个线程安全地进行访问的"线程安全"版本。

普遍性	您几乎可以创建到 任何 类型的对象的共享指针。

运行时安全	共享引用永远不会为null，且总是可以进行解引用。

不会产生引用循环	使用弱引用来断开引用循环。

表明用途	您可以轻松地区分对象 拥有者 和 观察者 。

性能	共享指针的性能消耗最小。 所有操作所占时间都是固定的。

强大的功能	支持针'const'、前置声明的不完全类型、类型转换等。

内存	所占内存大小是C++指针在64-位系统中所占内存的二倍 (外加了一个共享的16字节的引用控制器。)

### 创建自定义库的原因

std::shared_ptr (and even tr1::shared_ptr)不是在所有平台上都可用。

使得在所有编译器和平台上有更加一致的实现。

可以和其他虚幻容器及类型无缝地协作。

更好地控制平台特性，包括线程处理和优化。

我们想提供线程安全的功能 (以获得好的性能)。

我们已经添加了我们自己的改进(MakeShareable、赋值为NULL等)。

在我们的实现中不需要也不希望出现异常。

我们想在性能方面有更多的控制权 (内联函数、内存、虚函数的应用等)。

更轻松地进行调试(自由的代码注释等)。

在不需要的时候倾向于不引入新的第三方依赖。

### 辅助类和函数

在库中，以类和函数的形式提供了几个辅助功能，以便使得应用智能指针变得更加容易、更加直观。

辅助功能	描述

#### 类

TSharedFromThis	您可以让您自己的类继承这个类，来从“this”获得而一个TSharedRef 。

#### 函数

MakeShareable()	用于通过C++指针初始化共享指针(启用隐式转换) 。

StaticCastSharedRef()	静态类型转换效用函数，一般用于向下类型转换为派生类型。

ConstCastSharedRef()	将一个'const(常量的)'引用转换为'mutable（可变的）'智能引用。

DynamicCastSharedRef()	动态类型转换效用函数，一般用于向下类型转换为派生类型。

StaticCastSharedPtr()	静态类型转换效用函数，一般用于向下类型转换为派生类型。

ConstCastSharedPtr()	将一个'const(常量的)'引用转换为'mutable（可变的）'智能指针。

DynamicCastSharedPtr()	动态类型转换效用函数，一般用于向下类型转换为派生类型。

#### 智能指针实现细节

虚幻智能指针库中的各种智能指针类型的实现在性能、内存等方面都具有一些共有的特性。

##### 性能

当考虑使用共享指针时要时刻记住性能问题。共享指针的执行速度一般是非常快的；然而，这并不意味着任何地方都使用它。它们非常适用于某些高层系统或者工具编程，但不是很适合底层的引擎/路径渲染。

共享指针的在性能方面的一些优势有：

所有操作所占时间都是固定的。

共享指针解引用的速度和C++指针一样快。

复制共享指针永远不会分配内存。

线程安全的版本是无锁的。

和Boost或STL相比，其实现更快。

共享指针在性能方面的劣势包括：

创建及复制指针所带来的性能消耗。

引用计数内务处理。

共享指针使用的内存比C++指针多。

引用控制器的额外的堆分配。

由多个共享指针引用的每个独立对象都有性能消耗。

弱指针访问速度比共享指针访问速度略慢。

##### 内存使用量

所有的共享指针 (TSharedPtr, TSharedRef, TWeakPtr)都占8个字节(当针对32-位系统编译时)，包括：

C++ 指针(无符号32位整型)

引用控制器指针 (无符号32位整型)

TSharedFromThis 也占8个字节，因为它内嵌了弱指针。

引用控制器对象占12个字节(当针对32-位系统编译时)，包括：

C++ 指针(无符号32位整型)

共享引用计数(无符号32位整型)

弱引用计数(无符号32位整型)

无论有多少个 共享指针/弱指针 引用一个对象，都仅为每个对象创建一个引用控制器。

##### 反射

共享指针是非侵入式的，这意味着该类本身不知道它是否属于另一个共享指针（或引用）。一般，这是可以的，但有时候您想把当前实例作为共享引用访问。这种情况的解决方法是让该类继承于 TSharedFromThis<> 。

通过继承 TSharedFromThis<> ，您可以使用 AsSharedRef() 方法来将'this'转换为一个共享引用。 这对于总是返回共享引用的类工厂是非常有用的。

```c++
class FAnimation : public TSharedFromThis<FMyClass>
{
    void Register()
    {
        // Access a shared reference to 'this'
        TSharedRef<FMyClass> SharedThis = AsSharedRef();

        // Class a function that is expecting a shared reference
        AnimationSystem::RegisterAnimation( SharedThis );
    }
}
```



### 类型转换

您可以轻松地对共享指针(及引用)进行类型转换。和C++中的指针一样，向上类型转换是隐式的。

针对常量型指针进行类型转换(很不好，但有时候需要这样做):

ConstCastSharedPtr( ... )

静态类型转换(通常用于向下类型转换为派生类指针):

StaticCastSharedPtr( ... )

不支持动态类型转换(没有运行时类型信息)。 此时，您可以使用上面的静态类型转换。

### 线程安全性

常规的共享指针仅在单线程上访问是安全的。如果您需要多线程访问共享指针，那么请使用指针类的线程安全版本：

TThreadSafeSharedPtr

TThreadSafeSharedRef

TThreadSafeWeakPtr

TThreadSafeSharedFromThis<>

由于这些版本存在自动引用计数，所以运行速度略慢，但其行为和常规的C++指针几乎一样。

读取和复制时总是线程安全的。

写入/重置时必须进行同步才能确保安全。

如果您知道永远不会有多个线程访问您的指针，那么请不要使用线程安全版本。

### 应用详情

SharedPointerTesting.h 文件 (位于 [UE4RootLocation]/Engine/Source/Runtime/Core/Public/Templates/) 包含了使用共享指针和共享引用的各种示例。

##### 技巧

一般，当向一个新共享指针中传入一个C++指针时，您应该使用操作符“new”。

当传入智能指针作为函数参数时请使用 TSharedRef 或 TSharedPtr ，而不是使用TWeakPtr。

智能指针的“线程安全”版本运行速度略慢 -- 仅在需要的时候使用。

您可以按照所期望的方式把共享指针前置声明为不完全类型!

将会隐式地转换兼容的共享指针类型 (比如，向上类型转换)。

您可以创建一个针对 TSharedRef< MyClass > 的类型定义，以方便书写。

为获得最佳性能，要最小化调用 TWeakPtr::Pin (or conversions to TSharedRef/TSharedPtr) 的次数。

如果您的类继承 TSharedFromThis ，那么它可以将其本身返回为共享引用。

要想把一个指针向下类型转换为一个派生对象类，请使用 StaticCastSharedPtr() 函数。

共享指针完全支持 const 对象！

使用 ConstCastSharedPtr() 函数，您可以把一个 const(常量) 共享指针转换为可变的共享指针。

在深堆栈帧中，应该把共享指针转换为C++引用。共享指针最好用于成员引用，而不是堆栈临时变量。

和C++指针不同，共享指针不能进行内存复制，所以在使用共享指针数组时一定要考虑到这个问题。

##### 限制

共享指针和虚幻对象(UObject类)不兼容 !

目前仅是具有正常析构器的类型 (没有自定义的删除功能)。

还不支持动态分配的数组 (比如 MakeSharable( new int32[20] ))。

不支持将 TSharedPtr/TSharedRef 隐式地转换为布尔值。

##### 和其他实现的区别

类型名称和方法名称和虚幻引擎的基础代码保持了更好的一致性。

线程安全功能是可选的，而不是强制要求。

TSharedFromThis 返回一个共享 引用，而不是共享 指针 。

略掉了某些功能 (比如 use_count() 、 unique(), 等)。

不允许异常 (略掉了所有的相关功能)。

还不支持自定义分配器和自定义的删除功能。

我们的实现支持不能为null的共享指针(TSharedRef)。

添加了一些其他功能，比如 MakeShareable 和 NULL 赋值。

---

​			"Time after time have given me new courage to face life cheerfully, have been Kindness, *Beauty, and Truth.* "   ---Albert Einstein

---

