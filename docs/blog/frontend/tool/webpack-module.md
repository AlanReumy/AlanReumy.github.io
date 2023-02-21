# webpack 模块化原理

webpack打包的时候，允许我们使用各种模块化，最常用的就是 **CommonJS** 和 **Es Module** 这两种模块化

那么它是如何帮助我们实现了代码的模块化呢？

我们来研究一些它的原理，包括如下：

- `CommonJS` 模块化实现原理
- `ES Module` 模块化实现原理
- `CommonJS` 加载 `ES Module` 的原理
- `ES Module` 加载 `CommonJS` 的原理

## CommonJS 模块化实现原理

我们在 `index.js` 文件中编写如下代码：

```js
const { dateFormat, priceFormat } = require('./js/format');

console.log(dateFormat("abc"));
console.log(priceFormat("abc"));
```

打包后，我们整理完成后的代码如下：

![20220615144318](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615144318.png)

我们首先来看最开始具体执行代码的逻辑：

```js
!function () {
  // 1.加载./src/js/format.js
  const { dateFormat, priceFormat } = __webpack_require__("./src/js/format.js");
  console.log(dateFormat("abc"));
  console.log(priceFormat("abc"));
}();
```

我们接着来 `__webpack_require__` 函数里面干了什么：

```js
function __webpack_require__(moduleId) {
  // 1.判断缓存中是否已经加载过，加载过就直接返回
  if (__webpack_module_cache__[moduleId]) {
    return __webpack_module_cache__[moduleId].exports;
  }

  // 2.给module变量和__webpack_module_cache__[moduleId]赋值了同一个对象
  var module = __webpack_module_cache__[moduleId] = { exports: {} };

  // 3.加载执行模块
  __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

  // 4.导出module.exports {dateFormat: function, priceForamt: function}
  return module.exports;
}
```


这个函数中的 `__webpack_modules__` 是一个最上面定义的对象，里面将模块的路径作为 key，函数作为 value：

```js
var __webpack_modules__ = {
  "./src/js/format.js":
    (function (module) {
      const dateFormat = (date) => {
        return "2020-12-12";
      }
      const priceFormat = (price) => {
        return "100.00";
      }

      // 将我们要导出的变量, 放入到module对象中的exports对象
      module.exports = {
        dateFormat,
        priceFormat
      }
    })
}
```


## ES Module 模块化实现原理

我们在 `index.js` 文件中编写如下代码：

```js
import { sum, mul } from "./js/math";

console.log(mul(20, 30));
console.log(sum(20, 30));
```

打包后，我们整理完成后的代码如下：

![20220615145745](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615145745.png)

其中：

- `__webpack_modules__` 对象的作用和上面 `CommonJS` 中的对象完全一致
- `__webpack_require__` 函数和使用 `CommonJS` 打包后的 `__webpack_require__` 函数完全一致

接着我们来看看 `__webpack_modules__` 对象中的 key 对应的 value 中的函数做了什么

```js
var __webpack_modules__ = {
  "./src/index.js":
    (function (__unused_webpack_module, __webpack_exports__, __webpack_require__) {
      // 调用r的目的是记录时一个__esModule -> true
      __webpack_require__.r(__webpack_exports__);

      // _js_math__WEBPACK_IMPORTED_MODULE_0__ == exports
      var _js_math__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/js/math.js");

      console.log(_js_math__WEBPACK_IMPORTED_MODULE_0__.mul(20, 30));
      console.log(_js_math__WEBPACK_IMPORTED_MODULE_0__.sum(20, 30));
    }),
  "./src/js/math.js":
    (function (__unused_webpack_module, __webpack_exports__, __webpack_require__) {
      __webpack_require__.r(__webpack_exports__);

      // 调用了d函数: 给exports设置了一个代理definition
      // exports对象中本身是没有对应的函数
      __webpack_require__.d(__webpack_exports__, {
        "sum": function () { return sum; },
        "mul": function () { return mul; }
      });

      const sum = (num1, num2) => {
        return num1 + num2;
      }
      const mul = (num1, num2) => {
        return num1 * num2;
      }
    })
};
```

我们可以看到，在调用 `./src/index.js` 对应的函数中，调用了一个 `__webpack_require__.r` 函数：

它的作用是给模块做标记，表明是 `ES Module` 的模块

```js
!function () {
  // __webpack_require__这个函数对象添加了一个属性: r -> 值function
  __webpack_require__.r = function (exports) {
    if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
      Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
    }
    Object.defineProperty(exports, '__esModule', { value: true });
  };
}();
```

接着我们可以看到，在调用 `./src/js/math.js` 对应的函数中，调用了一个 `__webpack_require__.d`：

它的作用是给 `exports` 对象设置了一个代理，将本来存在于一个普通对象中模块暴露出来的变量，代理到 `exports` 对象上

```js
!function () {
  // __webpack_require__这个函数对象添加了一个属性: d -> 值function
  __webpack_require__.d = function (exports, definition) {
    for (var key in definition) {
      if (__webpack_require__.o(definition, key) && !__webpack_require__.o(exports, key)) {
        Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });
      }
    }
  };
}();
```

而这个函数中又使用了一个叫做 `__webpack_require__.o` 的函数，用来判断对象中是否有某个属性

```js
!function () {
  // __webpack_require__这个函数对象添加了一个属性: o -> 值function
  __webpack_require__.o = function (obj, prop) { return Object.prototype.hasOwnProperty.call(obj, prop); }
}();
```

## CommonJS 和 ES Module 相互导入模块化原理

我们在 `index.js` 文件中编写如下代码：

```js
// es module导出内容, CommonJS导入内容
const { sum, mul } = require("./js/math");

// CommonJS导出内容, es module导入内容
import { dateFormat, priceFormat } from "./js/format";

console.log(sum(20, 30));
console.log(mul(20, 30));

console.log(dateFormat("aaa"));
console.log(priceFormat("bbb"));
```

打包后，我们整理完成后的代码如下：

![20220615152850](https://codertzm.oss-cn-chengdu.aliyuncs.com/20220615152850.png)

我们直接看 `____webpack_modules__` 对应的 key

```js
var __webpack_modules__ = ({
  "./src/index.js":
    (function (__unused_webpack_module, __webpack_exports__, __webpack_require__) {
      "use strict";
      __webpack_require__.r(__webpack_exports__);
      var _js_format__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/js/format.js");
      var _js_format__WEBPACK_IMPORTED_MODULE_0___default = __webpack_require__.n(_js_format__WEBPACK_IMPORTED_MODULE_0__);
      
      // es module导出内容, CommonJS导入内容
      const math = __webpack_require__("./src/js/math.js");

      // CommonJS导出内容, es module导入内容
      console.log(math.sum(20, 30));
      console.log(math.mul(20, 30));
      console.log(_js_format__WEBPACK_IMPORTED_MODULE_0___default().dateFormat("aaa"));
      console.log(_js_format__WEBPACK_IMPORTED_MODULE_0___default().priceFormat("bbb"));
    }),
   "./src/js/format.js":
    (function (module) {
      const dateFormat = (date) => {
        return "2020-12-12";
      }
      const priceFormat = (price) => {
        return "100.00";
      }
      module.exports = {
        dateFormat,
        priceFormat
      }
    }),
  "./src/js/math.js":
    (function (__unused_webpack_module, __webpack_exports__, __webpack_require__) {
      
      __webpack_require__.r(__webpack_exports__);
      __webpack_require__.d(__webpack_exports__, {
        "sum": function () { return sum; },
        "mul": function () { return mul; }
      });
      const sum = (num1, num2) => {
        return num1 + num2;
      }

      const mul = (num1, num2) => {
        return num1 * num2;
      }
    })
});
```

可以看到，在路径对应的函数中，有一个 `__webpack_require__.n` 函数

它的作用是根据模块是 `ES Module` 还是 `CommonJS`，拿到对应的 `getDefaultExport` 函数

```js
!function () {
  // getDefaultExport function for compatibility with non-harmony modules
  __webpack_require__.n = function (module) {
    var getter = module && module.__esModule ?
      function () { return module['default']; } :
      function () { return module; };
    __webpack_require__.d(getter, { a: getter });
    return getter;
  };
}();
```

以上就是 webpack 最常见的两种模块化实现的原理。