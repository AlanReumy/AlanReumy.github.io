# nextTick 的原理

## nextTick 的作用

将回调推迟到下一个 DOM 更新周期之后执行。在更改了一些数据以等待 DOM 更新后立即使用它。

```js
import { createApp, nextTick, ref } from "vue";

const app = createApp({
  setup() {
    const h2 = ref(null);
    const message = ref("Hello!");
    const addMessage = async (newMessage) => {
      message.value = newMessage;
      console.log(h2.value.offsetHeight);
    };
  },
});
```

如果我们此时没有 `nextTick` ，直接输出 `h2.value.offsetHeight` ，会得到 0 的结果（假设 h2 原本没有高度）。而如果我们将 `addMessage` 修改如下，就可以达到正常输出高度的效果。

```js
const addMessage = async (newMessage) => {
  message.value = newMessage;
  nextTick(() => {
    console.log(h2.value.offsetHeight);
  });
};
```

其原因就是，addMessage 这个函数内部，正常来说是同步任务，而在 `Vue` 中，它将 watch、组件更新、生命周期回调放在微任务中，因此在同步任务执行完后，去清空微任务。会输出 0 的高度。

## nextTick 的原理

`nextTick` 实现的原理就是将回调函数中的所有东西放进微任务队列，因此在 watch、组件更新、生命周期回调这几个微任务出队后，会执行 `nextTick` 放入微任务队列的回调函数。
