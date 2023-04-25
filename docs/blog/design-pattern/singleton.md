# 单例模式

单例模式指的是保证一个类仅有一个实例，并提供一个访问它的全局访问点

## 实现思路

不管我们尝试去创建多少次，它都只给你返回第一次所创建的那唯一的一个实例。

基于静态方法的实现：

```js
class SingleDog {
  show() {
    console.log("我是一个单例对象");
  }
  static getInstance() {
    // 判断是否已经new过1个实例
    if (!SingleDog.instance) {
      // 若这个唯一的实例不存在，那么先创建它
      SingleDog.instance = new SingleDog();
    }
    // 如果这个唯一的实例已经存在，则直接返回
    return SingleDog.instance;
  }
}
```

也可以用闭包来实现:

```js
SingleDog.getInstance = function () {
  // 定义自由变量instance，模拟私有变量
  let instance = null;
  return function () {
    // 判断自由变量是否为null
    if (!instance) {
      // 如果为null则new出唯一实例
      instance = new SingleDog();
    }
    return instance;
  };
};
```

## 应用场景

### 实现一个 Storage

```js
class Storage {
  constrcutor() {
    if (!Storage.instance) {
      Storage.instance = new Storage();
    }
    return Storage.instance;
  }

  getItem(key) {
    return localStorage.getItem(key);
  }

  setItem(key, value) {
    return localStorage.setItem(key, value);
  }
}
```

### 实现一个全局模态框

```js
class Modal {
  constrcutor() {
    if (!Modal.instance) {
      Modal.instance = document.createElement("div");
      Modal.instance.innerHTML = "I'm a Modal";
      Modal.instance.id = "modal";
      Modal.instance.style.display = "none";
      document.body.appendChild(Modal.instance);
    }
    return Modal.instance;
  }
}

// 点击打开按钮展示模态框
document.getElementById("open").addEventListener("click", function () {
  // 未点击则不创建modal实例，避免不必要的内存占用;此处不用 new Modal 的形式调用也可以，和 Storage 同理
  const modal = new Modal();
  modal.style.display = "block";
});

// 点击关闭按钮隐藏模态框
document.getElementById("close").addEventListener("click", function () {
  const modal = new Modal();
  if (modal) {
    modal.style.display = "none";
  }
});
```
