# 一、Vue前端部分

## 1.项目搭建

- 通过`vue-cli`进行创建

```shell
# 全局安装 vue-cli
npm install -g @vue/cli
# 安装项目
vue create project_name
```

- 通过`npm`创建

```shell
npm init vue@latest
```

在当前目录下创建一个新的项目，并自动安装最新版本的`vue.js`。会生成一个基本的项目结构，但不会提供交互式的项目配置选项。

- 基于`vite`创建

```shell
npm init vite@latest
```

现代化的前端构建工具，会创建一个基本的项目结构，并安装Vite相关的依赖。

## 2.配置`eslint`

```shell
# pnpm创建eslint配置
pnpm create @eslint/config


# 选择eslint干什么？检查语法-发现问题-强制代码风格
How would you like to use ESLint? · style
# 项目模块类型？import/export
What type of modules does your project use? · esm
# 项目框架
Which framework does your project use? · vue
# 是否使用ts
Does your project use TypeScript? · No / Yes
# 运行环境
Where does your code run? · node
# 选择代码风格？自定义
How would you like to define a style for your project? · prompt
# eslint 配置文件格式
What format do you want your config file to be in? · JavaScript
# 缩进类型
What style of indentation do you use? · tab
# 引号类型？ 单or双
What quotes do you use for strings? · single
# 结束符类型？ Unix要好
What line endings do you use? · unix
# 是否使用分号
Do you require semicolons? · No / Yes
```

同时在settings.json文件中添加以下，保证每次保存自动格式化

```json
// 保存时自动格式化
editor.formatOnSave": true,
// 保存时自动修正
"editor.codeActionsOnSave": {
  "source.fixAll.eslint": "explicit"
},
```

可以在vue文件右键选`使用...格式化文档`，将`eslint`配置为默认格式化程序

最终`eslint.js`文件内容

```json
module.exports = {
	'env': {
		'es2021': true,
		'node': true
	},
	'extends': [
		'eslint:recommended',
		'plugin:@typescript-eslint/recommended',
		'plugin:vue/vue3-essential'
	],
	'overrides': [
		{
			'env': {
				'node': true
			},
			'files': [
				'.eslintrc.{js,cjs}'
			],
			'parserOptions': {
				'sourceType': 'script'
			}
		}
	],
	'parserOptions': {
		'ecmaVersion': 'latest',
		'parser': '@typescript-eslint/parser',
		'sourceType': 'module'
	},
	'plugins': [
		'@typescript-eslint',
		'vue'
	],
	'rules': {
		// 缩进类型
		'indent': ['error', 'tab'],
		// 行结束符类型 unix-LF
		'linebreak-style': ['error', 'unix'],
		// 引号类型
		'quotes': ['error', 'single'],
		// 结尾分号
		'semi': ['error','never'],
		// 数组和对象键值对最后一个逗号
		'comma-dangle': ['error', 'never'],
		// 数组对象之间的空格
		'array-bracket-spacing': ['error', 'never'],
		// 中缀操作符之间是否需要空格
		'space-infix-ops': true,
		// 对象字面量中冒号的前后空格
		"key-spacing": [0, { "beforeColon": false, "afterColon": true }],
	}
}

```

更多详细设置：https://blog.csdn.net/Snow_GX/article/details/92089358

## 3.引入`Ant Design Vue`(按需导入)

```shell
# 安装依赖
pnpm install ant-design-vue
# 安装按需导入依赖
pnpm install unplugin-vue-components -D
```

```ts
// vite.config.ts

import Components from 'unplugin-vue-components/vite'
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    Components({
      resolvers: [AntDesignVueResolver({importStyle:'less'})]
    })
  ],
})
```

# 二、.NET后端部分

## 1.DDD 领域驱动设计

- **应用层**-`Application`
  - 负责协调应用程序的任务和流程
  - 不包含具体的业务逻辑
  - 负责将用户输入转换为为领域层的调用

- **领域层**-`Domain`
  - 包含核心业务逻辑
  - 定义领域模型-对业务问题和业务规则的抽象
  - 关注业务核心概念和规则，与具体的技术实现无关
- **基础设施层**-`Infrastructure`
  - 与技术相关代码：数据库访问、文件系统访问等
  - 提供支持应用层和领域层的基础设施服务
- **展现层**-`Presentation`
  - 负责将系统状态呈现给用户，接受用户输入：页面、API断点
  - 不包含业务，通过调用应用层来实现用户请求的处理

![](C:\Files\Notes\Images\ProjectNote-1.png)

## 2..NET Standard和.NET Core

​	最开始.NET Framework只支持Windows，而mono是一个社区的跨平台实现，后来出了个.NET Core跨平台了，但是由于.NET Core和mono、.NET Framework是不同的，虽然mono能跑大部分的.NET Framework程序集，但是.NET Core不行而mono也不能跑.NET Core的程序集，.NET Core也不能跑mono和.NET Framework的程序集。

​	由于.NET对库函数的引用类似动态链接库，程序集内并不包含库函数的实现，只包含库函数的签名，然后运行的时候才去加载对应的有实现的程序集完成“链接”过程最后调用，于是.NET Standard就应运而生了。

​	NET Standard参考三个实现的情况，划定了一组API的子集，这组API在.NET Framework、mono和.NET Core上都有实现，然后使.NET Framework、mono和.NET Core都能加载.NET Standard程序集，这样当用户调用.NET Standard里的API的时候，会把调用转发到当前运行时的基础库的实现上。

​	这样一来，只要用户的代码`基于.NET Standard编写，就能同时在.NET Framework、mono、.NET Core上跑了`。

​	而如果要使用各自平台独有的API的话，则不能基于.NET Standard来编写代码，而需要基于.NET Framework、.NET Core或者mono来编写代码。

​	后来到了.NET Standard 2.1的时候，由于.NET Framework掉了队，不再新增新的功能，于是.NET Standard 2.1干脆不支持.NET Framework了，只支持mono和.NET Core。

​	再后来mono和.NET Core完成了基础库的统一，变成了新的.NET，于是.NET Standard的使命也结束了，只剩下一个统一的.NET。

>.NET Standard一般是用来开发`公用库`用的，兼容性好，能在很多.NET版本中使用
>
>.NET Core类库一般是用于写业务或者应用为主，他的功能相对多些

## 3.AutoInject

```c#
public static IServiceCollection AddScoped(this IServiceCollection services, Type serviceType, Type implementationType);
```

- `serviceType` 参数是服务的类型，即接口或抽象类。
- `implementationType` 参数是实现服务的具体类型。

即将`serviceType`类型的服务注册到服务容器中，在具体实现时使用`implementationType`类型

## 4.IdelBus

空闲对象管理器https://github.com/2881099/IdleBus

## 5.自定义特性类

```c#
/// <summary>
/// 自动注入特性
/// </summary>
[AttributeUsage(AttributeTargets.Class)]
public class AutoInjectAttribute : Attribute
{
    public string Key { get; set; }
    public ServiceLifetime Lifetime { get; set; }

    public AutoInjectAttribute(ServiceLifetime serviceLifetime = ServiceLifetime.Scoped, string key = "default")
    {
        Lifetime = serviceLifetime;
        Key = key;
    }
}
```

用于标识需要进行自动注入的类。

- `AttributeUsage(AttributeTargets.Class)`：
  - 这个特性类被标识为只能应用于类上，也就是说，它是一个类级别的特性

- `public string Key { get; set; }`：
  - 用于获取或设置注入的键（Key），这个键可能用于区分不同的服务
- `public ServiceLifetime Lifetime { get; set; }`：
  - 用于获取或设置服务的生命周期（Lifetime）

> 生命周期表示注入的服务在容器中的存活时间，可以是 Singleton（单例）、Scoped（作用域）、Transient（瞬时）等

在使用依赖注入容器进行服务注册时，通过解析这个特性，可以按照指定的键和生命周期进行服务的注册，例如：

```C#
[AutoInject(Key = "appService", Lifetime = ServiceLifetime.Singleton)]
public class MyApplicationService
{
    // ...
}
```

## 6.AutoMapper

- **AutoMapper的注入**

```c#
/// <summary>
/// AutoMapper 配置
/// </summary>
public static class AutoMapperConfig
{
    /// <summary>
    /// AutoMapper注入
    /// </summary>
    /// <param name="services"></param>
    /// <exception cref="ArgumentNullException"></exception>
    public static void AddAutoMapperConfiguration(this IServiceCollection services)
    {
        if (services == null) throw new ArgumentNullException(nameof(services));

        var autoMapperProfileList = GetAutoMapperProfiles();
        // 注入AutoMapper策略
        MapperConfiguration config = new MapperConfiguration(cfg =>
        {
            foreach (var profile in autoMapperProfileList)
                cfg.AddProfile(profile);
        });

        IMapper mapper = config.CreateMapper();
        services.AddSingleton(mapper);
    }

    /// <summary>
    /// 利用反射获取AutoMapper类型
    /// </summary>
    /// <returns></returns>
    public static IEnumerable<Type> GetAutoMapperProfiles()
    {
        var assemblies = Directory.GetFiles(AppDomain.CurrentDomain.BaseDirectory, "Info.*.AutoMapper.dll").Select(Assembly.LoadFrom).ToList();
        if (assemblies != null && assemblies.Count > 0)
        {
            List<Type> types = assemblies.Where(d => d.FullName != null && d.FullName.Split(',')[0].EndsWith("AutoMapper"))
                .SelectMany(x => x.GetTypes())
                .Where(t => t.IsClass && !t.IsAbstract && (t.BaseType?.FullName == "AutoMapper.Profile"))
                .ToList();
            return types;
        }
        else
            return Enumerable.Empty<Type>();
    }
}
```

- **AutoMapper的编辑和使用**

https://www.cnblogs.com/gl1573/p/13098031.html

## 7.Yitter/IdGenerator

雪花算法中的数字ID生成器

https://github.com/yitter/IdGenerator?tab=readme-ov-file

> 雪花算法：
>
> - 是一种生成分布式全局唯一ID的算法，生成的ID称为`Snowflake IDs`或`snowflakes`
> - 一个Snowflake ID有64比特。前41位是时间戳，表示了自选定的时期以来的毫秒数。 接下来的10位代表计算机ID，防止冲突。其余12位代表每台机器上生成ID的序列号，这允许在同一毫秒内创建多个Snowflake ID。最后以十进制将数字序列化。

## 8.Jwt认证及授权

- `Authentication`(认证):标识用户的身份，一般发生在登录的时候
- `Authorization`(授权):授予用户权限，指定用户能访问哪些资源；在认证之后

![](C:\Files\Notes\Images\ProjectNote-2.png)

[ASP.NET Core 实战：基于 Jwt Token 的权限控制全揭露 - 掘金 (juejin.cn)](https://juejin.cn/post/6844903769457557512?share_token=e15b4494-5777-44d4-bf9a-b9867554c8c0)

### 8.1 生成Token令牌

$$
Jwt{\quad}Token
\begin{cases}
A-header头信息
\begin{cases}
alg:加密算法名称\\
typ:JWT
\end{cases}\\
B-payload有效载荷
\begin{cases}
iss:发行者\\
exp:到期时间\\
sub:主题\\
aud:受众
\end{cases}\\
C-Signature签名:
是服务器验证传递数据是否有效安全的标准
\end{cases}
$$

- **添加基本配置**

```json
/* appsettings.json */
"Jwt": {
  "SecurityKey": "8602e9953dbe0ae4b82f49bbee9a6fc4", // 大于16位
  "Issuer": "Info.Storage",
  "Audience": "Info.Storage",
  "ExpireMinutes": "10080"
},
```

- **创建Jwt Token**

```c#
public JwtAuthorizationDto CreateJwt(JwtUserDto jwtUserDto)
{
    try
    {
        // 创建 JwtSecurityTokenHandler 实例，用于处理 JWT 操作
        JwtSecurityTokenHandler jwtSecurityTokenHandler = new JwtSecurityTokenHandler();
        SymmetricSecurityKey key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:SecurityKey"]!));

        DateTime authTime = DateTime.Now;
        DateTime expireTime = authTime.AddMinutes(Convert.ToDouble(_configuration["Jwt:ExpireMinutes"]));

        // 将用户信息添加到Claim(声明)中
        List<Claim> claims = new List<Claim>()
        {
            new Claim("userId",jwtUserDto.UserId.ToString()),
            new Claim("userName",jwtUserDto.UserName),
            new Claim("roleId",jwtUserDto.RoleId.ToString()),
            new Claim("account",jwtUserDto.Account),
            new Claim("roleName", jwtUserDto.RoleName)
        };
        // 签发一个加密后的用户信息凭证，用来标识用户身份
        SecurityTokenDescriptor tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims), // 创建声明信息
            Issuer = _configuration["Jwt:Issuer"], // Jwt token 的签发者
            Audience = _configuration["Jwt:Audience"], // Jwt token 的接受者
            Expires = expireTime, // 过期时间
            SigningCredentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256), // 创建 token
        };

        var token = jwtSecurityTokenHandler.CreateToken(tokenDescriptor);
        // 存储token信息
        var jwt = new JwtAuthorizationDto
        {
            Token = jwtSecurityTokenHandler.WriteToken(token),
            AuthTime = new DateTimeOffset(authTime).ToUnixTimeSeconds(),
            ExpireTime = new DateTimeOffset(expireTime).ToUnixTimeSeconds(),
        };
        return jwt;
    }
    catch (Exception ex)
    {
        LogHelper.Error(ex);
        return default;
    }
}
```

### 8.2 Jwt配置

包含授权方案设置：基于角色授权、基于声明授权、基于策略自定义授权

https://cloud.tencent.com/developer/article/1498360

```C#
/* JwtConfig.cs */
public static void AddJwtConfiguration(this IServiceCollection services, IConfiguration configuration)
{
    if (services == null) throw new ArgumentNullException(nameof(services));
    // 从配置中获取对应信息
    string? issuer = configuration["Jwt:Issuer"]; // 颁发者
    string? audience = configuration["Jwt:Audience"]; // 受众
    string? expire = configuration["Jwt:ExpireMinutes"]; // 过期时间
    string? securityKey = configuration["Jwt:SecurityKey"]; // 安全密钥
    // 配置信息转换
    TimeSpan expiration = TimeSpan.FromMinutes(Convert.ToDouble(expire));
    SecurityKey key = new SymmetricSecurityKey(securityKey == null ? new byte[0] : Encoding.UTF8.GetBytes(securityKey));
    // Jwt相关配置
    services.AddAuthorization(options => // 1.配置授权策略
    {
        options.AddPolicy("Policy.Default", policy => policy.Requirements.Add(new PolicyRequirement("Default")));
        options.AddPolicy("Policy.Admin", policy => policy.Requirements.Add(new PolicyRequirement("Admin")));
    }).AddAuthentication(s =>  // 2.配置身份验证
    {
        // 在身份验证成功后，默认使用的身份验证方案
        s.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        // 在请求处理过程中使用的默认身份验证方案
        s.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
        // 在发生身份验证挑战时使用的默认身份验证方案
        s.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    }).AddJwtBearer(s => // 3.配置Jwt Bearer (Token的鉴权逻辑)
    {
        // 配置 Jwt Bearer 的验证参数
        s.TokenValidationParameters = new TokenValidationParameters
        {
            ValidIssuer = issuer, // 颁发者的有效值
            ValidAudience = audience, // 受众的有效值
            IssuerSigningKey = key, // 用于验证签名的密钥
            ClockSkew = expiration, //允许的时钟偏差
            ValidateLifetime = true, // 是否验证令牌的生命周期
            RequireExpirationTime = true, // 是否要求令牌包含有效期
            ValidateIssuer = true, // 是否验证Issuer
            ValidateAudience = true, // 是否验证Audience
            ValidateIssuerSigningKey = true // 是否验证SecurityKey
        };
        // 配置 Jwt Bearer 的事件处理器
        s.Events = new JwtBearerEvents
        {
            // 收到消息时的触发的事件
            OnMessageReceived = context =>
            {
                if (context.Request.Method == "GET" && !string.IsNullOrWhiteSpace(context.Request.Query["token"]))
                    context.Token = context.Request.Query["token"];
                return Task.CompletedTask;
            },
            // 当身份验证失败时触发的事件
            OnAuthenticationFailed = context =>
            {
                // 令牌过期异常
                if (context.Exception.GetType() == typeof(SecurityTokenExpiredException))
                {
                    context.Response.StatusCode = 200;
                    context.Response.Headers.Append("WWW-AUthenticate", "Bearer error=\"token_refreshed\"");
                }

                return Task.CompletedTask;
            }
        };
    });

    services.AddSingleton<IAuthorizationHandler, PolicyHandler>();
}
```

- **策略声明**

```C#
public class PolicyRequirement : IAuthorizationRequirement
{
    public string RequiredRole { get; }

    public PolicyRequirement(string requiredRole)
    {
        RequiredRole = requiredRole;
    }
}
```

### 8.3 Jwt校验策略

```c#
public class PolicyHandler : AuthorizationHandler<PolicyRequirement>
{
    /// <summary>
    /// 授权方式(cookie, bearer, oauth, openid)
    /// </summary>
    public IAuthenticationSchemeProvider schemes { get; set; }

    /// <summary>
    /// 构造函数
    /// </summary>
    /// <param name="schemes"></param>
    public PolicyHandler(IAuthenticationSchemeProvider schemes)
    {
        this.schemes = schemes;
    }

    protected override async Task HandleRequirementAsync(AuthorizationHandlerContext context, PolicyRequirement requirement)
    {
        HttpContext? httpContext = context.Resource as HttpContext;
        var defaultAuthenticate = await schemes.GetDefaultAuthenticateSchemeAsync();
        if (defaultAuthenticate != null && httpContext != null)
        {
            // 验证签发的用户信息
            var result = await httpContext.AuthenticateAsync(defaultAuthenticate.Name);
            if (result.Succeeded)
            {
                string? roleNameClaim = context.User.Claims.FirstOrDefault(claim => claim.Type == "roleName")?.Value;
                if (!string.IsNullOrWhiteSpace(roleNameClaim) && roleNameClaim == requirement.RequiredRole || requirement.RequiredRole == "Default")
                    context.Succeed(requirement);
                else
                {
                    if (!httpContext.Response.Headers.ContainsKey("WWW-Authenticate"))
                    {
                        httpContext.Response.Headers.Append("WWW-Authenticate", "Bearer error=\"invalid_permission\"");
                    }
                    context.Fail();
                }
            }
            else
            {
                httpContext.Response.Headers.Append("WWW-Authenticate", "Bearer error=\"invalid_token\"");
                context.Fail();
            }
        }
    }
}
```

### 8.4 梳理

​	实际上整个流程就是，首先利用登录的用户获取其相关信息，基于此和配置文件创建Token；在Jwt的配置中设置授权策略，同时说明对于token的验证方式；通过实现Handler，自定义当识别到接口的授权时，对对应方案进行自定义校验。

## 9.数据库时区问题

时间数据插入数据库由于时区问题会自动少8个小时

```postgresql
set timezone='Asia/Shanghai';
```

