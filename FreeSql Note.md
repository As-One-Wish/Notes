`FreeSql`是功能强大的 `.NET ORM`，支持 `.NetFramework 4.0+`、`.NetCore 2.1+`、`Xamarin`等支持 NetStandard 所有运行平台。

# 特性

- 支持 CodeFirst 迁移；
- 支持 DbFirst 从数据库导入实体类，支持三种模板生成器；
- 采用 ExpressionTree 高性能读取数据；
- 支持深入的类型映射，比如 pgsql 的数组类型，堪称匠心制作；
- 支持丰富的表达式函数；
- 支持导航属性查询，和延时加载；
- 支持同步/异步数据库操作方法，丰富多彩的链式查询方法；
- 支持读写分离、分表分库，租户设计；
- 支持多种数据库，MySql/SqlServer/PostgreSQL/Oracle/Sqlite/Firebird/ClickHouse/QuestDB/达梦/南大通用GBase/神通/人大金仓/虚谷/翰高/华为GaussDB/MsAccess；

# 使用

## 数据库连接

```c#
public class ConfigOptions
{
    /**
     * 数据库连接字符串
     * 运行主机名-运行端口-连接用户名-连接密码-连接数据库-是否激活链接池-最小链接池数目
     **/
    public const string _DataBaseConnectStr =
        @"Host=localhost;Port=5432;Username=postgres;Password=root; Database=postgres;Pooling=true;Minimum Pool Size=1";

    public static IFreeSql? DataBaseConnect()
    {
        try
        {
            IFreeSql freeSql = new FreeSql.FreeSqlBuilder()
                .UseConnectionString(FreeSql.DataType.PostgreSQL, ConfigOptions._DataBaseConnectStr)
                .UseMonitorCommand(cmd => Console.WriteLine($"Sql:{cmd.CommandText}"))  // 监听SQL语句
                .UseAutoSyncStructure(true) // 自动同步实体结构到数据库,不会扫描程序集，只有CRUD时才会生成表，开发环境必备，生产环境慎用
                .Build();  // 务必定义成Singleton单例模式
            return freeSql;
        }
        catch (Exception ex)
        { 
            Console.WriteLine(ex.Message);
            return null;
        }
    }
}
```

**`Singleton`单例模式**：保证每一个类仅有一个实例，并为它提供一个全局访问点（程序中同一时刻最多存在该类的一个对象）

## 数据库操作

### 插入

**1、单条插入**

```c#
// 待添加信息
Company addCompany = new Company()
{
    CompanyName = "Learncenter",
    CreateTime = DateTime.Now
};

// 新增Company信息(返回受影响的行数)
int add_affect_rows = freeSql.Insert(addCompany).ExecuteAffrows();

// 新增后获取新增ID(有自增列则会返回ID)
long preId = freeSql.Insert(addCompany).ExecuteIdentity();

// 依赖FreeSql.Repository
/* 待补充 */
```

**2、批量插入**

当插入大批量数据时，内部采用分割分批执行的逻辑进行。分割规则如下（以PostgreSQL为例）：

|            | 数量 | 参数量 |
| :--------: | :--: | :----: |
| PostgreSQL | 5000 |  3000  |

数量：每批分割的大小，如批量插入 10000 条数据，在 mysql 执行时会分割为两批

参数量：为每批分割的参数量大小，如批量插入 10000 条数据，每行需要使用 5 个参数化，在 mysql 执行时会分割为每批 3000 / 5

```c#
List<Company> companies = new List<Company>()
{
    new Company()
    {
        CompanyName = "LearnCenter1",
        CreateTime = DateTime.Now   
    },
    new Company()
    {
        CompanyName = "LearnCenter2",
        CreateTime = DateTime.Now
    },
    new Company()
    {
        CompanyName = "LearnCenter3",
        CreateTime = DateTime.Now
    },
};

int add_affect_rows = freeSql.Insert(companies).ExecuteAffrows();
Console.WriteLine(add_affect_rows);

// BulkCopy
freeSql.Insert(companies).ExecutePgCopy();  
```

**3、插入指定的列**

```c#
// 插入指定某一列（只将addCompany中的在InsertColumns中说明的字段插入到数据库）
int add_effect_rows = freeSql.Insert(addCompany).InsertColumns(c => c.CreateTime).ExecuteAffrows();

// 指定插入某几列
int add_effect_rows = freeSql.Insert(addCompany).InsertColumns(c => new { c.CompanyName, c.CreateTime }).ExecuteAffrows();
```

**4、忽略列**

```c#
// 插入时忽略某一列(如果被忽略的列不能为空会报错)
int ignore_add_rows = freeSql.Insert(addCompany).IgnoreColumns(c=>c.CompanyName).ExecuteAffrows();

// 插入时忽略某几列
int ignores_add_rows = freeSql.Insert(addCompany).IgnoreColumns(c=>new {c.CompanyName, c.CreateTime }).ExecuteAffrows();
```

**5、字典插入**

```C#
// 字典插入(不用定义实体进行插入)
Dictionary<string, object> dict = new Dictionary<string, object>
{
    { "CompanyName", "dict_insert" },
    { "CreateTime", DateTime.Now }
};
freeSql.InsertDict(dict).AsTable(nameof(Company)).ExecuteAffrows();
```

**6、导入表数据**

`A`表数据到`B`表，要保证`A`、`B`表均存在（可以是同一个表），未被选择的列将会使用实体类中声明的默认值或者是C#默认值

```c#
// 导入表数据
int inTable_rows = freeSql.Select<Company>().Limit(10).InsertInto(null, a => new Company
{
    CompanyName = a.CompanyName,
    CreateTime = a.CreateTime
});
```

**7、API**

|       方法        |  返回值  |           参数           |                   描述                   |
| :---------------: | :------: | :----------------------: | :--------------------------------------: |
|    AppendData     | \<this\> | TI \| IEnumberable\<T1\> |            追加准备插入的实体            |
|  InsertIdentity   | \<this\> |            无            |              指明插入自增列              |
|   InsertColums    | \<this\> |          Lambda          |                只插入的列                |
|   IgnoreColums    | \<this\> |          Lambda          |                 忽略的列                 |
|  CommandTimeout   | \<this\> |           int            |            命令超时设置（秒）            |
|  WithTransaction  | \<this\> |      DbTransaction       |               设置事务对象               |
|  WithConnection   | \<this\> |       DbConnection       |               设置连接对象               |
|       ToSql       |  string  |            无            |          返回即将执行的SQL语句           |
|  ExecuteAffrows   |   long   |            无            |       执行SQL语句，返回影响的行数        |
|  ExecuteIdentity  |   long   |            无            |         执行SQL语句，返回子增值          |
|  ExecuteInserted  | List\<T> |            无            |      执行SQL语句，返回插入后的记录       |
| ExecutePgBulkCopy |   void   |            无            | PostgreSQL特有功能，执行Copy批量导入数据 |

### 删除

删除是一个非常危险的操作，FreeSql 对删除支持并不强大，默认仅支持单表、且有条件的删除方法。

若 `Where` 条件为空的时候执行，仅返回 `0` 或默认值，不执行真正的 SQL 删除操作。

**1、动态条件**（不用看，基本不用）

```c#
freeSql.Delete<User>(object dywhere)
```

`dywhere`支持：

```c#
// 主键值（new[] {主键值1，主键值2，···}）
freeSql.Delete<User>(new[] { 1, 2, 3 }).ExecuteAffrows();
// 实体对象（new User{}、new[]{new User, new User,···}）
freeSql.Delete<User>(new User { UserName = "CSharpLearner-17", Id = 17 }).ExecuteAffrows();
freeSql.Delete<User>(new User { UserName = "CSharpLearner-17" }).ExecuteAffrows();
freeSql.Delete<User>(new[] { new User { Id = 18 }, new User { Id = 19 } }).ExecuteAffrows();
```

存在问题：动态条件删除似乎只会按照ID进行删除，不受其他字段的影响

**2、删除条件**

出于安全考虑，没有条件不执行删除动作，避免误删除全表数据。删除全表数据：`fsql.Delete<T>().Where("1=1").ExecuteAffrows()`

```c#
freeSql.Delete<User>().Where(a => a.Id == 56).ExecuteAffrows();
freeSql.Delete<User>().Where(a => a.UserName == "CSharpLearner-27").ExecuteAffrows();

// 有问题
//freeSql.Delete<User>().Where("id = @id", new { id = 6 }).ExecuteAffrows(); // ?
// 解决-PostgreSQL是大小写敏感的，在SQL语句中要用双引号把字段名括起来
freeSql.Delete<User>().Where(@"""Id"" = @id", new { id = 841 }).ExecuteAffrows();

User uu = new User { Id = 14, UserName = "No" };
freeSql.Delete<User>().Where(uu).ExecuteAffrows();

List<User> users = new List<User>();
for(int i = 45; i < 50; ++i)
{
    users.Add(new User { Id = i });
}
freeSql.Delete<User>().Where(users).ExecuteAffrows();
```

**3、字典删除**

```c#
// 只有当字段中的字段匹配到一个记录上时才会删除
Dictionary<string, object> dict = new Dictionary<string, object>
{
    {"Id", 35 },
    { "UserName", "CSharpLearner-33" }
};
freeSql.DeleteDict(dict).AsTable("User").ExecuteAffrows();
// DELETE FROM "User" WHERE ("Id" = 35 AND "UserName" = 'CSharpLearner-33')
```

**4、`ISelect.ToDelete` 高级删除**

**5、`IBaseRepository`级联删除**

**6、API**

|      方法       |  返回值  |       参数        |                        描述                        |
| :-------------: | :------: | :---------------: | :------------------------------------------------: |
|      Where      | \<this\> |      lambda       |   表达式条件，仅支持实体基础成员(不包含导航对象)   |
|      Where      | \<this\> |   string,parms    | 原生sql语法条件，Where("id = @id", new { id = 1 }) |
|      Where      | \<this\> | T \| IEnumber\<T> |          传入实体或集合，将其主键作为条件          |
| CommandTimeout  | \<this\> |        int        |                 命令超时设置（秒）                 |
| WithTransaction | \<this\> |   DbTransaction   |                    设置事务对象                    |
| WithConnection  | \<this\> |   DbConnection    |                    设置连接对象                    |
|      ToSql      |  string  |        无         |               返回即将执行的SQL语句                |
| ExecuteAffrows  |   long   |        无         |            执行SQL语句，返回影响的行数             |
| ExecuteDeleted  | List\<T> |        无         |                                                    |

### 更新

**1、更新指定列**

```c#
freeSql.Update<Company>(38)
    .Set(a => a.CreateTime, DateTime.Now)
    .Set(a => a.CompanyName, "UpdateTest").ExecuteAffrows();

freeSql.Update<Company>(37)
    .Set(a=>new Company
    {
        CreateTime = DateTime.Now,
        CompanyName = "UpdateTest2",
    }).ExecuteAffrows();
// 出于安全考虑，没有条件不执行更新动作
freeSql.Update<Company>()
    .Set(a => new Company { CreateTime = DateTime.Now, CompanyName = "UpdateWhereTest" })
    .Where(a => a.Id == 36).ExecuteAffrows();
```

**2、更新实体**

- 推荐

只更新变化的属性，依赖`FreeSql.Repository`

```c#
var repos = freeSql.GetRepository<Company>();

// 先查询，更新只变化的字段
var item = repos.Where(a => a.Id == 37).First(); // 快照 item
item.CompanyName = "RepositoryTest";
repos.Update(item); // 对比快照时的变化，如果没有变化则不执行SQL语句，有变化则只更新有变化的字段

// 不查询直接使用状态管理更新
Company comp = new Company { Id = 38 };
repos.Attach(comp); // 此时快照comp
comp.CompanyName = "NoQueryUpdate";
repos.Update(comp);
```

**状态管理**

​	有些ORM是通过属性通知做的，对属性赋值后会记录状态，最终更新对象的时候就知道哪些属性是变化过的，缺点是对实体类有一定入侵。

​	`FreeSql.Repository`实现了对象的状态管理，对实体类没有入侵，原理是**在仓储中有一个对象副本，在最终更新对象的时候与副本对比，从而得到变化的属性**。

- 原始

```c#
// 原始方法
{
    Company new_company = new Company { Id = 37, CompanyName = "OldMethod", CreateTime = DateTime.Now };
    freeSql.Update<Company>().SetSource(new_company).ExecuteAffrows();

    Company new_company_1 = new Company { Id = 38, CompanyName = "OldMethod_38", CreateTime = DateTime.Now };
    /* 指定列更新 */
    freeSql.Update<Company>().SetSource(new_company_1).UpdateColumns(a => new { a.CompanyName }).ExecuteAffrows();

    Company new_company_2 = new Company { Id = 36, CompanyName = "OldMethon_39", CreateTime = DateTime.Now };
    /* 忽略列更新 */
    freeSql.Update<Company>().SetSource(new_company_2).IgnoreColumns(a => new { a.CompanyName }).ExecuteAffrows();

    /* 批量更新 */
    List<User> users = new List<User>();
    for (int i = 0; i < 3; ++i)
    {
        users.Add(new User { Id = 11 + i, UserName = $"CSharpLearner-{10 + i}", CreateTime = DateTime.Now });
    }
    freeSql.Update<User>().SetSource(users).ExecuteA
    // BulkCopy
    freeSql.Update<User>().SetSource(users).ExecutePgCopy();
```

指定`Set`列更新后，`SetSource`将会失效；`SetSource`默认依赖实体`IsPrimary`特性，临时主键可使用`SetSource(items, a=>a.Code)`

**3、根据DTO更新**

```c#
freeSql.Update<Company>()
    .SetDto(new { CompanyName = "DtoTest", CreateTime = DateTime.Now })
    .Where(a => a.Id == 37).ExecuteAffrows();

freeSql.Update<Company>()
    .SetDto(new Dictionary<string, object> { ["CompanyName"] = "DtoDictTest", ["CreateTime"] = DateTime.Now })
    .Where(a => a.Id == 38).ExecuteAffrows();
```

**DTO**：是在应用程序的不同层之间传输数据的一种模式，通常用于将数据从数据访问层传递到业务逻辑层或表示层，其目的是在不同层之间提供简单/轻量级的数据交换。

**4、`Set`  `SetSource`  `SetDto`**

三个是平级功能：

- `Set/SetRow`：在知道实体的时候使用，对应`update t set x = x`
- `SetSource`：更新整个实体，可以配合`UpdateColumns`、`IgnoreColumns`指定或忽略字段
- `SetDto`：是`Set`的批量操作

**5、字典更新**

```c#
Dictionary<string, object> dict = new Dictionary<string, object>()
{
    {"Id", 36 },
    {"CompanyName", "DictTest" },
    {"CreateTime", DateTime.Now }
};
freeSql.UpdateDict(dict).AsTable("Company").WherePrimary("ID").ExecuteAffrows();
```

**6、乐观锁和悲观锁**

- 乐观锁

假设在大多数情况下数据访问不会发生冲突，因此不会阻塞其他事务的执行；通过在数据更新时进行版本检查来保证数据的一致性。每个实体只支持一个乐观锁属性，在属性前标记特性：`[Column(IsVersion = true)]` 即可。

适用 `SetSource` 更新，每次更新 `version` 的值都会增加 `1`

- 悲观锁

假设在数据访问期间会发生冲突，因此在操作数据时会锁定资源，阻塞其他事务对该资源的访问；常见实现是通过数据库的锁机制来实现。

```c#
var user = fsql.Select<User>()
  .ForUpdate(true)
  .Where(a => a.Id == 1)
  .ToOne();
```

乐观锁适合于并发写入较少，冲突较少的场景，可以提高并发性能。而悲观锁适合于并发写入较多，冲突较频繁的场景，可以确保数据的一致性。

**7、`ISelect.ToUpdate` 高级更新**

**8、联表更新 `UpdateJoin`**

**9、API**

|      方法       | 返回值  |         参数         |                             描述                             |
| :-------------: | :-----: | :------------------: | :----------------------------------------------------------: |
|    SetSource    | \<this> | T1\| IEnumerable\<T> |                   更新数据，设置更新的实体                   |
|  IgnoreColumns  | \<this> |        Lambda        |                           忽略的列                           |
|       Set       | \<this> |    Lambda, value     |         设置列的新值，`Set(a => a.Name, "newvalue")`         |
|       Set       | \<this> |        Lambda        | 设置列的的新值为基础上增加，`Set(a => a.Clicks + 1)`，相当于 clicks=clicks+1 |
|     SetDto      | \<this> |        object        |                     根据 DTO 更新的方法                      |
|     SetRaw      | \<this> |    string, parms     | 自定义 SQL 语法，`SetRaw("title = @title", new { title = "newtitle" })` |
|      Where      | \<this> |        Lambda        |       表达式条件，仅支持实体基础成员（不包含导航对象）       |
|      Where      | \<this> |    string, parms     |    原生 sql 语法条件，`Where("id = @id", new { id = 1 })`    |
|      Where      | \<this> | T \| IEnumerable\<T> |               传入实体或集合，将其主键作为条件               |
| CommandTimeout  | \<this> |         int          |                       命令超时设置(秒)                       |
| WithTransaction | \<this> |    DbTransaction     |                         设置事务对象                         |
| WithConnection  | \<this> |     DbConnection     |                         设置连接对象                         |
|      ToSql      | string  |                      |                   返回即将执行的 SQL 语句                    |

### 插入和更新

**1、`IFreeSql.InsertOrUpdate`**

```c#
// 实体列表声明
List<Company> list = new List<Company>()
    {
        new Company(){Id=70, CompanyName="CSharpLearnCenter"},
        new Company(){CompanyName="JavaLearnCenter"}
    };
```

```c#
// IFreeSql.InsertOrUpdate
freeSql.InsertOrUpdate<Company>().SetSource(list)
    .IfExistsDoNothing()   // 如果存在则无操作==当数据不存在则插入
    .ExecuteAffrows();
// BulkCopy
freeSql.InsertOrUpdate<Company>().SetSource(list).ExecutePgCopy();

// FreeSql.Repository 之 InsertOrUpdate
var repos = freeSql.GetRepository<Company>();
repos.InsertOrUpdate(list[0]);
```

两者都是以ID主键为主：当实体不声明主键时，其值默认为0，此时认为此条数据不存在，直接插入；只有当明确声明ID主键且其值时数据库中已有的值，才认为数据已存在。

具体实现上，两者的机制不一样：对于前者，当实体类有自增属性时，批量`InsertOrUpdate`最多被拆成两次执行，内部计算出未设置子增值和有设置自增值的数据，分别执行`insert into`和`merge into`两种命令（采用事务执行）；对于后者，如果内部的状态管理存在数据，则更新，反正则插入，不支持批量操作。

### 插入

[查询 | FreeSql 官方文档](https://freesql.net/guide/select.html)

```c#
/**
 * 单表查询
 **/
{
    //List<Company> companies_1 = freeSql.Queryable<Company>().Where(a => a.Id == CSharpCompanyID).ToList();
    List<Company> companies_1 = freeSql.Select<Company>().Where(a => a.Id == CSharpCompanyID).ToList();
    CompaniesDisplay(companies_1);
}
/**
 * 多表查询
 **/
{
    var lis_1 = freeSql.Select<Company, User>()
        .InnerJoin((a, b) => a.Id == b.CompanyId)
        .Where(c => c.t1.Id == CSharpCompanyID)
        .Where(c => c.t2.CompanyId == CSharpCompanyID).ToList();
    CompaniesDisplay(lis_1);

    var lis_2 = freeSql.Select<Company, User>()
        .InnerJoin((a, b) => a.Id == b.CompanyId)
        .Where((c, u) => c.Id == CSharpCompanyID).ToList();
    CompaniesDisplay(lis_2);

    // 子表First/Count/Sum/Max/Min/Avg
    var list_3 = freeSql.Select<Company>().ToList(a => new
    {
        all = a,
        first = freeSql.Select<User>().Where(b => b.CompanyId == a.Id).First(b => b.Id),
        count = freeSql.Select<User>().Where(b => b.CompanyId == a.Id).Count(),
        sum = freeSql.Select<User>().Where(b => b.CompanyId == a.Id).Sum(b => b.Age),
        max = freeSql.Select<User>().Where(b => b.CompanyId == a.Id).Max(b => b.Age),
        min = freeSql.Select<User>().Where(b => b.CompanyId == a.Id).Min(b => b.Age),
        avg = freeSql.Select<User>().Where(b => b.CompanyId == a.Id).Avg(b => b.Age)
    });
    foreach (var item in list_3)
    {
        Console.WriteLine("--------------------------------------------");
        CompanyDisplay(item.all);
        Console.WriteLine($"First:{item.first},Count:{item.count},Sum:{item.sum},Max:{item.max},
                          Min:{item.min},Avg:{item.avg}");
    }

    // WhereCascade
    var list_4 = freeSql.Select<Company>()
        .LeftJoin<User>((c, u) => c.Id == u.CompanyId)
        .WhereCascade(x => x.Id == CPlusCompanyID)   // WhereCascade 说明是在关联完毕后的整体的一个筛选
        .ToList();
    /*SELECT a."Id", a."CompanyName", a."CreateTime" FROM "Company" a
        LEFT JOIN "User" u ON a."Id" = u."CompanyId" AND(u."Id" = 2) WHERE(a."Id" = 2)*/
    CompaniesDisplay(list_4);

}

/**
 * 嵌套查询
 **/
{
    // 查询分组第一条记录
    var list_1 = freeSql.Select<User>()
        .Where(a => a.Id < 500)
        .WithTempQuery(a => new
        {
            item = a,
            rownum = SqlExt.RowNumber().Over().OrderBy(a.Id).ToValue()
        })
        .Where(a => a.rownum == 1).ToList();
    /*SELECT* FROM(
        SELECT a."Id", a."Age", a."UserName", a."CreateTime", a."UserDetailId", a."CompanyId", row_number() 
        over(order by a."Id") "rownum"
        FROM "User" a WHERE(a."Id" < 80) ) a WHERE(a."rownum" = 1)*/
    foreach (var item in list_1)
    {
        Console.WriteLine("--------------------------------------");
        UserDisplay(item.item);
        Console.WriteLine(item.rownum);
    }

    // 如果数据库不支持开窗函数，可以使用嵌套查询解决
    var list_2 = freeSql.Select<User>()
        .Where(a => a.Id < 500)
        .GroupBy(a => a.CompanyId)
        .WithTempQuery(g => new { min = g.Min(g.Value.Id) })
        .From<User>()
        .InnerJoin((a, b) => a.min == b.Id)
        .ToList((a, b) => b);
    /*SELECT b."Id" as1, b."Age" as2, b."UserName" as3, b."CreateTime" as4, b."UserDetailId" as5, b."CompanyId" as6 FROM(
        SELECT min(a."Id") "min" FROM "User" a WHERE(a."Id" < 500) GROUP BY a."CompanyId" ) a 
        INNER JOIN "User" b ON a."min" = b."Id"*/
    UsersDisplay(list_2);
}

/**
 * 联合查询
 **/
{
    // 单表 Union ALL (求并集)
    var list_1 = freeSql.Select<User>().Where(a => a.Id == 1)
        .UnionAll(
            freeSql.Select<User>().Where(a => a.Id == 2),
            freeSql.Select<User>().Where(a => a.Id == 3)
        )
        //.Where(a => a.Id == 1 || a.Id == 2)
        .ToList();
    UsersDisplay(list_1);

    // 多表 Union ALL
    var list_2 = freeSql.Select<User, Company>()
        .Where((u, c) => c.Id == CSharpCompanyID)
        .WithTempQuery((a, b) => new { user = a, company = b })
        .UnionAll(
            freeSql.Select<User, Company>()
            .Where((u, c) => c.Id == CPlusCompanyID)
            .WithTempQuery((a, b) => new { user = a, company = b })
        )
        .Where(a => a.company.Id == CSharpCompanyID || a.company.Id == CPlusCompanyID)
        .ToList();
    foreach (var a in list_2)
    {
        UserDisplay(a.user);
        CompanyDisplay(a.company);
    }
}
// Linq To Sql
{
    // 将 ISelect 转为 IQueryable
    // Nuget引入程序集:FreeSql.Extensions.Linq
    IQueryable<Company> company_queryable = freeSql.Select<Company>().AsQueryable();

    // Linq方法查询
    var res_1 = company_queryable.Where(a => a.Id == 1).FirstOrDefault();

    // 将 IQueryable 还原为 ISelect
    ISelect<Company> company_select = company_queryable.RestoreToSelect();
  
  // Where
    var list_1 = (from a in freeSql.Select<Company>()
                  where a.Id == CSharpCompanyID
                  select a).ToList();
    // Select(指定字段)
    var list_2 = (from a in freeSql.Select<Company>()
                  where a.Id < 2
                  select new { a.Id }).ToList();
    // CaseWhen
    var list_3 = (from a in freeSql.Select<Company>()
                  where a.Id == CPlusCompanyID
                  select new
                  {
                      a.Id,
                      a.CompanyName,
                      testsub = new
                      {
                          time = a.Id > 10 ? "大于" : "小于等于"
                      }
                  }).ToList();
    // Join
    var list_4 = (from a in freeSql.Select<Company>()
                  join b in freeSql.Select<User>() on a.Id equals b.CompanyId
                  select a).ToList();
    // LeftJoin
    var list_5 = (from a in freeSql.Select<Company>()
                  join b in freeSql.Select<User>() on a.Id equals b.CompanyId into temp
                  from tc in temp.DefaultIfEmpty()
                  select a).ToList();
    // From(多表查询)
    var list_6 = (from a in freeSql.Select<Company>()
                  from b in freeSql.Select<User>()
                  where a.Id == b.CompanyId
                  select a).ToList();
    // GroupBy(分组)
    var list_7 = (from a in freeSql.Select<User>()
                  group a by new { a.CompanyId } into g
                  select new
                  {
                      g.Key.CompanyId,
                      cnt = g.Count(),
                      avg = g.Avg(g.Value.Age),
                      sum = g.Sum(g.Value.Age),
                      max = g.Max(g.Value.Age),
                      min = g.Min(g.Value.Age)
                  }).ToList();
}
/**
 * 分页查询
 **/
{
    // 同步分页
    var list_1 = freeSql.Select<User>().Where(a => a.Id > 10)
        .Count(out var total)
        .Page(1, 20).ToList();
    Console.WriteLine(total);
    UsersDisplay(list_1);
}

/**
 * 树型查询
 **/
{
    // 查询过滤条件后的数据，不带有层级关系
    var treeList1 = freeSql.Select<Menu>()
        .Where(m => m.MenuName == "一级菜单")
        .ToTreeList().ToList();

    // 不带Where条件可以显示出菜单的层级关系
    var treeList2 = freeSql.Select<Menu>().ToTreeList().ToList();
}
```

**开窗函数**（Window Function）也称为窗口函数或分析函数，是一类特殊的函数，用于在查询结果中执行聚合操作，并可以根据定义的窗口范围对数据进行分组和排序。开窗函数通常和"OVER"子句一起使用，该子句用于定义窗口的范围。

**Linq To Sql**：允许使用类似SQL的语法来查询和操作数据，但是它是在编译时继续类型检查，因此更加类型安全。意义如下：

- 类型安全：由于 LINQ 在编译时执行类型检查，可以在编译阶段捕获一些潜在的错误，例如拼写错误、类型不匹配等。这有助于在开发过程中减少一些运行时错误
- 易读性：LINQ 查询通常采用类似于 SQL 的语法，这使得查询代码更易读和维护。通过使用 LINQ，可以用更直观的方式表达查询意图，而不需要编写复杂的 SQL 语句
- 抽象数据库：使用 Linq To Sql，可以以一种数据库无关的方式查询数据，这样可以在不更改查询代码的情况下，轻松切换不同类型的数据库
- 效率和性能：虽然 LINQ 查询会被转换为 SQL 语句执行，但它们通常会受到 LINQ 提供者的优化，以提高查询性能。FreeSql 为 LINQ 提供了一个专门的提供者，以确保查询在数据库中以最佳方式执行
- 避免 SQL 注入：使用 LINQ 查询可以减少 SQL 注入攻击的风险。LINQ 提供者会处理输入参数，将它们转换为安全的查询

**导航属性**：指在实体类中定义的用于表达实体之间关系的属性。通过导航属性，可以轻松地在实体之间建立关联，类似于数据库中的外键关系。导航属性在查询和操作相关联的数据时非常有用，可以简化代码并提供更直观的数据访问方式。

```c#
// 实体类
[Table(Name = "authors")]
public class Author
{
    [Column(IsIdentity = true, IsPrimary = true)]
    public int Id { get; set; }

    [Column(Name = "name")]
    public string? Name { get; set; }

    // 导航属性，表示作者拥有的书籍列表
    [Navigate(nameof(Book.AuthorId))]
    public List<Book>? Books { get; set; }
}
[Table(Name = "books")]
public class Book
{
    [Column(IsIdentity = true, IsPrimary = true)]
    public int Id { get; set; }

    [Column(Name = "title")]
    public string? Title { get; set; }

    [Column(Name = "author_id")]
    public int AuthorId { get; set; }

    // 导航属性，表示书籍所属的作者
    [Navigate(nameof(AuthorId))]
    public Author? Author { get; set; }
}
// 操作
var authorWithBooks = freeSql.Select<Author>()
                             .IncludeMany(a => a.Books) // 使用 IncludeMany 方法加载导航属性 Books
                             .Where(a => a.Id == 1)
                             .First();
```

**贪婪加载**：[贪婪加载 | FreeSql 官方文档](https://freesql.net/guide/select-include.html)

## 事务

恢复和并发控制的基本单位（操作单元）。ACID特性：

- 原子性（Atomicity）：一个事务是一个不可分割的工作单位，事务中的操作要么都做，要么都不做
- 一致性（Consistency）：事务必须是使数据库从一个一致性状态变到另一个一致性状态
- 隔离性（Isolation）：一个事务内部的操作及使用的数据对并发的其他的事务是隔离的，并发执行的各个事务之间蹦年互相干扰
- 持久性（Durability）：一个事务一旦提交，其对数据库中数据的改变就是永久性的，后续其他操作或故障不应该对其有任何影响

**1、`UnitOfWork`事务**

```c#
/**
 * UnitOfWork事务
 **/
{
    IRepositoryUnitOfWork uow;
    try
    {
        freeSql.Insert(new User() // 事务外，直接插入
        {
            UserName = "test",
            Age = 23,
            CompanyId = 1,
            CreateTime = DateTime.Now,
            UserDetailId = 1
        }).ExecuteAffrows();

        using (uow = freeSql.CreateUnitOfWork()) // 事务内，待事务内操作全部正确才会提交
        {
            var companyRepo = freeSql.GetRepository<Company>();
            var userRepo = freeSql.GetRepository<User>();

            // 手工绑定单元
            companyRepo.UnitOfWork = uow;
            userRepo.UnitOfWork = uow;

            companyRepo.Insert(new Company()
            {
                CompanyName = "UnitOfWorkTest-1",
                CreateTime = DateTime.Now
            });

            User user = userRepo.Select.First();
          	// 字符长度超过限制，对该信息的更新会出错
            user.UserName = "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串";
            userRepo.Update(user);

            uow.Orm.Insert(new Company()
            {
                CompanyName = "UnitOfWorkTest-2",
                CreateTime = DateTime.Now
            }).ExecuteAffrows();

            /*
            --uow.Orm 和 freeSql 都是 IFreeSql
            --uow.Orm CURD 与 uow 是一个事务(可以理解为临时 IFreeSql)
            --freeSql 和 uow 不在一个事务
            */
            uow.Commit(); ;
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

**2、`DbContxt`事务**

```c#
/**
 * DbContext事务
 **/
{
    try
    {
        using (DbContext ctx = freeSql.CreateDbContext())
        {
            DbSet<Company> company = ctx.Set<Company>();
            DbSet<User> user = ctx.Set<User>();

            company.Add(new Company()
            {
                CompanyName = "DbContextTest-1",
                CreateTime = DateTime.Now
            });

            User upUser = user.Select.First();
            upUser.UserName = "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
                "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串";
            user.Update(upUser);

            ctx.SaveChanges(); // 相当于commit
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

**3、同线程事务**

同线程事务，由 fsql.Transaction 管理事务提交回滚（缺点：不支持异步），比较适合 WinForm/WPF UI 主线程使用事务的场景

同线程事务使用简单，需要注意的限制：

- 事务对象在线程挂载，每个线程只可开启一个事务连接，嵌套使用的是同一个事务；
- 事务体内代码不可以切换线程，因此不可使用任何异步方法，包括 FreeSql 提供的数据库异步方法（可以使用任何 Curd 同步方法）

```c#
/**
 * 同线程事务（不支持异步做法）
 **/
{
    try
    {
        freeSql.Transaction(() =>
        {
            freeSql.Insert(new Company()
            {
                CompanyName = "Transaction()同线程事务",
                CreateTime = DateTime.Now
            }).ExecuteAffrows();

            User upUser = freeSql.Select<User>().First();
            upUser.UserName = "超长字符串";
            int updateResult = freeSql.Update<User>().SetSource(upUser).ExecuteAffrows();

            if (updateResult < 0)
            {
                throw new Exception("事务回滚！");
            }
        });
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
        Console.WriteLine("事务已回滚");
    }
}
```

**4、外部事务**

事务由外部开启，使用 WithTransaction 将事务对象传入执行

```c#
/**
 * 外部事务
 **/
{
    // 获取DbTransaction
    // 同步方法using新语法
    using Object<DbConnection> conn = freeSql.Ado.MasterPool.Get();
    using DbTransaction transaction = conn.Value.BeginTransaction();

    try
    {
        freeSql.Insert(new Company()
        {
            CompanyName = "conn.Value.BeginTransaction 外部事务",
            CreateTime = DateTime.Now
        }).WithTransaction(transaction).ExecuteAffrows();

        User user = freeSql.Select<User>().First();
        user.UserName = "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
            "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
            "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串" +
            "超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串超长字符串";
        freeSql.Update<User>().SetSource(user).ExecuteAffrows();

        transaction.Commit();
    }catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

**5、更新排他锁**

```c#
freeSql.Transaction(() =>
  {
      var user = freeSql.Select<User>().ForUpdate(true).Where(a => a.Id == 1).ToOne();
  });
```

对于以上加了更新排他锁的查询语句，其生成的SQL语句为：

```sql
SELECT a."Id", a."Age", a."UserName", a."CreateTime", a."UserDetailId", a."CompanyId" 
	FROM "User" a WHERE (a."Id" = 1) limit 1 for update nowait
```

- `for update nowait`：

​		这个短语通常用于SQL语句的上下文，特别是在处理事务环境中的SELECT查询时。当执行带有“`for update await`”的SELECT语句时，它会在检索的行上获得共享锁。这些共享锁表示其他事务**仍然可以读取**这些行，但在当前事务完成之前**不能对其进行修改**。

​		该语句中的“`await`”部分表示当前事务**会等待**其他已经持有的锁释放后才会执行。这个语法主要用于避免死锁问题，因为在获取共享锁时，如果某些行已经被其他事务以排他锁（exclusive lock）持有，则当前事务会等待直到这些排他锁被释放。

## 导航属性

​		实体框架中的导航属性提供了一种在两个实体类型之间导航关联的方法。导航属性在概念模型中由`NavigationProperty`元素定义。针对对象参与到其中的每个关系，各对象均可以具有导航属性。使用导航属性，您可以在两个方向上导航和管理关系，如果重数为一或者零或一，则返回`EntityReference`，如果重数为多个，则返回`EntityCollection`，也可以选择单向导航，这种情况下可以删除导航属性。

```c#
/**
 * OneToOne
 * 一个User对应一个UserDetail
 * 没有提供太多边界，实际还需要手动去维护
 **/
{
    // 无事务添加
    freeSql.Delete<User>(freeSql.Select<User>().ToList()).ExecuteDeleted();
    freeSql.Delete<UserDetail>(freeSql.Select<UserDetail>().ToList()).ExecuteDeleted();

    var userRepository = freeSql.GetRepository<User>();
    var userDetailRepository = freeSql.GetRepository<UserDetail>();

    User user = new User()
    {
        UserName = "OneToOne",
        Age = new Random().Next(15, 30),
        CreateTime = DateTime.Now
    };
    userRepository.Insert(user);
    // 手动在UserDetail表中添加UserId，但是User表中没有UserDetailId
    UserDetail userDetail = new UserDetail()
    {
        Description = "NavigationProperty-OneToOne",
        Address = "localhost",
        UserId = user.Id
    };
    userDetailRepository.Insert(userDetail);

}

/**
 * OneToMany
 * 一个Company对应多个User
 **/
{
    freeSql.Delete<User>(freeSql.Select<User>().ToList()).ExecuteDeleted();
    freeSql.Delete<Company>(freeSql.Select<Company>().ToList()).ExecuteDeleted();

    // 级联新增
    // 新增公司同时新增多个用户
    var userRepository = freeSql.GetRepository<User>();
    var companyRepository = freeSql.GetRepository<Company>();

    // 配置级联新增
    companyRepository.DbContextOptions.EnableCascadeSave = true; // 级联保存

    Company company = new Company()
    {
        CompanyName = "NavigationProperty-OneToMany",
        CreateTime = DateTime.Now,
        UserList = new List<User>()
        {
            new User()
            {
                UserName = "张三",
                Age = new Random().Next(15, 45),
                CreateTime = DateTime.Now,
            },
            new User()
            {
                UserName = "李四",
                Age = new Random().Next(15,45),
                CreateTime = DateTime.Now
            }
        }
    };
    companyRepository.Insert(company);

    // 贪婪加载
    List<Company> companyList  = companyRepository.Select.IncludeMany(c=>c.UserList).ToList();

    // 延迟加载：需要某一块数据的时候才去查询，否则不查询
    var companyList1 = freeSql.Select<Company>().ToOne();
    /* 只是查到了Company信息
     * SELECT a."Id", a."CompanyName", a."CreateTime" FROM "Company" a limit 1*/

    foreach (var item in companyList1.UserList!) ; // !操作符覆盖编译器的警告信息
  	// 注意对应的实体要加 virtual 修饰来实现延迟加载
    /* 当时用到company中的UserList时才会去查
     * SELECT a."Id", a."Age", a."UserName", a."CreateTime", a."UserDetailId", a."CompanyId" 
     * FROM "User" a WHERE (a."CompanyId" = 16)*/
}
/**
 * ManyToOne
 * 相当于OneToMany的一个逆向
 * 没有提供便捷功能
 **/
{
    var companyRepository = freeSql.GetRepository<Company>();
    var userRepository = freeSql.GetRepository<User>();

    userRepository.DbContextOptions.EnableCascadeSave = true; // 级联保存

    var user = new User()
    {
        UserName = "张三",
        Age = new Random().Next(15, 45),
        CreateTime = DateTime.Now,
        CompanyInfo = new Company()
        {
            CompanyName = "ManyToOne",
            CreateTime = DateTime.Now
        }
    };

    userRepository.Insert(user);
}

/**
 * ManyToMany
 * 一个用户对应多个菜单；一个菜单对应多个用户
 * 引入第三张关系表--Map表
 * 只支持一对多的便捷处理，多对多还是需要手动维护
 **/
{
    var userRepository = freeSql.GetRepository<User>();
    userRepository.DbContextOptions.EnableCascadeSave = true;

    User user = new User()
    {
        UserName = "ManyToMany",
        CreateTime = DateTime.Now,
        UserMencMapInfo = new List<UserMenuMap>()
        {
            new UserMenuMap()
            {
                MenuInfo = new Menu
                {
                    MenuName = "菜单管理",
                    CreateTime = DateTime.Now,
                },
            },
            new UserMenuMap()
            {
                MenuInfo = new Menu
                {
                    MenuName = "用户管理",
                    CreateTime = DateTime.Now,
                }
            }
        }
    };
    User preUser = userRepository.Insert(user);

    var menuRepository = freeSql.GetRepository<Menu>();
    menuRepository.DbContextOptions.EnableCascadeSave = true;

    // 直接插入的话，会导致数据库中添加重复数据
    Menu menu = new Menu()
    {
        MenuName = "菜单管理",
        CreateTime = DateTime.Now,
        /* UserMenuMapInfo = new List<UserMenuMap>()
         {
             new UserMenuMap()
             {
                 UserId = preUser.Id
             }
         }*/
    };
    Menu preMenu = menuRepository.Insert(menu);

    // 直接去维护关系表
    UserMenuMap userMenuMap = freeSql.Select<UserMenuMap>().Where(c => c.UserId == preUser.Id).First();
    userMenuMap.MenuId = preMenu.Id;
    freeSql.Update<UserMenuMap>().SetSource(userMenuMap).ExecuteAffrows();

    // 贪婪加载
    List<User> userList = freeSql.Select<User>().IncludeMany(c => c.UserMencMapInfo).ToList();

    // 延迟加载
    var userList1 = freeSql.Select<User>().ToList();
}

/** 
 * 层级关系 Parent
 **/
{
    var menuRepository = freeSql.GetRepository<Menu>();
    menuRepository.DbContextOptions.EnableCascadeSave = true;

    Menu menu = new Menu()
    {
        MenuName = "菜单管理",
        CreateTime = DateTime.Now,
        Childs = new List<Menu>
        {
            new Menu()
            {
                MenuName= "菜单管理-二级-1",
                CreateTime= DateTime.Now,
                Childs = new List<Menu>()
                {
                    new Menu()
                    {
                        MenuName = "菜单管理-三级",
                        CreateTime = DateTime.Now,
                    }
                }
            },
            new Menu()
            {
                MenuName = "菜单管理-二级-2",
                CreateTime = DateTime.Now,
                Childs = new List<Menu>()
                {
                    new Menu()
                    {
                        MenuName = "菜单管理-三级-1",
                        CreateTime = DateTime.Now,
                    }
                }
            }
        }
    };
    Menu preMenu = menuRepository.Insert(menu);
}
```

## AOP

又叫面向切面编程，是一种编程范式，用于将横切关注点从主要业务逻辑中分离出来，以提高代码的模块化、可维护性和重用性。

- **切点**：程序执行过程中的特定点，例如方法的调用、属性的获取或设置等。
- **通知**：在切点上执行的代码逻辑，包括“前置通知”（Before Advice）、“后置通知”（After Advice）、“异常通知”（After-Throwing Advice）和“返回通知”（After-Returning Advice）等
- **切面**：包含切点和通知的组合，它定义了横切关注点的行为
- **织入**：将切面应用到目标代码中的过程。织入可以在编译时（Compile-Time Weaving）、类加载时（Load-Time Weaving）或运行时（Runtime Weaving）进行

在 FreeSql 中，AOP 功能主要是通过拦截器（Interceptors）实现的。拦截器是一种特殊的组件，它可以在执行数据库操作之前和之后对其进行拦截，从而允许在数据库操作之前或之后执行自定义逻辑。

```c#
/**
 * Insert/Update自动值处理--Aop.AuditValue
 **/
{
    //一般在数据库的设计中，通常包含四个字段：CreateTime CreateUser UpdateTime UpateUser
    freeSql.Aop.AuditValue += (s, e) =>
    {
        if (e.Column.CsName.Equals("CreateTime"))
        {
            e.Value = DateTime.Now;
        }
        Console.WriteLine("AuditValue Finish!");
    };

    Company company = new Company()
    {
        CompanyName = "AuditValue"
    };
    freeSql.Insert(company).ExecuteAffrows();
}

/**
 * CRUD命令执行前后触发--Aop.CurdBefore/Aop.CurdAfter
 **/
{
    // 监控SQL语句的执行效率
    freeSql.Aop.CurdBefore += (s, e) =>
    {
        Console.WriteLine("CurdBefore");
    };
    freeSql.Aop.CurdAfter += (s, e) =>
    {
        Console.WriteLine($"----ManagedThreadId:{Thread.CurrentThread.ManagedThreadId}\n" +
            $"----FullName:{e.EntityType.FullName}\n----ElapsedMilliseconds:{e.ElapsedMilliseconds}ms\n----{e.Sql}");
        if (e.ElapsedMilliseconds > 10)
        {
            Console.WriteLine("语句执行超过10ms");
        }
    };

    Company company = new Company()
    {
        CompanyName = "AuditValue"
    };
    freeSql.Insert(company).ExecuteAffrows();

}
```

## 过滤器

Apply 泛型参数可以设置为任何类型，当使用 Select/Update/Delete 方法时会进行过滤器匹配尝试（try catch）：

- 匹配成功的，将附加 where 条件；
- 匹配失败的，标记下次不再匹配，避免性能损耗；

```c#
freeSql.Aop.AuditValue += (s, e) =>
    {
        if (e.Column.CsName.Equals("CreateTime"))
            e.Value = DateTime.Now;
        if (e.Column.CsName.Equals("Age"))
            e.Value = new Random().Next(15, 50);
        Console.WriteLine("AuditValue Finish!");
    };

    Console.WriteLine("配置过滤器!");
    freeSql.GlobalFilter.Apply<User>("FilterOne", c => c.isDeleted == false);
    freeSql.GlobalFilter.Apply<User>("FilterTwo", c => c.Id > 0);

    freeSql.Insert(new User()
    {
        UserName = "FilterUser",
        isDeleted = false
    }).ExecuteAffrows();

    Console.WriteLine("过滤器生效！");
    List<User> users = freeSql.Select<User>().ToList();
    /* 过滤器设置条件在查询时自动加上
     * SELECT a."Id", a."Age", a."UserName", a."CreateTime", a."UserDetailId", a."CompanyId", a."isDeleted"
        FROM "User" a WHERE (a."isDeleted" = 'f') AND (a."Id" > 0)
    */

    List<Company> companies = freeSql.Select<Company, User>().Where((c, u) => c.Id.Equals(u.CompanyId)).ToList();
    /* 联表查询过滤器一样会启用
     * SELECT a."Id", a."CompanyName", a."CreateTime" FROM "Company" a, "User" b
        WHERE (b."isDeleted" = 'f') AND (b."Id" > 0) AND (a."Id" = b."CompanyId") AND (a."Id" > 0)
     */

    users[0].UserName = "Update-Filter";
    freeSql.Update<User>().SetSource(users[0]).ExecuteAffrows();
    /* 更新操作也会启用过滤器
     * UPDATE "User" SET "Age" = @p_0, "UserName" = @p_1, "CreateTime" = @p_2, "UserDetailId" = @p_3, "CompanyId" = @p_4, 
     "isDeleted" = @p_5 WHERE ("Id" = 1) AND ("isDeleted" = 'f') AND ("Id" > 0)
     */

    int DelRows = freeSql.Delete<User>(users).ExecuteAffrows();
    /* 删除操作也会启用过滤器
     * DELETE FROM "User" WHERE ("Id" IN (2,3,1,4)) AND ("isDeleted" = 'f') AND ("Id" > 0)
     */

    Console.WriteLine("禁用过滤器!");
    freeSql.Select<User>().DisableGlobalFilter("FilterOne").ToList();
    /* 过滤器禁用只在当前语句有效，DisableGlobalFilter()不带参数则表示禁用所有
     * SELECT a."Id", a."Age", a."UserName", a."CreateTime", a."UserDetailId", a."CompanyId", a."isDeleted"
        FROM "User" a WHERE (a."Id" > 0)
     */

    freeSql.GlobalFilter.ApplyIf<User>("FilterThreeOnly", () => 1 == 2, c => c.isDeleted == false);

    freeSql.Select<User>().ToList();
    /* ApplyIf  当第二个参数委托返回值为真过滤器才启用
     * SELECT a."Id", a."Age", a."UserName", a."CreateTime", a."UserDetailId", a."CompanyId", a."isDeleted" FROM "User" a
     */
```

