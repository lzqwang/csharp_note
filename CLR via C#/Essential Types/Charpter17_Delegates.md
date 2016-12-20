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
