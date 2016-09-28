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

## Extension methods

> what C#’s extension methods feature does. It allows you to define a static method that you can invoke using instance method syntax.

  C#拓展方法，允许开发者自定义可以被对象调用的静态方法

```
public static class StringBuilderExtensions {
  public static Int32 IndexOf(this StringBuilder sb, Char value) {
    for (Int32 index = 0; index < sb.Length; index++)
    if (sb[index] == value) return index;
    return -1;
  }
}
```
需要在static class中定义 static 方法，并且需要在方法中为第一个参数 加上 this，
编译器在编译代码时，当遇到一个对象调用了一个方法，首先会查找对象的Type中是否定义了该方法，如果有则调用，如果没有，则去static class中查找static的方法，该方法的第一个参数需要于此Type相同并且有this关键字标识
VS的智能提示会显式出拓展方法，并且在tooltip中会提示该方法是拓展的，这样在开发的过程中就可以很自然很方便的使用拓展方法了

### Rule and  Guideline

* C#中只有拓展方法，没有拓展属性拓展事件等等
* 拓展方法需要定义在非泛型静态的类中，对类名没有要求，对于方法要求最少有一个参数，而且只有第一个参数可以用this修饰
* 拓展方法不能定义在一个内部类中
* 为了提高效率，需要在使用拓展方法的类中，添加拓展方法所在的类的引用，使用 using classNamespace
* 如果不同的类中定义了相同的拓展方法，那么当对象调用这个方法的时候，编译器会出错，需要修改方法名使得两个拓展方法区别开，否则不能使用对象直接调用，只能显式的使用静态类来调用
* 有节制的使用拓展方法，因为拓展方法可以被一个类及其子类调用，如果给System.Object添加一个拓展方法，那么所有的类都会有这个方法，VS的智能提示会被此污染
* 拓展方法的一个versioning issue，如果在未来的版本中拓展的类本身实现了拓展方法，那么拓展方法将不会被调用，而是直接调用类中定义的方法

  拓展方法和正常的方法有一点区别在于，当调用的对象是null的时候，对于普通方法，会在进入方法之前就抛异常，而拓展方法则是在进入了方法内部之后才抛异常
  除了为类添加拓展方法，还可以为其他类型添加拓展方法
  1. 为接口添加拓展方法
```
public static void ShowItems<T>(this IEnumerable<T> collection) {
  foreach (var item in collection)
  Console.WriteLine(item);
}

//invoke
public static void Main() {
  // Shows each Char on a separate line in the console
  "Grant".ShowItems();
  // Shows each String on a separate line in the console
  new[] { "Jeff", "Kristin" }.ShowItems();
  // Shows each Int32 value on a separate line in the console
  new List<Int32>() { 1, 2, 3 }.ShowItems();
}
```
  *System.Linq.Enumerable* 是关于拓展方法的一个很好的应用，对 *IEnumerable<T>* 进行拓展
 2. 为委托添加拓展方法
```
public static void InvokeAndCatch<TException>(this Action<Object> d, Object o)
where TException : Exception {
  try { d(o); }
  catch (TException) { }
}
//And here is an example of how to invoke it.
Action<Object> action = o => Console.WriteLine(o.GetType()); // Throws NullReferenceException
action.InvokeAndCatch<NullReferenceException>(null); // Swallows NullReferenceException
```
3. 为枚举添加拓展方法，在枚举的章节详细介绍

  另外C#编译器可以声明一个委托指向对象的拓展方法
```
public static void Main () {
  // Create an Action delegate that refers to the static ShowItems extension method
  // and has the first argument initialized to reference the "Jeff" string.
  Action a = "Jeff".ShowItems;
  .
  .
  .
  // Invoke the delegate that calls ShowItems passing it a reference to the "Jeff" string.
  a();
}
```

### Extension attribute

当定义了一个拓展方法，C#编译器默认的会在这个方法，方法所在的类 以及所在的程序集上加一个attribute
```
// Defined in the System.Runtime.CompilerServices namespace
[AttributeUsage(AttributeTargets.Method | AttributeTargets.Class | AttributeTargets.Assembly)]
public sealed class ExtensionAttribute : Attribute {
}
```
通过这个attribute可以有助于编译器快速的查找拓展方法，上面的这个attribute定义在System.Core.dll 中，在编译的时候需要引用这个dll，即使代码中没有用到这个dll，但是在运行时不会再引用这个dll。

## Partial Methods

  Partial Methods的实现基于Partial Types, 但是与Partial Type又有所不同
  patial method 由两部分 一个是声明一个是实现，声明部分没有方法体，实现部分才有
  关于partial methods的 rule and  Guideline
* partial methods只能定义在partial class或者strut中
* partial methods的返回类型必须是void，参数中不能有out 类型的参数，可以有ref类型的，这里的限制主要是考虑到partial method可能不存在，在运行时使用会带来一些不必要的麻烦
* partial methods的两个方法必须有相同的方法名，参数
* 如果partial method的实现部分不存在，在代码中不能创建引用这个method的delegate
* partial methods默认都是private的， 而且C#编译器禁止显式的声明partial method为private

总的来说，分部类和分部方法应用的不是很广泛，除非一些特殊的情景，涉及到多人联合开发或者机器自动生成代码和人工后期实现的组合的时候可能会应用到
