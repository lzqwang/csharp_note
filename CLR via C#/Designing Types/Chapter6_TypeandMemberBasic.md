## The Different Kinds of Type Members

  * Constants -- 常量， 与Type相关而不是Type instance, Constants都是Static members
  * Fields -- 可以是static 或者 instance, static field 表示 Type的state，而instance field表示的是object的state，作者强烈建议将fields设置为private，这样保证Type和Object的state不被其他code所破坏
  * Instance Constructors -- 实例构造函数是一个method，用来初始化object的instance fields。
  * Type constructors -- 用来初始化Type的static fields
  * Methods -- method可以用来修改Type的state(static method) 和 Object的state(instance method)，一般的方法可以读写fields
  * Operator overloads -- 运算符重载， 其实是一个method，定义了object对于指定的运算符执行的逻辑，注意不是所有语言都支持运算符重载，所以它属于 CLS(Common language specification)
  * Conversion operators -- A conversion operator is a method that defines how to implicitly or explicitly cast or convert an object from one type to another type. 与运算符重载类似，这个也不属于CLS
  * Properies -- A property is a mechanism that allows a simple, field-like syntax for setting or querying part of the logical state of a type (static property) or object (instance property) while ensuring that the state doesn’t become corrupt. Properties can be parameterless (very common) or parameterful (fairly uncommon but used frequently with collection classes) 提供一种类似于field的语法形式，可以获取或者修改fields
  * Events -- 事件提供了一种机制，当stete change的时候事件调用一些方法，通常与 delegate一起使用。
    > A static event is a mechanism that allows a type to send a notification to one or more static or instance methods. An instance (nonstatic) event is a mechanism that allows an object to send a notification to one or more static or instance methods. Events are usually raised in response to a state change occurring in the type or object offering the event. An event consists of two methods that allow static or instance methods to register and unregister interest in the event. In addition to the two methods, events typically use a delegate field to maintain the set of registered methods    

  * Types -- Type内部还可以定义 Types，主要用来将庞大的Type进行拆分，把复杂的Type简单化

  注意上面这些Type Members是CLR中定义的，所以不论使用什么编程语言，最后都得生产IL，生成metatada，这些与CLR直接打交道的Type就只能是上面这几种，而CLR这里使用的metadata，正好实现了各种不同编程语言之间的互通有无，无缝连接。
  > Simply stated, metadata is the key to the whole Microsoft .NET Framework development platform; it enables the seamless integration of languages, types, and objects

## Type Visibility

  Type Visibility 可以设置为public或者internal， internal限制是同一个assembly中的type可以访问
  C#中提供Friend Assembly的机制，可以突破internal的访问限制
  ```
  [assembly:System.Runtime.CompilerServices.InternalsVisibleTo("Wintellect, PublicKey=12345678...90abcdef")]
  [assembly:System.Runtime.CompilerServices.InternalsVisibleTo("Microsoft, PublicKey=b77a5c56...1934e089")]
```

### Member Accessibility

  在Chapter1中已经提到过了 关于Type以及Type Member的访问控制，注意一下需要继承情况下的访问控制的情况
  > When a derived type is overriding a member defined in its base type, the C# compiler requires that the original member and the overriding member have the same accessibility.

  子类重写父类的member，必须保持相同的访问控制权限，注意这个是C#的限制而不是CLR的限制，CLR的限制如下

  > When deriving from a base class, the CLR allows a member’s accessibility to become less restrictive but not more restrictive.

  子类可以重写父类的访问控制,使访问限制更宽松，但不能使访问限制更严格。

## Static Classes

  Static Classes的几个限制
  * Static Class必须继承与System.Object,因为类的继承关系，只有在初始化了对象之后才有意义，静态类不能创建对象
  * 静态类不能实现接口的方法，因为接口的方法只能由对象来调用
  * 静态类只能定义静态成员, （fields, methods,properties and events）
  * 静态类不能用作为field， method parameter 以及local variables，这些都与对象有关

  静态类的主要用途是，用来作为工具类 比如 Math, Environment

## Partial Class, Structure and interface

  使用Partial的几种情况，
  * Same Type in different files, 每个开发可以负责开发自己的部分，最后生成一个Type
  * Same Type in Same File for diffenrent function， 可以从功能上来对Type进行划分，每个逻辑单元作为一个分部类
  * VS 中使用的 code  splitter， 常用的Asp.Net中的 applicaiton page，user control，等都有类似的应用

  需要注意的是 partial 只是C#编译器提供的一个语法，跟CLR没有关系，所以partial classes必须一起编译


## Component, Polymorphism and versioning

  C#中Assembly作为一个Component, 称为 Component Software Programming (CSP)
  Component可以升级，这里就用到了Version的概念，之前提到过 .NET 中的version 格式是 MajorVersion.MinorVersion.BuildNumber.RevisionNumber
  通过来说发布一个小版本，是为了修复之前版本的bug，会兼容之前的版本，而发布一个大的版本的话，则表示加了新的功能，可能不兼容之前的版本了。
  由于C#中提供了多态性，继承性等特性，在发布新版本需要考虑到之前的版本的兼容问题的时候，需要考虑到如何合理的使用这些特性。

  CLR调用virtual members时，使用的IL instruction有两个一个是 call 一个是 callvirt
  call 可以用于 调用 static， instance 以及virtual的 method
  callvirt 用于调用 instance 以及 virtual methods ，不包括static methods， callvirt会判断调用的对象是不是null，如果是null会抛异常，而由于这个判断的存在，大配置callvirt在效率上要比call 慢一些。
  作者的建议是在定义一个Class的时候，尽可能少用virtual，一个原因是callvirt和call的影响，另外一个更重要的原因是，使用virtual，子类可以继承重写这个方法，对于父类来说越少的virtual，就表示能更少的被子类所影响。

  关于合理的设计Type，作者给出了几点建议
  * 默认的应该将class编译成 sealed (C# compiler并不是这样设计的，需要显式的将class声明为sealed)，除非明确了这个class会被其他子类继承
  * 在class内部，把data fields定义为private
  * 在class内部，把method， property，events定义为private and nonvirtual，除非不得不暴露这个方法给外部
  * 一个OOP adage：面向对象格言-- when things get too complicated, make more types -- 但是不要在class中声明一个public的nested class
