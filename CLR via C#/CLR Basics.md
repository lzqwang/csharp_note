old content here
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
  ![Calling method](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/callingmethod.png)
  
