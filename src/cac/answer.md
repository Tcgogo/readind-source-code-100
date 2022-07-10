### 目录分析
```
cac
├─ .editorconfig
├─ .gitattributes
├─ .github
│    ├─ FUNDING.yml
│    └─ ISSUE_TEMPLATE.md
├─ .gitignore
├─ .prettierrc
├─ LICENSE
├─ README.md
├─ circle.yml
├─ examples
├─ index-compat.js
├─ jest.config.js
├─ mod.js
├─ mod.ts
├─ mod_test.ts
├─ package.json
├─ rollup.config.js
├─ scripts
│    └─ build-deno.ts
├─ src
│    ├─ CAC.ts
│    ├─ Command.ts
│    ├─ Option.ts
│    ├─ __test__
│    ├─ deno.ts
│    ├─ index.ts
│    ├─ node.ts
│    └─ utils.ts
├─ tsconfig.json
└─ yarn.lock
```
- .editorconfig: 维护一致的编码风格的配置文件
- .gitattributes: 
  - 定义文件的属性，主要属性有：
  - text：用于控制行尾的规范性。
  - eol：设置行末字符。（协同开发是容易出现行末字符不一致的情况，主要是git拉取代码时行尾规范的不同，导致代码强制转换，从而导致eslint报红）
    - eol=lf ，[回车] ：入库时将行尾规范为LF，检出时行尾不强制转换为 CRLF
    - eol=crlf，[换行、回车] ：入库时将行尾规范为LF，检出时将行尾转换为CRLF
  - diff：告诉 git 声明文件是否需要比较版本差异。
    - diff，强制视为文本文件
    - !diff，表示为非文本文件
    - 未定义
- .github: github 配置文件
  - `FUNDING.yml`: github 赞助配置文件
  - `ISSUE_TEMPLATE.yaml`: github yaml 模板配置文件
- .gitignore: 配置此文件可以让 git 对某些特定文件不追踪变化
- .prettierrc: prettier 的配置文件
- README：项目介绍文件
- circle.yaml: CircleCI 的配置文件
- examples: 示例
- index.compat.js: 主入口，主要是为了兼容 cjs
- jest.config.js: jest 配置文件
- mod.\*: 兼容 deno
- package.json: npm 规范的项目描述文件
- rollup.config.js: rollup 配置文件（主要是打包用的）
- tsconfig.json: TS 配置文件
- scripts: 项目中用到的脚本
- src: 项目主目录
- yarn.lock: yarn 的 filelock

### 分析一下 package.json 里面的字段都是干嘛的
- version：版本号
- name：包的名称
- description： 描述信息
- keywords：关键字
- author：作者
- homePage：官网地址
- main：指定了程序的主入口文件（ CommandJS导入），默认加载根目录的index.js文件（通过` require("moduleName") `加载）
- module：指定了程序的主入口文件（ESModule导入）
- scripts：脚本命令
- repository：npm包托管的地方，对于想贡献代码的人是有帮助的。
- types：为js文件指定主声明文件，如果主声明文件名是index.d.ts并且位置在包的根目录里（与index.js并列），就不需要使用`types`属性指定了。
- exports：允许你通过引用自己的 package name来定义 package 的入口文件, 对于所有在 `exports` 中定义的路径都必须是绝对路径。即 ./ 的形式。
  - 当一个 package 同时拥有 `exports` 和 `main` 字段时，在被以 package name 方式导入时，exports 的优先级较高。
- files：npm包作为依赖安装时要包括的文件，格式是文件正则的数组，["*"]代表所有文件。也可以使用 `npmignore` 来忽略个别文件。 `files`字段优先级最大，不会被 `npmignore` 和 `.gitignore` 覆盖。
- engine：表示你的项目所运行的node版本
- license：指定许可证
- devDependencies：npm包所依赖的构建和测试相关的npm包，放置到`devDependencies`，当使用 `npm install` 下载该包时，devDependencies中指定的包不会一并被下载。
- peerDependencies：指定npm包与主npm包的兼容性，当开发插件时是需要的
- engines：指定npm包可以使用的Node版本
- release：npm仓库主分支
- config：用来设置一些用于npm包的脚本命令会用到的配置参数。
- lint-staged：`lint-staged` 插件的配置
- husky：husky 插件的配置
```json
{
 "name": "cac",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./index-compat.js"
    },
    "./package.json": "./package.json",
    "./": "./"
  },
}

// 可以表示为
{
  "exports": {
    "cac": {
      "import": "cac/dist/index.mjs",
      "require": "cac/index-compat.js"
    },
    "cac/package.json": "cac/package.json",
    "cac/": "cac/"
  },
}

import Cac from "cac"; // from "cac/dist/index.mjs"
```

### mian和module的区别
- 引入module 字段主要解决的问题是 Tree Shaking
- 直接把 main 指向我们 ES6 格式的源码文件不就可以了吗？不行
  - 通常人们在使用打包工具的 `babel` 插件编译代码时都会屏蔽掉 node_modules 目录下的文件。因为按照约定大家发布到 npm 的模块代码都是基于 ES5 规范的，因此配置 `babel` 插件屏蔽 `node_modules` 目录可以极大的提高编译速度。但用户如果使用了我们发布的基于 ES6 规范的包就必须配置复杂的屏蔽规则以便把我们的包加入编译的白名单。
  - 如果用户是在 NodeJS 环境使用我们的包，那么极有可能连打包这一步骤都没有。如果用户的 NodeJS 环境又恰巧不支持 ES6 模块规范，那么就会导致代码报错。
- 所以 `mian` 指向打包成基于CommandJS规范的ES5的代码，而 module 指向 基于 ES6 模块规范的使用ES5语法书写的模块。
  - 如果npm包支持 `module` 字段则会优先使用 ES6 模块规范的版本，这样可以启用 Tree Shaking 机制。
  - 如果npm包不识别 `module` 字段则会使用我们已经编译成 CommonJS 规范的版本，也不会阻碍打包流程。

### 写一个库的 README 需要哪几个部分？
- 国际化
- 项目工程介绍
- 项目的使用效果图
- 项目特点
- 项目的基本结构（架构）
- 集成方式
- 使用方法
- 混淆
- 关于作者/组织及交流方式等信息。
- 贡献者/贡献组织
- 鸣谢
- 版权信息

### 开始生成一个README
- readme-md-generator 工具
- 开源项目地址：[https://github.com/kefranabg/readme-md-generator](https://github.com/kefranabg/readme-md-generator)


### Brackets 是如何实现的
- 通过匹配 `.startsWith('<')`，控制 `required` 变量是否为必须
- 通过匹配 `.startsWith('...')`, 控制 `variadic` 变量是否为可选参数


### [流程图](./flow/)