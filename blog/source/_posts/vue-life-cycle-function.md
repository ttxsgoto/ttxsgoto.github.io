---
title: Vue 生命周期函数
date: 2019-06-30 19:29:15
tags:
  - Vue
categories:
  - Frontend
---
记录vue 生命周期函数的学习

### 生命周期图示
![](https://cn.vuejs.org/images/lifecycle.png)
### 实例
```js
// 生命周期函数：就是vue实例在某一个时间点会自动执行的函数
var vm = new Vue({
    el: "#root",
    //template: "",
    data: {
        message: "hello world"
    },
    methods: {
        handleClick: function () {
            alert('xxxxx')
        }
    },
    // 生命周期函数
    // 在实例初始化之后，数据观测 (data observer) 和 
    // event/watcher 事件配置之前被调用
    beforeCreate: function () { // 自动执行
        console.log('beforeCreate')
    },
    // 在实例创建完成后被立即调用。在这一步，实例已完成以下的配置：
    // 数据观测 (data observer)，属性和方法的运算，
    // watch/event 事件回调。
    // 然而，挂载阶段还没开始，$el 属性目前不可见
    created: function () { // 自动执行
        console.log('created')
    },
    // 在挂载开始之前被调用：相关的 render 函数首次被调用。
    beforeMount: function () { // 页面还没有挂载，自动执行
        console.log(this.$el);
        console.log('beforeMount')
    },
    // el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子
    mounted: function () { // 页面挂载后，自动执行
        console.log(this.$el);
        console.log('mounted')
    },
    // 实例销毁之前调用。在这一步，实例仍然完全可用
    beforeDestroy: function () { 
     	// 当调用$destroy()方法时，还没有被销毁时，方法被触发
        console.log('beforeDestroy')
    },
    // Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，
    // 所有的事件监听器会被移除，所有的子实例也会被销毁
    destroyed: function () { // 当调用$destroy()方法时，完全销毁时方法被触发
        console.log('destroy')
    },
    beforeUpdate: function () { 
    	// 数据发生改变，还没有渲染之前执行该函数, vm.message= 'test'
        console.log('beforeUpdate')
    },
    updated: function () {  // 数据渲染之后执行该函数
        console.log('updated')
    }
})
```



