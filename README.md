# Source-code-Analysis-summary
关于一些插件的源码分析总结

### vue-devtools直接打开对应组件文件的原理总结
```
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

```