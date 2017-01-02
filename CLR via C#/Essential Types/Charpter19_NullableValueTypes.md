# Nullable Value Types

Value Types是不可以为null的，这也是value type的之所以被称为value type，应为要有value。
但是在一些情况下value type不能为null这个特性却为编程带来了许多的不方便之处，比如数据库中的数字类型的column与c#中的数据结构中的数字类型就无法很好匹配，因为数据库中是可以允许column的value是null的。另外一个例子是C#与其他语言编写的程序交互时可能出现问题，JAVA中的Date类型是可以为null，但是C#中不可以，所以当两种语言所写的程序在交互的时候就可能出现问题。为了提高Value type的可用性，C#中出现了 Nullable value types--  System.Nullable<T> structure

```
Nullable<Int32> x = 5;
Nullable<Int32> y = null;
Console.WriteLine("x: HasValue={0}, Value={1}", x.HasValue, x.Value);
Console.WriteLine("y: HasValue={0}, Value={1}", y.HasValue, y.GetValueOrDefault());
```

C#提供了简单语法来声明Nullable Value Types, 在value type后面加上问号，同时也支持使用运算符
```
Int32? x = 5;
Int32? y = null;

private static void Operators() {
  Int32? a = 5;
  Int32? b = null;
  // Unary operators (+ ++ - -- ! ~)
  a++; // a = 6
  b = -b; // b = null
  // Binary operators (+ - * / % & | ^ << >>)
  a = a + 3; // a = 9
  b = b * 3; // b = null;
  // Equality operators (== !=)
  if (a == null) { /* no */ } else { /* yes */ }
  if (b == null) { /* yes */ } else { /* no */ }
  if (a != b) { /* yes */ } else { /* no */ }
  // Comparison operators (<> <= >=)
  if (a < b) { /* no */ } else { /* yes */ }
}
```
使用C#的简化语法使用Nullable Value Types的时候还需要注意，编译器实际生成的IL要比我们所写的代码量要多的，因为IL中没有 ? 表示Nullable， 所以所有的变量都必须使用Nullable<T> 来声明和使用。

上面这种语法使用了一个问号，还有一种简化语法叫做 Null-Coalescing Operator 使用两个问号
> If the operand on the left is not null, the operand’s value is returned. If the operand on the left is null, the value of the right operand is returned. The null-coalescing operator offers a very convenient way to set a variable’s default value.

?? -- null-coalescing operator
var a=b??c;   等价于 var a=(b!=null)?b:c;

?? 虽然可以看作是 ? : 的语法糖，但是在实际运用中确实很方便，一个是用于为lambda expression赋值的时候，另外一个是用于多个值组合的时候

```
Func<String> f = () => SomeMethod() ?? "Untitled";
//vs
Func<String> f = () => { var temp = SomeMethod();
return temp != null ? temp : "Untitled";};

String s = SomeMethod1() ?? SomeMethod2() ?? "Untitled";
//vs
String s;
var sm1 = SomeMethod1();
if (sm1 != null) s = sm1;
else {
var sm2 = SomeMethod2();
if (sm2 != null) s = sm2;
else s = "Untitled";
}
```

CLR对于Nullable Value Types有一些特殊的支持
比如说boxing/unboxing的时候，CLR会判断其实际的value，如果是null的话就不做处理如果是value type的值则进行跟对应value type一样的处理
还有在GetType的时候，返回的不是Nullable<T> 而是直接返回 T，
另外CLR允许将Nullable<T>直接转换成T所实现的Interface，尽管Nullable 没有实现该接口
