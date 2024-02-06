# 一、公共语言运行时

## 1、相关概念区分

- .NET:以下技术的统称，是一个开发平台
- .Net Framework/.Net Core/Mono:3种将.Net技术打包发行的方式，其选择的组件不一定相同
- CLR:`Common Language Runtime`，一个**执行引擎**，用于进行一类程序（CLI），提供类型系统、垃圾回收、JIT等功能
- CLI:`Common Language Infrastructure`，一个经标准化组织认证的标准（ECMA-335），定义了CLR的基础功能和其上执行的程序的标准
- IL:`Intermediate Language`，用于将程序**直接**输入CLR的格式，包含二进制格式与其对应的文本格式；IL中的任何构造都与CLR内部严格一一对应
- CoreFX:随CLR一同发行的发行的一组子程序与类型，任何在CLR上运行的程序皆可调用；其中最底层的部分不能完全用IL表示，有些则是对CLR自身功能的调用，而上层可能依赖对底层的假设，因此其版本与CLR的版本是绑定的
- C#:一个编程语言，使用C语系风格；除了IL能直接表示的基本功能外，还提供了一些经过一定封装的功能；每个功能都可能需要假定CoreFX提供了特定的类型来完成；另两个对应的、由第一方支持的编程语言为F#和VB，其功能集和C#不完全一致
- Roslyn:第一方（标准制定者）实现的编译器，Mono也实现了一个（mcs），二者共同遵循C#语言标准
- .Net Standard:在不同的发行之间为CoreFX的公开接口（即使用方法）及行为规定标准，但不关心其具体实现细节，也允许每个发行提供一些额外内容

## 2、托管&&非托管

- **托管代码（Managed Code）：**
  - 托管代码是由CLR管理的代码。
  - 托管代码通常是用高级语言（如C#、VB.NET）编写的，这些代码会被编译成中间语言（IL）并存储在程序集中。
  - 托管代码受CLR的管理，因此具有更高的安全性、可靠性和跨平台性。
- **非托管代码（Unmanaged Code）：**
  - 非托管代码是不依赖CLR管理的代码，通常由底层语言（如C、C++）编写。
  - 非托管代码直接在计算机硬件上执行，不受CLR的控制和管理。
  - 由于非托管代码绕过了CLR的一些管理机制，可能导致内存泄漏、安全性问题等。
- **托管类型（Managed Types）：**
  - 托管类型是在CLR环境中定义和管理的数据类型。
  - 托管类型的对象由CLR负责内存分配和垃圾回收。
  - **引用类型**;**含引用类型或托管类型成员（字段、自动实现get访问器的属性）的结构**

- **非托管类型（Unmanaged Types）：**
  - 非托管类型是在非托管环境中定义的数据类型，通常由底层语言（如C、C++）定义。
  - 在与非托管代码交互的时候，C#代码可能需要使用一些特殊的机制（如Interop）来处理非托管类型。
  - **枚举**;**指针**;**不含引用类型及托管类型成员（字段、自动实现get访问器的属性）的结构**

# 二、语言相关

## 1.关于字典类型

- 导入命名空间

```c#
using System.Collections.Generic;
```

- 创建字典对象

```c#
Dictionary<key, value> DictionaryName = new Dictionary<key, value>();
```

`key-value`是对应类型名，同时要注意其访问修饰符

- 添加键值对

```c#
DictionaryName.Add(key, value);
```

- 访问值

```c#
// 使用索引器（[]）获取字典的值
int value = DictionaryName[key];
Console.WriteLine(value);

// 使用TryGetValue获取
int valiue;
if (DictionaryName.TryGetValue(key, out value))
{
	Console.WriteLine(bobScore);
}
```

- 更新值

```c#
DictionaryName[key] = newValue; 
```

- 删除键值对

```c#
DictionaryName.Remove(key);
```

- 遍历字典

```c#
foreach (var kvp in DictionaryName)
{
    Console.WriteLine(kvp.Key + ": " + kvp.Value);
}
```

## 2.关于扩展方法

- 扩展方法能够向现有类型中“添加”方法，而无需创建新的派生类型、重新编译或以其他方式修改原始类型
- 扩展方法是一种**静态方法**，但可以像扩展类型上的实例方法一样进行调用
- 扩展方法的第一个参数指定该方法作用于哪个类型，并且该参数以**this**为前缀
- 扩展方法只能访问所扩展类的`public`成员
- 必须是静态类才可以添加静态方法

```c#
namespace ExtensionMethods
{
    static class ExtensionMethods
    {
        static void Main(string[] args)
        {
            string preStr = "test";
            Console.WriteLine(preStr.AddSuffix());
        }

        public static string AddSuffix(this string str)
        {
            return str + "_suffix";
        }
    }
}
```

## 3.关于集合类型

- 点阵列`BitArray`：管理一个紧凑型的位置数组，使用布尔值表示。

```c#
using System.Collections;

namespace CollectionsApplication
{
    class Program
    {
        static void Main(string[] args)
        {
            // 创建两个大小为 8 的点阵列
            BitArray ba1 = new BitArray(8);
            BitArray ba2 = new BitArray(8);
            byte[] a = { 60 };
            byte[] b = { 13 };

            // 把值 60 和 13 存储到点阵列中
            ba1 = new BitArray(a);
            ba2 = new BitArray(b);

            BitArray ba3 = new BitArray(8);
            // 点阵列对象的And和Or方法会将结果保存在调用此方法的对象中，例如以下语句会将结果复制到ba1中
            ba3 = ba1.And(ba2);     //ba3 = ba1.Or(ba2);

            // ba3 的内容
            for (int i = 0; i < ba3.Count; i++)
            {
                Console.Write("{0, -6} ", ba3[i]);
            }
            Console.WriteLine();

        }
    }
}
```

## 4.关于枚举类型

```c#
using System;

namespace CSharp_Test
{
    internal class Program
    {
        public enum ver
        {
            None = 0,
            spring = 1,
            summer = 2,
            autumn = 3,
            winter = 4,
            all = spring | summer | autumn | winter
        }
        static void Main(string[] args)
        {
            var kk = ver.all;
            Console.WriteLine(kk);
            var aa = ver.spring;
            Console.WriteLine(aa);
            // 利用如下代码可以获得其对应值
            int aValue = (int)ver.spring;
            Console.WriteLine(aValue);
        }
    }
}
```

## 5.关于委托

- 可以执行方法的类型，调用委托变量时执行的就是变量指向的方法。

- `.NET`中定义了泛型委托`Action`(无返回值)和`Func`(有返回值)，一般不用自定义委托类型。

```c#
 using System;
namespace CSharp_Test
{   
    public class Program
    {
        // 声明 Function 委托类型
        delegate double Function (double x, double y);

        static double doubleOper(double x, double y) => x * y;

        // 参数传入函数的委托，类似于指针，但却是面向对象且类型安全
        static double DegTest(double res, Function f)
        {
            return res + f(res, res);
        }
        static void Main(string[] args)
        {
            double tag = 2.0;
            // 其对应参数是函数的名字，而是调用函数的形式 doubleOper(tag, tag)
            Console.WriteLine(DegTest(tag, doubleOper));
            // 或者可以使用匿名函数或者Lambda表达式创建委托
            Console.WriteLine(DegTest(tag, (double x, double y) => x * y));
        }
    }
}
```

**`Lambda`表达式**

## 6.关于运算符重载

```c#
public static T operator +(T a, T b){}
```

## 7.关于值类型和引用类型

- **值类型**： 值类型变量声明后，不管是否已经赋值，编译器为其分配内存。`byte,short,int,long,float,double,decimal,char,bool,struct`
- **引用类型**：`string`和`class`统称为引用类型。当声明一个类时，只在栈中分配一小片内存用于容纳一个地址，而此时并没有为其分配堆上的内存空间。当使用 $new$创建一个类的实例时，分配堆上的空间，并把堆上空间的地址保存到栈上分配的小片空间中。

## 8.关于特性和反射

1、**特性**：一种用于给代码元素（例如类、方法、属性、字段等）附加元数据的机制。特性提供了一种声明性的方法，用于描述代码元素的行为、特征或者配置信息。**类似于`Java`的注解**。

- 特性类的定义：通过定义特性类创建，特性类需要继承`System.Attribute`基类或其派生类。特性类可以定义特性的属性和方法，用于携带附加信息。

```c#
using System;

[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true)]
public class CustomAttribute : Attribute
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public CustomAttribute(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    // 可以定义其他方法或重写基类方法
}
```

- 特性的应用：可以应用到各种代码元素上，通过在代码元素前方使用**[特性名]**的方式来标记。

```c#
[Custom("John", 25)]
public class MyClass
{
    [Custom("Method1", 30)]
    [Custom("Method2", 40)]
    public void MyMethod()
    {
        // 方法体
    }
}
```

- 特性参数：特性可以接受参数，这些参数可以在特性类的构造函数中定义，并通过在特性应用时进行传递。

- 特性目标：特性可以应用于各种代码元素，如类、方法、属性、字段等。可以使用`AttributeTargets`枚举来指定特性的目标。

- 特性的多重应用：某些特性可以多次应用于同一个代码元素，通过设置特性类上的`AttributeUsage`特性的`AllowMultiple`属性为`true`来实现。

2、**反射**：一种强大的机制，允许程序在运行时获取和操作类型的信息，包括类、接口、方法、属性等。通过反射，可以动态地创建对象、调用方法、获取和设置属性等，而不需要在编译时明确知道类型的具体信息。简单的来说，例如有一个`Student`类，正常实例化可以填充或获得其中的姓名、年龄等信息，但是获取不到这个类的信息，比如类中字段的类型等。反射操作相对较慢，并且不够类型安全，因此在使用反射时需要谨慎，并尽可能避免频繁的反射操作。

- 类型的加载：在使用反射之前，需要加载要操作的类型。可以使用`Type`类的静态方法`GetType()`、`GetType(string typeName)`或`Assembly`类的方法`GetTypes()`等来获取类型的`Type`对象

```c#
Type type = typeof(MyClass);
Type type = Type.GetType("Namespace.MyClass");
Type[] types = assembly.GetTypes();
```

- 获取类型信息：通过`Type`对象，可以获取类型的各种信息，例如名称、命名空间、基类、实现的接口、方法、属性、字段等

```c#
Type type = typeof(MyClass);

string name = type.Name;
string namespace = type.Namespace;
Type baseType = type.BaseType;
Type[] interfaces = type.GetInterfaces();
MethodInfo[] methods = type.GetMethods();
PropertyInfo[] properties = type.GetProperties();
FieldInfo[] fields = type.GetFields();
```

- 创建对象：通过反射，可以动态地创建对象实例。可以使用`Activator`类的`CreateInstance()`方法来创建对象

```c#
Type type = typeof(MyClass);
object obj = Activator.CreateInstance(type);
```

- 调用方法：使用反射，可以在运行时调用方法。可以通过`MethodInfo`对象的`Invoke()`方法来调用方法

```c#
Type type = typeof(MyClass);
object obj = Activator.CreateInstance(type);

MethodInfo method = type.GetMethod("MyMethod");
method.Invoke(obj, null); // 调用方法
```

- 获取和设置属性：反射还可以用于获取和设置属性的值。可以使用`PropertyInfo`对象的`GetValue()`和`SetValue()`方法

```c#
Type type = typeof(MyClass);
object obj = Activator.CreateInstance(type);

PropertyInfo property = type.GetProperty("MyProperty");
object value = property.GetValue(obj);
property.SetValue(obj, newValue);
```

## 9.可空引用类型

```c#
namespace CommonExtensionMethods
{
    class Employee
    {
        public long Id { get; set; }
        public string Name { get; set; }   // 姓名
        public int Age { get; set; }        // 年龄
    }
}
```

在声明此类时，Name字段会报错：

```shell
CS8618：在退出构造函数时，不可为 null 的 字段“Name”必须包含非 null 值。请考虑将 属性 声明为可以为 null
```

​	引入“可空引用类型”是从编译器角度要求开发人员在编程的时候就考虑某个变量是否有可能为空，从而尽可能减少由空引用所带来的代码错误。

​	当使用引用类型的时候会产生此警告（例如`string`），但是如果在一个类中的构造函数中将一个引用类型的字段赋值了，就不会出现此警告，因为已经保证了此字段不会为空，如下Name字段：

```c#
class Employee
{
    public long Id { get; set; }
    public string Name { get; set; }   // 姓名
    public int Age { get; set; }        // 年龄

    public Employee(long id, string name) { Id = id; Name = name; }
}
```

或者是将`Name`声明为`string? Name`，表明允许Employee对象中的Name字段为空。

参考：https://www.cnblogs.com/daxnet/p/14456391.html

## 10.抽象方法VS虚方法

- 抽象函数不能具有功能，任何子类都必须提供自己的该方法的版本。重点是实现多态，同一个方法能做不同的事情
- 虚函数提供了一定的功能，子类如果够用则直接用，否则就 覆盖重写。当一个方法必须存在时。

## 11.拆箱装箱

装箱是将值类型转换为引用类型；拆箱是将引用类型转换为值类型。

[C#装箱和拆箱（Boxing 和 UnBoxing）_c# boxing unboxing-CSDN博客](https://blog.csdn.net/qiaoquan3/article/details/51439726)

## 12.协变和逆变

​	协变和逆变都是术语，前者指能够使用比原始指定的派生类型的派生程度更大（更具体的）的类型，后者指能够使用比原始指定的派生类型的派生程度更小（不太具体的）的类型。

> **协变**:IFoo<父类> = IFoo<子类>
>
> **逆变**:IBar<子类> = IBar<父类>

(1)协变

- 对于泛型类型参数，`out`关键字可指定类型参数是协变的。可以在泛型接口和委托中使用`out`关键字
- 协变就是对具体成员的输出参数进行一次类型转换，且类型转换的准则是[里氏替换原则](https://baike.baidu.com/item/里氏替换原则/3744239?fr=aladdin)

(2)逆变

- 对于泛型类型参数，`in`关键字可指定类型参数是逆变的。可以在泛型接口和委托中使用`in`关键字。
- 逆变就是对具体成员的输入参数进行一次类型转换，且类型转换的准则是[里氏替换原则](https://baike.baidu.com/item/里氏替换原则/3744239?fr=aladdin)

(3)Q&A

- **协变、逆变为什么只能针对泛型接口或者委托？而不能针对泛型类？**

   因为它们都只能定义方法成员（接口不能定义字段），而方法成员在创建对象的时候是不涉及到对象内存分配的，所以它们是类型（内存）安全的。

   为什么不针对泛型？因为泛型类是模板类，而类成员是包含字段的，不同类型的字段是影响对象内存分配的，没有派生关系的类型它们是不兼容的，也是内存不安全的。

- **协变、逆变为什么是类型安全的?**

   本质上是里氏替换原则，由里氏替换原则可知：派生程度小的是派生程度大的子集，所以子类替换父类的位置整个程序功能都不会发生改变。

- **为什么in、out只能是单向的（输入或输出）？**

   因为若类型参数同时为输入参数和输出参数，则必然会有一种转换不符合里氏替换原则，即将父类型的变量赋值给子类型的变量，这是不安全的所以需要明确指定in或out。

```c#
internal class Program
{
    private static void Main(string[] args)
    {
        IBar<object> barObj = new Bar();
        //IBar<string> barStr = (IBar<string>)barObj; /* Right*/
        IBar<string> barStr = barObj;  /* Wrong */

        IFoo<object> fooObj = new Foo();
        IFoo<string> fooStr = fooObj;

        barStr.Print(fooStr);
    }
}

internal interface IFoo<in T>
{ }

internal class Foo : IFoo<object>
{ }

//internal interface IBar<in T>  /* Right */
internal interface IBar<out T>   /* Wrong */
{
    void Print(IFoo<T> foo);
}

internal class Bar : IBar<object>
{
    public void Print(IFoo<object> foo)
    {
        throw new NotImplementedException();
    }
}
```

> ​	**将可变成员作为方法的输入参数，则当前成员的泛型参数可变性必须与输入成员的泛型参数可变性相反**。
>
> ​	**将可变成员作为方法的返回参数，则当前成员的泛型参数可变性必须与输出成员的泛型参数可变性相同**。

参考链接：[C# - 协变、逆变 看完这篇就懂了 - Virgil-Zhou - 博客园 (cnblogs.com)](https://www.cnblogs.com/VVStudy/p/11404300.html)

## Last.Tips

- 如果不显式制定访问修饰符（`public`、`private`），则默认为`private`
- 对于具有多个元素的数据类型对象，例如数组、字典等，直接输出会输出对应的类型
- `string`中的每个字符是`char`类型
- `Console.Read()`读取为`int`类型，`Console.ReadLine()`读取为`string`类型

