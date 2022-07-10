## cac基本概念
### option 的 --type <type> 和 --type [type] 有什么不同 ？ 
- 当在命令名中使用方括号时，尖括号表示必需的命令参数，而方括号表示可选参数。
- 当在选项名称中使用方括号时，尖括号表示需要一个字符串/数字值，而方括号表示该值也可以为 `true`。
- 源码通过 `required` 变量控制，如果是 <> 则为 `true`，是[]则为`false`，两种都不是 `isBoolean` 为 `true`，表示不需要参数
  
### Negated Options 是什么？option的名字格式是固定的吗 --no-[name] ?
- 要允许一个值为false的选项，你需要手动指定一个否定选项 
- 语法 `--no-[name]`
- 源码通过 `negated` 变量控制，如果匹配到 `optionName` 以 `no- ` 开头则设置 `this.config.default = true`

### Variadic Arguments ？
- 命令的最后一个参数可以是可变参数，而且只能是最后一个参数。
- 源码通过 `startsWith('...')` 控制 `variadic` 变量判断 option 是否有可变参数
  
### Dot-nested Options ? 
- 通过 点 ( . ) 嵌套的选项将合并为一个选项。（与commander不同点之一，commander不支持嵌套
```js
input: cli build3 --env.asd asd --env.foo foo

output: env: { ads: 'asd', foo: 'foo' } 
```
### Default Command ?
- 通过[...CommandName] 注册一个命令，当没有其他命令匹配时使用。


### new Cac()
[初始化流程图](./flow/new%20Cac.svg)
- `new Cac()`
- 初始化各种属性
- 初始化全局命令，`GlobalCommand` 类继承 `Command` 类，实际是 `new Command()`


### new Option()
[流程图](./flow/new%20Option.svg)
- `replace(/\.\*/g, '')`: 兼容点嵌套写法
- removeBrackets + sort: 通过去除括号和排序，获取参数列表和定义选项名称(取名称最长的)
- Negated Optionscamel: 通过 `startsWith('no-')` 判断 实现Negated Options, `negated ==false ? this.config.default = true`，判断参数是否必传 `<> or [] or  No arg needed`
- caseOptionName: 全部参数名转成驼峰命名

### command()
[流程图](./flow/command.svg)
- 创建一个命令实例。
- 第一参数为命令名称
- 第二个参数是命名描述
- 第三个参数是附加命令
  - `config.allowUnknownOptions：boolean` 允许未知选项
  - `config.ignoreOptionDefaultValue: boolean` 不要在解析选项中使用选项的默认值，只在帮助消息中显示它们。

### option()
[option流程图](./flow/option.svg)
- 添加一个全局的选项
- 第一个参数是选项名称
- 第二个参数是选项描述
- 第三个参数是附加命令
  - `config.default` 选项的默认值
  - `config.type: any[]` 当设置为[]时，该选项值返回数组类型。您还可以使用转换函数，如[String]，它将使用String调用选项值。

### action()
- 参数为一个回调函数
- parse 的第二个参数的 `run` 变量控制是否执行 callback
- 但命令被 parse 解析匹配到命令时，会通过 `runMatchedCommand` 方法查找并触发该命令对应的 action
- 多个 action 会被后面的 action 覆盖。源码只用了  `commandAction` 变量来存储对应的 action callback。

### parse()
[parse流程图](./flow/parse.svg)
- 第一个参数接收一个 `argv: string[]`, 默认是 process.argv
- 第二个参数是是否执行 `runMatchedCommand`, 默认是 true
- 解析用户输入的 argv，匹配对应的 command，触发对应的action和钩子

### version()
- `Type: (version: string, customFlags = '-v, --version') => CLI`
- `-v`，`--version`，输出版本号。

### help()
- `Type: (callback?: HelpCallback) => CLI`
- `-h`, `--help`，输出帮助消息。

### outputHelp()
- `Type: () => CLI`
- 输出帮助信息

### usage()
- 添加全局使用说明。子命令不使用它。
  
### Events(使用的 node 的 Events 模块)
- 监听命令触发
```ts
// 监听 foo 命名
cli.on('command:foo', () => {
  // Do something
})

// 默认命令
cli.on('command:!', () => {
  // Do something
})

// 未知命令
cli.on('command:*', () => {
  // Do something
})
```