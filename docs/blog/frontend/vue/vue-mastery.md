---
title: vue-Mastery 笔记
---

## render 函数

render 函数的基本使用:

```js
import { h } from "vue";
const App = {
  render() {
    return h(
      "div",
      {
        id: "foo",
      },
      [h("span", "world")]
    );
  },
};
```

v-if 的使用

```js
import { h } from "vue";
const App = {
  render() {
    if (this.ok) {
      return h(xxx);
    } else {
      return h(xxx);
    }
  },
};
```

v-for 的使用

```js
import { h } from "vue";
const App = {
  render() {
    return this.list.map((item) => {
      return h("div", { key: item.id }, item.text);
    });
  },
};
```

插槽的使用

```js
import { h } from "vue";
const App = {
  render() {
    // 返回的是一个vnode数组
    const slot = this.$slots.default() ? this.$slots.default() : [];
  },
};
```

## 静态节点提升

```html
<div>
  <div>hello world</div>
</div>
```

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220301193947.png)

被提升的节点只会创建一次，render 函数会随着组件的更新执行

优化点：

1. 被提升的节点只会创建一次，以后的每次渲染都会被复用
2. 在做 diff 算法的时候，不需要比较

## 对 props 的优化

```html
<div>
  <div @click="foo">button</div>
</div>
```

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220301195340.png)

优化点：

1. 由于 props 中的 onclick 的事件处理函数很少被修改，vue3 内部将其缓存起来，不做 diff 算法

## block tree 的优化

```html
<div>
  <div></div>
  <div></div>
  <div>
    <span>{{ msg }}</span>
  </div>
</div>
```

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220301201350.png)

这个根组件经过 createBlock 后，会有一个额外的属性，dynamicChildren，span 的这个 vnode 加入了 dynamicChildren 中

此时进行比较的话，不需要检查每个 vnode 的变化，而是直接去找 block，进行扁平数组的迭代

而且此时/_TEXT_/标志着，只需要检查文本内容，而不需要担心其他的 props 的变化

## mini-vue 的实现

```js
function h(tag, props, children) {
  return {
    tag,
    props,
    children,
  };
}

function mount(vnode, container) {
  const el = (vnode.el = document.createElement(vnode.tag));
  // props
  if (vnode.props) {
    for (const key in vnode.props) {
      const value = vnode.props[key];
      if (key.startsWith("on")) {
        el.addEventListener(key.slice(2).toLowerCase(), key);
      } else {
        el.setAttribute(key, value);
      }
    }
  }
  // children
  if (vnode.children) {
    if (typeof vnode.children === "string") {
      el.textContent = vnode.children;
    } else {
      vnode.children.forEach((child) => {
        mount(child, el);
      });
    }
  }
  container.appendChild(el);
}

function patch(n1, n2) {
  if (n1.tag === n2.tag) {
    const el = (n2.el = n1.el);
    // props
    const oldProps = n1.props || {};
    const newProps = n2.props || {};

    for (const key in oldProps) {
      const oldValue = oldProps[key];
      const newValue = newProps[key];
      if (oldValue !== newValue) {
        if (key.startsWith("on")) {
          el.addEventListener(key.slice(2).toLowerCase(), newValue);
        } else {
          el.setAttribute(key, newValue);
        }
      }
    }

    for (const key in newProps) {
      const oldValue = oldProps[key];
      if (key.startsWith("on")) {
        el.removeEventListener(key.slice(2).toLowerCase(), oldValue);
      } else {
        el.removeAttribute(key);
      }
    }

    // children
    const oldChildren = n1.children || [];
    const newChildren = n2.children || [];
    if (typeof newChildren === "string") {
      if (typeof oldChildren === "string") {
        el.textContent = newChildren;
      } else {
        el.innerHTML = newChildren;
      }
    } else {
      if (typeof oldChildren === "string") {
        newChildren.forEach((child) => {
          mount(child, el);
        });
      } else {
        const commonLength = Math.min(oldChildren.length, newChildren.length);

        for (let i = 0; i < commonLength; i++) {
          patch(oldChildren, newChildren);
        }

        if (newChildren.length > oldChildren.length) {
          newChildren.slice(oldChildren.length).forEach((child) => {
            mount(child, el);
          });
        }

        if (newChildren.length < oldChildren.length) {
          oldChildren.slice(newChildren.length).forEach((child) => {
            el.removeChild(child.el);
          });
        }
      }
    }
  } else {
    // replace
  }
}

// 响应式的实现
let activeEffect;
class Dep {
  subscriber = new Set();
  depend() {
    if (activeEffect) {
      this.subscriber.add(activeEffect);
    }
  }

  notify() {
    this.subscriber.forEach((effect) => {
      effect();
    });
  }
}

function watchEffect(effect) {
  activeEffect = effect;
  effect();
  activeEffect = null;
}

const targetMap = new WeakMap();

function getDeps(target, key) {
  let depsMap = targetMap.get(target);
  if (!depsMap) {
    depsMap = new Map();
    targetMap.set(target, depsMap);
  }

  let dep = depsMap.get(key);
  if (!dep) {
    dep = new Dep();
    depsMap.set(key, dep);
  }

  return dep;
}

const reactiveHandlers = {
  get(target, key, receiver) {
    const dep = getDeps(target, key);
    dep.depend();
    return Reflect.get(target, key, receiver);
  },
  set(target, key, value, receiver) {
    const dep = getDeps(target, key);
    const result = Reflect.set(target, key, value, receiver);
    dep.notify();
    return result;
  },
};

function reactive(raw) {
  return new Proxy(raw, reactiveHandlers);
}
```

## composition API

useFetch 案例

```html
<script src="https://cdn.bootcdn.net/ajax/libs/vue/3.2.31/vue.global.js"></script>

<div id="app"></div>

<script>
  const { ref, createApp, watchEffect } = Vue;
  function useFetch(getUrl) {
    const data = ref(null);
    const error = ref(null);
    const isPending = ref(true);

    watchEffect(() => {
      isPending.value = true;
      data.value = null;
      error.value = null;
      fetch(getUrl())
        .then((res) => res.json())
        .then((_value) => {
          data.value = _value;
          isPending.value = false;
        })
        .catch((err) => {
          error.value = err;
          isPending.value = false;
        });
    });

    return {
      data,
      error,
      isPending,
    };
  }

  const Post = {
    template: `
      <div v-if="isPending">loading...</div>
      <div v-else-if="error">{{ error }}</div>
      <div v-else>{{ data }}</div>
    `,
    props: ["id"],
    setup(props) {
      const { data, error, isPending } = useFetch(
        () => `https://jsonplaceholder.typicode.com/todos/${props.id}`
      );
      return {
        data,
        error,
        isPending,
      };
    },
  };
  const App = {
    components: {
      Post,
    },
    template: `
     <Post :id="id"></Post>
     <button @click="changeId">changeId</button>
    `,
    setup() {
      const id = ref(1);
      const changeId = () => {
        id.value++;
      };
      return {
        id,
        changeId,
      };
    },
  };

  createApp(App).mount("#app");
</script>
```
