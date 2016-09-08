# Shared Assemblies and Strongly Named Assemblies

  这一章节关注的是Assembly共享的问题，微软提倡Assembly的共用性，正常开发过程中引用 FCL便是很好的体现，除了微软官方提供的assembly，其他公司机构提供的assembly也是可以共享使用的，然而随即产生的一个问题便是，assembly随着时间的推移，使用者的增多，或多或少会需要fix bugs，add new feature，这样就涉及到了历史版本和新版本的兼容的问题，如果保证引用旧版本的assembly再升级了新的assembly之后，原来的功能完全不受影响，这是一个很重要的话题也是一个很难处理的问题，且听作者如何娓娓道来。

## Two kinds of Assemblies, two kinds of Deployment

  有两种类型的Assembly一种是强签名(strong named), 一种是弱签名(weakly named)
  而两种类型的deployment，一种是privately一种是globally
  > The real difference between weakly named and strongly named assemblies is that a strongly named assembly is signed with a publisher’s public/private key pair that uniquely identifies the assembly’s publisher.

  弱签名类型的assembly只能privately deploy，不能globally deploy

## Giving an assembly a strong name

  上面提到了，weakly named assembly和strongly named assembly的区别主要在于assembly是否用key pair进行签名，所以要把一个assembly 变成strongly named，只需要给他用key pair进行签名，那么首先需要一个key pair， 用SN.exe创建publc key pair file

  `SN –k MyCompany.snk`

  如果想查看生成的public key 使用下面两行命令

  `SN –p MyCompany.snk MyCompany.PublicKey sha256`

  生成MyCompany.PublicKey文件，然后执行

  `SN –tp MyCompany.PublicKey`

  可以得到如下输出

  > Microsoft (R) .NET Framework Strong Name Utility Version 4.0.30319.17929
Copyright (c) Microsoft Corporation. All rights reserved.
Public key (hash algorithm: sha256):
00240000048000009400000006020000002400005253413100040000010001003f9d621b702111
850be453b92bd6a58c020eb7b804f75d67ab302047fc786ffa3797b669215afb4d814a6f294010
b233bac0b8c8098ba809855da256d964c0d07f16463d918d651a4846a62317328cac893626a550
69f21a125bc03193261176dd629eace6c90d36858de3fcb781bfc8b817936a567cad608ae672b6
1fb80eb0
Public key token is 3db32f38c8b42c9a

另外SN.exe无法获取 private key的信息

接下来可以使用生成的key file，来为assembly签名

`csc /keyfile:MyCompany.snk Program.cs`

![Signing an assembly](https://raw.githubusercontent.com/lzqwang/csharp_note/master/CLR%20via%20C%23/screenshot/signingAssembly.png)

## The Global Assembly Cache

  > If an assembly is to be accessed by multiple applications, the assembly must be placed into a wellknown directory, and the CLR must know to look in this directory automatically when a reference to the assembly is detected.This well-known location is called the global assembly cache (GAC).

  GAC的位置是受制于.NET Framework 所以其location可能会随着.NET Framework的更新而改变， 通常在以下路径下可找到

  > %SystemRoot%\Microsoft.NET\Assembly

  使用GACUtil.exe来安装或者卸载GAC中的assembly

  注意只有强签名的assembly才能加入GAC中，运行GACUtil.exe需要管理员程序，相同Name的assembly也可以安装到GAC中，在GAC中会使用Assembly的public key token等信息来创建一个子路径，然后再把assembly放到子路径下。

## Building an Assembly that reference a strongly named assembly

  如果一个Assembly引用了一个强类型的assembly，比方说FCL中提供的dll，那么在编译时找的assembly位置和运行时加载的assembly不在同一个位置
  加载dll所查找的位置如下

  1. Working directory.
  2. The directory that contains the CSC.exe file itself. This directory also contains the CLR DLLs.
  3. Any directories specified using the /lib compiler switch.
  4. Any directories specified using the LIB environment variable

  当运行时，去GAC中加载dll
  之所以在编译的时候不去GAC中找Reference是因为GAC的location不是固定的，所以在指定 reference的时候需要指定 assembly在GAC中的具体位置，比较麻烦。
  虽然assembly相当于是在GAC和Combiler的路径下复制了两份，但是这两个路径下的assembly还是有所区别的
  在compiler 路径下的assembly是 machine agnostic的，也就是只包含Metdata信息不包含IL Code，也不缺分CPU是什么类型的。而在GAC中的assembly则需要包含Metadata和ILCode，并且需要为不同的CPU版本安装不同的assembly

## Strongly Named Assemblies Are Tamper-Resistant

  > When an assembly is installed into the GAC, the system hashes the contents of the file containing the manifest and compares the hash value with the RSA digital signature value embedded within the PE file (after unsigning it with the public key). If the values are identical, the file’s contents haven’t been tampered with.
  In addition, the system hashes the contents of the assembly’s other files and compares the hash values with the hash values stored in the manifest file’s FileDef table. If any of the hash values don’t match, at least one of the assembly’s files has been tampered with, and the assembly will fail to install into the GAC.

  将程序集安装到GAC的时候会检查比较manifest文件的hash value和签名时生成的RSA 数字签名的hash value是否相等，除此之外，对于程序集中的其他文件也需要其hash value与manifest文件中FileDef表里保存的value是否相同

  当一个应用要绑定一个assembly的时候，CLR首先根据这个assembly的name/version/public key token 等信息去GAC中load，如果找到了则返回这个Assembly所在的路径，以供运行时访问，如果在GAC中没有找到会继续到应用的工作路径(working directory)下去找，如果应用是使用MSI安装的，CLR会让MSI去load，如果上述都没有找到程序集，那么会排出 *System.IO.FileNotFoundException* .

  当CLR不是从GAC中load assembly的时候，CLR会在load assembly的时候去比较hash value(跟install assembly into GAC时的check一样),这样虽然会有一定的效率影响，但是确保所引用的assembly是正确的没有被修改的。
  这里注意到，如果是加载GAC中的assembly，因为其在安装的时候已经做过了防干扰的check，在load的时候就不会再被check了，而相反如果不在GAC中的assembly则每次加载的时候都需要check，这点区别也算是GAC的优势之一

## Delayed Signing

  Delayed Signing允许assembly只包含Public Key但不包含Private Key，保证其可以在GAC中正常安装，通常在开发测试阶段使用此策略，等待正式发布的时候再使用private key进行签名

  `csc /keyfile:MyCompany.PublicKey /delaysign MyAssembly.cs`

  让CLR不对生成的assembly进行hash和comparison，只需运行一次即可，不需要每次build dll都运行

  `SN.exe –Vr MyAssembly.dll`

  正式发布assembly的时候，用private key进行签名

  `SN.exe -Ra MyAssembly.dll MyCompany.PrivateKey`

  然后需要恢复对assembly的hash和comparison

  `SN.exe –Vu MyAssembly.dll`

  如果private key是使用CSP保存的，那么上述指令需要有相应的变化

  > If your public/private key pair is in a CSP container, you’ll have to specify different switches to the CSC.exe, AL.exe, and SN.exe programs: When compiling (CSC.exe), specify the /keycontainer switch instead of the /keyfile switch; when linking (AL.exe), specify its /keyname switch instead of its /keyfile switch; when using the Strong Name program (SN.exe) to add a private key to a delay-signed assembly, specify the –Rc switch instead of the –R switch. SN.exe offers additional switches that allow you to perform operations with a CSP.

  Delayed Signing 还有一个用处是与混淆(obfuscate)有关，因为混淆的代码不可以是签名之后的，所以可以先混淆然后再签名

## Privately deploy strongly named Assembly

  尽管GAC中部署Assembly有许多优点，但是并不是要求所有的强签名程序集都在GAC中部署，如果程序集确实是需要shared，那么应该在GAC中部署，但是如果不需要Shared给别的程序，那么完全可以只在working directory部署即可。
  另外作者还介绍了一种privately deployment的但是也可以被程序共享的方法，在config文件中指定引用的assembly的codebase。codebase可以是任何路径，如果是一个web url，那么CLR会把相应的assembly下载到本地，之后在使用assembly的时候会比较本地的assembly与url上的assembly的时间，如果需要更新，就会重新下载，否则直接使用当前的

  > the CLR will automatically download the file and store it in the user’s download cachea subdirectory under C:\Users\UserName\LocalSettings\Application Data\Assembly, where UserName is the name of the Windows user account currently signed on).


## How the Runtime resolve Type Reference

  Resolve Type Reference 有3种情况
  1. 相同assembly 相同file
  2. 相同assembly 不同file
  3. 不同assembly 不同file

## Advanced Administrative Control (Configuration)

  使用config file中其他的节点可以配置assembly加载

  ```
  <?xml version="1.0"?>
    <configuration>
      <runtime>:      
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
          <probing privatePath="AuxFiles;bin\subdir" />
          <dependentAssembly>
            <assemblyIdentity name="SomeClassLibrary" publicKeyToken="32ab4ba45e0a69a1" culture="neutral"/>
            <bindingRedirect oldVersion="1.0.0.0" newVersion="2.0.0.0" />
            <codeBase version="2.0.0.0" href="http://www.Wintellect.com/SomeClassLibrary.dll" />
          </dependentAssembly>
          <dependentAssembly>
            <assemblyIdentity name="TypeLib" publicKeyToken="1f2e74e897abbcfe" culture="neutral"/>
              <bindingRedirect oldVersion="3.0.0.0-3.5.0.0" newVersion="4.0.0.0" />
              <publisherPolicy apply="no" />
        </dependentAssembly>
      </assemblyBinding>
    </runtime>
</configuration>
```
bindingRedirect-- 表示load new version的assembly替换old version
codeBase -- 表示assembly的路径，可以是任意路径
publisherPolicy -- 默认这个选项是 yes， assmbly的publisher在更新了assembly之后可以生成一个config跟new version的 assembly一起发布，这样引用了这个assembly的其他application就会按照config中的配置去load new version assembly，如果不想这么做的话，需要把这个值设置为no

另外注意 只有对assembly进行更新的时候才需要创建publisher policy assembly
> A publisher should create a publisher policy assembly only when deploying an update or a service pack version of an assembly. When doing a fresh install of an application, no publisher policy assemblies should be installed.
