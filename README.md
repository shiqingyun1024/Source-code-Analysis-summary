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
```