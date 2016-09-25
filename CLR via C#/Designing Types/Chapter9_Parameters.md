# Parameters

## Optional and Named Parameters

  Rule and Guidelines
  * Parameters default value可以使用在methods，constructor methods以及c# indexer还可以应用在委托的定义
  * 有default value的parameters必须定义在required value的parameters之后，另外需要注意params array parameters必须定义在所有的parameters之后而且不能有default value
  * Default Values必须是编译时常量，这样才能通过编译，可以使用default 或者new用来声明一个value type，对于reference type只能使用null
  * 注意不要修改parameters的name，如果调用方使用named parameter方式来传参的话，如果参数名修改了，会导致调用方编译时出错
  * 注意修改parameters的default values可能会对外围的调用方产生影响，尽量使用 0/null来作为default values
  * ref/out 标记的参数不能指定default value
  调用的时候注意事项
  * 使用named parameters可以不按照方法中参数的顺序来传参，但是named parameters必须出现在参数列表的最后面 
  * 使用named parameters传参需要保证所有的required parameters的值都有赋值
  * 对于有default value的parameter如果不想传参进去 不可以这么调用M(a,,b)，不能用两个逗号来告诉编译器第二个参数的值使用default value，只能通过named parameters来实现
  * 对于ref out parameters可以使用下面的方式来传参
```
M(ref int num);
int a=5;
M(num: ref a);
```

  当调用COM component的代码时，C#允许在传参的时候不必添加ref/out，但是对于不是COM component的方法必须添加ref/out
  
  由于Optional Parameters是C#特有的，所有需要编译器将这些特殊的语法编译成CLR可以执行的IL。
  当编译器发现parameter有default value，会为这个parameter添加两个attribute System.Runtime.InteropServices.OptionalAttribute and System.Runtime.InteropServices.DefaultParameterValueAttribute 并把它们存到result file的metadata中，同时将default value传给DefaultParameterValueAttribute的构造函数中， 当编译器看到调用方法的时候没有指定方法的参数，就会到 metadata中去找default value

## Implicitly Typed Local Variables

  > C# supports the ability to infer the type of a method’s local variable from the type of expression that is used to initialize it
  
  C#中提供了一种机制可以通过variable的初始化的代码来判断variable的type，也就是var的功能
  注意var不能用 null来赋值
 
  使用var提供了一些便捷，尤其是在操作泛型集合，循环的逻辑中使用的很多
  在VS中当使用var声明了一个变量之后，tooltip上马上会提示当前变量的type

  注意var只能用在Methods中，不能用来定义method parameter或者type field

  对比var 和 dynamic， 
  var只能用在方法内部，但是dynamic可以用于local variable，fields以及arguments
  不能cast expression to var，可以cast expression to dynamic
  var声明的变量必须初始化，但是dynamic声明的变量可以不初始化

## Passing parameters as reference to method

  理解值传递和引用传递
  默认CLR参数都是值传递，即使是一个引用类型，传递的只是这个参数的地址，所以如果不改变引用类型参数的本身，只改变其属性的话，这些属性是可以在函数结束之后，被调用代码继续使用的，但是如果这个引用类型参数被重新复制了，那么调用的函数不会随之而改变  
  C#中提供了 ref out, 表示参数是引用传递而不是值传递
  ref -- 调用方需要在调用函数之前为参数赋值
  out -- 调用函数在返回之前必须为参数赋值

  CLR对待ref out是没有区别的，只是C#编译器需要这两个标志
  另外C#中允许使用ref/out来重载方法，但是只能2选1，因为 CLR中这两个是一样的效果

 ``` 
public sealed class Point {
static void Add(Point p) { ... }
static void Add(ref Point p) { ... }
static void Add(out Point p) { ... }  // error ： 重载方法不能只通过ref 和 out来区分
}
 ```
  
  ref/out使用在值类型和引用类型上的时候稍微有些区别，可能会造成误解
  如果不使用ref/out 对于值类型的参数，在方法中无论做什么修改都不会影响到调用方
  而对于引用类型，只要参数值没有被重新赋值，那么对此对象进行的操作，在方法结束之后，调用方是可以被影响的，因为引用类型执行的是一个内存地址的对象，这个对象被修改了，但是内存地址并没变。
  但是如果参数值被重新赋值的话，那么相当于新开辟了一块内存地址，这个时候无论对这个对象做什么操作，调用方都不会受影响，因为调用方引用的对象地址和方法中修改的对象并不是一个。 
  所以只有当方法中需要对引用类型的参数值重新赋值的情况下，才需要考虑使用ref/out。
  一个场景是交换两个引用类型的对象的值。

  >For some other examples that use generics to solve this problem, see System.Threading’s Interlocked class with its CompareExchange and Exchange methods.

 ## Passing a Variable Number of Arguments to a Method
 
   使用params 关键字，表示可以接受一个可变数量的参数
   > The params keyword tells the compiler to apply an instance of the System.ParamArrayAttribute custom attribute to the parameter. 
  
  params的原理也是通过编译器添加attribute，
  c#编译器在调用方法的时候，先去找没有ParamArrayattribute标记的方法，如果没有找到，再去找被attribute标记的方法
  使用params是有一定的效率影响的，需要在堆上分配一个数组存放参数值，除非传的是一个null
  所以使用params的应该是less common的，对于common的case要提供相应的方法，对于比较特殊少见的case，提供一个params的方法实现。

## Parameters and Return type guideline

  总结： 
  参数类型 尽量使用接口或者基类 
  返回类型 尽量使用更具体的类
  特殊情况时，比如想要在不改变调用方的逻辑，只改变函数内的实现的话，可以考虑将返回类型设计成相比对弱的类型
  这里的强 弱的概念是指 类的可拓展的余地 
  比如 IList<String> 和 List<String> 前者就相对弱一些
  而跟IEnumerable<String> 相比 IList<String> 算强的类型

## Const-ness

  C#中不支持将method 和 parameters 标记为const
  