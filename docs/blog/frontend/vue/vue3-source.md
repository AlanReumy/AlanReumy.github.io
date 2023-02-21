---
title: vue3源码分析
---

## 虚拟DOM的优势

现如今的三大框架都是使用虚拟DOM来对真实DOM进行抽象，这样做的好处有：

1. 方便对其操作
2. 方便进行跨平台的操作，将虚拟节点渲染成你想要的节点

## 虚拟DOM的渲染过程

![image-20220113202216941](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220113202216941.png)

## Vue源码的核心

Vue源码有三大核心：

1. Compiler模块：用来编译模板系统
2. Runtime模块：也就是渲染器模块
3. Reactivity模块：响应式系统

![image-20220113202733444](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220113202733444.png)

## Mini-Vue的实现

主要从三部分实现：

1. 渲染系统模块
2. 响应式系统模块
3. 应用程序入口模块

### 渲染系统的实现

主要包括三个功能：

1. h函数：返回一个vnode对象
2. mount函数：用于将vnode对象挂载到DOM上
3. patch函数：用于将两个vnode对比，决定如何处理新的vnode

#### h函数

直接返回一个vnode对象即可:

```js
const h = (tag, props, children) => {
  // vnode -> Object
  return {
    tag,
    props,
    children,
  };
};
```

#### mount函数

mount函数的步骤主要有四步：

1. 创建真实DOM元素
2. 处理vnode的props中的属性
3. 处理vnode的children
4. 将vnode挂载到真实DOM上

```js
const mount = (vnode, container) => {
  // 1. create element
  const el = (vnode.el = document.createElement(vnode.tag));

  // 2. handle Props
  if (vnode.props) {
    for (const key in vnode.props) {
      const value = vnode.props[key];
      if (typeof value === "string") {
        el.setAttribute(key, value);
      } else {
        el.addEventListener(key.slice(2).toLowerCase(), value);
      }
    }
  }

  // 3. handle children
  if (typeof vnode.children === "string") {
    el.textContent = vnode.children;
  } else {
    vnode.children.forEach((item) => {
      mount(item, el);
    });
  }

  // 4. mount on container
  container.appendChild(el);
};
```

#### patch函数

patch函数的实现主要有四步：

1. 比较标签是否相同
2. 如果标签不同，则处理props，处理新的props，删除旧的props
3. 继续处理children属性，对不同的类型进行比较、判断

```js
const patch = (n1, n2) => {
  // 1. diff tag
  if (n1.tag !== n2.tag) {
    const n1ElParent = n1.el.parent;
    n1ElParent.removeChild(n1.el);
    mount(n2, n1ElParent);
  } else {
    const el = (n2.el = n1.el);

    const oldProps = n1.props || {};
    const newProps = n2.props || {};

    // 2. handle newProp
    for (const key in newProps) {
      const newValue = newProps[key];
      const oldValue = oldProps[key];
      // dif  f propsValue
      if (newValue !== oldValue) {
        if (typeof newValue === "string") {
          el.setAttribute(key, newValue);
        } else {
          el.addEventListener(key.slice(2).toLowerCase(), newValue);
        }
      }
    }

    // 3. delete oldProp
    for (const key in oldProps) {
      const oldValue = oldProps[key];
      // 3.1 remove all event
      if (key.startsWith("on")) {
        el.removeEventListener(key.slice(2).toLowerCase(), oldValue);
      }
      // 3.2 key not in newProps
      if (!(key in newProps)) {
        el.removeAttribute(key);
      }
    }

    // 4.handle children
    const oldChildren = n1.children || [];
    const newChildren = n2.children || [];

    if (typeof newChildren === "string") {
      // 4.1 newChildren -> String oldChildren -> String
      if (typeof oldChildren === "string") {
        el.textContent = newChildren;
      } else {
        // 4.2 newChildren -> String oldChildren -> Array
        el.innerHTML = newChildren;
      }
    } else {
      // 4.3 newChildren -> Array oldChildren -> String
      if (typeof oldChildren === "string") {
        newChildren.forEach((item) => {
          mount(item, el);
        });
      } else {
        // 4.4 newChildren -> Array oldChildren -> Array
        const commonLength = Math.min(oldChildren.length, newChildren.length);
        // 4.4.1 patch all
        for (let i = 0; i < commonLength; i++) {
          patch(oldChildren[i], newChildren[i]);
        }

        // 4.4.2 newChildren.length > oldChildren.length
        if (newChildren.length > oldChildren.length) {
          newChildren.slice(oldChildren.length).forEach((item) => {
            mount(item, el);
          });
        }

        // 4.4.3 newChildren.length < oldChildren.length
        if (newChildren.length < oldChildren.length) {
          oldChildren.slice(newChildren.length).forEach((item) => {
            el.removeChild(item.el);
          });
        }
      }
    }
  }
};

```

### reactivity函数

### 创建app对象的过程

1. 创建app对象

   1. 调用ensureRenderer函数，创建渲染器

   2. 调用baseCreateRender函数，里面包含render函数的实现，此函数的返回值为：

      ![image-20220116104342910](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220116104342910.png)

   3. creatAppAPI调用的返回值是一个函数，该函数的返回值才是返回的真正的app对象

   4.

2. 重写app对象的mount方法
3. 返回app对象

### 挂载根组件的过程

1. 调用app对象中重写的mount方法
2. 调用createAppAPI返回的函数，此函数返回的app对象的mount方法
3. 判断是否挂载
4. 创建根组件的vnode
5. 调用render函数，渲染vnode
6. 进入render函数，调用patch函数，此函数实现的是创建或者更新组件
7. 判断类型
8. 调用processComponent函数，对组件进行处理
9. 调用mountComponent挂载组件
10. 调用createComponentInstance函数创建组件的实例
11. 调用setupComponent函数，此函数是对组件的props/data/methods/computed等进行初始化处理，并且内部实现对vue2的options api的兼容
12. 调用setupRenderEffect，此函数是设置和渲染有副作用的函数，用来监听组件数据的变化
13. 判断组件是否挂载
14. 获取组件的subTree
15. 调用patch函数
16. 判断类型
17. 是element类型
18. 调用processElement，对元素类型进行处理
19. 调用mountElement，此函数用来挂载元素
20. 根据vnode创建元素
21. 判断情况，如果子节点是一个数组，调用mountChildren挂载子元素
22. 挂载完成

> 组件的vnode和instance的区别：
>
> ​    组件的vnode：虚拟dom中的虚拟节点
>
> ​    组件的instance：保存组件的状态：如data、methods、computed等

### setupComponent函数的内部细节

1. 判断是否是一个有状态的组件
2. 初始化props和slots
3. 调用setupStatefulComponent函数，对有状态的组件进行设置
4. 判断是否有setup函数
5. 如果setup有参数, 那么会创建一个setupContext上下文, 这个上下文可以在函数中被使用
6. 调用callWithErrorHandling函数得到setup函数的结果
7. 调用handleSetupResult，处理setup函数的结果
8. 调用finishComponentSetup，此函数内部调用compile函数对template进行处理运行时模板，完成组件实例化设置

### compile的过程

1. 其实就是调用了一个叫baseCompile的函数
2. 对template进行解析生成ast树
3. 进行ast转换
4. 生成代码
