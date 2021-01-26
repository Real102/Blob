# require.context 的妙用

## 前言

我们在开发 vue 项目时，一般都会利用一些全局组件、Vuex 等进行辅助开发，时间久了，使用的辅助工具等会越来越多，每新增一个都要 import、 export， 非常的麻烦并且不方便管理。这个时候我们就需要借助 webpack 提供的一个函数：`require.context()`来帮助自己管理这些插件

使用 require.context 自动导入指定目录下的所有文件

### require.context(directory, useSubdirectories, RegExp, mode)

`directory:` 指定的文件目录，可以是相对路径/绝对路径  
`useSubdirectories:` 是否遍历该目录下的所有文件（Boolean）  
`regExp:` 正则表达式，用于匹配文件

此外`require.context()`还有三个属性：

`resolve` 是一个函数，它返回被解析后得到的模块 id  
`keys` 也是一个函数，它返回一个数组，包括所有可能被此 context module 处理的请求（主要借助这个属性来对文件做进一步处理）  
`id` 执行环境的 id，返回的是一个字符串

### 使用场景

有如下目录结构及对应文件内容

```json
|—— test
|   |—— index.js
|   |—— a.js
|   |—— b.js
|   └── c.js
```

```javascript
// a.js
export const func1 = () => {
	return "this is a.js"
}
```

```javascript
// b.js
export const func2 = () => {
	return "this is b.js"
}
```

```javascript
// c.js
export default {
	func3() {
		return "this is c.js"
	}
}
```

```javascript
// index.js
let apiList = {} // 定义临时变量，用于存储暴露出的函数
let context = require.context("./", false, /\.js$/)
context.keys().forEach(item => {
	// index.js刚好与其他文件同级，需要过滤，因为这个文件不导出任何变量或方法
	if (item === "./index.js") return
	// 当模块是以export default方式导出，会挂在default属性下，需要特别处理
	let temp = context(item).default || context(item)
	// Object.assign：如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性覆盖前面的属性
	// 确保指定目录下暴露的所有方法都不重名
	Object.assign(apiList, temp)
})
// console.log(apiList)
```

至此就已经把 `./` 当前目录下所有文件暴露出的变量或方法都已经存在 `apiList`，这样就可以做统一操作，避免重复写 import

## 参考文档

-   [webpack 中文文档 🚀](https://webpack.docschina.org/guides/dependency-management/#requirecontext)
