# Properties

## Parameterless Properties

  如果没有Properties的机制，可以使用Type的Public Fields来操作Type的数据，但是并不推荐这么使用，而是推出了Properties的机制
  不推荐使用公有变量的原因是 面向对象编程的一大特点是数据的封装， 公有变量的使用可以使得外围代码随意修改变量的值，与封装的特性背道而驰
  作者建议在设计Type的时候，将所有的Type fields设置为private的，通过method将field暴露给外围，这样可以检测控制调用方传入的参数是否合理，从而决定是否要对field进行修改。
  使用方法作为最初的数据封装的实现，但是有两个劣势
   一是需要写更多的code 二是调用的时候需要使用调用方法的方式。
   C#中Properties的机制部分的解决了第一个缺陷，完全的解决了第二个缺陷
   Properties可以使用任意的访问修饰符 ，同时可以定义在Interface中
   Proeperty需要有一个name和type，并且type不能是void，可以定义get and set，也可以只定义get，作为一个只读的属性，或者只定义set 作为一个只写的属性
   通过Property会操作一个field，成为backing fields，但是也不完全这样，可以不操作任何field，也可以操作多个fields

   编译器编译Properties时，会生成以下部分
   * A method representing the property’s get accessor method. This is emitted only if you define a get accessor method for the property.
   * A method representing the property’s set accessor method. This is emitted only if you define a set accessor method for the property.
   * A property definition in the managed assembly’s metadata. This is always emitted.

   前两部分对应的是get set的 访问方法，对于不支持Properites的语言，也可以调用方法。最后一个部分，property definition 定义了 property 与 accessor method的对应关系，在反射的时候会用到(*System.Reflection.PropertyInfo*)，CLR其实不会用到.

### Automatically Implemented Properties

```
public sealed class Employee {
  // This property is an automatically implemented property
  public String Name { get; set; }
  private Int32 m_Age;
  public Int32 Age {
    get { return(m_Age); }
    set {
    if (value < 0) // The 'value' keyword always identifies the new value.
    throw new ArgumentOutOfRangeException("value", value.ToString(),
    "The value must be greater than or equal to 0");
    m_Age = value;
    }
  }
}
```

  作者表示不喜欢这个feature，因为以下几个原因
  * 使用AIPs必须显式的为其进行初始化，
  * 需要序列化和反序列化的Type不要使用AIPs feature，因为编译器在序列化AIP的时候，会决定property的backing field的name，而且每次重新编译的话，这个name可能会改变
  * Debug的时候，不能在AIP上加断点
> For a single property, the AIP feature is an all-or-nothing deal.

  使用AIP feature，是需要同时指定get 和 set方法，否则这个Property没有实际意义

### Defining Properties Intelligently

  Properties使用起来像Fields，但是其实际上是Methods，所以Properties很容易产生一些疑惑。关于Properties在使用的过程中可能出现的问题
  * Properties可以设置成 read-only 或者 write-only的, 但是fields都是read&write
  * 使用Properties可能会抛异常，但是使用fields不会
  * Properties不能用来作为ref/out 参数
  * 执行Properties method可能会花费很长时间
  * 多次调用同一Property，得到的结果可能是不同的， DateTime.Now
  * 使用Property可能带来Site Effects
  * Propperty 需要额外的内存，同时可能返回一个和Objectde状态无关的引用，调用方修改这个引用但是实际上无法影响到object

### Object and Collection Initializers

  声明同时初始化
```
Employee e = new Employee() { Name = "Jeff", Age = 45 };
```

  C#允许省略 { 前的 ()

```
Employee e = new Employee{ Name = "Jeff", Age = 45 };
```

如果Property的Type是集合类型可以如下初始化
```
Classroom classroom = new Classroom {
  Students = { "Jeff", "Kristin", "Aidan", "Grant" }
};
```

### Anonymous Types

  匿名类型，声明的时候可以不指定Type，而是类似于Javascript中声明object的语法

```
// Define a type, construct an instance of it, & initialize its properties
var o1 = new { Name = "Jeff", Year = 1964 };
```  
  C#编译器在编译的时候会为这个匿名类型创建一个Type出来，因为编译器知道这个对象中每个属性的type，所以new一个新的匿名类型的对象但是其使用的属性type已经编译过了，编译器会直接使用编译过的Type

```
var a = new { Name = "Louis", Age = 22 };    
var b = new { Name = "LouisWang", Age = 22 };
var c = new { Age = 22, Name = "Louis" };
var at = a.GetType().Name;   // "<>f__AnonymousType0`2"  a and c are the same type but b is different
var bt = b.GetType().Name;   // "<>f__AnonymousType1`2"  a and c are the same type but b is different
```
  匿名类型在LINQ中起到了很关键的作用
```
String myDocuments = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
var query =
    from pathname in Directory.GetFiles(myDocuments)
    let LastWriteTime = File.GetLastWriteTime(pathname)
    where LastWriteTime > (DateTime.Now - TimeSpan.FromDays(7))
    orderby LastWriteTime
    select new { Path = pathname, LastWriteTime };// Set of anonymous type objects

foreach (var file in query)
  Console.WriteLine("LastWriteTime={0}, Path={1}", file.LastWriteTime, file.Path);
```

  匿名类型只能使用在方法内部，不能作为方法的参数或者返回类型

### Tuple  

  Tuple解决了上面提到的匿名类型不能作为方法的参数和返回类型的问题
  最简单的Tuple 是Tuple<T1> 支持一个参数，最复杂的是Tuple<T1,T2,T3,T4,T5,T6,T7,TRest>
  注意这个TRest是一个Tuple<T> 类型，也就是说如果需要更多个参数，可以再创建一个Tuple对象传给TRest
```
var t = Tuple.Create(0, 1, 2, 3, 4, 5, 6, Tuple.Create(7, 8));
Console.WriteLine("{0}, {1}, {2}, {3}, {4}, {5}, {6}, {7}, {8}",t.Item1, t.Item2, t.Item3, t.Item4, t.Item5, t.Item6, t.Item7,
t.Rest.Item1.Item1, t.Rest.Item1.Item2);  // 注意这里t.Rest.Item1 是因为Rest是一个Tuple<T1> 类型
```

  另外还有一个 System.Dynamic.ExpandoObject 对象， 可以在运行时添加删除属性

## Parameterful Properties

  上面介绍的Properties作为方法是没有参数的，所以作者称之为无参属性，而与之对应的有参属性，又被称为索引器
  索引器提供的语法类似于数组, 使用[] ，就像是C#提供的对[]的运算符重载
  索引器至少有一个参数，参数类型可以是除了void之外的 data type，另外索引器可以有多个参数
  > An example of an indexer that has more than one parameter can be found in the System.Drawing.Imaging.ColorMatrix class, which ships in the System.Drawing.dll assembly.

  索引器不能仅靠返回类型来区分参数类型相同的方法

```
Class CustomDictionary
{
  public string this[Int32 index]                        //Error	1	The type 'App.CustomDictionary' already contains a definition for 'Item'
  {
    // At least one accessor method is defined here
  }
  public int this[string value]                         //Error	2	The type 'App.CustomDictionary' already contains a definition for 'Item'
  {
    // At least one accessor method is defined here
  }
  public string Item(int index)
  {
    // to do...
  }
}   
```
  编译上面代码会失败，提示当前Type中已经定义了Item
  使用IndexerName 属性可以为索引器方法命名，如下
```     
public sealed class BitArray {
  [IndexerName("Bit")]               //默认的索引器没有指定方法名，编译器在编译的时候默认使用Item作为方法名，如果
  public string this[Int32 index]                       
  {
    // At least one accessor method is defined here
  }

  [IndexerName("Bit")]               //如果一个Type中定义了多个索引器，需要保证所有索引器的IndexerName是相同的，否则编译出错
  public int this[string value]
  {
    // At least one accessor method is defined here
  }
}
```
> Remember, C# won’t compile the code if it contains parameterful properties with different names
