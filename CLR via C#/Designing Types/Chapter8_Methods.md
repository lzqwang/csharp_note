# Methods

## Instance Constructors and Classes (Reference Types)

  类的构造函数不能被继承，所以不能被new virtual override 来修饰
  如果类的构造函数没有提供，编译器会使用默认的构造函数，调用基类的没有参数的构造函数
  ```
  public class SomeType {
  }
  //it is as though you wrote the code as follows.
  public class SomeType {
    public SomeType() : base() { }
  }
  ```

  但是如果基类没有无参的构造函数，上面这种默认的构造函数就会出现异常，在编译时会不通过
  另外如果类是abstract的，编译器生成的default 构造函数是protected，否则是public的
  对于static 类，编译器不会生成默认的构造函数

  注意在一些情况下可以创建出一个object而不是用构造函数，比如调用Object的 MemberwiseClone方法，还有在序列化反序列化的时候也不是使用构造函数来创建对象的

  不要在构造函数中调用虚函数
  下面是一个合理使用构造函数的例子，在声明field的时候不赋值，而是在一个common的 构造函数中赋值，然后其他的构造函数再去调用这个common 构造函数，因为，如果直接给field声明的时候赋值，会使得生成的IL变大。
  ```
  internal sealed class SomeType {
    // Do not explicitly initialize the fields here.
    private Int32 m_x;
    private String m_s;
    private Double m_d;
    private Byte m_b;
    // This constructor sets all fields to their default.
    // All of the other constructors explicitly invoke this constructor.
    public SomeType() {
      m_x = 5;
      m_s = "Hi there";
      m_d = 3.14159;
      m_b = 0xff;
    }
    // This constructor sets all fields to their default, then changes m_x.
    public SomeType(Int32 x) : this() {
      m_x = x;
    }
    // This constructor sets all fields to their default, then changes m_s.
    public SomeType(String s) : this() {
      m_s = s;
    }
    // This constructor sets all fields to their default, then changes m_x & m_s.
    public SomeType(Int32 x, String s) : this() {
      m_x = x;
      m_s = s;
    }
}
```

## Instance Constructors and Structures (Value Types)

  Value Type可以没有构造函数，编译器也不会默认的为Value Types创建出来构造函数，同时C#不允许定义无参的构造函数，编译的时候会出错
  > Error		Structs cannot contain explicit parameterless constructors

  注意这个异常是C#的限制而不是CLR的限制，如果使用其他的编程语言，是可以定义无参的构造函数

  Value Type中不可以直接在声明field的时候为field赋值
  ```
  internal struct SomeValType {
    // You cannot do inline instance field initialization in a value type.
    private Int32 m_x = 5;
  }
  ```

  如果是有参数的构造函数，在函数内部实现中必须为所有的data field赋值，否则编译时也会出错，这样的目的是保证fields先被赋值然后才能被使用。

  > Error		Field 'ConsoleApplication1.Structure.name' must be fully assigned before control is returned to the caller

  Value Type中的 this 表示的是Value Type的instance，可以read/write，但是在reference Type中this是readonly的

## Type Constructors

  Type Constructors相对于Instance Constructors而言，前者用来创建Type，后者用来创建instance.
  Type Constructor也是静态构造函数，一个Type中只能有一个静态构造函数而且不能有参数

  ```
  internal sealed class SomeRefType {
      static SomeRefType() {
      // This executes the first time a SomeRefType is accessed.
    }
  }
  internal struct SomeValType {
      // C# does allow value types to define parameterless type constructors.
      static SomeValType() {
      // This executes the first time a SomeValType is accessed.
    }
  }
  ```
  静态构造函数都是private(C# 编译器默认),而且不允许修改访问限制符

  > Note Because the CLR guarantees that a type constructor executes only once per AppDomain and is thread-safe, a type constructor is a great place to initialize any singleton objects required by the type.

  CLR保证了Type constructor在AppDomain中只运行一次，并且是线程安全的，所以适合在这里初始化 Type中的单例对象

  > In fact, because the CLR is responsible for calling type constructors, you should always avoid writing any code that requires type constructors to be called in a specific order.

  因为type constructor是由CLR来调用的，所以不要让其按照规定的顺序被调用

  >Note Although C# doesn’t allow a value type to use inline field initialization syntax for instance fields, it does allow you to use it for static fields. In other words, if you change the SomeType type above from a class to a struct, the code will compile and work as expected.

  上一节提到了Value Type不支持在声明field的时候直接赋值，但是对于 static fields，是可以直接声明+赋值的

  在C++中有构造函数和析构函数，在C#中有没有类似的析构函数机制呢，答案是否定的，Type unload 发生在 AppDomain unload的情况下，这种情况下 Type是unreachable的，GC会回收Type所占用的memory。
  > All is not lost, however. If you want some code to execute when an AppDomain unloads, you can register a callback method with the System.AppDomain type’s DomainUnload event.

  作者提到了在AppDomain的unload event中获取可以执行一些unload的操作。

## Operator Overload Methods

  运算符重载的方法
  运算符重载规定方法必须是 public and static
  CLR中并不支持运算符重载，所以这些都是C#编译器的功劳，下面两表是C# 中可重载的运算符
  C# Unary Operators and Their CLS-Compliant Method Names

C# Operator Symbol| Special Method Name |Suggested CLS-Compliant Method Name
--------|--------------|---------
\+ |op_UnaryPlus| Plus
\- |op_UnaryNegation |Negate
!| op_LogicalNot |Not
~ |op_OnesComplement |OnesComplement
++ |op_Increment |Increment
-- |op_Decrement |Decrement
(none)| op_True |IsTrue { get; }
(none)| op_False |IsFalse { get; }

C# Binary Operators and Their CLS-Compliant Method Names

C# Operator Symbol| Special Method Name |Suggested CLS-Compliant Method Name
-----------|-------------|----------
\+ |op_Addition| Add
\- |op_Subtraction |Subtract
\* |op_Multiply |Multiply
/ |op_Division| Divide
% |op_Modulus| Mod
& |op_BitwiseAnd |BitwiseAnd
\|| op_BitwiseOr |BitwiseOr
^ |op_ExclusiveOr |Xor
<< |op_LeftShift |LeftShift
>> | op_RightShift |RightShift
==| op_Equality |Equals
!|  op_Inequality |Equals
<| op_LessThan |Compare
>| op_GreaterThan |Compare
<=| op_LessThanOrEqual| Compare
>=| op_GreaterThanOrEqual |Compare

  除了运算符重载，C#中还提供了类型转换重载方法
  默认的C#中的一些基本类型可以进行隐式或者显式的类型转换，对已自定义的Type，也可以自己实现这两种方式的转换
  > method. In C#, you use the implicit keyword to indicate to the compiler that an explicit cast doesn’t have to appear in the source code in order to
emit code that calls the method. The explicit keyword allows the compiler to call the method only when an explicit cast exists in the source code.

  使用 implicit 来声明可以隐式转换的方法， 使用 explicit来声明需要显式转换的方法
  下面看例子
```
public sealed class Rational {
    // Constructs a Rational from an Int32
    public Rational(Int32 num) { ... }
    // Constructs a Rational from a Single
    public Rational(Single num) { ... }
    // Converts a Rational to an Int32
    public Int32 ToInt32() { ... }
    // Converts a Rational to a Single
    public Single ToSingle() { ... }
    // Implicitly constructs and returns a Rational from an Int32
    public static implicit operator Rational(Int32 num) {
      return new Rational(num);
    }
    // Implicitly constructs and returns a Rational from a Single
    public static implicit operator Rational(Single num) {
      return new Rational(num);
    }
    // Explicitly returns an Int32 from a Rational
    public static explicit operator Int32(Rational r) {
      return r.ToInt32();
    }
    // Explicitly returns a Single from a Rational
    public static explicit operator Single(Rational r) {
      return r.ToSingle();
    }
}

// How to use the methods

public sealed class Program {
  public static void Main() {
    Rational r1 = 5; // Implicit cast from Int32 to Rational
    Rational r2 = 2.5F; // Implicit cast from Single to Rational
    Int32 x = (Int32) r1; // Explicit cast from Rational to Int32
    Single s = (Single) r2; // Explicit cast from Rational to Single
  }
}
```
> the metadata for the four conversion operator methods looks like this.
public static Rational op_Implicit(Int32 num)
public static Rational op_Implicit(Single num)
public static Int32 op_Explicit(Rational r)
public static Single op_Explicit(Rational r)

   注意上面最后的两个方法，只有返回类型不同，这种语法在CLR中是支持的，但是在大多数编程语言中是不支持的包括c#
   当使用显式类型转换的时候，C#会生成调用显式类型转换的方法，但是在使用as  和 is的时候不会。
