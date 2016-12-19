# Enumerated Types and Bit Flags

## Enumerated Types

枚举类型的出现提高了代码的可读性和维护性，同时由于其是Type，在编译的过程中会进行语法的检查，可以及时发现使用不当的地方。

枚举类型可以定义一个underlyingType， 默认是int类型的，可以自定义为byte，sbyte,short,int,long...
> Every enumerated type has an underlying type, which can be a byte, sbyte, short, ushort, int (the most common type and what C# chooses by default), uint, long, or ulong. Of course, these C# primitive types correspond to FCL types.

枚举的ToString 方法可以显示出枚举的Name/Symbol和Value,这里我们设定Name是枚举的文字表示而Value则是数字表示
ToString可用的format
* null -- general format
* G -- general format
* D -- decimal format
* X -- hex format

另外注意当枚举类型的underlying Type不同，在hex format时显示出来的位数也是不同的
> Note When using hex formatting, ToString always outputs uppercase letters. In addition,the number of digits output depends on the enum’s underlying type:
2 digits for byte/sbyte, 4 digits for short/ushort, 8 digits for int/uint, and 16 digits for long/ulong. Leading zeros are output if necessary

Enum提供了Static 方法用来获取Enum instance的value和name，同时也提供了Parse 和 TryParse 方法来将name和value转成enum对象

另外还有一个IsDefined 方法 可以用来判断给定的value/name是否是在Enum Type中定义了。不过这个方法不要滥用，一个原因是这个方法内部使用反射实现，效率上会有差，二来只能区分大小写。

关于将枚举类型定义在什么位置，通常情况，要把枚举类型和使用该类型的其他类型定义在相同level，而不是嵌套在使用类型的内部定义。除非为了避免Name Confilict，可以考虑内嵌来定义

## Bit Flags

为枚举类型添加 Flags Attribute，使枚举支持位标志
> When defining an enumerated type that is to be used to identify bit flags, you should, of course, explicitly assign a numeric value to each symbol. Usually, each symbol will have an individual bit turned on.

```
[Flags, Serializable]
public enum FileAttributes {
  ReadOnly = 0x00001,
  Hidden = 0x00002,
  System = 0x00004,
  Directory = 0x00010,
  Archive = 0x00020,
  Device = 0x00040,
  Normal = 0x00080,
  Temporary = 0x00100,
  SparseFile = 0x00200,
  ReparsePoint = 0x00400,
  Compressed = 0x00800,
  Offline = 0x01000,
  NotContentIndexed = 0x02000,
  Encrypted = 0x04000,
  IntegrityStream = 0x08000,
  NoScrubData = 0x20000
}
```

当枚举类型被Flags属性标记时，ToString 方法会有所不同，如下所示
1. The set of numeric values defined by the enumerated type is obtained, and the numbers are sorted in descending order.
2. Each numeric value is bitwise-ANDed with the value in the enum instance, and if the result equals the numeric value, the string associated with the numeric value is appended to the output string, and the bits are considered accounted for and are turned off. This step is repeated until all numeric values have been checked or until the enum instance has all of its bits turned off.
3. If, after all the numeric values have been checked, the enum instance is still not 0, the enum instance has some bits turned on that do not correspond to any defined symbols. In this case, ToString returns the original number in the enum instance as a string.
4. If the enum instance’s original value wasn’t 0, the string with the comma-separated set of symbols is returned.
5. If the enum instance’s original value was 0 and if the enumerated type has a symbol defined with a corresponding value of 0, the symbol is returned.
6. If we reach this step, “0” is returned.

如果枚举类型没有Flags 属性，在ToString的时候 使用 F 参数，仍然可以输出和Flags Enum Type一样的效果

> You should never use the IsDefined method with bit flag–enumerated types. It won’t work for two reasons:
■■ If you pass a string to IsDefined, it doesn’t split the string into separate tokens to look up; it will attempt to look up the string as through it were one big symbol with commas in it. Because you can’t define an enum with a symbol that has commas in it, the symbol will never be found.
■■ If you pass a numeric value to IsDefined, it checks whether the enumerated type defines a single symbol whose numeric value matches the passed-in number. Because this is unlikely for bit flags, IsDefined will usually return false.

## Adding Methods to Enumerated Types

Enum类型不支持自定义Method，可以借助拓展方法的功能来实现，看个例子

```
internal static class FileAttributesExtensionMethods {
  public static Boolean IsSet(this FileAttributes flags, FileAttributes flagToTest) {
    if (flagToTest == 0)
    throw new ArgumentOutOfRangeException("flagToTest", "Value must not be 0");
    return (flags & flagToTest) == flagToTest;
  }
  public static Boolean IsClear(this FileAttributes flags, FileAttributes flagToTest) {
    if (flagToTest == 0)
    throw new ArgumentOutOfRangeException("flagToTest", "Value must not be 0");
    return !IsSet(flags, flagToTest);
  }
  public static Boolean AnyFlagsSet(this FileAttributes flags, FileAttributes testFlags) {
    return ((flags & testFlags) != 0);
  }
  public static FileAttributes Set(this FileAttributes flags, FileAttributes setFlags) {
    return flags | setFlags;
  }
  public static FileAttributes Clear(this FileAttributes flags, FileAttributes clearFlags) {
    return flags & ~clearFlags;
  }
  public static void ForEach(this FileAttributes flags, Action<FileAttributes> processFlag) {
    if (processFlag == null) throw new ArgumentNullException("processFlag");
    for (UInt32 bit = 1; bit != 0; bit <<= 1) {
      UInt32 temp = ((UInt32)flags) & bit;
      if (temp != 0) processFlag((FileAttributes)temp);
    }
  }
}

//how to use the extension methods
FileAttributes fa = FileAttributes.System;
fa = fa.Set(FileAttributes.ReadOnly);
fa = fa.Clear(FileAttributes.System);
fa.ForEach(f => Console.WriteLine(f));
```
