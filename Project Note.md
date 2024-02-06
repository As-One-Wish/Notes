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

