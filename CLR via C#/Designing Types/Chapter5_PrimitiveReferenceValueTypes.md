# Primitive, Reference, and Value Types

  这一章节介绍C#中的原始类型(基本类型)，引用类型以及值类型

## Primitive Types

  编译器中默认支持的类型被称为基本类型, 而且编译器会直接将这些类型跟FCL中的Types进行匹配
  >Any data types the compiler directly supports are called primitive types. Primitive types map directly to types existing in the 112 PART II Designing Types Framework Class Library (FCL)

  从这个解释可以看出来，所谓的原始类型/基本类型，是对于编译器而言，换句话说是每个编程语言自己规定了哪些是基本类型，而CLR中是没有基本类型这个概念的，下面是C#语言规定的基本类型

  Primitive|FCL Type|CLS-Compliant|Description
  -----|----------|---|----------
  sbyte| System.SByte| No| Signed 8-bit value
  byte| System.Byte| Yes| Unsigned 8-bit value
  short| System.Int16| Yes| Signed 16-bit value
  ushort| System.UInt16| No| Unsigned 16-bit value
  int| System.Int32| Yes| Signed 32-bit value
  uint| System.UInt32| No| Unsigned 32-bit value
  long| System.Int64| Yes| Signed 64-bit value
  ulong| System.UInt64| No| Unsigned 64-bit value
  char| System.Char| Yes| 16-bit Unicode character (char never represents an 8-bit value as it would in unmanaged C++.)
  float| System.Single| Yes| IEEE 32-bit floating point value
  double| System.Double| Yes| IEEE 64-bit floating point value
  bool| System.Boolean| Yes| A true/false value
  decimal| System.Decimal| Yes| A 128-bit high-precision floating-point value commonly used for financial calculations in which rounding  errors can’t be tolerated. Of the 128 bits, 1 bit represents the sign of the value, 96 bits represent the value itself, and 8 bits represent the power of 10 to divide the 96-bit value by (can be anywhere from 0 to 28). The remaining bits are unused.
  string| System.String| Yes| An array of characters
  object| System.Object| Yes| Base type of all types
  dynamic| System.Object| Yes| To the common language runtime (CLR), dynamic is identical to object. However, the C# compiler allows dynamic variables to participate in dynamic dispatch by using a simplified syntax. For more information, see “The dynamic Primitive Type” section at the end of this chapter.

  >The C# language specification states, “As a matter of style, use of the keyword is favored over use of the complete system type name.” I disagree with the language specification; I prefer to use the FCL type names and completely avoid the primitive type names

  C#语言规范中声明，从风格上来讲，使用keyword要优于使用完整的Type Name，但是作者却持反对意见，原因在于什么呢
  1. 程序员会因为keyword的name和FCL中的Type Name而疑惑，比如说string vs String ,到底该用哪一个，还有一个例子是关于int的疑惑，有人认为int在32-bit 表示的是32-bit value而在64-bit 机器上表示的是64-bit value。其实int是和System.Int32匹配，不会跟操作系统有关系，作者认为使用FCL Type Name的话可以避免这些不必要的疑惑产生
  2. C#中的long 关键字和别的语言有冲突，甚至有的编程语言不把long视为关键字，这样一来对代码的可读性可能会有一定的影响，尤其是对于习惯于其他编程语言的人
  3. FCL中有许多方法是的名字中包含TypeName，这个时候使用Type Name比使用keyword 看起来更好理解
  4. 许多程序员专写C#程序而忘了CLR还支持其他的语言，比如C++中关于long的定义就会和C#产生冲突

  基本类型的类型转换，不必遵循上一章节提到的关于Casting Type的rule，必须是Type或者其子类才可以转
  C# 编译器对于基本类型做了一些格外的逻辑判断，只要能确保是安全的转换就可以隐式的直接转，比如数字类型，在不损失精度的情况下可以直接隐式转，而对于可能损失精度的转换则需要显式的操作。

### Checked and Unchecked Primitive Type Operations

  数字类型的运算的逻辑中涉及到对于overflow的处理，不同的编程语言对于overflow的处理也有差异，C#中在编译的时候可以让程序员选择是否检查overflow，
![Configure Overfllow Check](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/CheckOverflowinVS.png)

  默认的VS是不对于Overflow进行check的，如果出现overflow的情况呢，
  鉴于这个配置是作用于全局的，可能不够灵活，所以可以使用 checked 和 unchecked 来对局部的代码进行判断， 注意一点 checked 和 uncheced 可以写成作用域的形式， 但是只是对其中直接调用的方法有效，如果作用域中的调用了其他的函数，这个函数不会被checked或者unchecked所影响，而取决于函数实现中的checked或者unchecked

```
//第一种写法
Byte b=100;
b=checked((Byte)(b+200));

//第二种写法 , 对于CallOtherFunc 来说，checked是没有作用的
checked{  
  Byte b=100;
  b=(Byte)(b+200);
  b=CallOtherFunc();
}
```

关于数字运算以及checked/uncheckedde 使用，作者给出的建议是
* 尽量使用 signed data type 来替换 unsigned data type，因为大部分FCL中的函数返回值都是signed data， 而且 unsigned data type 不是 CLS-Compliant
* 如果不希望代码逻辑处理错误的输入数据，导致overflow的出现，在代码中适时的使用 checked
* 如果overflow是在代码中没有不好的影响，是可以接受的结果，那么可以使用unchecked
* 可以不使用 checked/unchecked，这样表示开发者不关心overflow这块，如果有异常抛出那还好，如果没有异常，那可能就操作错误数据了

作者的建议是对于效率影响不是要求特别高的情况，还是建议check overflow的，抛出异常总比代码中出现了错误的数据而继续使用要好的多。


## Reference Types & Value Types

  CLR中支持两种类型的Type， 引用类型和值类型。总体来说CLR中大部分都是引用类型，但是不能把所有类型都设计成引用类型，因为引用类型需要在 memory heap中分配资源，而且要为额外的fields分配资源，如果所有类型都是引用类型的话，效率肯定会差很多。
  而值类型不同于引用类型，值类型不需要从memory heap上分配资源创建出来，而是直接在thread stack上进行创建而且使用之后，可以自行销毁，不需要等待垃圾回收机制来处理。
  关于两者的定义如下

  > any type called a class is a reference type.  On the other hand, the documentation refers to each value type as a structure or an enumeration.

  class 都是引用类型， struct 和 enum 都是值类型
  之前说过所有的CLR Type都需要继承自System.Object， 其中System.ValueType 和 System.Enum 是继承自System.Object的两个抽象类，所用的struct都是继承自System.ValueType而所有的enum都是继承自System.Enum。

  什么情况下要使用值类型而取代引用类型
  * Type中不包含会改变Type中fields的方法
  * Type不继承于其他Type
  * Type不能被其他Type继承
  * Type的instances大小都很小(16 bytes左右)
  * Type的instances很大，但是不会作为函数的参数或者返回值

  > by default, arguments are passed by value

  这里关注下，函数的arguments默认是使用值传递的，  这个跟之后要讲到的ref/out有关

  Value Type和 Reference Type不同之处总结如下
  * Value Type 有 boxed 和 unboxed 两种表现形式， Ref Type只有boxed 一种
  * Value Type继承自System.ValueType， 这个基类重写了 下面三个函数，
  ```
        //  Indicates whether this instance and a specified object are equal.
        //
        // Parameters:
        //   obj:
        //     The object to compare with the current instance.
        //
        // Returns:
        //     true if obj and this instance are the same type and represent the same value;
        //     otherwise, false.
        [SecuritySafeCritical]
        public override bool Equals(object obj);
        //
        // Summary:
        //     Returns the hash code for this instance.
        //
        // Returns:
        //     A 32-bit signed integer that is the hash code for this instance.
        [SecuritySafeCritical]
        public override int GetHashCode();
        //
        // Summary:
        //     Returns the fully qualified type name of this instance.
        //
        // Returns:
        //     A System.String containing a fully qualified type name.
        public override string ToString();
  ```
    如果自己定义一个value type，考虑效率方面的因素，应该重写Equals 和 GetHasCode 两个方法
    > Due to performance issues with this default implementation, when defining your own value types, you should override and provide explicit implementations for the Equals and GetHashCode methods.

  * Value Type不能被继承，所以不能有virtual的methods，也不能有abstract methods，所有的methods默认都是 sealed
  * Value Type和 Reference Type 在初始化的时候也不同，Value Type的变量总是会有一个默认的初始值，而Ref Type初始化的时候是null，Value Type不会出现空引用的异常，因为跟没就没有pointer这一说，但是C#中提供了 Nullable<> 这个类， 可以实现类似的功能，但是本质上来说 只是包装了一层外壳
  * Value Type 变量在赋值的时候，执行的是 field-by-field的copy，而 Ref Type在赋值的时候，只是copy 一个memory address
  * 鉴于上一条所说，Ref Type可以出现多个变量指向同一个memory address，这样修改一个其他的变量会随之改变，而Value Type不会出现这种情况
  * Value Type 不是在memory heap上分配资源，可以在执行之后自行销毁，不需要等垃圾回收机制来处理

### How the CLR Controls the Layout of a Type’s Fields

  CLR中如何控制Type的field在memory上如何排列的，使用 StructLayout 这个attribute可以指定数据的排列存储方式
  > Microsoft’s C# compiler selects LayoutKind.Auto for reference types (classes) and LayoutKind.Sequential for value types (structures). It is obvious that the C# compiler team believes that structures are commonly used when interoperating with unmanagedcode, and for this to work, the fields must stay in the order defined by the programmer.However, if you’re creating a value type that has nothing to do with interoperability with unmanaged code, you could override the C# compiler’s default.

  LayoutKind.Auto -- CLR 自动选择最优的方式来存储
  LayoutKind.Sequential -- 按照fields声明的顺序来存储，主要是考虑到 structures可能用于和非托管代码的交互
  LayoutKind.Explicit -- 需要为每个field 指定FieldOffset的值

  总的来说，这个控制，可以优化CLR的存储，减少内存的使用量，但是实际上使用到的频率很小，因为现在内存一般都比较充足，这点优化可能效果上不是特别明显，不过对于特别消耗内存的时候，可以考虑下这里提供的方式来进行一定的优化

## Boxing and Unboxing Value Types

  上面提到过 Value Types有两种表现形式，boxed 和 unboxed，两种表现形式互相转换就是 boxing 和 unboxing的过程
  boxing： unboxed -> boxed Value Type 变成了Reference Type, 分以下三步
  >  
  1. Memory is allocated from the managed heap. The amount of memory allocated is the sizerequired by the value type’s fields plus the two additional overhead members (the type object pointer and the sync block index) required by all objects on the managed heap.
  2. The value type’s fields are copied to the newly allocated heap memory.
  3. The address of the object is returned. This address is now a reference to an object; the value type is now a reference type.

  unboxing： boxed -> unboxed, 相比于boxing， unboxing的开销要小一些
  >
  1.  The address of the fields in the boxed object is obtained. This process is called unboxing.
  2.  The values of these fields are copied from the heap to the stack-based value type instance.

  Internally, here’s exactly what happens when a boxed value type instance is unboxed
  >
  1. If the variable containing the reference to the boxed value type instance is null, a NullReferenceException is thrown.
  2. If the reference doesn’t refer to an object that is a boxed instance of the desired value type,an InvalidCastException is thrown.

  对于上面的第二条，boxed 和 unboxed 转换时，Type必须是一样的，Type Cast在unboxing的过程中是无效的

  C#不允许修改Boxed Value Type中的fields value，但是借助于Interface，可以实现类似的效果，但是并不推荐这么使用structure， 如之前提到过的，使用structure的首要条件是，不存在method修改其fields

  值类型在使用过程中，如果使用不当会产生很多次的boxing/unboxing 对内存和效率都有一定程度的影响，了解了Value Type的本质以及使用中可能出现的坑，才能更好的使用它

### Object Equality and Identity

  System.Object 提供的默认的Equals方法
  ```
  public class Object {
    public virtual Boolean Equals(Object obj) {
    // If both references point to the same object,
    // they must have the same value.
    if (this == obj) return true;
    // Assume that the objects do not have the same value.
    return false;
  }
}
```
  这个方法只是判断两个object是否指向同一为内存位置，这个equals 成为 identity，而不是 value equality.
  下面这个方法是理想的Equals的实现
```
public class Object {
  public virtual Boolean Equals(Object obj) {
    // The given object to compare to can't be null
    if (obj == null) return false;
    // If objects are different types, they can't be equal.
    if (this.GetType() != obj.GetType()) return false;
    // If objects are same type, return true if all of their fields match
    // Because System.Object defines no fields, the fields match
    return true;
  }
}
```
  但是由于CLR没有这么实现Equals方法，而Equals方法本身是可以被子类重写的，所以这个System.Object的Equals方法就不再会被使用，为了解决这个问题，System.Object 提供了一个静态方法, 用来判断 identity
```
public class Object {
  public static Boolean ReferenceEquals(Object objA, Object objB) {
    return (objA == objB);
  }
}
```
> You should always call ReferenceEquals if you want to check for identity (if two references point to the same object).
You shouldn’t use the C# == operator (unless you cast both operands to Object first) because one of the operands’ types could overload the == operator, giving it semantics other than identity

  System.ValueType 重写了Object的Equals 方法，提供了判断Value Equality的方式
  在判断的过程中有一步需要调用Type中包含的fields的Equals方法，实现这个逻辑使用到了Reflection，所以在一定程度上对效率会造成影响。所以建议读者在实现自己的Value Type的Equals方法时，不要调用Base.Equals.
  Equals四个属性
  * reflexive -- 自反,反身  x.Equals(x) is  true
  * symmetric -- 对称 x.Equals(y) and y.Equals(x) 结果相同
  * transitive -- 传递 x.Equals(y) is true, y.Equals(z) is true, then x.Equals(z) must return true
  * consistent -- 一致性 对比的两个值在比较的过程中不能被改变，而且返回值必须是true or false

  另外，还有两点关于Equals的
  1. Have the type implement the System.IEquatable<T> interface’s Equals method -- 实现IEquatable<T> 接口，这个接口支持泛型，在调用的时候可以省去类型转换这一步，对于Value Type来说，省去了boxing/unboxing 的开销，所以更推荐使用
  2. 运算符重载 == 和 != ,使用重写的Equals方法来进行运算符重载


## Object Hash Codes

  CLR提供了一个虚函数用来 GetHashCode， 同时建议如果重写了Equals方法那么也需要重写GetHashCode ，原因是对于Equals的两个对象来说，他们的HashCode也应该是相同的
  实现HashCode的几个原则
  * 使用一个好的随机分配的算法，提高Hash Table的效率
  * 可以调用基类的GetHashCode的方法，但是不建议调用Object和ValueType的
  * 实现的算法里至少要用到一个实例成员(instance field)
  * 使用到的instance field应该是恒定的，初始化的时候就有值，而且这个值在整个对象的生命周期中不再改变
  * 算法应该执行的越快越好
  * 具有相同value的对象返回的HashCode必须一样

  GetHashCode 取得的value不可以保存起来用于之后的逻辑，尤其是.NET Framework中提供的实现，因为随着版本的更新，GetHashCode的算法可能会改变，导致hashcode的值与之前缓存的出现不同

## The dynamic Primitive Type

  dynamic 提供了一种运行时获取Type实际类型的方式
  > you cannot write methods whose signature differs only by dynamic and Object

  不可以用dynamic和Object来区分两个method

  dynaic 声明的field，method parameter以及 method return type，在编译的时候, 编译器会把这种类型转成Object同时加一个 属性，但是
  > compiler converts this type to the System.Object type and applies an instance of System.Runtime.CompilerServices.DynamicAttribute to the field, parameter, or return type in metadata

  dynamic和var的区别

  > You cannot cast an expression to var but you can cast an expression to dynamic. You must explicitly initialize a variable declared using var, whereas you do not have to initialize a variable declared with dynamic.

  下面两段代码
  ```
    // use dynamic
    dynamic expObj = sender as ExpandoObject;
    Console.WriteLine("Name:{0}", expObj.Name); // expObj is a dynamic object， Name will be resolved at run time, no error occur in compile time
    // use var
    var expObj = sender as ExpandoObject;
    Console.WriteLine("Name:{0}", expObj.Name);  // Compile failed  
  ```
  使用var声明的dynamic对象，编译时会报错
  
  > Error：'System.Dynamic.ExpandoObject' does not contain a definition for 'Name' and no extension method 'Name' accepting a first argument of type //'System.Dynamic.ExpandoObject' could be found (are you missing a using directive or an assembly reference?)

  dynamic 由于是运行时执行，所以需要在运行时中有一个 runtime binders 来获知dynamic所对应的实际type，在C#中 runtime binders存在于 Microsoft.CSharp.dll，这个dll在 csc.rsp文件中默认会引用。而这个dll又会引用System.dll以及System.Core.dll 如果涉及到与COM 对象的交互还会引用 System.Dynamic.dll

  > And when the payload code executes, it generates dynamic code at run time; this code will be in an in-memory assembly called “Anonymously Hosted DynamicMethods Assembly.” The purpose of this code is to improve the performance of dynamic dispatch in scenarios where a particular call site is making many invocations using dynamic arguments that have the same runtime type.

  运行时执行 dynamic code实在内存中的“Anonymously Hosted DynamicMethods Assembly.” 这样做是为了提升效率。

  尽管如此，可以看到使用dynamic的开销还是存在的，所以在使用dynamic的时候 要考虑到效率的问题，是否必须要使用dynamic。

  > One of the limitations of dynamic is that you can only use it to access an object’s instance members because the dynamic variable must refer to an object.

  dynamic的一个限制是，只能访问Object的instance members，包括变量，属性以及方法，但是对于static的方法属性则无法使用，作者写了一个类来实现了这个功能，其中用到了reflection来实现。
  ```
internal sealed class StaticMemberDynamicWrapper : DynamicObject {  //继承自DynamicObject，重写其中的虚方法，在重写的方法中利用反射去获取到static的members
    private readonly TypeInfo m_type;
    public StaticMemberDynamicWrapper(Type type) { m_type = type.GetTypeInfo(); }
    public override IEnumerable<String> GetDynamicMemberNames() {
      return m_type.DeclaredMembers.Select(mi => mi.Name);
    }
    public override Boolean TryGetMember(GetMemberBinder binder, out object result) {
      result = null;
      var field = FindField(binder.Name);
      if (field != null) { result = field.GetValue(null); return true; }
      var prop = FindProperty(binder.Name, true);
      if (prop != null) { result = prop.GetValue(null, null); return true; }
      return false;
    }
    public override Boolean TrySetMember(SetMemberBinder binder, object value) {
      var field = FindField(binder.Name);
      if (field != null) { field.SetValue(null, value); return true; }
      var prop = FindProperty(binder.Name, false);
      if (prop != null) { prop.SetValue(null, value, null); return true; }
      return false;
    }
    public override Boolean TryInvokeMember(InvokeMemberBinder binder, Object[] args,  out Object result) {
      MethodInfo method = FindMethod(binder.Name);
      if (method == null) { result = null; return false; }
      result = method.Invoke(null, args);
      return true;
    }
    private MethodInfo FindMethod(String name, Type[] paramTypes) {
      return m_type.DeclaredMethods.FirstOrDefault(mi => mi.IsPublic && mi.IsStatic
      && mi.Name == name
      && ParametersMatch(mi.GetParameters(), paramTypes));
    }
    private Boolean ParametersMatch(ParameterInfo[] parameters, Type[] paramTypes) {
      if (parameters.Length != paramTypes.Length) return false;
      for (Int32 i = 0; i < parameters.Length; i++)
      if (parameters[i].ParameterType != paramTypes[i]) return false;
      return true;
    }
    private FieldInfo FindField(String name) {
      return m_type.DeclaredFields.FirstOrDefault(fi => fi.IsPublic && fi.IsStatic && fi.Name == name);
    }
    private PropertyInfo FindProperty(String name, Boolean get) {
      if (get)
      return m_type.DeclaredProperties.FirstOrDefault(
      pi => pi.Name == name && pi.GetMethod != null &&
      pi.GetMethod.IsPublic && pi.GetMethod.IsStatic);
      return m_type.DeclaredProperties.FirstOrDefault(
      pi => pi.Name == name && pi.SetMethod != null &&
      pi.SetMethod.IsPublic && pi.SetMethod.IsStatic);
    }
}
  ```

### ExpandoObject

  ExpandoObject是 System.Core.dll中提供的一个类，可以在运行时为这个object添加members

  ```
    // Summary:
    //     Represents an object whose members can be dynamically added and removed at
    //     run time.
    public sealed class ExpandoObject : IDynamicMetaObjectProvider, IDictionary<string, object>, ICollection<KeyValuePair<string, object>>, IEnumerable<KeyValuePair<string, object>>, IEnumerable, INotifyPropertyChanged
    {
        // Summary:
        //     Initializes a new ExpandoObject that does not have members.
        public ExpandoObject();
    }
  ```

  reference:[Building C# objects dynamically](https://www.oreilly.com/learning/building-c-objects-dynamically)

  使用ExpandoObject跟写js的感觉差不多了，省去了定义声明初始化的一些逻辑，但是对内存和效率以及程序的稳定性健壮性有一定的影响，所以在使用之前还好应该根据实际情况来权衡下利弊。
