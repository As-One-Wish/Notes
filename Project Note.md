# 一、Vue前端部分

## 1.基础项目搭建

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

## 2.Vben Admin模板项目搭建

- 获取项目代码

```shell
git clone https://github.com/anncwb/vue-vben-admin.git
```

- 安装依赖

```shell
cd vue-vben-admin
git checkout thin # Important！！
pnpm install
```

- 运行

```shell
pnpm serve
```

## 3.配置`eslint`

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
	env: {
		es2021: true,
		node: true,
		browser: true
	},
	extends: [
		'eslint:recommended',
		'plugin:@typescript-eslint/recommended',
		'plugin:vue/vue3-essential'
	],
	overrides: [
		{
			env: {
				node: true
			},
			files: ['.eslintrc.{js,cjs}'],
			parserOptions: {
				sourceType: 'script'
			}
		}
	],
	parserOptions: {
		ecmaVersion: 'latest',
		parser: '@typescript-eslint/parser',
		sourceType: 'module',
		ecmaFeatures: {
			jsx: true
		}
	},
	plugins: ['@typescript-eslint', 'vue'],
	rules: {
		// 缩进类型
		indent: ['error', 'tab'],
		// 行结束符类型 windows-CRLF
		'linebreak-style': ['error', 'windows'],
		// 引号类型
		quotes: ['error', 'single'],
		// 结尾分号
		semi: ['error', 'never'],
		// 数组和对象键值对最后一个逗号
		'comma-dangle': ['error', 'never'],
		// 数组对象之间的空格
		'array-bracket-spacing': ['error', 'never'],
		// 中缀操作符之间是否需要空格
		'space-infix-ops': ['error'],
		// 对象字面量中冒号的前后空格
		'key-spacing': ['error', { beforeColon: false, afterColon: true }],
		// clear eslintvue/comment-directive 错误解决
		'vue/comment-directive': 'off',
		// 设置html缩进为两个
		'vue/html-indent': ['error', 'tab'],
		// 配置ts参数声明类型格式冒号前后空格
		'@typescript-eslint/type-annotation-spacing': [
			'error',
			{
				before: false,
				after: true,
				overrides: {
					arrow: { before: true, after: true }
				}
			}
		],
		// 不能有多余的空格
		'no-multi-spaces': ['error'],
		// 不能有不规则的空格
		'no-irregular-whitespace': ['error'],
		//生成器函数*的前后空格
		'generator-star-spacing': ['error'],
		// 关闭 ts any 类型警告
		'@typescript-eslint/no-explicit-any': ['off'],
		// 允许 {} 的使用
		'@typescript-eslint/ban-types': [
			'error',
			{
				extendDefaults: true,
				types: {
					'{}': false,
					Object: false
				}
			}
		],
		// 关闭禁用未声明的变量
		'no-undef': 'off',
		// 关闭组件命名规则
		'vue/multi-word-component-names': 'off',
		// 非空代码块
		'no-empty': 'off'
	}
}

```

更多详细设置：https://blog.csdn.net/Snow_GX/article/details/92089358

## 4.tsconfig.json配置

```json
{
    "compilerOptions": {
    // 设置编译后的 JavaScript 代码目标版本为 ESNext
    "target": "ESNext",
    // 指定生成的模块系统类型为 ESNext 模块
    "module": "ESNext",
    // 指定模块解析策略为 Node.js 模块解析
    "moduleResolution": "Node",
    // 不从默认库文件（lib.d.ts）中排除任何类型
    "noLib": false,
    // 开启所有严格的类型检查选项
    "strict": true,
    // 允许编译 JavaScript 文件
    "allowJs": true,
    // 允许导入 JSON 文件作为模块
    "resolveJsonModule": true,
    // 允许导入没有默认导出的模块时，使用默认导出
    "allowSyntheticDefaultImports": true,
    // 生成源映射文件
    "sourceMap": true,
    // 启用 ES 模块导入和导出的Interop
    "esModuleInterop": true,
    // 检查未使用的局部变量
    "noUnusedParameters": true,
    // 检查未使用的函数参数
    "noUnusedLocals": true,
    // 跳过对声明文件的检查
    "skipLibCheck": true,
    // 移除编译后的 JavaScript 文件中的注释
    "removeComments": true,
    // 不允许隐式的any类型
    "noImplicitAny": false,
    // 保留JSX代码，不进行转换
    "jsx": "preserve",
    // 关闭对函数参数的严格检测
    "strictFunctionTypes": false,
    // 启用实验性的装饰器支持
    "experimentalDecorators": true,
    "baseUrl": ".",
    "declaration": false,
    "types": [
      "vite/client"
    ],
    "paths": {
      "@/*": [
        "src/*"
      ],
      "#/*": [
        "types/*"
      ]
    }
  },
  "include": [
    "tests/**/*.ts",
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "types/**/*.d.ts",
    "types/**/*.ts",
    "build/**/*.ts",
    "build/**/*.d.ts",
    "mock/**/*.ts",
    "vite.config.ts"
  ],
  "exclude": ["node_modules", "tests/server/**/*.ts", "dist", "**/*.js"]
}

```

## 5.引入`Ant Design Vue`(按需导入)

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

## 6.Hooks

基于组合式API，允许站在组件的逻辑代码中更好的组织和复用代码；本质上是一组可复用的函数，可以钩入Vue组件的生命周期。

**定义：**

```ts
import { ref, onMounted, onUnmounted } from 'vue';

export function useMousePosition() {
  const x = ref(0);
  const y = ref(0);

  function updatePosition(event) {
    x.value = event.clientX;
    y.value = event.clientY;
  }

  onMounted(() => {
    window.addEventListener('mousemove', updatePosition);
  });

  onUnmounted(() => {
    window.removeEventListener('mousemove', updatePosition);
  });

  return { x, y };
}
```

**使用：**

```vue
<template>
  <div>
    Mouse position: X={{ x }}, Y={{ y }}
  </div>
</template>

<script setup>
import { useMousePosition } from './useMousePosition';

const { x, y } = useMousePosition();
</script>
```

## 7.路径别名

`vite.config.ts`

```ts
export default defineConfig({
	plugins: [
		vue(),
		...
	],
	resolve: {
		alias: {
			'@': resolve(__dirname, 'src'),
			'#': resolve(__dirname, 'types')
		}
	}
})
```

`tsconfig.json`

```json
{  
  "compilerOptions": {
    "baseUrl": ".", // 必须
    "paths": {
      "@/*": ["src/*"],
      "#/*": ["types/*"]
    }
  }
}
```

## 8.import.meta.glob

vite独有的功能，利用`import.meta.glob()`直接导入所有模块，但使用前要进行类型定义

**`tsconfig.json`**

```json
{
  "compilerOptions": {
    "types": ["vite/client"]
  }
}
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

[领域基元](https://zhuanlan.zhihu.com/p/340911587)

[应用架构](https://zhuanlan.zhihu.com/p/343388831)

### 1.1 Domain Primitive

在一个特定领域里，拥有精准定义的、可自我验证的、拥有行为的Value Object

- **将隐性的概念显性化** -- Make Implicit Concepts Explicit
- **将隐性的上下文显性化** -- Make Implicit Context Explicit
- **封装多对象行为** -- Encapsulate Multi-Object Behavior

### 1.2 实现业务逻辑三种方式

#### 1.2.1 事务脚本

思路同**面向过程编程**一样，要达到目标，需要哪些步骤，然后一步步操作

#### 1.2.2 贫血模型

只包含数据，不包含业务逻辑的类（Entity），就叫做贫血模型（Anemic Domain Model）。其具体的业务逻辑集中在Service中，将数据和操作分开，破坏了面向对象的封装特性，是一种典型的面向过程的编程风格

#### 1.2.3 充血模型

数据和对应的业务逻辑被封装在同一个类中，典型的面向对象编程风格。

> 在基于贫血模型的传统开发中，Service层包含Service类和BO类两部分，BO是贫血模型，只包含数据，不包含具体的业务逻辑，业务逻辑集中在Service类中；
>
> 在基于充血模型的DDD开发模式中，Service层包含Service类和Domain类两部分，Domain就相当于贫血模型中的BO，区别在于Domain既包含数据，也包含业务逻辑
>
> **基于贫血模型的传统开发，重Service轻BO；基于充血模型的DDD开发模式，轻Service重Domain**

### 1.3 Anti-corruption Layer

**防腐层**，一个上下文通过一些适配和转换与另一个上下文交互。

![](C:\Files\Notes\Images\ProjectNote-3.png)

是一种在不同应用间转换的机制。创建一个防腐层，以根据客户端自己的领域模型为客户提供功能；该层通过其现有接口与另一个系统通信，几乎不需要对其进行任何修改。

在不共享相同领域模型的不同子系统之间实施防腐层（或外观或适配器层），此层转换一个子系统向另一个子系统发出的请求。使用防腐层模式可确保应用程序的设计不受限于对外部子系统的依赖。

因此，防腐层隔离不仅是为了自身领域模型免受其他领域模型的代码的侵害，还在于分离不同的域并确保它们在将来保持分离。

实际开发中提供的功能：

- **适配器**：通过适配器模式，可以将数据转化逻辑封装到ACL内部，降低对业务代码的侵入
- **缓存**：对于频繁调用且数据变更不频繁的外部依赖，通过在ACL里嵌入缓存逻辑，能够有效的降低对于外部依赖的请求压力
- **兜底**：如果外部依赖的稳定性较差，一个能够有效提升我们系统稳定性的策略是通过ACL起到兜底的作用，比如当外部依赖出问题后，返回最近一次成功的缓存或业务兜底数据
- **易于修改**：类似于之前的Repository，ACL的接口类能够很容易的实现Mock或Stub，以便于单元测试
- **功能开关**

## 2..NET Standard和.NET Core

最开始.NET Framework只支持Windows，而mono是一个社区的跨平台实现，后来出了个.NET Core跨平台了，但是由于.NET Core和mono、.NET Framework是不同的，虽然mono能跑大部分的.NET Framework程序集，但是.NET Core不行而mono也不能跑.NET Core的程序集，.NET Core也不能跑mono和.NET Framework的程序集。

由于.NET对库函数的引用类似动态链接库，程序集内并不包含库函数的实现，只包含库函数的签名，然后运行的时候才去加载对应的有实现的程序集完成“链接”过程最后调用，于是.NET Standard就应运而生了。

NET Standard参考三个实现的情况，划定了一组API的子集，这组API在.NET Framework、mono和.NET Core上都有实现，然后使.NET Framework、mono和.NET Core都能加载.NET Standard程序集，这样当用户调用.NET Standard里的API的时候，会把调用转发到当前运行时的基础库的实现上。

这样一来，只要用户的代码`基于.NET Standard编写，就能同时在.NET Framework、mono、.NET Core上跑了`。

而如果要使用各自平台独有的API的话，则不能基于.NET Standard来编写代码，而需要基于.NET Framework、.NET Core或者mono来编写代码。

后来到了.NET Standard 2.1的时候，由于.NET Framework掉了队，不再新增新的功能，于是.NET Standard 2.1干脆不支持.NET Framework了，只支持mono和.NET Core。

再后来mono和.NET Core完成了基础库的统一，变成了新的.NET，于是.NET Standard的使命也结束了，只剩下一个统一的.NET。

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

实际上整个流程就是，首先利用登录的用户获取其相关信息，基于此和配置文件创建Token；在Jwt的配置中设置授权策略，同时说明对于token的验证方式；通过实现Handler，自定义当识别到接口的授权时，对对应方案进行自定义校验。

## 9.数据库时区问题

时间数据插入数据库由于时区问题会自动少8个小时

```postgresql
set timezone='Asia/Shanghai';
```

## 10.DRY、KISS、YAGNI三原则

适用于软件设计各个层面

- `DRY `- Don’t Repeat Yourself

**不要总是用相同代码解决相同问题**。尽量在项目中减少重复的代码行、方法、模块。

- `KISS `- Keep It Simple & Stupid

**让代码简单直接**

- `YAGNI `- You Ain’t Gonna Need It

**适可而止**。只有在需要的时候才去添加额外的功能，不要过度设计

## 11 各类对象相关概念

- `DAO`（Data Access Object） **数据访问对象**

是一个数据访问接口，主要是与数据库交互

对照Infrastructure.Core -> Repository -> Entities

- `DTO`（Data Transfer Object） **数据传输对象**

一种设计模式，用于在软件应用程序的子系统或层之间传输数据

通常在数据访问层、业务逻辑层和表示层之间传输数据

主要目的是封装数据并在系统的不同部分之间高效传输

通常只包含数据字段和访问器（获取器和设置器）的方法，但不包含任何业务逻辑

对照Infrastructure.Core -> Entity

- `DO`（Domain Object） **领域对象**

从现实世界中抽象出来的有形或无形的业务实体

反映了业务领域的结构和行为，通常包含`业务逻辑`和`行为方法`

对照Domain -> Object

- `VO`（View Object） **视图对象**

用于展示层， 把某个指定页面（或组件）的所有数据封装起来

如果一个DTO对应一个VO则，DTO=VO；如果一个DTO对应多个VO，则展示层需要把VO转换为服务层对应方法所要求的DTO，传递给服务层

- `AO`（Application Object） **应用对象**

不是一个普遍使用的术语

在应用程序中承担重要角色的对象，但具体含义可能会因上下文而异

- `BO`（Business Object） **业务对象**

用于表示业务领域中的实体和概念的对象

是对业务需求的抽象和建模，代表了业务逻辑层中的实体和行为

> DO **VS** BO
>
> - 区别
>
>   - DO：表示领域中的实体或概念，是DDD中的一部分，用于建模领域的概念和规则；DO反映了约领域的结构和行为，通常用于表示业务逻辑层和数据访问层之间的数据结构
>   - BO：主要关注业务逻辑和业务规则的实现，是DDD中的一部分，代表了业务领域中的实体和概念；BO负责实现业务规则和行为，通常与业务逻辑密切相关
>
> - 联系：
>
>   两者都是表示业务领域中的实体和概念，在一定程度上是相互关联的；业务对象通常是领域对象的一种具体实现或者衍生物，它们可以直接映射到领域对象，或者是基于领域对象构建的

- `PO`（Persistent Object） **持久化对象**

在软件开发中用于数据持久化存储的对象；

通常与数据库中的表或文档等数据存储结构相对应，用于将应用程序中的数据持久化倒持久化存储介质中，或从持久化存储中检索数据

> 特征：
>
> - **映射到数据存储结构**
> - **持久化操作**
> - **与业务逻辑的解耦**：通常与业务逻辑对象相分离，以实现数据存储和业务逻辑的解耦

- `Entity`（概念实体模型）**实体类和模型**
- `View`（概念视图模型） **视图模型**

## 12.设计模式

**创建型模式**

> 用来解决对象实例化和使用的客户端耦合的模式，可以让客户端和对象实例化都独立变化，做到相互不影响

- [单例模式](https://blog.csdn.net/sinat_40003796/article/details/125594207)
- [简单工厂模式](https://blog.csdn.net/sinat_40003796/article/details/125558524)
- [工厂方法模式](https://blog.csdn.net/sinat_40003796/article/details/125848004)
- [抽象工厂模式](https://blog.csdn.net/sinat_40003796/article/details/125599352)
- [建造者模式](https://blog.csdn.net/sinat_40003796/article/details/125597300)
- [原型模式](https://blog.csdn.net/sinat_40003796/article/details/125622374)

**结构型模式**

> 主要研究的是类和对象的组合的问题；包括两种类型，一是类结构型模式：指的是采用继承机制来组合实现功能；二是对象结构性模式：指的是通过组合对象的方式来实现新的功能

- [适配器模式](https://blog.csdn.net/sinat_40003796/article/details/125635861) -- 注重转换接口，将不吻合的接口适配对接
- [桥接模式](https://blog.csdn.net/sinat_40003796/article/details/125640595) -- 注重分离接口与其实现，支持多维度变化

- [装饰模式](https://blog.csdn.net/sinat_40003796/article/details/125714980) -- 注重稳定接口，在此前提下为对象扩展功能
- [组合模式](https://blog.csdn.net/sinat_40003796/article/details/125723161) -- 注重统一接口，将“一对多”关系转化为“一对一”关系
- [外观模式](https://blog.csdn.net/sinat_40003796/article/details/125725068) -- 注重简化接口，简化组件系统与外部客户程序的依赖关系
- [享元模式](https://blog.csdn.net/sinat_40003796/article/details/125735655) -- 注重保留接口， 在内部使用共享技术对对象存储进行优化
- [代理模式](https://blog.csdn.net/sinat_40003796/article/details/125742301) -- 注重假借接口，增加间接层来实现灵活控制

**行为型模式**

> 主要讨论的是在不同对象之间划分责任和算法的抽象化的问题：
>
> --类的行为模式：使用继承关系在几个类之间分配行为
>
> --对象的行为模式：使用对象聚合的方式来分配行为

- [模板方法模式](https://blog.csdn.net/sinat_40003796/article/details/125756389)
- [命令模式](https://blog.csdn.net/sinat_40003796/article/details/125760071)
- [迭代器模式](https://star-302.blog.csdn.net/article/details/125762129)
- [观察者模式](https://blog.csdn.net/sinat_40003796/article/details/125778359)
- [中介者模式](https://blog.csdn.net/sinat_40003796/article/details/125780496)
- [状态模式](https://blog.csdn.net/sinat_40003796/article/details/125782291)
- [策略模式](https://blog.csdn.net/sinat_40003796/article/details/125797589)
- [职责链模式](https://blog.csdn.net/sinat_40003796/article/details/125799083)
- [访问者模式](https://blog.csdn.net/sinat_40003796/article/details/125840720)
- [备忘录模式](https://blog.csdn.net/sinat_40003796/article/details/125841880)
- [解释器模式](https://blog.csdn.net/sinat_40003796/article/details/125844433)

## 13.AutoUpdater.NET

[AutoUpdater.NET](https://github.com/ravibpatel/AutoUpdater.NET?tab=readme-ov-file)是一个类库，可将自动更新功能添加到桌面应用程序项目中。

### 13.1 **工作原理**

- AutoUpdater.NET从服务器获取包含更新信息的`xml`文件

- 当最新版本大于当前安装的版本，就会显示更新对话框
- 如果确认更新，则会根据xml文件中的路由下载更新文件进行安装

### 13.2 xml文件相关参数

- `version`:**必须**，提供应用程序最新版本（`X.X.X.X`）
- `url`:**必须**，提供最新应用程序的安装文件/压缩文件的路径
- `changelog`:应用程序更改日志的路径
- `mandatory`:强制性，将忽略"稍后提醒"和"跳过"选项
  - `mode`:模式，1-隐藏对话框关闭按钮，2-自动开始下载和更新应用程序
  - `minVersion`:当应用程序安装版本小于此处指定的最小版本时，才会触发强制选项
- `executable`:可执行文件路径
- `args`:为Installer提供命令行参数
- `checksum`:提供更新文件校验和，以检查文件完整性

### 13.3 应用实例

**开启本机IIS服务器**

![](C:\Files\Notes\Images\ProjectNote-4.png)

**创建更新相关文件**

![](C:\Files\Notes\Images\ProjectNote-5.png)

**更新配置文件**

```xml
<?xml version='1.0' encoding="UTF-8"?>
<item>
    <!--在版本标记之间提供应用程序的最新版本。版本必须为X.X.X.X格式。-->
    <version>1.0.0.2</version>
    <!--在url标签之间提供最新版本安装程序文件或zip文件的url。自动更新。NET下载这里提供的文件，并在用户按下Update按钮时安装它。-->
    <url>http://127.0.0.1/Downloads/TestDownload.zip</url>
    <!--在changelog标记之间提供应用程序更改日志的URL。如果你不提供变更日志的URL，那么更新对话框将不会显示变更日志。-->
    <changelog>http://127.0.0.1/Updates/UpdateLog.html</changelog>
    <!--如果你不想让用户跳过这个版本，可以将其设置为true。这将忽略“稍后提醒”和“跳过”选项，并在更新对话框中隐藏“稍后提醒”和“跳过”按钮。-->
    <mandatory>false</mandatory>
    <!--提供更新文件的校验和。如果你做这个autotoupater。NET将在执行更新过程之前比较下载文件的校验和，以检查文件的完整性。
    您可以在校验和标记中提供algorithm属性，以指定应该使用哪个算法来生成下载文件的校验和。目前支持MD5、SHA1、SHA256、SHA384和SHA512。-->
    <checksum algorithm="MD5">Update file Checksum</checksum>
</item>
```

**Winform程序配置**

首先安装`Autoupdater.NET.Official`包

```c#
using AutoUpdaterDotNET;
using System;
using System.Globalization;
using System.Reflection;
using System.Threading;
using System.Windows.Threading;

namespace AutoUpdateNET
{
    public partial class Form1 : DevExpress.XtraEditors.XtraForm
    {
        public Form1()
        {
            InitializeComponent();
            Assembly assembly = Assembly.GetEntryAssembly();
          	// 显示版本号
            label1.Text = $"Current Version : {assembly.GetName().Version}";
          	// 将当前线程的文化设置为特定文化,这会影响日期格式、货币格式以及 UI 使用的语言
            Thread.CurrentThread.CurrentCulture = Thread.CurrentThread.CurrentUICulture = CultureInfo.CreateSpecificCulture("zh");
            AutoUpdater.ShowRemindLaterButton = true;
          	// 设定计时器
            DispatcherTimer timer = new DispatcherTimer { Interval = TimeSpan.FromSeconds(3) };
            timer.Tick += delegate { AutoUpdater.Start("http://127.0.0.1/Updates/AutoUpdaterStarter.xml"); };
            timer.Start();
        }

        private void button1_Click(object sender, EventArgs e)
        {
            AutoUpdater.Start("http://127.0.0.1/Updates/AutoUpdaterStarter.xml");
        }
    }
}
```

