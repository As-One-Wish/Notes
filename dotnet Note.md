# .NET 概念

- **.NET Framework**：用于构建Windows桌面应用程序、ASP.NET Web应用程序和其他类型的应用程序，仅可在Windows上运行，并且不支持跨平台开发

- **.NET Core**：被设计为模块化和轻量级，允许开发人员构建可在多个平台上运行的应用程序，包括Windows、macOS和Linux，支持各种应用程序类型，包括Web应用程序、命令行工具、微服务等。与.NET Framework相比，.NET Core具有更高的性能和更好的可伸缩性


- **.NET Standard**：是一个规范，定义了一组公共API，允许开发人员编写可在不同的.NET实现中共享的代码，声明标准，但是具体的实现交由不同的平台各自实现

# 异步编程

## **异步方法**：用`async`关键字修饰的方法

> - 异步方法的返回值一般是`Task<T>`，`T`是真正的返回值类型，例如`Task<int>`。惯例，异步方法名字以`Async` 结尾
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

## **`async`、`await`原理**

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

## **异步中的线程切换**

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

## **缺少`async`修饰的方法**

> - `async`方法有一定的缺点：异步方法会生成一个类，运行效率没有普通犯法高；可能会占用非常多的线程
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

## **`CancellationToken`**

​	传播有关应取消操作的通知

```c#
namespace CancellationTokenTest
{
    internal class CancellationTokenTest
    {
        /*static async Task Main(string[] args)
        {
            CancellationTokenSource cts = new CancellationTokenSource();
            cts.CancelAfter(5000);
            // await DownloadHtmlAsync("https://youzack.com", 100);
            CancellationToken cToken = cts.Token;
            //await DownloadHtml1Async("https://youzack.com", 100, cToken);
            await DownloadHtml2Async("https://youzack.com", 100, cToken);
        }*/
        static void Main(string[] args)
        {
            CancellationTokenSource cts = new CancellationTokenSource();
            CancellationToken cToken = cts.Token;
            DownloadHtml2Async("https://youzack.com", 100, cToken);
            // 根据用户输入进行中止
            while (Console.ReadLine() != "q") { }
            cts.Cancel();
            Console.ReadLine();
        }
        // 正常下载
        static async Task DownloadHtmlAsync(string url, int n)
        {
            using (HttpClient client = new HttpClient())
            {
                for (int i = 0; i < n; ++i)
                {
                    string html_content = await client.GetStringAsync(url);
                    Console.WriteLine($"{DateTime.Now}:{html_content}");
                }
            }
        }
        // 添加定时取消下载
        static async Task DownloadHtml1Async(string url, int n, CancellationToken cancellationToken)
        {
            using (HttpClient client = new HttpClient())
            {
                for (int i = 0; i < n; ++i)
                {
                    string html_content = await client.GetStringAsync(url);
                    Console.WriteLine($"{DateTime.Now}:{html_content}");
                    /*if (cancellationToken.IsCancellationRequested)
                    {
                        Console.WriteLine("请求被取消！");
                        break;
                    }*/
                    // 抛出异常，需要手动处理异常，取消不及时
                    cancellationToken.ThrowIfCancellationRequested();
                }
            }
        }
		// 调用自含CancellationToken参数的函数
        static async Task DownloadHtml2Async(string url, int n, CancellationToken cancellationToken)
        {
            using (HttpClient client = new HttpClient())
            {
                for (int i = 0; i < n; ++i)
                {
                    // 交由GetAsync进行，取消及时
                    var resp = await client.GetAsync(url, cancellationToken);
                    string html_content = await resp.Content.ReadAsStringAsync();
                    Console.WriteLine($"{DateTime.Now}:{html_content}");
                }
            }
        }
    }
}
```

## **`WhenAll`** 和 **`WhenAny`**

​	顾名思义，`WhenAll`是提供的任务全部完成后视为该任务完成，所有任务的执行不会按照特定的顺序执行，而是会并发的执行，即所有的任务同时开始，最后返回的任务数组是有一定顺序的；`WheAny`是提供的任务中任何一个完成都视为该任务完成，其中任务一样是并发执行，最后返回一个完成的任务表示此任务的完成。

```C#
namespace WhenAllTest
{
    internal class WhenAllTest
    {
        static async Task Main(string[] args)
        {
            // 获取当前路径下的所有文件
            string[] files = Directory.GetFiles(@"D:\Files\Test");
            // 初始化一个Task数组，其中每一个Task的返回值是int
            Task<int>[] countTasks = new Task<int>[files.Length];
            for(int i=0;i<files.Length; ++i)
            {
                string pre_file_name = files[i];
                // 依次对每个文件创建Task，存入countTasks数组
                Task<int> preTask = GetCharsCount(pre_file_name);
                countTasks[i] = preTask;
            }
            // 利用WhenAll，待全部完成后视为任务完成
            int[] counts = await Task.WhenAll(countTasks);
            int sum = counts.Sum();
            Console.WriteLine(sum);
        }

        static async Task<int> GetCharsCount(string file_name)
        {
            string chars = await File.ReadAllTextAsync(file_name);
            Console.WriteLine($"{file_name}的字符个数为:{chars.Length}");
            return chars.Length;
        }
    }
}
```

## Tips

> - `async`是提示编译器为异步方法中的`await`代码进行分段处理的，而一个异步方法是否修饰了`async`对于方法的调用者来讲没区别，因此对于接口中的方法或者抽象方法不能修饰为`async`
> - 异步与`yield`：`yield return`不仅能够简化数据的返回，而且可以让数据处理流水线化，提升性能

```c#
namespace YieldTest
{
    internal class Program
    {
        static async Task Main(string[] args)
        {
            foreach (var s in Test1()) {
                Console.WriteLine(s);
            }
            await foreach (var s in Test2()) {
                Console.WriteLine(s);
            }
        }

        static IEnumerable<string> Test1()
        {
            List<string> list = new List<string>();
            list.Add("hello");
            list.Add("hwj");
            list.Add("good morning");
            return list;
        }

        // 旧版C#中,async方法中不能用yield:从C# 8.0开始,把返回值声明为IAsyncEnumerable(不要带Task),
        // 然后遍历的时候用await foreach()即可
        static async IAsyncEnumerable<string> Test2()
        {
            yield return "hello";
            yield return "hwj";
            yield return "good morning";
        }
    }
}
```



# LINQ

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

# 依赖注入

> 依赖注入（`DI`）是控制反转（`IOC`）思想的实现方式
>
> 依赖注入简化模块的组装过程，降低模块之间的耦合度

## 概念

- **服务**：对象
- **注册服务**：对象要预先注册，后续才能拿到
- **服务容器**：负责管理注册的服务（服务的管理者）
- **查询服务**：创建对象及关联对象（向服务容器要服务的过程）
- **对象生命周期**：Transient(瞬态)、Scoped（范围）、Singleton（单例）

> 根据类型来获取和注册服务。可以分别指定`服务类型`（service type）和`实现类型`（implementation type）。这两者可能相同，也可能不同。服务类型可以是类，也可以是接口，建议面向接口编程，更灵活。
>
> .NET控制反转组件取名为`DependencyInjection`,但它包含ServiceLocator的功能

## 生命周期

> 不要在长生命周期的对象中引用比它短的生命周期的对象，例如在Singleton对象中引用Transient对象

**生命周期的选择**

> - 如果类无状态：建议为Singleton
> - 如果类有状态，且有Scope控制：建议为Scoped
> - 使用Transient的时候要谨慎

```c#
static void Main(string[] args)
{
    ServiceCollection services = new ServiceCollection();
    services.AddScoped<TestServiceImpl1>();

    using (ServiceProvider sp = services.BuildServiceProvider())
    {
        TestServiceImpl1 t = sp.GetService<TestServiceImpl1>()!;
        t.Name = "tom";
        t.SayHi();

        TestServiceImpl1 t1 = sp.GetService<TestServiceImpl1>()!;
        t1.Name = "jerry";
        t1.SayHi();

        Console.WriteLine(object.ReferenceEquals(t, t1));

        t.SayHi();

        using (IServiceScope sp1 = sp.CreateScope())
        {
            // 在scope中获取Scope相关的对象
            TestServiceImpl1 s1 = sp1.ServiceProvider.GetService<TestServiceImpl1>()!;

            TestServiceImpl1 s2 = sp1.ServiceProvider.GetService<TestServiceImpl1>()!;

            Console.WriteLine(object.ReferenceEquals(s1, s2));
        }
    }
}
```

## 服务定位器

```c#
static void Main(string[] args)
{
    ServiceCollection services = new ServiceCollection();

    services.AddScoped<ITestService, TestServiceImpl1>();
    services.AddScoped<ITestService, TestServiceImpl2>();
    //services.AddScoped(typeof(ITestService), typeof(TestServiceImpl1));
    //services.AddSingleton(typeof(ITestService), new TestServiceImpl1());

    using (ServiceProvider sp = services.BuildServiceProvider())
    {
        /*
         * 如果找不到服务就返回null
         * GetService的类型要与ADD*的注册服务类型一致
         */
        ITestService ts = sp.GetService<ITestService>()!;
        //ITestService ts = (ITestService)sp.GetService(typeof(ITestService))!;
        /*
         * 找不到就直接抛异常
         */
        TestServiceImpl1 ts1 = sp.GetRequiredService<TestServiceImpl1>();

        IEnumerable<ITestService> tss = sp.GetServices<ITestService>();
        foreach (ITestService t in tss)
        {
            Console.WriteLine(t.GetType());
        }
        /*
         * 如果注册了多个服务，而获取是获取单个，以最后一个注册的为准
         */
        var tt = sp.GetService<ITestService>();


    }
}
```

## .NET依赖注入

**依赖注入具有“传染性”**

> - 如果类的对象是通过`DI创建`的：这个类的构造函数中声明的所有服务类型的参数都会被DI赋值
> - 如果一个对象是`手动创建`的：她的构造函数中声明的服务型参数就不会被自动赋值

.NET的DI默认是构造函数注入

**简单使用**

通过几个类来使用依赖注入

```c#
class Controller
{
    private readonly ILog log;
    private readonly IStorage storage;

    public Controller(ILog log, IStorage storage)
    {
        this.log = log;
        this.storage = storage;
    }

    public void Test()
    {
        this.log.Log("开始上传");
        this.storage.Save(".NET的依赖注入", "test.txt");
        this.log.Log("上传完毕");
    }
}
```

```c#
interface ILog
{
    public void Log(string message);
}

class LogImpl : ILog
{
    public void Log(string message)
    {
        Console.WriteLine($"日志:{message}");
    }
}
```

```c#
interface IConfig
{
    public string GetValue(string name);
}

class ConfigImpl : IConfig
{
    public string GetValue(string name)
    {
        return "hello" + name;
    }
}
class DBConfigImpl : IConfig
{
    public string GetValue(string name)
    {
        return $"Db helllo {name}";
    }
}
```

```c#
interface IStorage
{
    public void Save(string content, string name);
}

class StorageImpl : IStorage
{
    private readonly IConfig config;

    public StorageImpl(IConfig config)
    {
        this.config = config;
    }

    public void Save(string content, string name)
    {
        string server = config.GetValue("server");
        Console.WriteLine($"向服务器{server}的文件({name})上传了\"{content}\"");
    }
}
```

**主函数中服务注册及使用**

```c#
static void Main(string[] args)
{
    ServiceCollection services = new ServiceCollection();
    services.AddScoped<Controller>();
    services.AddScoped<ILog, LogImpl>();
    services.AddScoped<IStorage, StorageImpl>();
    //services.AddScoped<IConfig, ConfigImpl>();
    services.AddScoped<IConfig, DBConfigImpl>();

    using (ServiceProvider sp = services.BuildServiceProvider())
    {
        Controller c = sp.GetRequiredService<Controller>();
        c.Test();
    }
    Console.ReadKey();
}
```

​	综上，依赖注入实际上可以认为，我使用一个类时，不管这个类是什么样子，我就告诉框架我要这个类，我不需要自己new就可以使用，这样的好处是：1、如果声明对象的话，可能其构造函数的参数很难传入（比如是其他的类）；2、如果服务有修改，比如某接口的实现方法改了，我只需要在注册时修改注册函数，而不需要去修改业务代码

# 配置系统

## 配置读取

**1、直接读取**

> 需要安装：Microsoft.Extensions.Configuration  –基础包
>
> ​		Microsoft.Extensions.Configuration.Json	–Json对应

```c#
ConfigurationBuilder configBuilder = new ConfigurationBuilder();
configBuilder.AddJsonFile("config.json", optional: false, reloadOnChange: true);
IConfigurationRoot configRoot = configBuilder.Build();

string? name = configRoot["name"];
string? addr = configRoot.GetSection("proxy:address").Value;
string? addr = configRoot["proxy:address"];
```

**2、绑定读取**

> 需要安装：Microsoft.Extensions.Configuration.Binder

​	将对应节点绑定为一个对象，对象要提前声明

```c#
Proxy proxy = configRoot.GetSection("proxy").Get<Proxy>();
Console.WriteLine($"addr:{proxy.Address}, mask:{proxy.Netmask}");
```

**3、选项方式读取**(和DI结合)

> 需要安装：1、2中的 + Microsoft.Extensions.Options

​	读取配置的时候，DI要声明`IOptions<T>`、`IOptionsMonitor<T>`、`IOptionsSnapshot<T>`等类型

>- IOptions\<T\>:不会读取新的值
>- IOptionsMonitor\<T\>：会立即更新新的值
>- IOptionsSnapshot\<T>：会在一个范围内保持一致

```c#
/*****主函数中使用部分*****/
// DI 服务注册
ServiceCollection services = new ServiceCollection();
services.AddScoped<TestController>();

// configurationRoot声明
ConfigurationBuilder configBuilder = new ConfigurationBuilder();
configBuilder.AddJsonFile("config.json", optional: false, reloadOnChange: true);
IConfigurationRoot configRoot = configBuilder.Build();

/*
 * 注册Options服务，并将应用程序配置根中的对应配置项绑定到Config类型的实例中
 * 在应用程序中需要使用这些配置数据时，可以直接将Config类型的实例注入到其他组件中，从而方便地获取配置信息
 */
services.AddOptions().Configure<Config>(e=>configRoot.Bind(e));

using(var sp = services.BuildServiceProvider())
{
    var c = sp.GetRequiredService<TestController>();
    c.Test();
}

/*****TestController*****/
public class TestController
{
    private readonly IOptionsSnapshot<Config> optConfig;

    public TestController(IOptionsSnapshot<Config> optConfig)
    {
        this.optConfig = optConfig;
    }
    public void Test()
    {
        var config = optConfig.Value;
        Console.WriteLine($"Name:{config.Name},Age:{config.Age}\naddr:{config.proxy!.Address}\nmask:{config.proxy.Netmask}");
    }
}
```

## 其他配置

**1、命令行方式**

> 需要安装：Microsoft.Extensions.Configuration.CommandLine

**2、环境变量方式**

> 需要安装：Microsoft.Extensions.Configuration.EnvironmentVariables
