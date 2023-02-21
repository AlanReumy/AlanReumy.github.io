---
title: Vue面试题（个人总结）
---

## 1.组件间通信

组件间的通信可分为三种：

- 父子组件通信：props/$emit/$parent/ref/$attrs
- 兄弟组件通信：$parent/$root/eventbus/vuex
- 跨组件通信：eventbus/vuex/provide+inject

> 其中$on，$children在vue3中已经被移除了

## 2.v-for 和 v-if 优先级

- 优先级：在vue2中v-for的优先级高于v-if，在vue3中v-if的优先级高于v-for。
- 不应同时使用：根据官方文档，v-for和v-if不应该同时使用。如果你想实现相同的效果，应该使用computed后的数组来进行v-for。

> - [vue2源码](https://github1s.com/vuejs/vue/blob/HEAD/src/compiler/codegen/index.js#L65-L66)
> - [vue3源码](https://github1s.com/vuejs/core/blob/HEAD/packages/compiler-core/src/codegen.ts#L586-L587)

## 3.简述Vue生命周期

- 生命周期的概念：vue组件实例的生命周期被创建后都会经过一系列初始化的步骤，例如：数据监测，模板编译，挂载到dom上，以及数据变化后更新dom，这个过程中会在不同的时期运行不同的生命周期函数（life cycle hook），用户可以在生命周期函数里面添加自己的代码处理业务逻辑。

- 生命周期的各个阶段：

  | vue2          | vue3            | 描述                                  |
  | ------------- | --------------- | ------------------------------------- |
  | beforeCreate  | beforeCreate    | 组件实例被创建前                      |
  | created       | created         | 组件实例创建后                        |
  | beforeMount   | beforeMount     | 组件挂载之前                          |
  | mounted       | mounted         | 组件挂载之后                          |
  | beforeUpdate  | beforeUpdate    | 组件数据更新前                        |
  | updated       | updated         | 组件数据更新后                        |
  | beforeDestory | beforeUnmount   | 组件实例销毁之前                      |
  | destory       | unmounted       | 组件实例销毁后                        |
  | activated     | activated       | keep-alive 缓存的组件激活时           |
  | deactivated   | deactivated     | keep-alive 缓存的组件失活时           |
  |               | errorCaptured   | 捕获一个来自子孙组件的错误时被调用    |
  |               | renderTracked   | 跟踪虚拟 DOM 重新渲染时调用           |
  |               | renderTriggered | 当虚拟 DOM 重新渲染被触发时调用       |
  |               | serverPrefetch  | 组件实例在服务器上被渲染前调用（ssr） |

- 实际使用：

  **beforeCreate**：通常用于插件开发中执行一些初始化任务

  **created**：组件初始化完毕，可以访问各种数据，获取接口数据等

  **mounted**：dom已创建，可用于获取访问数据和dom元素；访问子组件等。

  **beforeUpdate**：此时`view`层还未更新，可用于获取更新前各种状态

  **updated**：完成`view`层的更新，更新后，所有状态已是最新

  **beforeunmount**：实例被销毁前调用，可用于一些定时器或订阅的取消

  **unmounted**：销毁一个实例。可清理它与其它实例的连接，解绑它的全部指令及事件监听器

- setup和created谁先执行：

  setup比created先执行，根据创建组件实例的流程：在[Vue3源码](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/component.ts#L648)，创建组件实例后，会执行setupComponent，其中会调用setupStatefulComponent函数，此函数会调用setup，而此时生命周期钩子还并未调用。

## 4.双向绑定使用及原理

- 是一个指令：vue中双向绑定是一个指令`v-model`，可以绑定一个响应式数据到视图，同时视图中变化能改变该值。
- `v-model`本质上是语法糖，默认情况下相当于`:value`和`@input`。使用`v-model`可以减少大量繁琐的事件处理代码，提高开发效率。
- 对于自定义组件时，`v-model` 在vue2和vue3的表现不同：表现在prop 和事件默认名称上：
  - prop：`value` -> `modelValue`；
  - 事件：`input` -> `update:modelValue`
