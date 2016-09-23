# Constants and fields

## Constants
  > A constant is a symbol that has a never-changing value

  Constants表示恒定，在C#中之前介绍过的原始类型可以用来定义Contansts，包括
  > In C#, the following types are primitives and can be used to define constants: Boolean, Char, Byte, SByte, Int16, UInt16, Int32, UInt32, Int64, UInt64, Single, Double, Decimal, and String.

  这里值得注意的是String类型。 因为String类型有需要的处理字符串的方法，无论对String做了什么处理，原来的string value都不会改变，这点很重要

  对于不是原始类型，可以定义一个constant的 value -- null

  ```
  public sealed class SomeType {
    // SomeType is not a primitive type but C# does allow
    // a constant variable of this type to be set to 'null'.
    public const SomeType Empty = null;
  }
  ```

  因为Constants是不变的，所以Constants的value会直接写到 metadata中，也就是在编译完成之后value就被写到metadata中不会改变了，这一点也会重要
  > You can’t use constants if you need to have a value in one assembly picked up by another assembly at run time (instead of compile time). Instead, you can use readonly fields,

##　fields

  Fields的访问修饰符

CLR Term| C# Term |Description
-------|----------|---------
Static |static |The field is part of the type’s state, as opposed to being part of an object’s state.
Instance| (default)| The field is associated with an instance of the type, not the type itself.
InitOnly |readonly |The field can be written to only by code contained in a constructor method. -- 只能在构造函数中被赋值，之后不能修改其值
Volatile |volatile |Code that accessed the field is not subject to some thread-unsafe optimizations that may be performed by the compiler, the CLR, or by hardware. Only the following types can be marked volatile: all reference types, Single, Boolean, Byte, SByte, Int16, UInt16, Int32, UInt32, Char, and all enumerated types with an underlying type of Byte, SByte, Int16, UInt16, Int32, or UInt32. Volatile fields are discussed in Chapter 29, “Primitive Thread Synchronization Constructs.” -- 用于多线程同步，在多线程的章节详解

  > Compilers and verification ensure that readonly fields are not written to by any method other than a constructor. Note that reflection can be used to modify a readonly field

  注意反射可以修改一个readonly field

  关于const和static readonly经常会被用于比较，其实const是static的这里没有疑问，但是readonly是在运行时获取field value这点有别于const。

  > When a field is of a reference type and the field is marked as readonly, it is the reference that is immutable, not the object that the field refers to. The following code demonstrates.

  当声明一个reference type field为readonly时， 表示这个reference是不能修改的，但是referece to的object是可以修改的
