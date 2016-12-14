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
