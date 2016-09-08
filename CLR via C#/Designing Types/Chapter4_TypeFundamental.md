# Type Fundamentals

## All type are derived from System.Object

  CLR Runtime要求所有的Type都必须继承自System.Object，
  在C#中声明一个class的时候，可以显式的声明该类继承与System.Object，但是一般都没有人这么写，默认的如果一个声明一个class而没有显式的声明其基类，默认的会将该class继承自System.Object
  ```
  class Implement         class Implement: System.Object
  {}                      {}
  ```
  这两种写法产生的效果是完全相同的。
  既然所有Type都要继承自System.Object，那么所有Type都继承了System.Object的Method

  Public Method |Description
  :-------------|--------------
Equals |Returns true if two objects have the same value. For more information about this method, see the “Object Equality and Identity” section in Chapter 5, “Primitive, Reference, and Value Types.”
GetHashCode |Returns a hash code for this object’s value. A type should override this method if its objects are to be used as a key in a hash table collection, like Dictionary. The method should provide a good distribution for its objects. It is unfortunate that this method is defined in Object because most types are never used as keys in a hash table; this method should have been defined in an interface. For more information about this method, see the “Object Hash Codes” section in Chapter 5.
ToString |The default implementation returns the full name of the type (this.GetType().FullName). However, it is common to override this method so that it returns a String object containing a representation of the object’s state. For example, the core types, such as Boolean and Int32, override this method to return a string representation of their values. It is also common to override this method for debugging purposes; you can call it and get a string showing the values of the object’s fields. In fact, Microsoft Visual Studio’s debugger calls this function automatically to show you a string representation of an object. *Note* that ToString is expected to be aware of the CultureInfo associated with the calling thread. Chapter 14, “Chars, Strings, and Working with Text,” discusses ToString in greater detail.
GetType |Returns an instance of a Type-derived object that identifies the type of the object used to call GetType. The returned Type object can be used with the reflection classes to obtain metadata information about the object’s type. Reflection is discussed in Chapter 23, “Assembly Loading and Reflection.” The GetType method is nonvirtual, which prevents a class from overriding this method and lying about its type, violating type safety.

Protected Method |Description
:----------------|------
MemberwiseClone |This nonvirtual method creates a new instance of the type and sets the new object’s instance fields to be identical to the this object’s instance fields. A reference to the new instance is returned.
Finalize |This virtual method is called when the garbage collector determines that the object is garbage and before the memory for the object is reclaimed. Types that require cleanup when collected should override this method. I’ll talk about this important method in much more detail in Chapter 21, “The Managed Heap and Garbage Collection.”

  CLR规定使用 *new* 来创建一个新的对象，下面是 *new* operator到底执行了什么
  1. 计算Type中的所有实例成员以及基类中的成员所需的字节数，另外还有一些额外的开销 *type object pointer* 和 *sync block index* CLR用来管理Object
  2. 为这个对象分配内存，所有的成员都给赋值为null或者0(zero)
  3. 初始化 *type object pointer* 和 *sync block index*
  4. 调用Type的构造函数，构造函数内部会调用基类的构造函数，一直到调用System.Object的构造函数(什么都不做，直接返回)
  完成上述操作之后， *new* 返回了一个引用或指针 (reference or pointer)。
  另外没有 *delete* 操作符来删除或清空 *new* 所创建的对象，只有等垃圾回收机制检测到对象已经不再被使用，会自动清空。

## Casting between Types

  CLR的一大功能特色是 Type Safety，在Type Casting方面，CLR有一些规范
  Type cast to Base Type时，可以隐式转换
  Type cast to Derived Type时，必须显式转换，因为这种转换在运行时可能会出错

  ```
  Implement CastType(Object obj)
  {
    Implement impObj=(Implement)obj;
    return impObj;
  }
  ```
  
  对于上面的代码，在编译的时候可以通过，但是在运行时，会去判断obj的Type是否是Implement或者其子类，如果不是的话，cast failed，throw exception
  另外建议修改上面Method的参数类型为 Implement，这样在编译时就可以check到一些非法转换而不需要等到运行时。
  经常使用到一个Cast Type的例子如下，参数类型为Stream，但是在实际中，是不能直接new一个stream对象的，但是作为stream对象的子类FileStream， MemoryStream等则可以被new出来，这样直接将子类对象传入函数中，无需显式的Cast Type
  ```
  void Create(Stream content)
  {
    //manipulate the stream content
  }
  ```
  
### Casting with C# *is* or *as* operators

  C#中提供了两个操作符来进行Casting相关的操作
  *is* 用于判断object是否从属于给定的Type，返回的结果是Boolean
  *as* 用于Casting Type，返回指定Type的对象或者无法Casting返回null，并不抛异常
  as的出现是在一定基础上，完善了 is的功能，如果使用is先判断再进行casting的话相当于要对object进行两次判断，而as则只需要一次判断即可

  >*Note* C# allows a type to define conversion operator methods, as discussed in the “Conversion  Operator Methods” section of Chapter 8, “Methods.” These methods are invoked only  when using a cast expression; they are never invoked when using C#'s as or is operator

  关于Casting，记住一点就是 子类可以隐式的转成父类， 父类(或者其他类)转成子类必须显式转，对于显式转换，编译时不会报错，运行时才会实际check是否能转成功

## Namespaces and Assemblies

  >Namespaces allow for the logical grouping of related types, and developers typically use them to make it easier to locate a particular type.

  CLR中是没有Namespace的概念的，C#中的Namespace更多起到一个分组的作用。

  关于Namespace Conflict问题有两种情况
  一个是两个Namespace下有相同的Type Name

  ```
    namespace ABC                   namespace BBC
    {                               {
      class Foo                       class Foo
      {}                              {}
    }                               }
  ```
  
  解决这个问题， 使用using声明的时候加以区分就可以了

  ```
    using ABCF=ABC.Foo;
    using BBCF=BBC.Foo;  
    class Program
    {
      static void Main()
      {
        ABCF.staticMethod();
        BBCF.staticMethod();
      }
    }
  ```
  
  下面这种case更特殊一些，如果引用的两个dll中namespace和type都相同

  ```
    namespace ABC                   namespace ABC
    {                               {
      class Foo                       class Foo
      {}                              {}
    }                               }
  ```
  
  这个问题其实比较罕见的，但是解决方法不算麻烦，需要使用 extern alias
  首先在reference的propert界面，给reference修改一下aliase,重名的references都需要改

  ![change reference aliases](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/referenceAliases.png)

  然后在code中最上面使用extern aliases 引用

  ```
    extern alias aliX;
    extern alias aliY;
    class Program
    {
      static void Main()
      {
        aliX.namespace.class.staticMethod();
        aliY.namespace.class.staticMethod();
      }
    }
  ```

  说了这么多Namespace，那么namespace和assembly究竟关系几何，其实2者没有什么必要联系，你想啊CLR压根就不知道有namespace这东西，所以从底层来说，这哥俩八杆子打不着
  在VS中可以看到这两个value可以分别赋值，互不影响

  ![Namespace&Assembly](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/namespaceAssembly.png)

## How Things Relate at Run Time

  这一部分作者试图解释在运行时，Type中的members是如何工作的，从下面的视图中可以看到，从3个部分来看运行时的工作流程

  ![Types at Runtime](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/typeatRuntime.png)

  图上包含了 code， thread stack已经heap 三部分。
  当code中的函数被调用时，假定调用发生在Thread stack中，thread中之前肯定已经执行过其他的方法，那么在调用code中的函数时，首先检查函数中使用到了哪些Type，如果这些Type之前没有使用到，需要进行初始化，将Types的信息在heap中创建出来。

  ![Types at Runtime](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/newTypeinHeap.png)

  使用new来创建一个Type 对象

  ![Types at Runtime](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/newObject.png)

  调用Type的static method返回 Type对象，直接找到Type中static method的IL Code执行

  ![Types at Runtime](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/callStaticMethod.png)

  返回新的Type对象的值赋值给了上一步中存放新创建出来的对象的变量，这样上一步中新创建的对象就没有被任何变量引用，垃圾回收机制会负责清除。
  下一行code是调用nonvirtual instance method

  ![Types at Runtime](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/callNonvirtualInstanceMethod.png)

  下一行code调用virtual instance method，对于虚函数，CLR会检查object的实际类型，这里是Manager，然后去看Manager是否重载了这个虚函数，如果是的调用重载的函数，如果没有重载，调用基类上的虚函数

  ![Types at Runtime](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/callVirtualInstanceMethod.png)

  根据上面的图示，CLR的Run Time简单流程过了一遍，基本可以搞清楚了。

  另外说一定，上面提到过的初始化每个Type都会格外初始化两个field *type object pointer* 和 *sync block index*
  这里着重介绍下 *type object pointer* -- 它指向的也是一个Type -- System.Type, 而System.Type也会有一个 *type object pointer* 指向其本身。
  System.Object 的GetType method 返回的就是 这个type。

  
