# Delegates

Delegates我们翻译成委托，也是C#中一大特色，作者大神将这个委托认为是callback， function callback，在Javascript中callback应用很广泛，翻译为中文就是回调函数，在完成一些操作之后可以回调这个函数，这样函数就和其他数据类型一样可以作为一个方法参数传给另外一个方法。
C#中以委托来实现function callback的功能，同时又把委托的机制做大做强
>Delegates ensure that the callback method is type-safe, in keeping with one of the most important goals of the common language runtime (CLR).
Delegates also integrate the ability to call multiple methods sequentially and support the calling of static methods as well as instance methods

## A first look at Delegates
> In short, unmanaged C/C++ callback functions are not type-safe (although they are a very lightweight mechanism).
In the .NET Framework, callback functions are just as useful and pervasive as in unmanaged Windows programming. However, the .NET Framework provides a type-safe mechanism called delegates

```
// Declare a delegate type; instances refer to a method that
// takes an Int32 parameter and returns void.
internal delegate void Feedback(Int32 value);    //定义一个委托，只要符合委托定义的返回值和方法参数类型的方法都可以作为参数使用

public sealed class Program {
  public static void Main() {
    StaticDelegateDemo();
    InstanceDelegateDemo();
    ChainDelegateDemo1(new Program());
    ChainDelegateDemo2(new Program());
  }
  private static void StaticDelegateDemo() {      // 委托参数可以是null，也可以是static 方法
    Console.WriteLine("----- Static Delegate Demo -----");
    Counter(1, 3, null);
    Counter(1, 3, new Feedback(Program.FeedbackToConsole));
    Counter(1, 3, new Feedback(FeedbackToMsgBox)); // "Program." is optional
    Console.WriteLine();
  }
  private static void InstanceDelegateDemo() {
    Console.WriteLine("----- Instance Delegate Demo -----");
    Program p = new Program();
    Counter(1, 3, new Feedback(p.FeedbackToFile));
    Console.WriteLine();
  }
  private static void ChainDelegateDemo1(Program p) {
    Console.WriteLine("----- Chain Delegate Demo 1 -----");
    Feedback fb1 = new Feedback(FeedbackToConsole);
    Feedback fb2 = new Feedback(FeedbackToMsgBox);
    Feedback fb3 = new Feedback(p.FeedbackToFile);
    Feedback fbChain = null;
    fbChain = (Feedback) Delegate.Combine(fbChain, fb1);
    fbChain = (Feedback) Delegate.Combine(fbChain, fb2);
    fbChain = (Feedback) Delegate.Combine(fbChain, fb3);
    Counter(1, 2, fbChain);
    Console.WriteLine();
    fbChain = (Feedback)
    Delegate.Remove(fbChain, new Feedback(FeedbackToMsgBox));
    Counter(1, 2, fbChain);
  }
  private static void ChainDelegateDemo2(Program p) {
    Console.WriteLine("----- Chain Delegate Demo 2 -----");
    Feedback fb1 = new Feedback(FeedbackToConsole);
    Feedback fb2 = new Feedback(FeedbackToMsgBox);
    Feedback fb3 = new Feedback(p.FeedbackToFile);
    Feedback fbChain = null;
    fbChain += fb1;
    fbChain += fb2;
    fbChain += fb3;
    Counter(1, 2, fbChain);
    Console.WriteLine();
    fbChain -= new Feedback(FeedbackToMsgBox);
    Counter(1, 2, fbChain);
  }
  private static void Counter(Int32 from, Int32 to, Feedback fb) {
    for (Int32 val = from; val <= to; val++) {
      // If any callbacks are specified, call them
      if (fb != null)
        fb(val);
    }
  }
  private static void FeedbackToConsole(Int32 value) {
    Console.WriteLine("Item=" + value);
  }
  private static void FeedbackToMsgBox(Int32 value) {
    MessageBox.Show("Item=" + value);
  }
  private void FeedbackToFile(Int32 value) {
    using (StreamWriter sw = new StreamWriter("Status", true)) {
      sw.WriteLine("Item=" + value);
    }
  }
}
```
委托中的协变 covariance 与逆变 contra-variance
covariance： 返回类型可以是定义类型的子类型
contra-variance：传入的参数类型可以是定义类型的父类型

Covariance and Contra-Variance  只适用于引用类型而不适用于值类型和void，不支持值类型的原因是值类型的内存结构与引用类型存在差异，好在，编译器会在你使用不当的时候提出错误

## Demystifying Delegates

委托使用起来看似比较简单，定义一个委托变量，然后把一个符合委托定义的方法作为参数使用就可以实现callback function，但是实际上CLR为实现委托了很多幕后工作
举个例子
```
internal delegate void Feedback(Int32 value);
```
定义了这样一个委托，编译器看到这个定义，实际上要生成一个class来实现
```
internal class Feedback : System.MulticastDelegate {
  // Constructor
  public Feedback(Object @object, IntPtr method);
  // Method with same prototype as specified by the source code
  public virtual void Invoke(Int32 value);
  // Methods allowing the callback to be called asynchronously
  public virtual IAsyncResult BeginInvoke(Int32 value,
  AsyncCallback callback, Object @object);
  public virtual void EndInvoke(IAsyncResult result);
}
```
> The BeginInvoke and EndInvoke methods are related to the .NET Framework's Asynchronous Programming Model which is now considered obsolete and has been replaced by tasks that I discuss in Chapter 27, “Compute-Bound Asynchronous Operations.

我们定义的delegate都是继承自System.MulticastDelegate 而它又继承自 System.Delegate

>Basically, because delegates are classes, a delegate can be defined anywhere a class can be defined.

MulticastDelegate’s Significant Non-Public Fields

Field Type | Description
-----------------|-----------
\_target | System.Object When the delegate object wraps a static method, this field is null. When the delegate objects wraps an instance method, this field refers to the object that should be operated on when the callback method is called. In other words, this field indicates the value that should be passed for the instance method’s implicit this parameter.
\_methodPtr |System.IntPtr An internal integer the CLR uses to identify the method that is to be called back.
\_invocationList | System.Object This field is usually null. It can refer to an array of delegates when building a delegate chain (discussed later in this chapter).

注意使用static method和instance method的区别体现在 \_target 这里，static method 的target 是null，而instance method的target就是instance对象

## Using Delegates to Call Back Many Methods (Chaining)

委托链表，一个委托对象中可以绑定多个回调函数，上面的表中第三个field \_invocationList 用来绑定多个回调函数
可以使用 Delegate.Combine 方法或者直接使用 + 运算符 来增加一个回调函数
使用 Delegate.Remove 方法或则会 -= 运算符 可以删除一个回调函数

>When the　loop is complete, the result variable will contain only the result of the last delegate called (previous　return values are discarded); this value is returned to the code that called Invoke.

如果委托是有返回值的，那么委托链表中的最后一个回调函数的返回值会成为整个委托链表的返回值，如果想要获取每个回调函数的返回值，需要遍历每一个委托链表中的回调函数然后将返回值缓存起来。
另外如果委托链中有一个回调函数出现异常，那么会影响到后续的回调函数的执行。
C#中为了规避这些问题，提供了 GetInvocationList的方法可以返回所有的回调函数，然后依次运行回调函数，这个时候就可以自定义一些返回值和异常处理的逻辑

## Generic Delegates

FCL中提供了一系列有返回值和无返回值的泛型委托，这样在实际开发过程中使用FCL中的委托类型就可以了 不需要自己定义
```
public delegate void Action(); // OK, this one is not generic
public delegate void Action<T>(T obj);
public delegate void Action<T1, T2>(T1 arg1, T2 arg2);
public delegate void Action<T1, T2, T3>(T1 arg1, T2 arg2, T3 arg3);
...
public delegate void Action<T1, ..., T16>(T1 arg1, ..., T16 arg16);

public delegate TResult Func<TResult>();
public delegate TResult Func<T, TResult>(T arg);
public delegate TResult Func<T1, T2, TResult>(T1 arg1, T2 arg2);
public delegate TResult Func<T1, T2, T3, TResult>(T1 arg1, T2 arg2, T3 arg3);
...
public delegate TResult Func<T1,..., T16, TResult>(T1 arg1, ..., T16 arg16);
```

> When using delegates that take generic arguments and return values, contra-variance and covariance come into play, and it is recommended that you always take advantage of these features because they have no ill effects and enable your delegates to be used in more scenarios

使用泛型委托的时候，注意巧用参数和返回值的协变与逆变


## C#’s Syntactical Sugar for Delegates

* Syntactical Shortcut #1: No Need to Construct a Delegate Object -- 可以不定义delegate对象，直接把回调函数当作参数传递
* Syntactical Shortcut #2: No Need to Define a Callback Method (Lambda Expressions) -- 除了不定delegate对象，回调函数甚至都可以不定义，使用lambda 表达式 搞定
* Syntactical Shortcut #3: No Need to Wrap Local Variables in a Class Manually to Pass Them to a Callback Method -- callback方法中可以直接使用已经定义好的变量，不需要再把这些变量传递给回调函数中。 这里lambda 表达式中可以直接使用外面定义的变量，而如果不使用lambda表达式而是另外一个方法的话，就必须通过一个helper class将这些变量传递给这个方法中。 实际上，CLR在编译lambda表达式的时候也确实是按照这样的方式来重写了这里的代码

Lambda 表达式的出现一定程度上成全了C#的在使用委托的时候可以使用的语法糖，但是lambda表达式也不能不分场合的滥用，一般来讲代码只被引用一次而且代码行数不超过3行的情况下，使用lambda表达式是很合适的

## Delegates and reflection

可以通过反射来动态获取委托类型，同时提供了动态调用的方法，
> Using reflection APIs, you must first acquire a MethodInfo object referring to the method you want to create a delegate to. Then, you call the CreateDelegate method to have it construct a new object of a Delegate-derived type identified by the first parameter, delegateType.

>System.Delegate’s DynamicInvoke method allows you to invoke a delegate object’s callback method, passing a set of parameters that you determine at run time. When you call DynamicInvoke,it internally ensures that the parameters you pass are compatible with the parameters the callback method expects. If they’re compatible, the callback method is called. If they’re not, an ArgumentException is thrown. DynamicInvoke returns the object the callback method returned

Delegate与Reflection结合的情况下，通常不能明确的获取delegate的实际返回值或参数，这个时候使用 DynamicInvoke 方法，可以在运行时调用委托的回调函数，当然如果回调函数的参数和传进去的参数不匹配的话， ArgumentException 会抛出。这个方法在介绍 Event篇章中自定义的EventSet 中得到了实际应用。
