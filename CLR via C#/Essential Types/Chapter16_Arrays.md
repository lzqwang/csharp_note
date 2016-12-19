# Arrays

数组可以有一维数组，多维数组以及 不规则数组(jagged Arrays)
> When possible, you should stick with single-dimensional, zero-based arrays, sometimes referred to as SZ arrays, or vectors. Vectors give the best performance because you can use specific Intermediate Language (IL) instructions

## Initializing Array Elements

初始化数组对象
```
string[] names= new string[] {"Louis","Veronica"};
```
> use C#’s implicitly typed local variable (var) feature to simplify the code a little

可以使用 var来简化
```
var names= new string[]{"Louis","Veronica"};
```
> use C#’s implicitly typed array feature to have the compiler infer the type of the array’s elements. Notice the following line has no type specified between new and [].

可以隐藏数据类型，让编译器根据数组元素的类型来确定数组的类型
```
var names=new []{"Louis","Veronica",null};
```
上面这段代码，编译器会根据数组元素的类型，找到最近匹配的数据类型，上面这个例子是两个String，一个null，null可以转为其他引用类型，所以编译器找到最佳匹配类型就是string
如果换成下面这个代码
```
var names=new []{"Louis","Veronica",123}; // Error	No best type found for implicitly-typed array
```
编译器根据数组元素的类型，找不到最佳匹配类型，上面这种情况唯一能匹配的类型是Object，但是这样需要对元素进行装箱操作
> C# compiler team thinks that boxing array elements is too heavy-handed for the compiler to do for you implicitly, and that is why the compiler issues the error

还可以有如下这种声明方式
```
string[] names={"Louis","Veronica"};

var names={"Louis","Veronica"};  // Error		Cannot initialize an implicitly-typed local variable with an array initializer
```

最后介绍了一个例子
```
// Using C#’s implicitly typed local, implicitly typed array, and anonymous type features:
var kids = new[] {new { Name="Louis" }, new { Name="Veronica" }};
// Sample usage (with another implicitly typed local variable):
foreach (var kid in kids)
  Console.WriteLine(kid.Name);
```

## Casting Arrays

> CLR allows you to implicitly cast the source array’s element type to a target type. For the cast to succeed, both array types must have the same number of
dimensions, and an implicit or explicit conversion from the source element type to the target element type must exist.
The CLR doesn’t allow the casting of arrays with value type elements to any other type. (However, by using the Array.Copy method, you can create a new array and populate its elements in order to obtain the desired effect.)

使用Array.Copy不仅可以将数组元素从source array copy to target array，同时还能正确的处理内存的重叠区域。看看copy以下几招
* ■■ Boxing value type elements to reference type elements, such as copying an Int32[] to an Object[].
* ■■ Unboxing reference type elements to value type elements, such as copying an Object[] to an Int32[].
* ■■ Widening CLR primitive value types, such as copying elements from an Int32[] to a Double[].
* ■■ Downcasting elements when copying between array types that can’t be proven to be compatible based on the array’s type, such as when casting from an Object[] to an IFormattable[]. If every object in the Object[] implements IFormattable, Copy will succeed.

## All Arrays Implicitly Implement IEnumerable, ICollection, and IList

>The CLR team didn’t want System.Array to implement IEnumerable<T>, ICollection<T>, and IList<T>, though, because of issues related to multi-dimensional arrays and non-zero–based arrays.Defining these interfaces on System.Array would have enabled these interfaces for all array types.
Instead, the CLR performs a little trick: when a single-dimensional, zero–lower bound array type is created, the CLR automatically makes the array type implement IEnumerable<T>, ICollection<T>, and IList<T> (where T is the array’s element type) and also implements the three interfaces for all of the array type’s base types as long as they are *reference types*.

```
Object
Array (non-generic IEnumerable, ICollection, IList)
  Object[] (IEnumerable, ICollection, IList of Object)
  String[] (IEnumerable, ICollection, IList of String)
  Stream[] (IEnumerable, ICollection, IList of Stream)
    FileStream[] (IEnumerable, ICollection, IList of FileStream)
  .
  . (other arrays of reference types)
  .
```
> Note that if the array contains value type elements, the array type will not implement the interfaces for the element’s base types.

数组元素的类型是引用类型或者值类型，会影响数组的继承关系

## Passing and  Returning Arrays

当Array作为参数传给方法或者作为返回值从方法返回的时候，需要考虑是否直接将Array传入或者返回，还是将Array 处理之后再传入/返回， 需要注意的是使用Array.Copy实际上是浅拷贝，也就是，通过这里的参数一样可以修改数组元素的value。
另外如果返回的数组元素是空的情况下，可以返回null或者length为0的数组，建议返回后者，这样调用方可以少写一个判断语句，避免造成空引用。

## Creating Non-Zero Lower Bound Arrays

通常在创建数组的时候只需要指定数组的维度以及长度，因为默认index都是从0开始的 C#中提供了一个方法可以创建出Non-Zero Lower Bound的数组，用到的方法如下
```
// I want a two-dimensional array [2005..2009][1..4].
Int32[] lowerBounds = { 2005, 1 };
Int32[] lengths = { 5, 4 };
Decimal[,] quarterlyRevenue = (Decimal[,])Array.CreateInstance(typeof(Decimal), lengths, lowerBounds);
```

Array.CreateInstance 这个方法有很多重载，使用的过程中可以查看VS的智能提示了解不同重载的用法

## Array InternalsVisibleTo

上面提到了Non-Zero lower bound Array，下面来看下Array的分类，分为以下两种
* ■■ Single-dimensional arrays with a lower bound of 0. These arrays are sometimes called SZ (for single-dimensional, zero-based) arrays or vectors.
* ■■ Single-dimensional and multi-dimensional arrays with an unknown lower bound.

SZ和SNZ的区别是，两者的type有所区别
比如声明一个String数组，如果是SZ 那么它的type显示出来是 String[] ; 而SNZ的type是 String[\*]
星号告诉CLR编译器这个数组不是一个SZ，不能用C#的语法来直接获取数组元素

> Accessing the elements of a single-dimensional, zero-based array is slightly faster than accessing the elements of a non-zero–based, single-dimensional array or a multi-dimensional array

访问SZ/vectors 要不访问SNZ或则会多维数组要高一些

> First, there are specific IL instructions—such as newarr, ldelem, ldelema, ldlen, and stelem—to manipulate single-dimensional, zero-based arrays, and these special IL instructions cause the JIT compiler to emit optimized code.
Second, in common situations, the JIT compiler is able to hoist the index range–checking code out of the loop, causing it to execute just once

> So if performance is a concern to you,you might want to consider using an array of arrays (a jagged array) instead of a rectangular array.

## Unsafe Array Access and Fixed-Size Array

Unsafe array access is very powerful because it allows you to access:
* ■■ Elements within a managed array object that resides on the heap (as the previous section demonstrated).
* ■■ Elements within an array that resides on an unmanaged heap. The SecureString example in Chapter 14, “Chars, Strings, and Working with Text,” demonstrated using unsafe array access on an array returned from calling the System.Runtime.InteropServices.Marshal class’s SecureStringToCoTaskMemUnicode method.
* ■■ Elements within an array that resides on the thread’s stack

> The *stackalloc* statement can be used to create a single-dimensional, zero-based array of value type elements only, and the value type must
not contain any reference type fields.

To embed an array directly inside a structure, there are several requirements:
* ■■ The type must be a structure (value type); you cannot embed an array inside a class (reference type).
* ■■ The field or its defining structure must be marked with the unsafe keyword.
* ■■ The array field must be marked with the fixed keyword.
* ■■ The array must be single-dimensional and zero-based.
* ■■ The array’s element type must be one of the following types: Boolean, Char, SByte, Byte, Int16, UInt16, Int32, UInt32, Int64, UInt64, Single, or Double.

```
using System;
public static class Program {
  public static void Main() {
    StackallocDemo();
    InlineArrayDemo();
  }
  private static void StackallocDemo() {
    unsafe {
      const Int32 width = 20;
      Char* pc = stackalloc Char[width]; // Allocates array on stack
      String s = "Jeffrey Richter"; // 15 characters
      for (Int32 index = 0; index < width; index++) {
      pc[width - index - 1] =
      (index < s.Length) ? s[index] : '.';
      }
      // The following line displays ".....rethciR yerffeJ"
      Console.WriteLine(new String(pc, 0, width));
    }
  }
  private static void InlineArrayDemo() {
    unsafe {
      CharArray ca; // Allocates array on stack
      Int32 widthInBytes = sizeof(CharArray);
      Int32 width = widthInBytes / 2;
      String s = "Jeffrey Richter"; // 15 characters
      for (Int32 index = 0; index < width; index++) {
      ca.Characters[width - index - 1] =
      (index < s.Length) ? s[index] : '.';
      }
      // The following line displays ".....rethciR yerffeJ"
      Console.WriteLine(new String(ca.Characters, 0, width));
    }
  }
}

internal unsafe struct CharArray {
  // This array is embedded inline inside the structure
  public fixed Char Characters[20];
}

```
