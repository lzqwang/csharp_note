# Exceptions and State Management

## Defining "Exception"
定义一个Type的时候通常用名词来命名，比如Stream，Thread, 而定义Type中的members(包括Properties，Methods，Events),其中表示Action的通常使用动词，比如Begin，Read，Write等
当一个action member不能完成既定的任务这时候会抛出一个异常。
> Important An exception is when a member fails to complete the task it is supposed to perform as indicated by its name.

## System.Exception

Public Properties of the System.Exception Type

Property |Access |Type |Description
----|---|---|-------
Message |Read-only |String |Contains helpful text indicating why the exception was thrown. The message is typically written to a log when a thrown exception is unhandled. Because end users do not see this message, the message should be as technical as possible so that developers viewing the log can use the information in the message to fix the code when producing a new version.
Data |Read-only |IDictionary |A reference to a collection of key-value pairs. Usually, the code throwing the exception adds entries to this collection prior to throwing it; code that catches the exception can query the entries and use the information in its exception-recovery processing.
Source |Read/write |String |Contains the name of the assembly that generated the exception.
StackTrace |Read-only |String |Contains the names and signatures of methods called that led up to the exception being thrown. This property is **invaluable** for debugging.
TargetSite |Read-only |MethodBase |Contains the method that threw the exception.
HelpLink |Read-only |String |Contains a URL (such as file://C:\MyApp\Help.htm#MyExceptionHelp) to documentation that can help a user understand the exception. Keep in mind that sound programming and security practices prevent users from ever being able to see raw unhandled exceptions, so unless you are trying to convey information to other programmers, this property is seldom used.
InnerException |Read-only |Exception |Indicates the previous exception if the current exception were raised while handling an exception. This read-only property is usually null. The Exception type also offers a public GetBaseException method that traverses the linked list of inner exceptions and returns the originally thrown exception.
HResult |Read/write |Int32 |A 32-bit value that is used when crossing managed and native code boundaries. For example, when COM APIs return failure HRESULT values, the CLR throws an Exception-derived object and maintains

> Important When you throw an exception, the CLR resets the starting point for the exception; that is, the CLR remembers only the location where the most recent exception object was thrown.

CLR会重新设置exception的starting point，所以在catch了exception之后，如果想要把exception再抛出去，注意使用正确的方式
```C#
private void SomeMethod() {
  try { ... }
  catch (Exception e) {
  ...
    throw e; // CLR thinks this is where exception originated.     FxCop reports this as an error
  }
}

private void SomeMethod() {
  try { ... }
  catch (Exception e) {
  ...
    throw; // This has no effect on where the CLR thinks the exception originated. FxCop does NOT report this as an error
  }
}

private void SomeMethod() {
  Boolean trySucceeds = false;
  try {
    ...
    trySucceeds = true;
  }
  finally {
    if (!trySucceeds) { /* catch code goes in here */ }
  }
}
```

从上面的代码中可以看出，如果想要在SomeMethod外层获取到excetion的原始信息，需要使用第二种方式。而为了避免throw exception带来的影响，使用第三种方式，可以不必担心exception被reset starting point。
> If the CLR can find debug symbols (located in the .pdb files) for your assemblies, the string returned by System.Exception’s StackTrace property or System.Diagnostics.StackTrace’s ToString method will include source code file paths and line numbers. This information is incredibly useful for debugging

在debug模式下， StackTrace的ToString方法会包含源代码的文件路径以及行数，对于调试很有帮助
有时候会发现有些执行的方法在StackTrace中并没有体现出来，这个有两个原因，一个是he stack is really a record of where the thread should return to, not where the thread has come from. 另一个原因是编译器为了减少从调用和返回方法时的开销，可以将方法内联(inline method)，这样会导致stack trace显示的方法不准确

可以为method 指定 特定MethodImpl Attribute 让编译器不inlime method

## FCL-Defined Exception classes
```
System.Exception
  System.AggregateException
  System.ApplicationException
    System.Reflection.InvalidFilterCriteriaException
    System.Reflection.TargetException
    System.Reflection.TargetInvocationException
    System.Reflection.TargetParameterCountException
    System.Threading.WaitHandleCannotBeOpenedException
    System.Diagnostics.Tracing.EventSourceException
  System.InvalidTimeZoneException
  System.IO.IsolatedStorage.IsolatedStorageException
  System.Threading.LockRecursionException
  System.Runtime.CompilerServices.RuntimeWrappedException
  System.SystemException
    System.Threading.AbandonedMutexException
    System.AccessViolationException
    System.Reflection.AmbiguousMatchException
    System.AppDomainUnloadedException
    System.ArgumentException
      System.ArgumentNullException
      System.ArgumentOutOfRangeException
      System.Globalization.CultureNotFoundException
      System.Text.DecoderFallbackException
      System.DuplicateWaitObjectException
      System.Text.EncoderFallbackException
    System.ArithmeticException
      System.DivideByZeroException
      System.NotFiniteNumberException
      System.OverflowException  
      System.ArrayTypeMismatchException
    System.BadImageFormatException
    System.CannotUnloadAppDomainException
    System.ContextMarshalException
    System.Security.Cryptography.CryptographicException
      System.Security.Cryptography.CryptographicUnexpectedOperationException  
    System.DataMisalignedException
    System.ExecutionEngineException
    System.Runtime.InteropServices.ExternalException
      System.Runtime.InteropServices.COMException
      System.Runtime.InteropServices.SEHException      
    System.FormatException
      System.Reflection.CustomAttributeFormatException
    System.Security.HostProtectionException
    System.Security.Principal.IdentityNotMappedException
    System.IndexOutOfRangeException
    System.InsufficientExecutionStackException
    System.InvalidCastException
    System.Runtime.InteropServices.InvalidComObjectException
    System.Runtime.InteropServices.InvalidOleVariantTypeException
    System.InvalidOperationException
      System.ObjectDisposedException
    System.InvalidProgramException
    System.IO.IOException
      System.IO.DirectoryNotFoundException
      System.IO.DriveNotFoundException
      System.IO.EndOfStreamException
      System.IO.FileLoadException
      System.IO.FileNotFoundException
      System.IO.PathTooLongException
    System.Collections.Generic.KeyNotFoundException
    System.Runtime.InteropServices.MarshalDirectiveException
    System.MemberAccessException
      System.FieldAccessException
      System.MethodAccessException
      System.MissingMemberException
      System.MissingFieldException
      System.MissingMethodException
    System.Resources.MissingManifestResourceException
    System.Resources.MissingSatelliteAssemblyException
    System.MulticastNotSupportedException
    System.NotImplementedException
    System.NotSupportedException
      System.PlatformNotSupportedException
    System.NullReferenceException
    System.OperationCanceledException
      System.Threading.Tasks.TaskCanceledException
    System.OutOfMemoryException
      System.InsufficientMemoryException
    System.Security.Policy.PolicyException
    System.RankException
    System.Reflection.ReflectionTypeLoadException
    System.Runtime.Remoting.RemotingException
      System.Runtime.Remoting.RemotingTimeoutException
    System.Runtime.InteropServices.SafeArrayRankMismatchException
    System.Runtime.InteropServices.SafeArrayTypeMismatchException
    System.Security.SecurityException
    System.Threading.SemaphoreFullException
    System.Runtime.Serialization.SerializationException
    System.Runtime.Remoting.ServerException
    System.StackOverflowException
    System.Threading.SynchronizationLockException
    System.Threading.ThreadAbortException
    System.Threading.ThreadInterruptedException
    System.Threading.ThreadStartException
    System.Threading.ThreadStateException
    System.TimeoutException
    System.TypeInitializationException
    System.TypeLoadException   
      System.DllNotFoundException
      System.EntryPointNotFoundException
      System.TypeAccessException
    System.TypeUnloadedException
    System.UnauthorizedAccessException
      System.Security.AccessControl.PrivilegeNotHeldException
    System.Security.VerificationException
    System.Security.XmlSyntaxException
  System.Threading.Tasks.TaskSchedulerException
  System.TimeZoneNotFoundException   
```
  关于System.Exception的继承 最初的设计是 只有两个type继承分别是System.SystemException 和 System.ApplicationException
  但是实际上这个规定 Microsoft自己也没有遵守，导致了好多exception class继承的不对，结果最初设计的这两个exception type显得没有什么意义了，但是又不能删掉了，因为有的用户已经继承自这两个type了

## Throw an Exception

1. 考虑Exception的继承问题，尽量少的继承基类，但是也不能直接使用System.Exception
2. 第二个需要考虑的问题就是，为Exception赋上有意义的message

## Defining Your Own Exception Class

通常自定义一个Exception的时候，只是继承自一个已有的Exception class然后为这个Exception设计一个能阐述异常信息的Message，而实际上要自定义一个Exception class需要支持序列化，作者提供了一个泛型版的Excetion class，可以作为一个模板，使用者只需要自定义多个ExceptionArgs的子类用来表示多个exception type即可
``` C#
    [Serializable]
    sealed class Exception<TExceptionArgs> : Exception, ISerializable where TExceptionArgs : ExceptionArg
    {
        private const String c_args = "Args"; // For (de)serialization
        private readonly TExceptionArgs m_args;
        public TExceptionArgs Args { get { return m_args; } }
        public Exception(String message = null, Exception innerException = null)
            : this(null, message, innerException)
        { }
        public Exception(TExceptionArgs args, String message = null, Exception innerException = null)
            : base(message, innerException)
        { m_args = args; }

        // This constructor is for deserialization; since the class is sealed, the constructor is
        // private. If this class were not sealed, this constructor should be protected
        [SecurityPermission(SecurityAction.LinkDemand,        Flags = SecurityPermissionFlag.SerializationFormatter)]
        private Exception(SerializationInfo info, StreamingContext context)
            : base(info, context)
        {
            m_args = (TExceptionArgs)info.GetValue(c_args, typeof(TExceptionArgs));
        }

        // This method is for serialization; it’s public because of the ISerializable interface
        [SecurityPermission(SecurityAction.LinkDemand,
        Flags = SecurityPermissionFlag.SerializationFormatter)]
        public override void GetObjectData(SerializationInfo info, StreamingContext context)
        {
            info.AddValue(c_args, m_args);
            base.GetObjectData(info, context);
        }

        public override String Message
        {
            get
            {
                String baseMsg = base.Message;
                return (m_args == null) ? baseMsg : baseMsg + " (" + m_args.Message + ")";
            }
        }

        public override Boolean Equals(Object obj)
        {
            Exception<TExceptionArgs> other = obj as Exception<TExceptionArgs>;
            if (other == null) return false;
            return Object.Equals(m_args, other.m_args) && base.Equals(obj);
        }

        public override int GetHashCode() { return base.GetHashCode(); }
    }

    [Serializable]
    class ExceptionArg
    {
        public virtual string Message { get { return string.Empty; } }
    }
```
在自定义Exception的时候只需要自定义一个CustomExceptionArg 继承自ExceptionArg，可以添加需要的信息，这样在序列化的时候这些信息也可以保留，而如果是直接继承自System.Exception 是无法保证这些自定义的属性可以被正确序列化的

## Trading Reliability for Productivity

这一部分作者语重心长的阐述了CLR中的异常处理机制,由于C#属于高级语言，C#编译器其实帮助开发者做了许多事情，而且FCL中提供了许多的library帮助开发者完成功能。所以开发者所写的代码其实是在CLR的底层逻辑之上的，也就说开发者所能处理的异常非常有限，因为大部分异常是FLC中的class抛出的，作为开发者就算捕获了异常可能也无法针对这个异常做出有效的措施。但是CLR会尽可能的做到稳定，以至于让开发者觉得只需要处理好自己的异常就可以了。
作者这里的副标题以可靠性换功能性-- 意思就是由于CLR帮助开发者封装好了底层的实现，所以我们可能会觉得CLR的实现可能有不可靠的地方，但是与此同时 开发者又可以有效的利用CLR提供的各种功能使得开发程序变得更加容易，所以这是不可兼得的问题，既然选择了这种开发平台那就只能接受这种设定，充分的将其优势发挥出来

## Exception Guidelines and Best practices

* **Use finally liberally**
finally 语句很好用而且没有什么毒副作用，当一个方法执行完成之后，可以利用finally 语句完成一些清理的工作，即使出现了异常也可以执行finally语句，可以恢复一些corrupted state
但是需要注意finally语句中不要执行大段的业务逻辑
* **Don't Catch everything**
开发者在捕获异常的时候，应该是提前预判到了异常的产生然后针对特定的异常做出相应的处理，所以不推荐只捕获System.Exception, 因为这样就没有针对性了，作者的建议是 如果在catch System.Exception 的代码里面将exception又throw了，这种做法还是可以接受的，但是如果只是catch而没有throw相当于是在错误已经发生的同时，还继续执行，可能会导致意料之外的结果。我们在开发的过程中经常出现catch System.Exception的情况，以后要针对这个问题好好反思一下
* **Recovering Gracefully from an Exception**
如果开发者明确的知道 代码中可能会出现的异常类型，那么在异常处理的时候，就需要针对特定的异常给出相应的友好的提示。
* **Backing Out of a Partially Completed Operation When an Unrecoverable Exception Occurs—Maintaining State**
如果一个函数功能需要调用其他的函数来完成，那么有可能出现部分函数执行成功而另外有部分函数执行失败了，针对这种失败的情况，应该在出现错误的时候恢复state至函数开始运行之前的状态，针对这种情况，需要catch所有的exception，但是不要吞掉exception，而是直接将exception re-throw给调用方。
* **Hiding an Implementation Detail to Maintain a “Contract”**
有时候需要将实际抛出的异常隐藏而抛出另外一个异常出去，这种情况通常出现在，调用方对实现方的逻辑不是非常清楚，不知道实现中所具体涉及到的逻辑，这个时候实现的逻辑中如果有异常出现，则通常不要把异常直接抛出给调用方，因为调用方根本不知道这个异常出现的前因后果，所以在实现的逻辑中将异常进行一定的包装然后再抛出就可以了。


## Unhandled exceptions

>That is, when an application gets an unhandled exception, Windows writes an entry to the system’s event log. You can see this entry by opening the Event Viewer application and then looking under the Windows Logs ➔ Application node in the tree

如果程序中出现了没有处理的异常，CLR的policy是，将异常信息写入到系统的event log中

>you can get more interesting details about the problem by using the Windows Reliability Monitor applet

使用windows自带的Reliability Monitor可以看到更详细的log信息，在windows下 search reliability history，即可打开Reliability Monitor

## Debugging Exceptions

VS中可以配置对Exception的处理方式
VS-> Debug -> Exceptions
可以配置 Thrown ,User-handled
> If an exception type’s Thrown check box is not selected, the debugger will also break where the exception was thrown, but only if the exception type was not handled

## Exception-Handling Performance Considerations

关于是否使用异常处理机制，不同人有不同的看法，而不支持使用异常处理的人大部分是因为异常处理对效率有很大的影响，作者对于这个问题的观点是，坚决支持使用异常处理的，尤其是对于使用CLR的情况下，CLR，FCL是肯定会抛出异常的，这样就必须要在外围处理了。
另外作者举了 Int32的 Parese / TryParse的例子
作者建议是不要优先使用 TryXXX类型的方法， 因为这种方法虽然本意是返回 true/false来表示方法运行的结果，但是这个方法本身还是可能会出现异常，是在方法内部无法处理的异常。
所以规范的object model，是使用 non-TryXXX方法，除非客户对于异常处理的效率问题很不满意，执意要改的情况下才考虑使用 TryXXX的方式

## Constrained Execution Regions (CERs)

CLR中提供了一种方式，可以先预判catch 和 finaly 中执行的方法是否会出现异常，再决定是否执行try中的code

```
private static void Demo2() {
    // Force the code in the finally to be eagerly prepared
    RuntimeHelpers.PrepareConstrainedRegions(); // System.Runtime.CompilerServices namespace
    try {
        Console.WriteLine("In try");
    }
    finally {
        // Type2’s static constructor is implicitly called in here
        Type2.M();
    }
}
public class Type2 {
    static Type2() {
      Console.WriteLine("Type2's static ctor called");
    }
    // Use this attribute defined in the System.Runtime.ConstrainedExecution namespace
    [ReliabilityContract(Consistency.WillNotCorruptState, Cer.Success)]
    public static void M() { }
}
```

>PrepareConstrainedRegions -- it will eagerly compile the code in the try’s catch and finally blocks. The JIT compiler will load any assemblies, create any type objects, invoke any static constructors, and JIT any methods. If any of these operations result in an exception, then the exception occurs before the thread enters the try block.
However, the JIT compiler only prepares methods that have the ReliabilityContractAttribute applied to them with either Consistency.WillNotCorruptState or Consistency.MayCorruptInstance because the CLR can’t make any guarantees about methods that might corrupt AppDomain or process state.

使用PrepareConstrainedRegions 方法可以提前编译try catch finally中的code，如果编译出现异常，那么就直接不执行try中的code了。但是不是所有的方法都预先编译，只有使用ReliabilityContract 属性中的Consistency.WillNotCorruptState or Consistency.MayCorruptInstance 修饰的方法才可以。


## Code Contracts

* *Preconditions* Typically used to validate arguments
* *Postconditions* Used to validate state when a method terminates either due to a normalreturn or due to throwing an exception
* *Object Invariants* Used to ensure an object’s fields remain in a good state through an object’sentire lifetime

使用code contract可以避免异常的出现，在可能会出现异常的变量，方法，或者逻辑之前使用合适的code contract。
在实际的开发过程中，使用code contract的机会其实很少，通常我们会选用自己惯用的一套逻辑来进行参数验证，避免异常的出现，而如果异常出现了，也有一套异常处理的机制来跟踪记录。
