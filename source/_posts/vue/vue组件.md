---
title: Vue组件
date: 2018-05-11 11:02:00
tags: vue
category: vue
---
# vue组件
-------------

vue组件主要是封装dom、dom所需要展示的数据及其的展现方式的封装。组件可以扩展 HTML 元素，封装可重用的代码。
 
## veu组件全局注册
组件在注册之后，便可以作为自定义元素在一个实例的模板中使用。注意确保在初始化根实例之前注册组件：

    <div id="example">
      <my-component></my-component>
    </div>

    // 注册
    Vue.component('my-component', {
        template: '<div>A custom component!</div>'
    })
    // 创建根实例
    new Vue({
      el: '#example'
    })
会被渲染为       

    <div id="example">
      <div>A custom component!</div>
    </div>
## 局部注册
你不必把每个组件都注册到全局。你可以通过某个 Vue 实例/组件的实例选项 components 注册仅在其作用域中可用的组件：

    var Child = {
      template: '<div>A custom component!</div>'
    }
    new Vue({
      // ...
      components: {
        // <my-component> 将只在父组件模板中可用
        'my-component': Child
      }
    })
这种封装也适用于其它可注册的 Vue 功能，比如指令。


