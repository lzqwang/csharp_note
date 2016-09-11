### CLR's execution model 

  既然是要介绍.NET Framework，说明读者选择了.NET Framework作为开发平台，这是开始编程的第一步。
  然后呢，需要选择编程语言，不同的编程语言会有不同的能力，比如说，使用非托管的C/C++可以更接近于底层系统

> have low-level control of the system

  而托管的语言则有不同的侧重倾向，或主攻UI方向，或者善于与COM objects或者数据库打交道
  CLR-- Common language runtime -- 公共语言运行时 -- 顾名思义，可以被多管理种编程语言运用，CLR提供了诸多公共的功能和机制，
  比如内存分配管理，线程机制，异常处理等等。但是CLR其实是不关注什么编程语言也不无需要关心，因为不管是什么编程语言最后到CLR上运行的时候，都需要编译成CLR能理解能运行的模块-- execution model。
  既然CLR跟编程语言无关，那么怎么决定编程语言的优劣之分呢
  作者认为主要是语言的编译器和分析器，编程语言都会制定自己的一套语法，编译器负责将按照规定语言编写的程序编译成CLR可以运行的代码
  好的编程语言大概就是那些可以将source code准确的编程，对于source code中的错误找出来并给出相应的修改建议。

> So, what is the advantage of using one programming language over another?
Well, I think of compilers as syntax checkers and “correct code” analyzers. They examine your source
code, ensure that whatever you’ve written makes some sense, and then output code that describes
your intention.

先不谈编程语言的优劣之分，重点放在execution model上，不同的语言通过各自编译器的被编译成Managed module

> A managed module is a standard 32-bit Windows portable executable (PE32) file or a standard 64-bit Windows portable executable (PE32+) file that requires the CLR to execute
> By the way, managed assemblies always take advantage of Data Execution Prevention(DEP) and Address Space Layout Randomization (ASLR) in Windows; these two features improve the security of your whole system.

![Compiling source code into managed module](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/compiling%20source%20code%20into%20managed%20model.png)

  managed module的组成部分包括 PE32/PE32+ header, CLR header, IL Code 以及 Metadata.
  ![Parts of Managed Module](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/PartsofManagedModule.png)
  编译器在生成IL Code的同时，需要生成Metadata，Metadata与IL Code都是内嵌在exe/dll之中，两者永远是同步的
  Meatadata有以下几个作用
  * 在编译时metadata移除本地c/c++ header以及 library文件，因为所有的引用信息都在IL所在的文件里，编译器可以直接通过读取文件中的metadata来获取这些信息
  * Metadata所包含的信息在VS的只能提示中起到关键作用，vs通过分析metadata中的信息得以知道类，方法，属性，变量以及方法的参数信息
  * CLR 的code verification process 通过metadata来确定代码是不是只包括 'type-safe' operations
  * Metadata允许序列化和反序列化的实现
  * Metadata允许垃圾回收器来追踪对象的生命周期，对于任意对象，垃圾回收器可以在metadata中找到该对象中的变量引用的其他对象
  微软的C++编译器是唯一允许在一个managed module中同时使用managed and unmanaged code的, 可以在托管代码中调用本地的c/c++非托管代码

## Compiling Managed Module into Assemblies

  CLR并不是直接使用Managed Module，而是需要将Managed Module编译成Assembly程序集,程序集可以保护多个Manged Module，以及其他的一些resource 文件。
![Compiling Managed Module into Assemblies](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/CompilingModuleintoAssembly.png)
  程序集可以将逻辑和物理文件解耦，比如程序集中包含的物理文件如果很少会被用到，那么只有当逻辑中用到那些文件的时候才会去下载。
  另外程序集中对引用的其他程序集(包括版本号)有加以说明，所以程序集是'self-describing'，这样在部署程序集的时候就不需要其他的文件就可以知道程序集的依赖关系

## Loading the Common Language Runtime

  Assembly可以是可执行的程序(exe)或者是程序中引用的dll，而CLR则负责assembly的运行，而CLR是依托于.NET Framework，那么就要求.NET Framework必须安装在host机器上。
  查看一台机器是否安装了.NET　Framework的方法是 去%SystemRoot%/System32 路径下查找 mscoree.dll 文件，如果存在说明安装了。另外由于.NET Framework有多个版本可以共存，所以想要查看具体安装了哪些版本，可以使用CLRVer.exe
  **关于32-bit 和 64-bit process**
  通常情况下编写的程序可以在32位和64位机器上运行，那么编译的时候默认的选项是anycpu，如果代码中一些特殊的地方需要指定cpu的类型，可以在VS中配置使用32位或者64位，另外有一个prefer 32-bit的选项，如果是勾选上了，那么即使在64位机器上运行的也是32位的程序。另外如果没有特殊的要求的话，默认都是使用anycpu+prefer 32-bit 这种模式的，因为VS不支持63-bit程序的 edit-and-continue，也就是说使用32-bit的程序在使用vs调试的过程中会方便一些。
  64-bit windows可以运行32-bit process,有个专门的名称叫 wow64(for Windows on Windows 64)
  开始运行程序之前，先根据程序的头文件中的信息来决定当前系统是否可以运行该程序，如果可以运行那么是在32-bit或者64-bit下运行(这里只针对64-bit windows)
  ![Platforms of Modules](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/PlatformonResultingModules.png)

  根据相应的platform，process相对应的路径下去load MSCorEE.dll， 然后调用该dll中的一个发放，这个方法会load CLR，加载assemblies,等一些初始化的工作，等初始化结束之后，回调用process的entry point，比如Main函数，来执行process中的代码，这样process就启动运行了。
  > the process’s primary thread calls a method defined inside MSCorEE.dll. This method initializes the CLR, loads the EXE assembly, and then calls its entry point method (Main). At this point, the managed application is up and running.

  如果unmanaged application调用Win32 LoadLibrary来加载一个managed assembly，windows会去初始化CLR(如果CLR还没有初始化的话)，但是有一点需要注意的地方是，通过这种方式加载Assembly会比直接运行assembly受限制，比如说目标assmebly指定需要在x86下运行，那么这个时候如果调用的程序是64-bit，那么这个assembly就无法加载。而如果直接执行这个assmebly的话，就会在wow64下运行于64-bit windows。

## Executing Assembly Code

  Assembly包含Metadata以及IL， IL是CPU-independent machine language，是微软同其他商业和学术的语言编译器磋商之后创造出来的。 IL可以算作一门面向对象的语言，它可以创建对象，操纵对象甚至还可以抛出和捕获异常。
  通常程序员编写的程序是要比IL高级的c#, VB, F#等等，但是微软也支持直接使用IL来编写程序，IL　Assembler　ILAsm.exe以及　IL　Disassembler，　ILDasm.exe
  IL实现了CLR提供的所有的功能特性，但是更高级的编程语言则未必会实现所有这些功能特性，会一些语言有自己的着重点。再来看这本书的名字 CLR via C#，也就是说作者要从C#的角度来讲解CLR，那么CLR中的功能C#中实现的或者没有实现的作者都会进行讲解
  > The only way for you to know what facilities the CLR offers is to read documentation specific to the CLR itself. In this book, I try to concentrate on CLR features and how they are exposed or not exposed by the C# language.

  由于CLR对于编程语言的使用是不限制的，所以其实可以使用不同的编程语言来开发一套产品在CLR上运行，不同语言之间可以很好的集成是CLR的一个很出色的特性。

  执行方法，需要将IL转成CPU指令，这个工作是由CLR中的JIT(Just In Time) 编译器完成的.
  ![Calling method](
  https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/Callingmethod.png)

  当第一次调用方法时，需要JIT Compiler编译出相对应的CPU指令，之后再调用相同的方法，就不再需要JIT Compiler再编译一次，而是直接调用刚才生成好的CPU指令即可。
  C# 编译器有两个开关可以影响JIT Compiler，一个是 /optimize 另一个是 /debug
  /optimize - 表示IL code不需要优化， /optimize + 则是需要优化， 通常不优化的方式可以在debug的时候进行单步调试而优化之后则不能进行单步调试，但是生成的IL Code会简洁许多。
  /debug 则是表示是否需要优化JIT Native Code， 如果/optimize + 那么无论/debug 如何都会优化JIT Native Code， 如果/optimize -，那么/debug - 会优化JIT Native Code，因为不需要Debug的话，优化代码可以使代码更简洁运行速度有所提升，而/degbug +/full/pdfonly 则不会优化JIT Native Code

  > When you create a new C# project in Visual Studio, the Debug configuration of the project has */optimize- and /debug:full* switches, and the Release configuration has */optimize+ and /debug:pdbonly* switches specified.

  但是可以在VS的Property->Build->Output-> Advanced 中设置 /debug

  c/c++的developers通常会对Managed Code的编译运行这个过程哦开销表示担心。因为unmanaged code编译的过程是有区别的，而且调用的时候可以直接调用。但是在Managed Code的编译运行则要分两步走，1是将source code尽可能的编译成 IL ， 2是在运行的时候需要在Run Time将IL编译成CPU指令，这样一来就需要消耗更多的内存和CPU时间。
  作者作为由C/C++背景转向CLR的人也对这个额外开销抱有怀疑，但是经过他微软的优化，发现整个开销已经控制在最小的范围而且对效率的影响很小。同时作者还认为Managed code在效率上要优于unmanaged code
  * JIT Compiler知道所运行环境的具体CPU型号(书中例子是奔腾4，估计现在要升级了), 在把IL编译成CPU指令的时候，就可以使用CPU中提供的更高性能的指令使得执行的速度更快，而unmanaged compiler则做不到，只能按照最低标准的CPU指令来编译，所以一些特别的指令则没有被使用到
  * JIT Compiler可以预判某些代码的执行结果，尤其是一些if 判断，如果判断时认为其always false就可以不编译if中的代码，从而提升效率 ```
    if (numberOfCPUs > 1) {
    ...
    }  ```
  * CLR后续可能会推出Recompile的功能，在run time的同时 recompile code来提升效率

  对于目前的效率还不满意的读者，可以使用NGEN.exe来将Assembly的IL code生成CPU 指令，这样可以省去了编译的过程中的消耗，但是有一点需要注意
  >  Note that NGen.exe must be conservative about the assumptions it makes regarding the actual execution environment, and for this
reason, the code produced by NGen.exe will not be as highly optimized as the JIT compiler–produced code.

  也就是说NGen.exe在执行的时候不会像JIT compiler运行时候那么的贴近于实际环境，所以NGen.exe在有些判断上面会显得保守。

  > In addition, you may want to consider using the `System.Runtime.ProfileOptimization` class. This class causes the CLR to record (to a file) what methods get JIT compiled while your application is running.

  [An solution that use ProfileOptimization  to improve app launch performance](https://blogs.msdn.microsoft.com/dotnet/2012/10/18/an-easy-solution-for-improving-app-launch-performance/)

### IL and verification

  IL 是stack-based and typeless
  > In my opinion, the biggest benefit of IL isn’t that it abstracts away the underlying CPU. The biggest benefit IL provides is application robustness and security

  读者认为IL提供最大的优势是应用的健壮性和安全性。 CLR在把IL编译成CPU指令时，会执行verification的过程，这个过程主要是通过检查IL code来确保所有的code都是安全的，这其中的检查就包括每个方法被调用时指定的参数个数和类型是否正确，每个方法的返回类型是否正确，以及每个方法是否有返回值等等。而verification的过程中需要使用到Managed Module的metadata中的所有对象方法以及引用的对象的信息

### Unsafe Code

  C#编译器生成的是safe code，但是也允许使用unsafe code，所谓unsafe code就是unmanaged code
  > Unsafe code is allowed to work directly with memory addresses and can manipulate bytes at these addresses

  使用unsafe code就可能会造成一些问题，所以CLR对于使用到unsafe code的方法，必须要加unsafe 关键字 同时在编译代码的时候需要使用 /unsafe 编译开关

  > Microsoft supplies a utility called PEVerify.exe, which examines all of an assembly’s methods and   notifies you of any methods that contain unsafe code.

## Native code generation tool NGen.exe

  > The NGen.exe tool that ships with the .NET Framework can be used to compile IL code to native code when an application is installed on a user’s machine. Because the code is compiled at install time, the CLR’s JIT compiler does not have to compile the IL code at run time.

  使用NGen.exe可以减少程序的启动时间(startup time)以及工作单元(working set)
  NGen.exe 看起来很好，因为它提前把程序编译成了CLR可以直接使用的文件，但是需要注意NGen.exe可能会有以下的潜在问题
  * 没有知识产权保护，使用NGen files的同时必须包含IL Code
  * NGen files 同步的问题， CLR在加载NGen files的时候，会对比files与当前环境的信息，如果CLR Vesion，CPU type，windows operation system等条件有改变的话，CLR不会继续使用NGen files而是使用JIT Compiler重新编译。
  * 较差的执行效率 使用NGen编译出的文件，不会具有JIT Compiler根据当前环境做出一些假设的能力，所以最后编译生成的code会差一些，本来一些编译时就可以做的判断需要运行时才能执行，对效率上有一定的影响。

  综上而言，使用NGen未必就能提升多少效率，有时候还可能适得其反，所以在选择之前，最好实际的操作比较一下，确实有提升效率的话 再选择使用。
  作者给出的建议是，对于serverside的应用，只有在第一个client请求的时候会出现效率问题，因为首次需要初始化的时间会比较长，之后的使用则不会受影响，而是用GNen这种方式，也仅仅在首次请求的时候有一定的优化作用，之后大部分时间里则没有效果。所以使用NGen基本上没有什么意义。而对于client side 应用，NGen的效果可能会好一些，尤其是那些可能同时被多个程序使用的assembly。微软对于大型的client 应用还有一个专门的tool
   > Managed Profile Guided Optimization tool (MPGO.exe)

  用来分析程序在启动的时候需要什么，然后生成一份profile，之后NGen可以利用这个profile对程序来更好的优化。


## The Frameworkf Class Library

  .Net Framework中包含了 FCL， FCL中是微软提供的许多实用的类，方便程序员在编程过程中很好的使用微软技术，而使用这些微软技术可以开发以下这些类型的应用
  * Web Service  
  * Web Forms/MVC Html based applications(web sites)
  * Rich Windows GUI applications -- WPF， Win Forms
  * Windows console applications
  * Windows services
  * Database stored procedures
  * Component library

  FCL中提供的部分类，可以被开发者继承使用，使得开发更容易
  ![FCL Namespaces](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/FCLNamespaces.png)

  作者在这里提到了，本书的目标旨在介绍CLR，面向的是所以利用.NET平台开发应用的程序员。其他的一些专项数据会专注于某一种类的应用来展开，他认为这种方式是
  > learn from the top down

  而这本书的学习则相反
  > learn from the bottom up

## The Common Type System

  CLR is all about Types， Type定义了程序中可以使用的功能，同时也定义了一种编程语言如何与其他编程语言通信的机制，由于Type是CLR之根本，所以微软定义了正式的说明书-- Common Type System (CTS) 用于规范Type的使用
  本书的Part II中会对Type进行详细剖析，这里只提一个概要。
  > The CTS specification states that a type can contain zero or more members

  这里提到的members包括
  * Field  -- 表示一个对象的状态的一部分，由类型和命名来作为标识
  * Method -- 对对象进行操作的一个函数，通常用于修改对象的状态。包含名称，参数及其类型(包括顺序)，以及是否有返回值，如果有返回值则规定返回值的类型
  * Property -- 对于调用方，property跟Field类似，而对于实现方来说则跟Method很像。Properties allow an implementer to validate input parameters and
object state before accessing the value and/or calculating a value only when necessary. They also allow a user of the type to have simplified syntax. Finally, properties allow you to create read-only or write-only “fields."
  * Event -- 一种通知的机制，用于当前对象和其他对象之间。

  Type的访问限制 包括Public， Assembly(internal in C#) 以及 private
  而Type的member也有访问限制
  * private 最低的访问权限，只能被相同Type中的其他member方法
  * Family  可以被继承的Types访问，无论是否在同一个Assembly (protected in C#)
  * Assembly 可以被同一Assembly中的Types访问 (internal in most languages)
  * Family and Assembly 同一个Assembly中的继承Types可以访问，但是C#和一些其他的语言中并没有实现这一限制
  * Family or Assembly 可以被继承类或者同assmebly的Types访问 (protected internal in C# )
  * Public 随便访问

  另外还有一些CTS rule, 比如一个Type只能继承一个base class， 还有Type必须继承自System.Object

## The Common Language specification

  CLR是支持多语言的，也就给不同语言编写的程序在CLR上进行集成提供了可能性，但是不同的语言有自己的语法和标准，就使得互相直接的集成变得很麻烦，为了统一管理规范，就有了CLS。
  CLS是CLR上所支持的多个语言的交集，也就是说每个语言自己独特定义的语法或者类型或者判断条件，如果在别的语言上无法支持那么就是违背了CLS。
  [Cross-Language Interoperability](https://msdn.microsoft.com/en-us/library/730f1wy3.aspx)
  > In the CLR, every member of a type is either a field (data) or a method (behavior). This means that every programming language must be able to
access fields and call methods. Certain fields and certain methods are used in special and common ways. To ease programming, languages typically offer additional abstractions to make coding these common programming patterns easier

  使用ILDASM.exe(location : %\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.6 Tools) 来分析assembly，可以查看assembly中的field和method

## Interoperability with Unmanaged Code

  CLR支持3种与Unmanaged Code互操作
  * Managed Code可以调用dll中的unmanaged function -- 使用的是P/Invoke(Platfor invoke)机制，FCL中有许多调用kernel32.dll， User32.dll的实现
  * Managed Code可以使用已经存在的com组件(server) -- 使用 TlbImp.exe(跟ILDASM.exe路径相同)可以将com组件转成成相对应的managed library
  * Unmanaged Code可以使用Managed Code (server)

  Microsoft now makes available the source code for the Type Library Importer tool and a P/Invoke Interop Assistant tool to help developers needing to interact with native code.
  [Managed, Native, and COM Interop Team](http://clrinterop.codeplex.com/)
