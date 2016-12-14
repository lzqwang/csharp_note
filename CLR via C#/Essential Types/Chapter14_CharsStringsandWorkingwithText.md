# Chars, Strings and Working with text

## Chars
Char Type中提供了很多个static的方法，可以获知当前char的类型，可以转换
  > IsDigit, IsLetter, IsWhiteSpace, IsUpper, IsLower, IsPunctuation, IsLetterOrDigit, IsControl, IsNumber, IsSeparator, IsSurrogate, IsLowSurrogate, IsHighSurrogate, and IsSymbol. Most of these methods call GetUnicodeCategory internally and simply return true or false accordingly

GetUnicodeCategory 返回一个枚举类型-- UnicodeCategory， 其中列举了可能出席的char的类型

> GetNumericValue, returns the numeric equivalent of a character

char可以和数字进行转换，有三种方式可以实现
* Casting -- 直接进行类型转换
* Convert -- 使用Convert类来转换
* IConvertible 接口 -- The Char type and all of the numeric types in the .NET Framework Class Library (FCL) implement the IConvertible interface. 可以用此接口将Char与number进行相互转换

## String

关于String类型， String是class而不是struct，String是 immutable，即恒定类型，一个String对象声明定义之后其value就不会改变了，这一点很重要，尤其是在可以对String 对象进行各种转换截取拼接查询等操作之后，String的值还是不变的，当然如果使用 = 重新赋值了的话 就是另外一回事了。

### Construc String
String 是CLR中保留的原始类型，和数字类型类似可以直接使用字符串来构造一个String对象，而且不能使用New 来构造。

> To concatenate several strings together at run time, avoid using the + operator because it creates multiple string objects on the garbage-collected heap. Instead, use the System.Text.StringBuilder type.

不建议使用 + 来拼接多个string，因为这样会创建多个string 对象， 推荐使用 StringBuilder 来完成拼接字符串的操作

> Finally, C# also offers a special way to declare a string in which all characters between quotes are considered part of the string. These special declarations are called verbatim strings and are typically used when specifying the path of a file or directory or when working with regular expressions

C#提供了一种特殊的方式来为string赋值， 使用 @ 符号， 那么双引号之间的所有字符都会被当作string的一部分，如果不

### Strings are immutable

> Having immutable strings also means that there are no thread synchronization issues when manipulating or accessing a string. In addition, it’s possible for the CLR to share multiple identical String contents through a single String object. This can reduce the number of strings in the system—thereby conserving memory usage—and it is what string interning (discussed later in the chapter) is all about.

String的恒定性，带来的好处是在多线程的情况下，访问操作string对象不会产生线程同步的问题。另外CLR可以减少string对象所占用的内存，这也是string interning的核心

### Compare Strings

Compare Strings 是经常会用到的操作，比较string的方法有很多重载大部分会用到 StringComparison 枚举，少部分会用到 CompareOptions 枚举
```
public enum StringComparison {
CurrentCulture = 0,
CurrentCultureIgnoreCase = 1,
InvariantCulture = 2,
InvariantCultureIgnoreCase = 3,
Ordinal = 4,
OrdinalIgnoreCase = 5
}

[Flags]
public enum CompareOptions {
None = 0,
IgnoreCase = 1,
IgnoreNonSpace = 2,
IgnoreSymbols = 4,
IgnoreKanaType = 8,
IgnoreWidth = 0x00000010,
Ordinal = 0x40000000,
OrdinalIgnoreCase = 0x10000000,
StringSort = 0x20000000
}
```

使用CompareOptions的方法中同时还需要指定CultureInfo.
通常最快的比较方法是使用 Ordinal 或者  OrdinalIgnoreCase
> On the other hand, when you want to compare strings in a linguistically correct manner (usually for display to an end user), you should use StringComparison.CurrentCulture or StringComparison.CurrentCultureIgnoreCase

>Important For the most part, StringComparison.InvariantCulture and StringComparison.InvariantCultureIgnoreCase should not be used.

大部分情况下不会使用 StringComparison.InvariantCulture and StringComparison.InvariantCultureIgnoreCase

> If you want to change the case of a string's characters before performing an ordinal comparison, you should use String’s ToUpperInvariant or ToLowerInvariant method

推荐使用ToUpperInvariant而不是ToLowerInvariant, 微软内部对 ToUpperInvariant 进行了优化，而且FCL中在IgnoreCase的compare中会把字符串ToUpperInvariant

```
Boolean Equals(String value, StringComparison comparisonType)
static Boolean Equals(String a, String b, StringComparison comparisonType)
static Int32 Compare(String strA, String strB, StringComparison comparisonType)
static Int32 Compare(string strA, string strB, Boolean ignoreCase, CultureInfo culture)
static Int32 Compare(String strA, String strB, CultureInfo culture, CompareOptions options)
static Int32 Compare(String strA, Int32 indexA, String strB, Int32 indexB, Int32 length, StringComparison comparisonType)
static Int32 Compare(String strA, Int32 indexA, String strB, Int32 indexB, Int32 length, CultureInfo culture, CompareOptions options)er
static Int32 Compare(String strA, Int32 indexA, String strB, Int32 indexB, Int32 length, Boolean ignoreCase, CultureInfo culture)
Boolean StartsWith(String value, StringComparison comparisonType)
Boolean StartsWith(String value,
Boolean ignoreCase, CultureInfo culture)
Boolean EndsWith(String value, StringComparison comparisonType)
Boolean EndsWith(String value, Boolean ignoreCase, CultureInfo culture)
```
> String’s other comparison methods—CompareTo (required by the IComparable interface), CompareOrdinal, and the == and != operators—should also be avoided

除了上面列出的方法之外，其他的方法包括 == , !=  不推荐使用，因为这些方法没有显式的指定比较的条件，cultureinfo，是否区分大小写，而不同的方法可能有不同的default条件
> For example, by default, CompareTo performs a culture-sensitive comparison, whereas Equals performs an ordinal comparison. Your code will be easier to read and maintain if you always indicate explicitly how you want to perform your string comparisons

所以最好在比较的时候指明条件，这样让代码的意图简单明了

关于CultureInfo
The .NET Framework uses the System.Globalization.CultureInfo type to represent a language/country pair
CultureInfo 有两个重要的属性
* CurrentUICulture -- 用于显示的cultureInfo 默认的是与CurrentCulture一样的，可以修改
* CurrentCulture -- 除了CurrentUICulture之外的cultureinfo包括日期时间 等

在考虑cultureinfo的情况下，compare string要更复杂，在某些语言中，一些特殊的字符在比较的时候会转换成相应的英语字符
> *Note* When the Compare method is not performing an ordinal comparison, it performs character expansions. A character expansion is when a character is expanded to multiple characters regardless of culture. In the above case, the German Eszet character ‘ß’ is always expanded to ‘ss.’ Similarly, the ‘Æ’ ligature character is always expanded to ‘AE.’ So in the code example, the second call to Compare will always return 0 regardless of which culture I actually pass in to it

### String interning

```
public static String Intern(String str);
public static String IsInterned(String str);
```

使用Intern可以让不同的string 对象在使用相同的字符串时，访问的是内存堆上的同一个资源。在实现上使用了hash table来记录string 对象是否已经interning

利用String Interning机制，在比较字符串的时候可以提升一定的效率节省一定的内存，比较下面两个例子
```
// nomral way to compare strings
private static Int32 NumTimesWordAppearsEquals(String word, String[] wordlist) {
Int32 count = 0;
for (Int32 wordnum = 0; wordnum < wordlist.Length; wordnum++) {
if (word.Equals(wordlist[wordnum], StringComparison.Ordinal))
count++;
}
return count;
}

private static Int32 NumTimesWordAppearsIntern(String word, String[] wordlist) {
// This method assumes that all entries in wordlist refer to interned strings.
word = String.Intern(word);
Int32 count = 0;
for (Int32 wordnum = 0; wordnum < wordlist.Length; wordnum++) {
if (Object.ReferenceEquals(word, wordlist[wordnum]))
count++;
}
return count;
}
```

### Examining a String’s Characters and Text Elements

利用String以及StringInfo类 对 String对象进行各种操作，clone, copy, substring, insert, remove  等等
关于这些方法他们有一个共同点，就是他们都是返回一个新的string 对象。 *String is immutable*

## Constructing a String Efficiently

由于String 的immutable 恒定性， FCL 提供了另外一个Type StringBuilder 来更高效的创建String对象。注意这里只是用StringBuilder来创建String对象，至于方法参数中仍然要使用String。

### Obtaining a String Representation of an Object: ToString

ToString 方法用来显示string。

```
public interface IFormattable {
  String ToString(String format, IFormatProvider formatProvider);
}
```
 > IFormattable’s ToString method takes two parameters. The first, format, is a string that tells the method how the object should be formatted. ToString’s second parameter, formatProvider, is an instance of a type that implements the System.IFormatProvider interface. This type supplies specific culture information to the ToString method.

 第二参数主要是用来指定culture info

 FCL中很多的type都提供了多种ToString format，常用的是DateTime 和 Number 类型的 format
 DateTime type supports
 * “d” for short date,
 * “D” for long date,
 * “g” for general,
 * “M” for month/day,
 * “s” for sortable,
 * “T” for long time,
 * “u” for universal time in ISO 8601 format,
 * “U” for universal time in full date format,
 * “Y” for year/month, and others

 all of the built-in numeric types support
 * “C” for currency,
 * “D” for decimal,
 * “E” for exponential(scientific) notation,
 * “F” for fixed-point,
 * “G” for general,
 * “N” for number,
 * “P” for percent,
 * “R” for round-trip,
 * “X” for hexadecimal

 使用 String.Format 方法和 StringBuilder 对象的 AppendFormat方法可以将多个对象，按照指定的格式输出。

### Providing Your Own Custom Formatter

可以自定义一个 formatter 可以自定义rule， 看下面的例子
```
class CustomFormatter : ICustomFormatter, IFormatProvider   //需要继承两个接口
{
    public string Format(string format, object arg, IFormatProvider formatProvider)
    {
        string s;
        IFormattable formattable = arg as IFormattable;
        if (formattable == null)
        {
            s = arg.ToString();
        }
        else
        {
            s = formattable.ToString(format, formatProvider);
        }
        if (arg.GetType() == typeof(Int32))
        {
            return String.Format("<B>{0}</B>", s);
        }
        else
        {
            return s;
        }
    }

    public object GetFormat(Type formatType)   //  这个方法会先触发，根据formatType如果是customFormatter就返回，如果不是返回当前culture下的
    {
        if (formatType == typeof(ICustomFormatter))
        {
            return this;
        }
        else
        {
            return System.Threading.Thread.CurrentThread.CurrentCulture.GetFormat(formatType);
        }
    }
}
```

## Encodings: Converting Between Characters and Bytes

Encode/Decode 发生在strings需要和stream相互转换的时候
首先介绍CLR中支持的Encoding 类型
两个最常用的是UTF-16和 UTF-8
* ■■ UTF-16 encodes each 16-bit character as 2 bytes. It doesn’t affect the characters at all, and no compression occurs—its performance is excellent. UTF-16 encoding is also referred to as *Unicode* encoding. Also note that UTF-16 can be used to convert from little-endian to big-endian and vice versa. -- 每个16位的字符都是2个字节
* ■■ UTF-8 encodes some characters as 1 byte, some characters as 2 bytes, some characters as 3 bytes, and some characters as 4 bytes. Characters with a value below 0x0080 are compressed to 1 byte, which works very well for characters used in the United States. Characters between 0x0080 and 0x07FF are converted to 2 bytes, which works well for European and Middle Eastern languages. Characters of 0x0800 and above are converted to 3 bytes, which works well for East Asian languages. Finally, surrogate pairs are written out as 4 bytes. UTF-8 is an extremely popular encoding, but it’s less efficient than UTF-16 if you encode many characters with values of 0x0800 or above.

除此之外还有一些不常用的encoding
* ■■ UTF-32 encodes all characters as 4 bytes. This encoding is useful when you want to write a simple algorithm to traverse characters and you don’t want to have to deal with characters taking a variable number of bytes. For example, with UTF-32, you do not need to think about surrogates because every character is 4 bytes. Obviously, UTF-32 is not an efficient encoding in terms of memory usage and is therefore rarely used for saving or transmitting strings to a file or network. This encoding is typically used inside the program itself. Also note that UTF-32 can be used to convert from little-endian to big-endian and vice versa.
* ■■ UTF-7 encoding is typically used with older systems that work with characters that can be expressed using 7-bit values. You should avoid this encoding because it usually ends up expanding the data rather than compressing it. The Unicode Consortium has deprecated this encoding.
* ■■ ASCII encodes the 16-bit characters into ASCII characters; that is, any 16-bit character with a value of less than 0x0080 is converted to a single byte. Any character with a value greater than 0x007F can’t be converted, so that character’s value is lost. For strings consisting of characters in the ASCII range (0x00 to 0x7F), this encoding compresses the data in half and is very fast (because the high byte is just cut off). This encoding isn’t appropriate if you have characters outside of the ASCII range because the character’s values will be lost.

> The Default property returns an object that is able to encode/decode using the user’s code page as specified by the Language For Non-Unicode Programs option of the Region/Administrative dialog box in Control Panel. (See the GetACP Win32 function for more information.) However, using the Default property is discouraged because your application’s behavior would be machine-setting dependent, so if you change the system’s default code page or if your application runs on another machine, your application will behave differently.

Encoding class 里面可以获取到 UTF-16，UTF-8 以及上面提到的不常用的encoding type，另外还有一个Default Encoding, 这个Default Encoding跟机器的设置有关，所以在使用Encoding的时候不要用Default，因为Default有不确定性，可能会让程序出现非预期的结果，所以通常不建议使用。

### Secure String

> Never put the cotents of a secure string into a string

SecureString 没有提供ToString 方法，也不建议将SecureString convert to string，因为这样就会把敏感信息暴露出来

TABLE--Methods of the Marshal Class for Working with Secure Strings

Method to Decrypt SecureString to Buffer |Method to Zero and Free Buffer
--|--
SecureStringToBSTR |ZeroFreeBSTR
SecureStringToCoTaskMemAnsi |ZeroFreeCoTaskMemAnsi
SecureStringToCoTaskMemUnicode |ZeroFreeCoTaskMemUnicode
SecureStringToGlobalAllocAnsi |ZeroFreeGlobalAllocAnsi
SecureStringToGlobalAllocUnicode |ZeroFreeGlobalAllocUnicode

使用System.Runtime.InteropServices.Marshal 可以将SecureString 的content解析出来，但是这个类是unsafe code，在编译的时候需要allow unsafe code， 在VS中可以设置 properties-- build -- allow unsafe code 勾选
