# Custom Attributes

> one of the most innovative features the Microsoft .NET Framework has to offer: custom attributes Custom attributes allow you to declaratively annotate your code constructs, thereby enabling special features. Custom attributes allow information to be defined and applied to almost any metadata table entry. This extensible metadata information can be queried at run time to dynamically alter the way code executes

Custom Attributes可以声明在class， method， field， property，定义一些信息在metadata table中，在运行时可以获取到这些定义的信息，从而动态的改变代码的运行逻辑。


## Using Custom Attributes

> The first thing you should realize about custom attributes is that they’re just a way to associate additional information with a target. The compiler emits this additional information into the managed module’s metadata. Most attributes have no meaning for the compiler; the compiler simply detects the attributes in the source code and emits the corresponding metadata.

custom attributes只是一种将格外的信息与target关联起来的一种方式，编译器将这些格外的信息生成到managed module的metadata中。

> The CLR allows attributes to be applied to just about anything that can be represented in a file’s metadata.Most commonly, attributes are applied to entries in the following definition tables: TypeDef(classes, structures, enumerations, interfaces, and delegates), MethodDef (including constructors),ParamDef, FieldDef, PropertyDef, EventDef, AssemblyDef, and ModuleDef.

```
using System;
[assembly: SomeAttr] // Applied to assembly  -- prefix is mandatory
[module: SomeAttr] // Applied to module -- prefix is mandatory

[type: SomeAttr] // Applied to type
internal sealed class SomeType<[typevar: SomeAttr] T> { // Applied to generic type variable
[field: SomeAttr] // Applied to field
public Int32 SomeField = 0;
[return: SomeAttr] // Applied to return value  -- prefix is mandatory
[method: SomeAttr] // Applied to method
public Int32 SomeMethod(
[param: SomeAttr] // Applied to parameter
Int32 SomeParam) { return SomeParam; }
[property: SomeAttr] // Applied to property
public String SomeProp {
[method: SomeAttr] // Applied to get accessor method
get { return null; }
}
[event: SomeAttr] // Applied to event
[field: SomeAttr] // Applied to compiler-generated field  -- prefix is mandatory
[method: SomeAttr] // Applied to compiler-generated add & remove methods
public event EventHandler SomeEvent;
}
```
可以在attribute前面加前缀用来表示该attribute的作用域，如果不写前缀的话，编译器会自动的识别对应的target，但是对于一些attribute编译器无法自动识别出来的，就必须要加前缀来说明

所有的Attributes都必须继承自System.Attribute。
基于所有的Attributes都是Class，那么每个Class都应该有构造函数，由于Attribute 声明的特殊性，c#中为Attribute的构造函数提供了特殊的语法
```
[DllImport("Kernel32", CharSet = CharSet.Auto, SetLastError = true)]
```
第一个参数为构造函数中必须的参数，成为 positional parameter，另外两个是public field或property，可以在这里赋值，当然也可以不赋值，这些成为 named parameter

可以为一个target指定多个attribute，当指定多个Attributes时，attributes的顺序不会造成任何影响，Attributes需要包含在[]中，如果是多个attributes，只需要在中间用逗号隔开不同的attributes也可以给每个attribute单独加到括号里，但是需要加小括号
```
[Serializable][Flags]
[Serializable, Flags]
[FlagsAttribute, SerializableAttribute]
[FlagsAttribute()][Serializable()]
```

## Defining Your Own Attribute Class

> You should think of an attribute as a logical state container. That is, while an attribute type is a class, the class should be simple.
The class should not offer any public methods, events, or other members

Custom Attribute本身也是一个class，所以定义个Custom Attribute class的时候，也可以为这个class声明attribute，AttributeUsageAttribute，
positional parameter是  AttributeTargets, 可用的枚举值如下
```
[Flags, Serializable]
public enum AttributeTargets {
Assembly = 0x0001,
Module = 0x0002,
Class = 0x0004,
Struct = 0x0008,
Enum = 0x0010,
Constructor = 0x0020,
Method = 0x0040,
Property = 0x0080,
Field = 0x0100,
Event = 0x0200,
Interface = 0x0400,
Parameter = 0x0800,
Delegate = 0x1000,
ReturnValue = 0x2000,
GenericParameter = 0x4000,
All = Assembly | Module | Class | Struct | Enum |
Constructor | Method | Property | Field | Event |
Interface | Parameter | Delegate | ReturnValue |
GenericParameter
}
```
另外还提供了两个属性 AllowMultiple and Inherited
AllowMultiple 控制同一个attribute是否可以在一个target上声明多次，
Inherited 控制attribute是否应用在已声明的class的子类或者重载方法 Inherited, indicates if the attribute should be applied to derived classes and overriding methods when applied on the base class.

```
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited=true)]
  internal class TastyAttribute : Attribute {
}
[Tasty][Serializable]
internal class BaseType {
  [Tasty] protected virtual void DoSomething() { }
}
internal class DerivedType : BaseType {
  protected override void DoSomething() { }
}
```
上面代码中TastyAttribute的Inherited属性为true， 所以DerivedType也应用了TastyAttribute，重载的DoSomething方法也算应用了Tasty属性
> Be aware that the .NET Framework considers targets only of classes, methods, properties, events, fields, method return values, and parameters to be inheritable. So when you’re defining an attribute type, you should set Inherited to true only if your targets include any of these targets

如果定义一个Custom Attribute但是没有声明AttributeUsage，编译器会使用AttributeUsage 的默认构造函数

## Attribute Constructor and Field/Property Data Types

>When defining an attribute class’s instance constructor, fields, and properties, you must restrict yourself to a small subset of data types. Specifically, the legal set of data types is limited to the following:Boolean, Char, Byte, SByte, Int16, UInt16, Int32, UInt32, Int64, UInt64, Single, Double,
String, Type, Object, or an enumerated type.

定义attibute 类的构造函数，变量，属性时，需要注意数据类型，可用的类型有 Boolean, Char, Byte, SByte, Int16, UInt16, Int32, UInt32, Int64, UInt64, Single, Double,
String, Type, Object, or an enumerated type。
另外一位数组也是可以使用的，但是尽量避免使用数组
> However, you should avoid using arrays because a custom attribute class whose constructor takes an array is not CLS-compliant

```
using System;
internal enum Color { Red }
[AttributeUsage(AttributeTargets.All)]
internal sealed class SomeAttribute : Attribute {
public SomeAttribute(String name, Object o, Type[] types) {
  // 'name' refers to a String
  // 'o' refers to one of the legal types (boxing if necessary)
  // 'types' refers to a 1-dimension, 0-based array of Types
  }
}
[Some("Jeff", Color.Red, new Type[] { typeof(Math), typeof(Console) })]
internal sealed class SomeType { }
```

> the best way for developers to think of custom attributes: instances of classes that have been serialized to a byte stream that resides in metadata.
Later, at run time, an instance of the class can be constructed by deserializing the bytes contained in the metadata. In reality, what actually happens is that the compiler emits the information necessary to create an instance of the attribute class into metadata.

## Detecting the Use of a Custom Attribute

Attribute本身是没有用的，使用Attribute是在运行时，而且需要针对自定义的attribute添加特殊的逻辑，否则code原来的逻辑不会发生任何改变。如何在运行时判断code中是否定义了Attribute，使用到了Reflection -- 反射

>So if you define your own attribute classes, you must also implement some code that checks for the existence of an instance of your attribute class (on some target) and then execute some alternate code path. This is what makes custom attributes so useful!

FCL提供了方法用来attribute是否存在
使用 IsDefined方法可以最快速的判断一个attribute是否定义了，因为这个方法不会去创建attribute 对象
另外两个方法 GetCustomAttributes 和 GetCustomAttribute 可以返回attribute对象

```
private static void ShowAttributes(MemberInfo attributeTarget) {
  var attributes = attributeTarget.GetCustomAttributes<Attribute>();
  Console.WriteLine("Attributes applied to {0}: {1}",  attributeTarget.Name, (attributes.Count() == 0 ? "None" : String.Empty));
  foreach (Attribute attribute in attributes) {
    // Display the type of each applied attribute
    Console.WriteLine(" {0}", attribute.GetType().ToString());
    if (attribute is DefaultMemberAttribute)
      Console.WriteLine(" MemberName={0}", ((DefaultMemberAttribute) attribute).MemberName);
    if (attribute is ConditionalAttribute)
      Console.WriteLine(" ConditionString={0}", ((ConditionalAttribute) attribute).ConditionString);
    if (attribute is CLSCompliantAttribute)
        Console.WriteLine(" IsCompliant={0}", ((CLSCompliantAttribute) attribute).IsCompliant);
    DebuggerDisplayAttribute dda = attribute as DebuggerDisplayAttribute;
    if (dda != null) {
      Console.WriteLine(" Value={0}, Name={1}, Target={2}",
      dda.Value, dda.Name, dda.Target);
    }
  }
  Console.WriteLine();
}
```

## Detecting the Use of a Custom Attribute Without Creating Attribute-Derived Objects

上面介绍的Detect custom attribute的方法，是会调用Attribute的构造函数，并且可以调用property的set 方法，在某些权限控制比较严格的地方，可以使用一种替代方式获取custom attribute而不需要调用attribute的构造函数，使用System.Reflection.CustomAttributeData class

```
private static void ShowAttributes(MemberInfo attributeTarget) {
  IList<CustomAttributeData> attributes = CustomAttributeData.GetCustomAttributes(attributeTarget);
  Console.WriteLine("Attributes applied to {0}: {1}",
  attributeTarget.Name, (attributes.Count == 0 ? "None" : String.Empty));
  foreach (CustomAttributeData attribute in attributes) {
    // Display the type of each applied attribute
    Type t = attribute.Constructor.DeclaringType;
    Console.WriteLine(" {0}", t.ToString());
    Console.WriteLine(" Constructor called={0}", attribute.Constructor);
    IList<CustomAttributeTypedArgument> posArgs = attribute.ConstructorArguments;
    Console.WriteLine(" Positional arguments passed to constructor:" +  ((posArgs.Count == 0) ? " None" : String.Empty));
    foreach (CustomAttributeTypedArgument pa in posArgs) {
      Console.WriteLine(" Type={0}, Value={1}", pa.ArgumentType, pa.Value);
    }
    IList<CustomAttributeNamedArgument> namedArgs = attribute.NamedArguments;
    Console.WriteLine(" Named arguments set after construction:" +  ((namedArgs.Count == 0) ? " None" : String.Empty));
    foreach(CustomAttributeNamedArgument na in namedArgs) {
      Console.WriteLine(" Name={0}, Type={1}, Value={2}",  na.MemberInfo.Name, na.TypedValue.ArgumentType, na.TypedValue.Value);
    }
    Console.WriteLine();
  }
  Console.WriteLine();
}

```
## Conditional Attribute Classes

鉴于一些Attribute class只是应用于某些特定的场景，比如使用FxCop的时候检测的attribute只是在使用这个tool的时候有用，在code实际运行的时候是没有任何用处的，但是这个attribute的存在却增加了metadata的size从而影响效率。所幸，针对这种情况可以使用 System.Diagnostics.ConditionalAttribute 有条件的选择要用的attribute

```
[Conditional("TEST")][Conditional("VERIFY")]
public sealed class CondAttribute : Attribute {
}
[Cond]
public sealed class Program {
  public static void Main() {
    Console.WriteLine("CondAttribute is {0}applied to Program type.",
    Attribute.IsDefined(typeof(Program),
    typeof(CondAttribute)) ? "" : "not ");
  }
}
```

编译器只要在TEST和VERIFY标识下才会生出attribute的信息，但是attribute class的定义和实现则需要存在于assembly中
> When a compiler sees an instance of the CondAttribute being applied to a target, the compiler will emit the attribute information into the metadata only if the TEST or VERIFY symbol is defined when the code containing the target is compiled. However, the attribute class definition metadata and implementation is still present in the assembly.




## Matching Two Attribute Instances Against Each Other

获取到了Attribute之后，就需要判断Attribute里的field或property的值。System.Attribute 提供了Equals 方法用来判断 Attribute对象是否相等。
> If the types are identical, then Equals uses reflection to compare the values of the two attribute objects’ fields (by calling Equals for each field). If all the fields match, then true is returned; otherwise, false is returned. You might override Equals in your own attribute class to remove the use of reflection, improving performance.

除了 Equals 方法之外，FCL还提供了一个virtual method -- Match用来判断两个attribute对象是否匹配，默认的Match方法就是调用Equals方法然后返回结果，可以重载Match方法来实现自己定义的逻辑

```
[AttributeUsage(AttributeTargets.Class)]
internal sealed class AccountsAttribute : Attribute {
  private Accounts m_accounts;
  public AccountsAttribute(Accounts accounts) {
    m_accounts = accounts;
  }

  public override Boolean Match(Object obj) {
    // If the base class implements Match and the base class
    // is not Attribute, then uncomment the following line.
    // if (!base.Match(obj)) return false;
    // Since 'this' isn't null, if obj is null,
    // then the objects can't match
    // NOTE: This line may be deleted if you trust
    // that the base type implemented Match correctly.
    if (obj == null) return false;
    // If the objects are of different types, they can't match
    // NOTE: This line may be deleted if you trust
    // that the base type implemented Match correctly.
    if (this.GetType() != obj.GetType()) return false;
    // Cast obj to our type to access fields. NOTE: This cast
    // can't fail since we know objects are of the same type
    AccountsAttribute other = (AccountsAttribute) obj;
    // Compare the fields as you see fit
    // This example checks if 'this' accounts is a subset
    // of others' accounts
    if ((other.m_accounts & m_accounts) != m_accounts)
    return false;
    return true; // Objects match
  }

  public override Boolean Equals(Object obj) {
    // If the base class implements Equals, and the base class
    // is not Object, then uncomment the following line.
    // if (!base.Equals(obj)) return false;
    // Since 'this' isn't null, if obj is null,
    // then the objects can't be equal
    // NOTE: This line may be deleted if you trust
    // that the base type implemented Equals correctly.
    if (obj == null) return false;
    // If the objects are of different types, they can't be equal
    // NOTE: This line may be deleted if you trust
    // that the base type implemented Equals correctly.
    if (this.GetType() != obj.GetType()) return false;
    // Cast obj to our type to access fields. NOTE: This cast
    // can't fail since we know objects are of the same type
    AccountsAttribute other = (AccountsAttribute) obj;
    // Compare the fields to see if they have the same value
    // This example checks if 'this' accounts is the same
    // as other's accounts
    if (other.m_accounts != m_accounts)
    return false;
    return true; // Objects are equal
  }

  // Override GetHashCode since we override Equals
  public override Int32 GetHashCode() {
    return (Int32) m_accounts;
  }
}
[Accounts(Accounts.Savings)]
internal sealed class ChildAccount { }
[Accounts(Accounts.Savings | Accounts.Checking | Accounts.Brokerage)]
internal sealed class AdultAccount { }

public sealed class Program {
  public static void Main() {
    CanWriteCheck(new ChildAccount());
    CanWriteCheck(new AdultAccount());
    // This just demonstrates that the method works correctly on a
    // type that doesn't have the AccountsAttribute applied to it.
    CanWriteCheck(new Program());
  }
  private static void CanWriteCheck(Object obj) {
    // Construct an instance of the attribute type and initialize it
    // to what we are explicitly looking for.
    Attribute checking = new AccountsAttribute(Accounts.Checking);
    // Construct the attribute instance that was applied to the type
    Attribute validAccounts = obj.GetType().GetCustomAttribute<AccountsAttribute>(false);
    // If the attribute was applied to the type AND the
    // attribute specifies the "Checking" account, then the
    // type can write a check
    if ((validAccounts != null) && checking.Match(validAccounts)) {
      Console.WriteLine("{0} types can write checks.", obj.GetType());
    } else {
      Console.WriteLine("{0} types can NOT write checks.", obj.GetType());
    }
  }
}
```
