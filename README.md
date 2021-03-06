# Source-code-Analysis-summary
关于一些插件的源码分析总结

## vue-devtools直接打开对应组件文件的原理总结
```
首先先参考这篇博客：据说 99% 的人不知道 vue-devtools 还能直接打开对应组件文件？本文原理揭秘
#<https://lxchuan12.gitee.io/open-in-editor/#_1-1-%E7%9F%AD%E6%97%B6%E9%97%B4%E6%89%BE%E4%B8%8D%E5%88%B0%E9%A1%B5%E9%9D%A2%E5%AF%B9%E5%BA%94%E6%BA%90%E6%96%87%E4%BB%B6%E7%9A%84%E5%9C%BA%E6%99%AF>

1.学习内容
vue-devtools打开对应文件的原理以及vscode的调试方式

2.准备
- vscode
- vue-devtools Vue3版本
- vue3 demo项目

3.Vue-devtools是什么
对于vue开发者来说。对这个再熟悉不过了，如果不熟悉的参考一下文档vue-devtools<https://devtools.vuejs.org/>。
工具的主要功能有
- 查看组件
- 查看数据，修改数据以便模拟不同数据时候页面的表现形式
- 查看vuex的事件和数据
- 打开组件对应的vue文件
- 查看render code
- 查看vue组件对应的dom

这次主要是探寻第4点的原理

4.学习目的
- 掌握vscode的调试方式
- 掌握vue-devtools打开组件对应的vue文件的原理

目前我使用的技术栈主要是Vue.js，所以vue-devtools是经常使用的，下面我会先讲一下我安装、调试、分析这三个过程遇到的问题，我的一些思考和收获。
1、安装
vue-devtools的官网链接：https://devtools.vuejs.org/
官网上有详细的安装教程。
项目中vue.config.js的配置为：
// 1.导入launch-editor-middleware包
let openInEditor = require('launch-editor-middleware')
// 在devServer选项中，注册/__open-in-editorHTTP 路由：
module.exports = {
    devServer:{
        open:true,
        port:8020,
        // 猜测要启动的编辑器。您还可以使用该editor选项指定编辑器应用程序 openInEditor('code')
        before (app) {
            app.use('/__open-in-editor', openInEditor('code'))
        }
    }
}
安装过程遇到的问题：
1.1、安装调试vue3.0的vue-devtools之后没有生效
原因：我之前安装过调试vue2.x的vue-devtools，然后我又安装了调试vue3.0的vue-devtools，同时存在两个vue-devtools，结果只有调试vue2.x的vue-devtools生效，后面安装的调试vue3.0的vue-devtools，根本没有生效。我在vue-devtools官网上的常见问题中找到了答案。如下图：
所以删除了调试vue2.x的vue-devtools就可以了。调试vue3.0的vue-devtools同样可以调试vue2.x。

1.2、点击下图中的按钮没有反应。
在vscode编辑器中会报这样的错误：
Could not open App.vue in the editor. 
To specify an editor, specify the EDITOR env variable or add "editor" field to your Vue project config.
解决方案：
打开命令面板 ，输入 ‘>shell command’ ，找到: “Install ‘code’ command in PATH command”。点击即可.

1.3、但是当我再次打开vscode编辑器的时候，还是需要先安装code命令，再次安装的时候报
没有权限，于是我打开了所有的权限，再次安装的时候还是报这个错误。

解决方案：经过自己的摸索，先进行卸载，然后再进行安装。
先卸载：
再进行安装：
这样就可以了，我不知道其他小伙伴有没有遇到这个问题，我在网上也搜了很多关于权限限制的问题，但是都没有生效，我想是不是我电脑上有一些配置影响了，现在还没找到根本原因，不过不影响运行和调试。

2、调试
经过安装之后，可以成功的在编辑器中打开对应的组件文件。

先说一下原理：
通过源码发现，vue-devtools做的事情其实并不复杂，其中利用了nodejs中的child_process执行子进程拿到文件路径，再利用命令（mac和linux使用ps x，Window使用Get-Process）去查找编辑器来打开，当然也可以自己指定编辑器。

流程先走一遍
1、vue3-project/package.json中有一个调试按钮。

选择 serve vue-cli-service serve

2、搜索 'launch-editor-middleware'这个中间件
当然这个中间件是在node_modules下的文件中，一般来说搜索不到node_modules下的文件，需要设置下。当然也有个简单做法。就是「排除的文件」右侧旁边有个设置图标「使用“排查设置”与“忽略文件”」，点击下。这时就搜到了vue3-project/node_modules/@vue/cli-service/lib/commands/serve.js中有使用这个中间件。

3、在启动项目的时候会先走到这个断点处

Vue CLI 3中开箱即用具体源码实现。
// vue3-project/node_modules/@vue/cli-service/lib/commands/serve.js
// 46行
const launchEditorMiddleware = require('launch-editor-middleware')
// 192行
before (app, server) {
        // launch editor support.
        // this works with vue-devtools & @vue/cli-overlay
        // 请求会走到这里，
        app.use('/__open-in-editor', launchEditorMiddleware(() => console.log(
          `To specify an editor, specify the EDITOR env variable or ` +
          `add "editor" field to your Vue project config.\n`
        )))
        // allow other plugins to register middlewares, e.g. PWA
        api.service.devServerConfigFns.forEach(fn => fn(app, server))
        // apply in project middlewares
        projectDevServerOptions.before && projectDevServerOptions.before(app, server)
  },

4、启动项目后，点击这个按钮

5、Network中发出一个请求
会看到Network中发送了一个请求http://localhost:8021/_open-in-editor?file=src/components/HelloWorld.vue，并且带有查询参数?file=src/components/HelloWorld.vue?file=src/components/HelloWorld.vue?file=src/components/HelloWorld.vue

6、请求发出之后，调试会走到了这里，请求发出之后，经过第3步的处理，调用了launchEditorMiddleware，所以会走到这里
先看调试截图。

我们再看一下整个launch-editor-middleware中间件的代码
// vue3-project/node_modules/launch-editor-middleware/index.js
const url = require('url')
const path = require('path')
const launch = require('launch-editor')

module.exports = (specifiedEditor, srcRoot, onErrorCallback) => {
  if (typeof specifiedEditor === 'function') {
    onErrorCallback = specifiedEditor
    specifiedEditor = undefined
  }

  if (typeof srcRoot === 'function') {
    onErrorCallback = srcRoot
    srcRoot = undefined
  }
  // process.cwd() 方法返回 Node.js 进程的当前工作目录。
  srcRoot = srcRoot || process.cwd()
  // 打印srcRoot  输出：/Users/shi/Documents/vue-devtools源码分析/open-in-editor/vue3-project

  // 闭包函数
  return function launchEditorMiddleware (req, res, next) {
    // 解析出请求路径中所带的参数，这样就得到了文件名
    const { file } = url.parse(req.url, true).query || {}
    if (!file) {
      res.statusCode = 500
      res.end(`launch-editor-middleware: required query param "file" is missing.`)
    } else {
      // path.resolve(srcRoot, file) 
      // 输出的是/Users/shi/Documents/vue-devtools源码分析/open-in-editor/vue3-project/src/components/HelloWorld.vue
      launch(path.resolve(srcRoot, file), specifiedEditor, onErrorCallback)
      res.end()
    }
  }
}

从代码中可以看出最后调用了launch-editor。
我们分析一下launch-editor-middleware/index.js这个文件
从上面第3步我们得知launchEditorMiddleware接收的是这样一个箭头函数 ()=>console.log(....)，那就是说传入的specifiedEditor是一个箭头函数，然后specifiedEditor再赋值给onErrorCallback，如果srcRoot是一个函数，也会采取同样的赋值方式（这种切换参数的写法，在很多源码中都很常见。为的是方便用户调用时传参。虽然是多个参数，但可以传一个或者两个）。如果没有传入srcRoot，那么srcRoot就会被赋值 Node.js 进程的当前工作目录。最后返回的是一个闭包函数，在这个函数中，首先先解析出请求路径中所带的参数，这样就得到了文件名。然后进行判断，最后把这个文件完整的文件路径名（/Users/shi/Documents/vue-devtools源码分析/open-in-editor/vue3-project/src/components/HelloWorld.vue）和箭头函数传入launch-editor中。
7、launch-editor
通过断点调试进入到launch-editor中，先看一下launch-editor中的代码（只是粘贴了index.js中的launchEditor函数代码）

// vue3-project/node_modules/launch-editor/index.js
function launchEditor (file, specifiedEditor, onErrorCallback) {
  // 解析文件路径，行号，列号
  const parsed = parseFile(file)
  let { fileName } = parsed
  const { lineNumber, columnNumber } = parsed
  // 判断目录中是否存在这个文件
  if (!fs.existsSync(fileName)) {
    return
  }

  if (typeof specifiedEditor === 'function') {
    onErrorCallback = specifiedEditor
    specifiedEditor = undefined
  }

  // onErrorCallback是一个console箭头函数，用wrapErrorCallback包裹了一层，
  // 所以需要看看wrapErrorCallback这个函数执行了什么
  onErrorCallback = wrapErrorCallback(onErrorCallback)
  // 猜测当前正在使用的编辑器
  const [editor, ...args] = guessEditor(specifiedEditor)
  if (!editor) {
    // 如果没有编辑器，就会在控制台打印我们刚开始看的报错信息
    // Could not open App.vue in the editor. 
    // To specify an editor, specify the EDITOR env variable or add "editor" field to your Vue project config.
    onErrorCallback(fileName, null)
    return
  }

  if (
    process.platform === 'linux' &&
    fileName.startsWith('/mnt/') &&
    /Microsoft/i.test(os.release())
  ) {
    fileName = path.relative('', fileName)
  }

  if (lineNumber) {
    const extraArgs = getArgumentsForPosition(editor, fileName, lineNumber, columnNumber)
    args.push.apply(args, extraArgs)
  } else {
    args.push(fileName)
  }

  if (_childProcess && isTerminalEditor(editor)) {
    _childProcess.kill('SIGKILL')
  }

  if (process.platform === 'win32') {
    _childProcess = childProcess.spawn(
      'cmd.exe',
      ['/C', editor].concat(args),
      { stdio: 'inherit' }
    )
  } else {
    _childProcess = childProcess.spawn(editor, args, { stdio: 'inherit' })
  }
  _childProcess.on('exit', function (errorCode) {
    _childProcess = null

    if (errorCode) {
      onErrorCallback(fileName, '(code ' + errorCode + ')')
    }
  })

  _childProcess.on('error', function (error) {
    onErrorCallback(fileName, error.message)
  })
}

wrapErrorCallback函数很精妙
wrapErrorCallback 返回给一个新的函数，onErrorCallback就成了一个新的函数，wrapErrorCallback 执行时，再去执行 onErrorCallback(cb)。


```
## vue3-工具函数源码解读
```
1、准备
这一期主要是解读Vue3源码中实用的基础工具函数，大部分都是日常见到或者用到的，可以巩固我们的js基础知识，而且我们也能学到如何读开源项目。先通读几遍川哥写的文章初学者也能看懂的 Vue3 源码中那些实用的基础工具函数，感觉很赞，需要多向川哥学习。下面我会跟着川哥的思路分析每一个基础函数，然后注上自己的理解和扩展。最后做一个总结。
2、调试
2.1 读开源项目 贡献指南
阅读开源项目的 README.md 和贡献指南 contributing.md
贡献指南写了很多关于参与项目开发的信息。比如怎么跑起来，项目目录结构是怎样的。怎么投入开发，需要哪些知识储备等。
我觉得这两个文件对想阅读源码的开发者来说十分重要。README.md 描述的是项目的基本信息，它可以快速了解这个项目的全貌。贡献指南 contributing.md 会包含如何参与项目开发，项目打包/运行命令，项目目录结构等等，它能帮助你更好地调试/参与开发源码。在 contributing.md 中我看到了一些比较感兴趣的知识点，比如打包构建格式/配置，包依赖处理，不过这次主要是阅读工具函数，所以后续再抽时间进行有关组件库的学习。
2.2 打包构建项目代码
安装完依赖，直接运行yarn build就可以打包 Vue3 的项目代码了，打包的产物如下（以 shared 模块为例）：

这里的 cjs，esm 是 JS 里用来实现【模块化】的不同规则，JS 的模块化标准还有 amd，umd，iife。
CJS，CommonJS，只能在 NodeJS 上运行，使用 require("module") 读取并加载模块，不支持浏览器。
ESM，ECMAScript Module，现在使用的模块方案，使用 import export 来管理依赖，浏览器直接通过 <script type="module"> 即可使用该写法。NodeJS 可以通过使用 mjs 后缀或者在 package.json 添加 "type": "module" 来使用。
2.3 生成 sourcemap 调试 vue-next 源码
在贡献指南 contributing.md 文件中描述了如何生成 sourcemap 文件：添加【--sourcemap】参数即可。

packages/vue/dist/vue.global.js.map 就是 sourcemap 文件了。
sourcemap 是一个信息文件，里面储存着位置信息，转换后的代码的每一个位置，所对应的转换前的位置。有了它，出错的时候，出错工具将直接显示原始代码，而不是转换后的代码，方便调试。
3、工具函数
3.11 isArray 判断数组

关于prototype、__proto__，可以看一下我的这篇文章详解prototype、__proto__和constructor

3.15 isFunction 判断是不是函数
下面的isPromise等函数中有用到


3.18 isObject 判断是不是对象

3.19 isPromise 判断是不是 Promise
学到了，判断是不是 Promise是这样的判断的，之前我写的比这个复杂，掌握好promise很重要

3.20 objectToString 对象转字符串

3.21 toTypeString 对象转字符串

3.22 toRawType主要用于精准的判断数据类型，实现方式是：对象转字符串后 截取后几位

判断数据类型的方法和原理请参考我的这篇博客：判断js数据类型的四种方法和原理
3.23 isPlainObject 判断是不是纯粹的对象

3.24 isIntegerKey 判断是不是数字型的字符串key值

3.25 makeMap && isReservedProp
传入一个以逗号分隔的字符串，生成一个 map(键值对)，并且返回一个函数检测 key 值在不在这个 map 中。第二个参数是小写选项。

3.26 cacheStringFunction 缓存

这个函数也是和上面 MakeMap 函数类似。只不过接收参数的是函数。 《JavaScript 设计模式与开发实践》书中的第四章 JS单例模式也是类似的实现。=== 关于单例模式，我需要好好总结一下了。

3.27 hasChanged 判断是不是有变化

3.28 invokeArrayFns 执行数组里的函数
这种应用场景对我来说很少见，不过确实学习到了。

为什么这样写，我们一般都是一个函数执行就行。
数组中存放函数，函数其实也算是数据。这种写法方便统一执行多个函数。
3.29 def 定义对象属性

3.30 toNumber 转数字

3.31 getGlobalThis 全局对象
确实是很特殊的写法，很值得借鉴。

获取全局 this 指向。
初次执行肯定是 _globalThis 是 undefined。所以会执行后面的赋值语句。
如果存在 globalThis 就用 globalThis。MDN globalThis
如果存在self，就用self。在 Web Worker 中不能访问到 window 对象，但是我们却能通过 self 访问到 Worker 环境中的全局对象。
如果存在window，就用window。
如果存在global，就用global。Node环境下，使用global。
如果都不存在，使用空对象。可能是微信小程序环境下。
下次执行就直接返回 _globalThis，不需要第二次继续判断了。这种写法值得我们学习。
总结
通过这次阅读vue3.0的工具函数源码，学习到了很多东西。
```