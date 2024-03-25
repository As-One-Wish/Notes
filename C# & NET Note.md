# 一、.NET 概念

- **.NET Framework**：用于构建Windows桌面应用程序、ASP.NET Web应用程序和其他类型的应用程序，仅可在Windows上运行，并且不支持跨平台开发

- **.NET Core**：被设计为模块化和轻量级，允许开发人员构建可在多个平台上运行的应用程序，包括Windows、macOS和Linux，支持各种应用程序类型，包括Web应用程序、命令行工具、微服务等。与.NET Framework相比，.NET Core具有更高的性能和更好的可伸缩性


- **.NET Standard**：是一个规范，定义了一组公共API，允许开发人员编写可在不同的.NET实现中共享的代码，声明标准，但是具体的实现交由不同的平台各自实现

# 二、公共语言运行时

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

## 3、AppDomain-AppContext-Application-Assembly

- `AppDomain`
  - 是.NET运行时中的一个隔离容器，用于加载、执行和卸载托管代码
  - 提供了一种在同一个进程中隔离和管理不同部分的机制，增强了应用程序的安全性、稳定性和可维护性
  - 是执行代码的环境，每个.NET程序至少有一个`AppDomain`
- `AppContext`
  - 是.NET Framework中的一个类，用于提供应用程序级别的配置和上下文信息
  - 提供了一种获取和设置应用程序级别配置信息的方式，例如线程池大小、语言环境等
  - 提供了一种管理应用程序和上下文信息的机制，与应用程序的配置和环境相关
- `Application`
  - 是.NET框架中的一个类，用于管理正在运行的应用程序的实例
  - 提供了一种访问应用程序级别信息和功能的方式，例如管理应用程序的生命周期、处理异常等
  - 通常与GUi应用程序(如Windows Forms、WPF等)相关联，提供了一些用于控制应用程序行为的方法和事件
- `Assembly`
  - 一种部署单元，包含了执行代码、资源和元数据；通常以DLL或EXE文件形式存在
  - 代码的物理单元，可以被加载到应用程序域中并执行；一个应用程序域可以加载和执行多个程序集


> ​	`AppDomain`和`AppContext`都是用于管理应用程序的执行环境和上下文，但它们的作用略有不同。`AppDomain`主要用于隔离和执行托管代码，`AppContext`则用于管理应用程序级别的配置信息
>
> ​	`Application`类提供了一些用于管理正在运行的应用程序实例的方法和属性，通常位于GUI应用程序中，但与`AppDomain`和`AppContext`相比，它更多的关注应用程序本身的生命周期管理

# 三、配置

​	.NET中的配置是使用一个或多个[配置提供程序](#configProv)执行的。配置提供程序使用各种配置源从键值对读取配置数据。

​	给定一个或多个配置源，`IConfiguration`类型提供配置数据的统一视图

![](C:\Files\Notes\Images\C#&NET-1.png)

## 1.<span id="configProv">配置提供程序</span>

​	是一种机制，用于从不同的配置源中读取应用程序的配置信息并将其提供给应用程序。`ConfigurationBuilder`类用于构建配置对象，并且可以添加不同类型的配置提供程序来读取配置：

- `AddJsonFile`--Json文件

```c#
IConfigurationBuilder cfgBuilder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory()) // 设置配置文件基本路径
  	// optional: 配置文件是否是可选(不存在是否引发异常)
  	// reloadOnChange: 配置文件修改时是否重新加载配置
    .AddJsonFile("appsettings.json", optional: true, reloadOnChange:true);

IConfiguration configuration = cfgBuilder.Build();
```

- `AddXmlFile`--Xml文件

```c#
IConfigurationBuilder cfgBuilder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddXmlFile("appsettings.xml", optional: true, reloadOnChange:true);

IConfiguration configuration = cfgBuilder.Build();
```

- `AddEnvironmentVariales`--环境变量

```c#
IConfigurationBuilder cfgBuilder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddEnvironmentVariables();

IConfiguration configuration = cfgBuilder.Build();
```

- `AddCommandLine`--命令行参数

```c#
IConfigurationBuilder cfgBuilder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddCommandLine(args);

IConfiguration configuration = cfgBuilder.Build();
```

## 2.选项模式

​	选项模式使用类来提供对相关设置组的[强类型访问](#stronglyTypedAccess)。当配置设置[由方案隔离到单独的类](#schemeToClass)时，应用遵循两个重要软件工程原则：

- **接口分隔原则(ISP)或封装**:依赖于配置设置的方案(类)仅依赖于其使用的配置设置
- **关注点分离**:应用的不同部件的设置不彼此依赖或相互耦合

​	选项模式可以通过`IOptions<TOptions>`接口实现，其中泛型类型参数`TOptions`被约束为`class`；

​	在使用选项模式时，选项类：

- 必须是非抽象类，有一个公共无参数构造函数
- 包含要绑定的公共读写属性(不绑定字段)

```c#
public record HttpApiSettings
{
    /// <summary>
    /// BaseUrls字典
    /// </summary>
    public Dictionary<string, string> BaseUrls { get; set; }
    /// <summary>
    /// 过期时间
    /// </summary>
    public int GlobalOverTime { get; set; } = int.MinValue;
}
```

> - <span id="stronglyTypedAccess">`强类型访问`</span>(Strongly Typed Access):指通过在编译时制定类型来访问数据或对象的方式
>
>   其优点是编译器可以在编译时对类型进行检查，确保类型的正确性和一致性，从而提高了代码的安全性和可靠性
>
> - <span id="schemeToClass">`由方案隔离到单独的类`</span>:将应用程序的配置设置从原始的配置文件中提取，并将其映射到一个单独的.NET类中

- **`相关依赖`**

  - Microsoft.Extensions.Configuration

  - Microsoft.Extensions.Configuration.Json

  ​	解决`.SetBasePath()`的报错

  - Microsoft.Extensions.DependencyInjection

  - Microsoft.Extensions.Options

  - Microsoft.Extensions.Options.ConfigurationExtensions

  ​	解决`services.Configure<TOptions>()`的转化问题

  - System.Configuration.ConfigurationManager

## 3.选项模式样例

- **配置文件**--`appsettings.json`

```json
{
  "PersonalInfo": {
    "Name": "山鬼不识字",
    "Account": "AsOneWish",
    "Gender": 1,
    "Age": 23
  }
}
```

>**关于appsettings.json文件在生成解决方案时不会自动生成到输出目录问题：**
>
>```xml
><ItemGroup>
>  	<None Update="appsettings.json">
>    		<CopyToOutputDirectory>Always</CopyToOutputDirectory>
>  	</None>
></ItemGroup>
>```

- **程序入口文件**--`Program.cs`

​	读取配置文件，获取其中`PersonalInfo`节点，映射到配置类，注册输出类

```c#
public static void ConfigureService(ServiceCollection services)
{
    IConfigurationBuilder cfgBuilder = new ConfigurationBuilder()
        .SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("appsettings.json");

    IConfiguration configuration = cfgBuilder.Build();

    services.Configure<PersonalInfo>(configuration.GetSection("PersonalInfo"));
    services.AddScoped<Test1>();
}
```

​	服务获取并调用，输出配置信息

```c#
private static void Main(string[] args)
{
    ServiceCollection services = new ServiceCollection();
    ConfigureService(services);

    ServiceProvider builder = services.BuildServiceProvider();
    Test1 testClass = builder.GetRequiredService<Test1>();

    testClass.Display();
}
```

- **输出类**

​	选项模式的使用，并对配置信息进行输出

```c#
public class Test1
{
    private readonly PersonalInfo _personalInfo;

    public Test1(IOptions<PersonalInfo> options)
    {
        _personalInfo = options.Value;
    }

    public void Display()
    {
        Console.WriteLine($"Account: {_personalInfo.Account}\nName: {_personalInfo.Name}\nAge: {_personalInfo.Age}\nGender: {_personalInfo.Gender}");
    }
}
```

# 四、异步编程

## 1.**异步方法**：用`async`关键字修饰的方法

> - 异步方法的返回值一般是`Task<T>`，`T`是真正的返回值类型，例如`Task<int>`。惯例，异步方法名字以`Async`结尾
> - 即使方法没有返回值，也最好把返回值声明为非泛型的`Task`
> - 调用泛型方法时，一般在方法前面加上`await`，这样拿到的返回值就是泛型指定的`T`类型
> - 异步方法的传染性：一个方法中如果有`wait`调用，则这个方法也必须修饰为`async`
> - 涉及到延时时，尽量不要用`Thread.Sleep()`，要使用`await Task.Delay()`

```c#
using System.Text;

namespace Asynchronous1
{
    internal class Asynchronous1
    {
        static async Task Main(string[] args)
        {
            //这是文件读写的同步写法
            /*string file_name = @"D:\Files\Test\Asynchronous1.txt";
            File.WriteAllText(file_name, "这是同步的写法！");

            string file_content = File.ReadAllText(file_name);
            Console.WriteLine(file_content);*/

            string file_name = @"D:\Files\Test\Asynchronous1.txt";  
            StringBuilder sb = new StringBuilder();
            for(int i = 0; i < 100000; ++i)
                sb.AppendLine("这个测试不加await会报错");

            // 涉及到异步async时，对于使用async类型的方法时注意一定要加await(会有些情况不用加)
            await File.WriteAllTextAsync(file_name, sb.ToString());

            // ReadAllTextAsync的返回值是Task类型，但是加上await之后会自动从Task中取出
            string file_content = await File.ReadAllTextAsync(file_name); 
            Console.WriteLine(file_content);
        }
    }
}
```

## 2.**`async`、`await`原理**

​	带有`async`的方法会被C#编译器编译成一个类，其中主要根据`await`调用来进行状态的切分（参考状态机），对`async`方法的调用会被拆分为对`MoveNext`的调用。其工作原理可以概括一下几个步骤：

> 1. 当调用一个异步方法时，该方法会立即启动执行，并返回一个代表异步操作的任务（`Task`）
> 2. 遇到第一个 `await` 关键字时，异步方法会将控制权交还给调用方，表示暂时挂起。同时，异步方法会注册一个回调，用于在异步操作完成后恢复执行
> 3. 异步操作继续在后台执行，直到完成。完成后，异步方法会使用异步操作的结果继续执行
> 4. 当异步操作完成后，回调会通知调度器，并在适当的时机恢复异步方法的执行
> 5. 异步方法继续执行，从 `await` 关键字后的位置开始



```C#
namespace AsyncPrinciple
{
    internal class AsyncPrinciple
    {
        static async Task Main(string[] args)
        {
            string html_path = "https://taobao.com";
            using (HttpClient client = new HttpClient())
            {
                string html_content = await client.GetStringAsync(html_path);
                Console.WriteLine(html_content);
            }

            string file_content = "AsyncPrinciple";
            string file_path = @"D:\Files\Test\AsyncPrinciple.txt";
            await File.WriteAllTextAsync(file_path, file_content);
            Console.WriteLine(file_content + "--写入成功！");
            string str = await File.ReadAllTextAsync(file_path);
            Console.WriteLine("文件内容:" + str);
        }
    }
}
```

## 3.**异步中的线程切换**

​	`await`调用的等待期间，`.NET`会把当前的线程返回给线程池，等异步方法调用执行完毕后，框架会从线程池再取出来一个线程执行后续的代码；到要等待的时候，如果发现已经执行结束了，那就没必要再切换线程了，剩下的代码就继续在之前的线程上继续执行了。

```c#
using System.Text;

namespace ThreadSwitch
{
    internal class ThreadSwitch
    {
        static async Task Main(string[] args)
        {
            // 查看当前线程的线程ID
            Console.WriteLine(Thread.CurrentThread.ManagedThreadId);
            StringBuilder sb = new StringBuilder();
            // 如果次数较少，也就是说这个操作所消耗的时间较短，可能不会切换进程
            for(int i = 0; i < 10000; ++i)
            {
                sb.Append("线程切换测试！");
            }
            await File.WriteAllTextAsync(@"D:\Files\Test\ThreadSwitch.txt", sb.ToString());
            // 查看当前线程的线程ID
            Console.WriteLine(Thread.CurrentThread.ManagedThreadId);
        }
    }
}
```

​	但是异步方法并不代表多线程。异步方法的代码并不会自动在新线程中执行，除非把代码放到新线程中执行。

```C#
namespace NoneThread
{
    internal class NoneThread
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine("Pre_Thread_ID:"+Thread.CurrentThread.ManagedThreadId);
            double res = await CalcAsync(5000);
            Console.WriteLine($"res:{res}");
            Console.WriteLine("Las_Thread_ID:" + Thread.CurrentThread.ManagedThreadId);
        }
        public static async Task<double> CalcAsync(int n)
        {
            // 此时并不会切换线程，当作同步的方法来执行了
            /*Console.WriteLine("CalcAsync_Thread_ID:" + Thread.CurrentThread.ManagedThreadId);
            double result = 0;
            Random rand = new Random();
            for(int i = 0; i < n * n; ++i)
            {
                result += rand.NextDouble();
            }
            return result;*/

            // 此时相当于自己新开一个线程
            return await Task.Run(() =>
            {
                Console.WriteLine("CalcAsync_Thread_ID:" + Thread.CurrentThread.ManagedThreadId);
                double result = 0;
                Random rand = new Random();
                for (int i = 0; i < n * n; ++i)
                {
                    result += rand.NextDouble();
                }
                return result;
            });
        }
    }
}
```

## 4.**缺少`async`修饰的方法**

> - `async`方法有一定的缺点：异步方法会生成一个类，运行效率没有普通方法高；可能会占用非常多的线程
> - 返回`Task`的不一定都要标注`async`，标注`async`只是可以更方便的`await`
> - 如果一个异步方法只是对别的异步方法调用的转发，并没有太多复杂的逻辑（比如等待`A`的结果，再调用`B`；把`A`调用的返回值拿到内部做一些处理再返回），那么就可以去掉`async`关键字。

```C#
namespace NoneAsync
{
    internal class NoneAsync
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine(await ReadAsync(1));
        }
        // 带async的方法
        /*static async Task<string> ReadAsync(int num)
        {
            if (num == 1)
                return await File.ReadAllTextAsync(@"D:\Files\Test\Asynchronous1.txt");
            else if (num == 2)
                return await File.ReadAllTextAsync(@"D:\Files\Test\Asynchronous2.txt");
            else
                throw new ArgumentException();
        }*/
        // 不带async的方法
        static  Task<string> ReadAsync(int num)
        {
            if (num == 1)
                return  File.ReadAllTextAsync(@"D:\Files\Test\Asynchronous1.txt");
            else if (num == 2)
                return  File.ReadAllTextAsync(@"D:\Files\Test\Asynchronous2.txt");
            else
                throw new ArgumentException();
        }
    }
}
```

## 5.多线程

- **进程：**一般指程序中运行程序，实际作用是为程序在执行过程中创建好所需的环境和资源
- **线程：**是进程的一个实体，是CPU用来调度的最小单元，一个进程可以拥有多个线程
- **单线程：**进程中只有一个线程，只执行一个线程
- **多线程：**同一时刻，可以执行多个线程
- **并发：**一段时间内，同时做多件事情

> 并发是`同一时刻，只执行一个线程`，但是多个线程被快速的交替执行，在宏观上有多线程同时执行的假效果，但是在微观上只是把时间分成若干顿，使多个线程快速的交替执行

- **并行：**同一时刻，做多件事情

> 同一时刻，有多个线程在多个处理器上同时执行，无论从宏观还是微观来看，这些线程都是一起执行的

>单处理器下，多线程一定是并发执行的；
>
>多处理器下，当线程数小于等于处理器个数时，多线程是并行的；反之可能是并行或并发

- **同步：**等待前一个任务结束后，再执行下一个任务，无并发或者并行概念
- **异步：**多个任务，同时执行
- **多线程同步：**当有一个线程对内存某一块地址操作时，不允许其他线程对这个内存地址进行操作，直到该线程操作完成
- **并发三大特性**
  - **可见性：**当多个线程访问同一个变量时，一个线程修改了这个变量值，其他线程能够立刻看到修改后的值
  - **原子性：**即一个操作或者多个操作，要么全部执行(执行过程中不被任何因素打断)，要么都不执行
  - **有序性：**即程序的执行按照代码的先后顺序执行
- **死锁**
  - **互斥性：**当一个资源被线程使用的时候，别的线程不能使用
  - **不可抢占性：**资源请求者不可强制从资源拥有者中抢夺资源
  - **占有且等待性：**资源请求者在等待其他资源时，保持对原有资源的占有
  - **循环等待性：**线程1等待线程2占有的资源，线程2等待线程1占有的资源

# 五、LINQ

 基于[委托](D:\Files\Note\C# Note.md)和[Lambda](D:\Files\Note\C# Note.md)

```c#
// Employee 类
namespace CommonExtensionMethods
{
    class Employee
    {
        public long Id { get; set; }
        // 默认启用了可空引用类型，例如声明为string后此变量就不能为null，只有显式声明string？才可以为null
        public string? Name { get; set; }   // 姓名
        public int Age { get; set; }        // 年龄
        public bool Gender { get; set; }    // 性别
        public int Salary { get; set; }     // 薪水
        public override string ToString()
        {
            return $"ID={Id},Name={Name},Age={Age},Gender={Gender},Salary={Salary}";
        }
    }
}
```

常用扩展方法

```C#
static List<Employee> employees = new List<Employee>();
static void Main(string[] args)
{
    init();
    // Where-->根据条件判定每个元素，符合要求则会返回至结果
    IEnumerable<Employee> items_one = employees.Where(a => a.Age > 30);
    foreach (Employee item in items_one) { Console.WriteLine(item); }
    Console.WriteLine();
    // Count-->获取满足条件的记录条数
    Console.WriteLine(employees.Count(b => b.Gender && b.Age < 30));
    Console.WriteLine();
    // Any-->是否至少有一条数据
    Console.WriteLine(employees.Any(c => c.Gender == false && c.Age > 40));
    Console.WriteLine();
    /*Single-->获取有且只有一条满足要求的数据
    SingleOrDefault-->获取最多只有一条满足要求的数据
    First-->至少有一条，返回第一条
    FirstOrDefault-->返回第一条或者默认值*/
    Employee items_two = employees.Single(d => d.Name == "张三");
    Console.WriteLine(items_two);
    Console.WriteLine();
    // Order[By]-->正序排序  OrderByDescending-->倒序排序  排序结果是新的数据，对原数据无修改
    IEnumerable<Employee> items_three = employees.OrderBy(e => e.Age);
    foreach (Employee item in items_three) { Console.WriteLine(item); }
    Console.WriteLine();
    // ThenBy、ThenByDescending-->多规则排序
    IEnumerable<Employee> items_four = employees.OrderBy(e => e.Age).ThenBy(e => e.Salary);
    foreach (Employee item in items_four) { Console.WriteLine(item); }
    Console.WriteLine();
    // Skip-->跳过n条数据   Take-->获取n条数据
    IEnumerable<Employee> items_five = employees.Skip(3).Take(3);
    foreach (Employee item in items_five) { Console.WriteLine(item); }
    Console.WriteLine();
    /***************** 聚合函数 *****************/
    // Max Min Average Sum Count
    // GroupBy-->分组  返回值IGrouping类型
    IEnumerable<IGrouping<int, Employee>> items_six = employees.GroupBy(e => e.Age);
    foreach (IGrouping<int, Employee> g in items_six)
    {
        Console.WriteLine(g.Key);
        foreach (Employee item in g) { Console.WriteLine(item); }
        Console.WriteLine("-------------------------------------");

    }
    Console.WriteLine();
    /***************** 匿名类型 *****************/
    // 匿名类型最终编译后是一个确定的类型，类型的名字由编译器定义
    var obj = new { Name = "匿名类型", Description = "没有类型名的" };
    Console.WriteLine(obj.Name + "," + obj.Description);
    /***************** 投影操作 *****************/
    // Select
    IEnumerable<string> items_seven = employees.Select(e => e.Name + "," + e.Age + "," + (e.Gender ? "男" : "女"));
    foreach (string s in items_seven) { Console.WriteLine(s); }
    Console.WriteLine();
    // 投影+匿名类型
    var items_eight = employees.Select(e => new { XingMing = e.Name, NianLing = e.Age });
    foreach (var s in items_eight) { Console.WriteLine($"姓名:{s.XingMing},年龄:{s.NianLing}"); }
    Console.WriteLine();
    /***************** 集合转换 *****************/
    // ToArray()  ToList()
    IEnumerable<Employee> items_nine = employees.Where(e => e.Salary > 6000);
    List<Employee> employees_list = items_nine.ToList();
    Employee[] employees_array = items_nine.ToArray();
    /***************** 查询语法 *****************/
    // 方法语法
    var items_ten = employees.Where(e => e.Salary > 6000).OrderBy(e => e.Age)
        .Select(e => new { e.Age, e.Name, XB = e.Gender ? "男" : "女" });
    foreach (var item in items_ten) { Console.WriteLine(item); }
    Console.WriteLine();
    // 查询语法
    var items_eleven = from e in employees where e.Salary > 5500 select new { e.Age, e.Name, XB = e.Gender ? "男" : "女" };
    foreach (var item in items_eleven) {  Console.WriteLine(item); }

}
static void init()
{
    employees.Add(new Employee { Id = 1, Name = "张三", Age = 18, Gender = true, Salary = 5000 });
    employees.Add(new Employee { Id = 2, Name = "李四", Age = 45, Gender = true, Salary = 4500 });
    employees.Add(new Employee { Id = 3, Name = "王五", Age = 23, Gender = false, Salary = 8000 });
    employees.Add(new Employee { Id = 4, Name = "赵六", Age = 46, Gender = true, Salary = 7000 });
    employees.Add(new Employee { Id = 5, Name = "孙七", Age = 35, Gender = false, Salary = 6400 });
    employees.Add(new Employee { Id = 6, Name = "钱八", Age = 26, Gender = true, Salary = 5600 });
    employees.Add(new Employee { Id = 7, Name = "周九", Age = 26, Gender = false, Salary = 7700 });
    employees.Add(new Employee { Id = 8, Name = "吴十", Age = 26, Gender = false, Salary = 5500 });
}
```

# 六、依赖注入

> 依赖注入（`DI`）是控制反转（`IOC`）思想的实现方式
>
> 依赖注入简化模块的组装过程，降低模块之间的耦合度

## 1.依赖关系反转

**应用程序中的依赖关系方向应该是抽象的方向，而不是实现详细信息的方向。**

- 直接依赖项关系

​	编译时依赖关系顺着运行时执行的方向流动：若类A调用类B的方法，类B调用类C的方法，则在编译时，类A将取决于类B，类B又取决于类C

![](C:\Files\Notes\Images\C#&NET-2.png)

- 反转依赖项关系 

​	A可以调用B实现的抽象上的方法，让A可以在运行时调用B，而B又在编译时依赖于A控制的接口；运行时，程序执行的流程不变，但接口引入意味着可以轻松插入这些接口的不同实现；**高层模块不再依赖于底层模块，而是依赖于抽象接口**

![](C:\Files\Notes\Images\C#&NET-3.png)

## 2.控制反转

- 一种软件设计原则，是**依赖关系倒置的一种实现方式**；反转了组件对外部资源的控制，将控制权从应用程序代码中转移到了外部容器或框架中
- 在控制反转中，组件的创建、管理和生命周期由容器和框架来控制，而不是由组件自己来控制，这使得组件之间的依赖关系更加松散
- 控制反转的一个常见实现是依赖注入，它通过将依赖对象通过构造函数、属性或方法参数传递给组件来实现依赖关系的注入

> 控制反转是依赖关系倒置的一种实现方式，它通过将控制权转移到外部容器或框架中来实现依赖关系的倒置，而依赖关系倒置更关注于高层模块对抽象的依赖，而不是具体的实现。

## 3.WebApplication & IHostService

- `WebApplication`
  - 通常用于构建和配置ASP.NET Core Web应用程序，允许定义中间件、路由和请求管道，以处理传入的HTTP请求
  - Web API、Web应用程序、微服务、单页应用程序(SPA)后端
- `IHostService`
  - 用于定义应用程序中托管的服务；实现`IHostService`接口的服务可以在应用程序启动时启动，并在应用程序关闭时停止
  - 通常用于在应用程序启动或关闭时执行一些初始化或清理操作，例如启动后台任务、初始化数据库连接等

## 4.辅助角色服务

- **后台服务**:引用BackgroundService类型

- **托管服务**:实现IHostedService或引用IHostedService本身
- **长时间运行服务**:持续运行的任何服务
- **Windows服务**:Windows服务基础结构
- **辅助角色服务**:引用[辅助角色服务模版](#SupportingRoleServiceTemplate)

> <span id="SupportingRoleServiceTemplate">**辅助角色服务模版**</span>:模板由`Program`和`Worker`类构成
>
> ```c#
> using App.WorkerService;
> 
> HostApplicationBuilder builder = Host.CreateApplicationBuilder(args);
> builder.Services.AddHostedService<Worker>();
> 
> IHost host = builder.Build();
> host.Run();
> ```
>
> 前面的`Program`类：
>
> - 创建一个`HostApplicationBuilder`
> - 调用`AddHostedService`以将`Worker`注册为托管服务
> - 从生成器生成`IHost`
> - 在运行应用的`host`实例上调用`Run`

## 5.生命周期

> 不要在长生命周期的对象中引用比它短的生命周期的对象，例如在Singleton对象中引用Transient对象

**生命周期的选择**

> - 如果类无状态：建议为Singleton
> - 如果类有状态，且有Scope控制：建议为Scoped
> - 使用Transient的时候要谨慎

## 6.样例

**作为注册的测试类**

```c#
namespace DITest
{
    public interface ITestService
    {
        public void Display();
    }

    public class TestService1 : ITestService
    {
        public void Display()
        {
            Console.WriteLine("This is TestService1！！");
        }
    }

    public class TestService2 : ITestService
    {
        public void Display()
        {
            Console.WriteLine("This is TestService2！！");
        }
    }
}
```

**测试依赖注入中构造函数自解析赋值**

```c#
namespace DITest
{
    public class Worker
    {
        private readonly ITestService _testService;

        public Worker(ITestService testService)
        {
            this._testService = testService;
        }

        public void Run()
        {
            _testService.Display();
        }
    }
}
```

**函数入口-服务注册及获取**

```c#
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace DITest
{
    internal class Program
    {
        public static void Main(string[] args)
        {
            HostApplicationBuilder app = Host.CreateApplicationBuilder(args);

            app.Services.AddSingleton<ITestService, TestService1>();
            app.Services.AddSingleton<ITestService, TestService2>();
            app.Services.AddSingleton<Worker>();

            IHost host = app.Build();

            IServiceProvider sp = host.Services;
            Worker worker = sp.GetRequiredService<Worker>();
            worker.Run();

            host.Run();
        }
    }
}
```

对于同一个接口的不同实现，后续注册的会覆盖前一个

## 7.总结

**依赖注入具有“传染性”**

> - 如果类的对象是通过`DI创建`的：这个类的构造函数中声明的所有服务类型的参数都会被DI赋值
> - 如果一个对象是`手动创建`的：她的构造函数中声明的服务型参数就不会被自动赋值

.NET的DI默认是构造函数注入

​	综上，依赖注入实际上可以认为，我使用一个类时，不管这个类是什么样子，我就告诉框架我要这个类，我不需要自己new就可以使用，这样的好处是：1、如果声明对象的话，可能其构造函数的参数很难传入（比如是其他的类）；2、如果服务有修改，比如某接口的实现方法改了，我只需要在注册时修改注册函数，而不需要去修改业务代码

# 七、GC

垃圾回收机制，为了高效、便捷和安全地管理内存，C#中采用了自动垃圾回收机制，由系统代理内存管理，从而提高开发效率和避免内存泄漏等问题。

- **标记-清理：**从根对象开始，根据对象的引用关系递归标记可达对象，在清理阶段则对不可达对象进行内存清理，某些GC还会在清理阶段进行内存压缩从而减少内存碎片化。
- **分代管理：**在托管堆上根据所创建对象的生命周期进行分类，刚创建的对象被称为“第一代”，在上一次“标记-清理”阶段存活下来的对象会被分类为“第二代，以此类推，分代管理的目的是为了`控制对象进行“标记-清理”的频率从而提高GC的效率`。

# 八、委托&事件

## 1.委托

是对指定签名的函数的引用，通过委托可以实现将函数作为参数传递或者间接调用函数，委托是**类型安全**的，仅指向与其声明时指定签名相匹配的函数。

委托可以分为单播委托和多播委托，二者的区别在于是对单个方法还是一组方法的引用

- **多播委托**：实际上是一个类实例，其中定义了一个函数引用列表用于存储订阅的函数。当调用多播委托时，将由CLR来遍历该函数引用列表，并按照订阅顺序依次调用函数。

## 2.事件

一种特殊的多播委托，其相比于普通的多播委托更加安全，事件将多播委托的调用权限隔离在其所在类的内部，并对外部关闭了直接通过赋值符号"="修改多播委托实例的入口，使得外部调用者仅能够进行基本的函数订阅和取消订阅的操作。

# 九、重载&重写

## 1.重载

`编译时多态`，重载函数的名称相同但参数列表不同，在调用时编译器会自动根据传递的参数列表适配指定形式的重载函数。

## 2.重写

`运行时多态`，子类重写父类的方法，在调用时根据实例对象的类型而适配重写函数。

- 派生类可以重写的基类方法
  - 基类中使用`virtual`关键字进行限定的方法(简称"虚方法")
  - 在派生类中使用new关键字对与基类同名的方法进行重写(简称"隐藏方法")
  - 基类是抽象类，抽象类中使用`abstract`关键字进行限定的方法(简称"抽象方法")
- `virtual`关键字

​	用于定义虚方法，基类中的virtual方法可以被直接或间接派生类`选择性`重写

- `new`关键字

​	用于隐藏方法(派生类对基类隐藏同名方法)，直观的区别就在于声明为基类而实例化为派生类的对象将无法调用派生类的隐藏方法，但是声明与实例化均为派生类的对象则可以调用派生类的隐藏方法

- `abstract`关键字

​	用于声明抽象类和抽象方法，且抽象方法仅存在于抽象类中，直接派生类被要求必须完成积累中抽象方法的定义，除非直接派生类也为抽象类

- `seald`关键字

​	用于声明密封类和密封方法，密封类无法被继承，密封方法无法被重写

# 十、语言相关

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

## Tips

- 如果不显式制定访问修饰符（`public`、`private`），则默认为`private`
- 对于具有多个元素的数据类型对象，例如数组、字典等，直接输出会输出对应的类型
- `string`中的每个字符是`char`类型
- `Console.Read()`读取为`int`类型，`Console.ReadLine()`读取为`string`类型

