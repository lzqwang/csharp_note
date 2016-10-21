# Generics

  Generics泛型，提供了一种算法重用的机制，泛型提供了下述的优势
  * Source Code Protection, 程序员不需要直接访问算法的source code就可以直接使用该算法
  * Type Safety，泛型在使用的时候，需要指定type类型，编译器会获知当前使用的type，如果在方法中使用了其他type会出现编译错误
  * Clearner Code, 因为type safety，所以使用泛型的时候可以省去很多类型转换
  * Better Performance，泛型中使用value type，不需要装箱拆箱的操作，可以减少垃圾回收的次数，对效率有一定的提升

## Generics in the Framework Class Library

> Microsoft recommends that programmers use the generic collection classes and now discourages use of the non-generic collection classes for several reasons. First, the non-generic collection classes are not generic, and so you don’t get the type safety, cleaner code, and better performance that you get when you use generic collection classes. Second, the generic classes have a better object model than the non-generic classes. For example, fewer methods are virtual, resulting in better performance,and new members have been added to the generic collections to provide new functionality.

  微软建议使用泛型的集合，首先非泛型的集合，失去了上面提到的使用泛型带来的好处，包括类型安全，代码整洁以及效率提升等
  另外使用泛型类型比非泛型类型提供了更好的对象模型，在泛型类型中可以少用一些虚函数，提高了效率的同时也对代码进行了封装。

## Generics Infrastructure

  泛型是在CLR 2.0版本中引入的，为了支持泛型微软需要大动干戈，需要修改 IL， JIT， 编程语言(C# VB...) ，VS Debuger， VS智能提示等等这些内容。

### Open and Closed Types
> A type with generic type parameters is called an open type, and the CLR does not allow any instance of an open type to be constructed

  通常泛型类都是Open Type，不能直接用Open Type来声明创建一个对象，只有指定了实际使用类型之后，将其变成Closed Type，才能创建对象。
  每个Open Type可以有多个closed type，而每个closed type有属于其本身的static field以及static constructor
  CLR提供了限制Generic type的机制，但是注意不能将generic type限制为枚举类型
### Generic Types and Inheritance
  泛型和继承，注意区分泛型和继承之间的关系。
  泛型类可以继承自其Base　Type。 然后这个泛型类作为OpenType又可以衍生出多个Closed　Type，这些Closed　Type也是Base Type的子类但不是Open Type的子类

### Generic Types Identities
  泛型在书写代码的时候 因为要包含 < > 这些符号，可能在书写上略显繁琐，处于代码简洁简单方面的考虑，可以使用using 来声明一个泛型类型的别名。也可以使用var 来声明一个变量虽然这样只是少写一次 < >
  但是千万不要整一个非泛型的class继承自泛型，然后再使用这个非泛型的class，这样做一来更罗嗦了，代码写着是简洁了但是逻辑确是冗余的，二来这个非泛型class和泛型class就不是同一个type了

## Generic Interfaces/ Delegates  
> ■■ Invariant Meaning that the generic type parameter cannot be changed. I have shown only invariant generic type parameters so far in this chapter.
  ■■ Contra-variant Meaning that the generic type parameter can change *from a class to a class derived from it*. In C#, you indicate contra-variant generic type parameters with the *in* keyword. Contra-variant generic type parameters can appear only in input positions such as a method’s argument.
  ■■ Covariant Meaning that the generic type argument can change *from a class to one of its base classes*. In C#, you indicate covariant generic type parameters with the *out* keyword. Covariant generic type parameters can appear only in output positions such as a method’s return type.

  泛型可以用于接口和委托中，使得泛型的应用更加广泛
  在泛型接口/委托中，会出现in/ out ，这两个分别称为逆变(contra-variant)和协变 (covariant)
  in -- contra-variant -- 一般用于方法参数中，表示传入的参数可以是声明类型的子类
  out -- covariant 一般用于方法的返回值，表示传出的返回值可以是声明的类型的父类。
  Contra-variant: 逆变-- in-- 可以将class变成其子类
  Covariant: 协变，out-class可以变成其父类
  这样的设计，可以使得一套泛型接口/委托可以被更多的复用
  使用了in /out 的话，方法参数中就不能使用ref 和 out了

>In general, a covariant type parameter can be used as the return type of a delegate, and contravariant type parameters can be used as parameter types.
For an interface, covariant type parameters can be used as the return types of the interface's methods, and contravariant type parameters can be used as the parameter types of the interface's methods.

  in/out (逆变/协变) 应用在接口和委托上的时候效果是相反的

  ```
  // example of covariant
  public interface IEnumerable<out T> : IEnumerable
  // usage :
  IEnumerable<Derived> d = new List<Derived>();
  IEnumerable<Base> b = d;

  // example of contra-variant
  public interface IComparable<in T>
  //usage :
  IComparable<Base> b= new Base();
  IComparable<Derived> d= b;
  ```  



## Constraints
  * Primary Constraint  可以指定0/1 , 从 class or struct 中 2选1
  * Second Constraints  可以指定0或多个接口作为制约
  * Constructor Constraints, 使用 new() 来约束泛型类型必须要有一个非抽象的无参数的构造函数
