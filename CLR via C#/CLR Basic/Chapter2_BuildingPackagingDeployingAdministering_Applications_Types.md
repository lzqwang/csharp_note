#Building, Packaging, Deploying, and Administering Applications and Types

## .NET Framework Deployment Goal

  长久以来，windows系统以不稳定和复杂而臭名昭著
  第一个原因是不稳定，在安装程序的时候，会出现DLL冲突(DLL hell),导致新装的程序使已存在的程序崩溃
  第二个原因是复杂，是说安装一个程序的时候，会涉及到拷贝文件到指定路径下，还可能修改注册表信息，创建桌面或者菜单快捷方式等等，这个安装是很复杂的，因为所有的东西不是一个简单的整体而是分散在不同的角落，以至于当卸载程序的时候，都会怀疑这个程序是不是真的完全卸载干净了。
  另外一个原因则是安全性，安装一个程序时，程序中包含的文件可能在系统中做一些用户不知道的操作。

  针对以上这些问题，.NET Framework是如何应对的呢
  首先，.NET Framework对于DLL hell的问题进行彻底的解决，通过Chapter3的内容可以更好的了解这一方面。另外对于上面提到的第二个原因，.NET Framework花了很长时间去解决安装程序的时候文件分布分散的问题，比如Types现在不需要注册表中的设置，但是快捷链接还是需要的。至于安全性， .NET Framework引入了Security Model叫做 code access security, 它允许hosts去设置权限，可以控制加载的组件可以做哪些操作。
  总之，作者的意思就是windows早些年因为一些问题被诟病，现在.NET Framework的出现就是这些问题的克星，要提windows力挽狂澜。

## Building Types into Model

  Program.cs 代码如下
  `public sealed class Program {
    public static void Main() {
      System.Console.WriteLine("Hi");
    }
   }`
   把Program.cs build成为一个exe，执行下面的命令
   `csc.exe /out:Program.exe /t:exe /r:MSCorLib.dll Program.cs`

  其中/r 是reference， 因为引用了 *System.Console* ,由于MSCorlib.dll是common使用率很高所以默认都会加载这个dll，可以不显式的表示出来。

### Response FileInfo

  使用csc.exe的时候可以将 参数写在response file中，然后让csc.exe在执行的时候去response file中获取相应的参数即可
  针对上面的例子，可以写一个 MyResponse.rsp
  `/out:Program.exe
  /t:exe
   /r:MSCorLib.dll
  `
  然后执行
  `csc.exe @MyResponse.rsp Program.cs`
  在 .NET　Framework安装路径下会有一个默认的csc.rsp文件，其中的内容包含了许多 reference, 这样user在自己使用csc.exe的时候就不需要格外指定reference了。
  可以使用 /noconfig 来忽略本地或者全局的 response file。
  另外关于 /r 中指定的都是reference是相对路径的，编译器会到以下这些路径下寻找
  * Working directory.
  * The directory that contains the CSC.exe file itself. MSCorLib.dll is always obtained from this directory. The path looks something like this: %SystemRoot%\Microsoft.NET\Framework\v4.0.#####.
  * Any directories specified using the /lib compiler switch.
  * Any directories specified using the LIB environment variable.

## A brief look at Metadata

  > The metadata is a block of binary data that consists of several tables. There are three categories of tables: definition tables, reference tables, and manifest tables

  *Definition Table*

  Metadata defination table name| Description 
  :-----------------------------|--------------
  ModuleDef|包含了Module的入口(entry)，这个入口有包括了module的文件名，后缀名，versionID
  TypeDef|包含了每个Type的入口，而每个入口又包括了Type的name，基类型，访问限制(public,private..),同时还包括了在其他表中的索引(Methodef,FieldDef,PropertyDef以及EventDef)
  MethodDef|Module中每个Method的入口，入口包括method name，flags(public，private,abstract, virtual,static,final etc..),以及在IL Code中可以找到该方法的位置。同时每个Entry还包括了ParamDef的信息，详细的定义了Method的parameters信息
  FieldDef|Module中每个field的entry，包括了field的name，type以及flags
  ParamDef|Module中每个parameter的entry，包括了parameter的name，type以及flags(in,out，retval etc)
  PropertyDef|property entry
  EventDef|event entry

  *Reference Table*

  Metadata Reference Table name| Description
  :----------------------------|-----------
  AssemblyDef|Module中引用的每个assembly的entry，Contains one entry for each assembly referenced by the module. Each entry includes the information necessary to bind to the assembly: the assembly’s name (without path and extension), version number, culture,and public key token (normally a small hash value generated from the publisher’s public key, identifying the referenced assembly’s publisher). Each entry also contains some flags and a hash value. This hash value was intended to be a checksum of the referenced assembly’s bits. The CLR completely ignores this hash value and will probably continue to do so in the future.
  ModuleRef|Contains one entry for each PE module that implements types referenced by this module. Each entry includes the module’s file name and extension (without path). This table is used to bind to types that are implemented in different modules of the calling assembly’s module.
  TypeRef|Contains one entry for each type referenced by the module. Each entry includes the type’s name and a reference to where the type can be found. If the type is implemented within another type, the reference will indicate a TypeRef entry. If the type is implemented in the same module, the reference will indicate a ModuleDef entry. If the type is implemented in another module within the calling assembly, the reference will indicate a ModuleRef entry. If the type is implemented in a different assembly, the reference will indicate an AssemblyRef entry.
  MemberRef|Contains one entry for each member (fields and methods, as well as property and event methods) referenced by the module. Each entry includes the member’s name and signature and points to the TypeRef entry for the type that defines the member

  使用ILDASM可以查看上面提的的table的具体详细信息
  > To see the metadata in a nice, human-readable form, select the View/MetaInfo/Show! menu item (or press Ctrl+M)

  使用ILDASM还可以查看统计信息
  > select the ILDasm’s View/Statistics menu item

## Compiling Modules to form an Assembly

  >To summarize, an assembly is a unit of reuse, versioning, and security. It allows you to partition your types and resources into separate files so that you, and consumers of your assembly, get to determine which files to package together and deploy. After the CLR loads the file containing the manifest, it can determine which of the assembly’s other files contain the types and resources the application is referencing. Anyone consuming the assembly is required to know only the name of the file containing the manifest; the file partitioning is then abstracted away from the consumer and can change in the future without breaking the application’s behavior

  Manifest Metadata Table Name| Description
  :---------------------------|-------------------------
  AssemblyDef |Contains a single entry if this module identifies an assembly. The entry includes the assembly’s name (without path and extension), version (major, minor, build, and revision), culture,flags, hash algorithm, and the publisher’s public key (which can be null).
  FileDef | Contains one entry for each PE and resource file that is part of the assembly (except the file containing the manifest because it appears as the single entry in the AssemblyDef table).The entry includes the file’s name and extension (without path), hash value, and flags. If this assembly consists only of its own file, the FileDef table has no entries.
  ManifestResourceDef| Contains one entry for each resource that is part of the assembly. The entry includes the resource’s name, flags (public if visible outside the assembly and private otherwise), and an index into the FileDef table indicating the file that contains the resource file or stream. If the resource isn’t a stand-alone file (such as a .jpg or a .gif), the resource is a stream contained within a PE file. For an embedded resource, the entry also includes an offset indicating the start of the resource stream within the PE file.
  |ExportedTypesDef| Contains one entry for each public type exported from all of the assembly’s PE modules. The entry includes the type’s name, an index into the FileDef table (indicating which of this assembly’s files implements the type), and an index into the TypeDef table. Note: To save file space, types exported from the file containing the manifest are not repeated in this table because the type information is available using the metadata’s TypeDef table|

  C# 编译器使用下面这些 target时，/t[arget]:exe, /t[arget]:winexe, /t[arget]: appcontainerexe, /t[arget]:library, or /t[arget]:winmdobj
  会生成Assembly包含manifest 信息，
  如果使用 /t[target] :module ,只编译出一个PE 不包含Metadata manifest tables，生成的文件 后缀名为 .netmodule

  如果之后想把 module 加到Assembly中， 只需要加上 /addmodule 参数
  > csc /out:MultiFileLibrary.dll /t:library /addmodule:RUT.netmodule FUT.cs

  *VS 中不支持MultiFileLibrary，如果要实现这种效果只能使用命令行*

  使用AL.exe可以把 .netmodule build 成 assembly，也可以将source files加到assmebly中
  `csc /t:module /r:MultiFileLibrary.dll Program.cs
   al /out:Program.exe /t:exe /main:Program.Main Program.netmodule`

## Assembly Version Resource Information

  build assembly的时候，需要指定assembly的一些信息，使用VS开发C#的应用，默认会生成一个 *AssemblyInfo.cs* 文件，里面包含了Assembly的信息，直接修改这个文件里面的内容即可。
  关于VesionNumber的定义

       |Major Version|Minor Version|Build Number|Revision Number
  -----|-------------|-------------|------------|---------------
  example|2|1|789|2

  Major Version 和 Minor Version比较好理解也是我们经常使用到的，关于这个Build Number和Revision Number可以这样理解，如果这个Assembly每天都会build，那么Build Number就需要每一天增加1，而Revision Number则是当每天build的次数不止一次的时候使用，表示当天build的次数。

  通过查看 *AssemblyInfo.cs* 会发现一下三种version
  `[assembly: AssemblyVersion("1.0.0.0")]
   [assembly: AssemblyFileVersion("1.0.0.0")]
   [assembly:AssemblyInformationalVersion("1.0.0.0")]`

  这其中只有AssemblyVersion是出现在Metadata中的，CLR会用这个version去寻找reference或者dependency。
  另外两个则主要是起到一个解释说明的作用，

## Culture

  关于程序的Culture，通常情况下如果程序包含多种语言的资源文件，微软建议是不指定Assembly的culture，这种情况下assembly被视为 culture-neutral
  然后就可以根据culture-neutral的assembly创建 culture-specifed的assembly,这种Assembly被称为 satellite assembly
  > Assemblies that are marked with a culture are called satellite assemblies.
  For these satellite assemblies, assign a culture that accurately reflects the culture of the resources placed in the assembly. You should create one satellite assembly for each culture you intend to support

  通常使用AL.exe来生成satellite assembly，因为不需要包含code，只需要包含特定语言的资源文件即可。
  通常情况下，assmebly不应该引用satellite assmebly而是引用culture-neutral的assembly，如果必须要使用satellite assembly中的member，可以使用反射技术

## Simple Application Deployment (Privately Deployed Assemblies)  

  对于Windows Store Apps，packaging rule非常严格，VS将application中用到的所有assemblies 打包到一个.appx文件中，不同的user安装app的时候，如果是同一个version那么是共用一个.appx， 如果是不同version的话则会在不同的路径下创建出 不同version的 .appx文件。
  而对于Desktop application, assmebly不需要进行打包操作，可以直接将assemblies复制到其他路径下，就可以运行，至于卸载，只需要把assemblies文件都删除即可。
  另外可以使用 其他的一些机制对assmebly进行整合，比如 .cab(typically used for Internet download scenarios to compress files and reduce download times)
  或者可以将assemblies打包到MSI中(by the Windows Installer service (MSIExec.exe))

## Simple Administrative Control (Configuration)

  application可以有一个相关的 config文件(xml格式)用来配置一些程序运行时可能用到的信息
  比如说Program.exe里面引用了 MultiFileLibrary.dll，但是MultiFileLibrary.dll保存在另外一个路径下，运行Program.exe的时候会提示找不到MultiFileLibrary.dll而报错
  `AppDir directory (contains the application’s assembly files)
    Program.exe
    Program.exe.config (discussed below)
      AuxFiles subdirectory (contains MultiFileLibrary’s assembly files)
        MultiFileLibrary.dll
        FUT.netmodule
        RUT.netmodule`

解决这个问题可以利用 Program.exe.config
  `<configuration>
    <runtime>
      <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
        <probing privatePath="AuxFiles" />
      </assemblyBinding>
    </runtime>
  </configuration>`

  probing -- 表示会到'AuxFiles' 这个路径下去寻找assembly bingding

  另外注意一点，默认的CLR查找assemblybing时会查以下两个路径
  `appdir/assemblyName.dll
   appdir/assemblyName/assemblyName.dll`
  然后才会去找config文件中配置的 probing path
  使用Fuslogvw.exe 这个tool可以查看 load assembly binding的相关信息
  [Fuslogvw.exe (Assembly Binding Log Viewer)](https://msdn.microsoft.com/en-us/library/e74a18c4)

  默认的.NET Framework每个version会提供一个 machine.config 其中包括了common的设置，一般来说不要修改这里的内容，一些特殊的设置可以在application 自己的config中进行配置
  
