# React.memo 实现原理

`React.memo` 的核心原理是**浅比较**和**缓存**。在每次渲染时，`React.memo` 会比较组件上一次渲染时的 props 和当前的 props。如果发现 props 没有变化，就直接使用上一次渲染的结果，而不再重新渲染组件。这使得组件的渲染开销显著减少。

1. 什么是 `React.memo`？

   `React.memo` 是一个高阶组件，它接受一个组件作为参数，并返回一个新的组件。它的行为类似于 `PureComponent`，但用于函数组件，而 `PureComponent` 是用于类组件的。`React.memo` 会对传入组件的 `props` 进行浅比较，当 `props` 没有变化时，它会跳过渲染和更新过程。

以下是 `React.memo` 的工作步骤：

1. **创建一个记忆组件**：

   - `React.memo` 接受一个函数组件 `Component` 作为参数，并返回一个新的**记忆组件**（Memoized Component）。

2. **浅比较 props**：

   - 记忆组件在接收到新的 props 时，会将新的 props 和上一次的 props 进行浅比较（`shallow comparison`）。
   - 如果两个 props 对象相等（即每个键值对都相等），则跳过组件渲染，直接返回之前的渲染结果。
   - 如果有任何 props 发生变化，则重新渲染组件，并更新缓存的渲染结果。

3. **缓存结果**：

   - 记忆组件会缓存上一次渲染的结果（虚拟 DOM），如果下一次渲染发现 props 没有变化，它会复用这个缓存的结果。

### 3. **如何进行浅比较？**

- 浅比较（Shallow Comparison）指的是对 props 对象的第一层进行比较，即比较对象中每个 key 的值是否相等。
- 这意味着如果 props 是**原始类型**（如数字、字符串、布尔值等）或**引用不变的对象**，浅比较可以有效判断是否需要重新渲染。
- 但是，如果 props 是一个**嵌套对象**（例如数组、对象等引用类型），即使内部数据相同，只要引用发生变化，浅比较也会认为它们不同，从而触发重新渲染。

### 4. **自定义比较函数**

React 允许在 `React.memo` 中传入一个自定义的比较函数（`areEqual`），用于比较旧 props 和新 props。这在需要进行深度比较或特殊比较逻辑时非常有用：

```javascript
const MyComponent = React.memo(
  (props) => {
    /* 渲染逻辑 */
  },
  (prevProps, nextProps) => {
    // 自定义比较逻辑
    return prevProps.value === nextProps.value;
  }
);
```

- 如果自定义比较函数返回 `true`，表示 props 没有变化，跳过渲染。
- 如果返回 `false`，表示 props 发生了变化，需要重新渲染。

### 5. `React.memo` **的优点**

- **性能优化**：通过避免不必要的渲染，`React.memo` 可以减少组件的渲染次数，从而提升性能。
- **适用于函数组件**：在没有 `React.memo` 之前，只有类组件可以使用 `PureComponent` 进行优化，现在函数组件也有了类似的优化手段。

### 6. **使用注意事项**

- **浅比较的局限性**：`React.memo` 的浅比较在处理复杂的嵌套对象或数组时效果有限。若 props 包含大量嵌套对象或数组，需要手动进行深度比较或使用不可变数据结构来确保引用不变。
- **适用场景**：`React.memo` 最适合用于**纯展示组件**或**不频繁更新的组件**。对于频繁更新的组件，过多的比较反而可能带来性能开销。

### 7. **工作流程总结**

- 调用 `React.memo`，返回一个记忆组件。
- 当该组件接收到新的 props 时，与之前的 props 进行浅比较（或使用自定义比较函数）。
- 如果 props 没有变化，跳过渲染，直接使用缓存结果；如果 props 发生变化，则重新渲染组件并更新缓存。

### 8. **示例**

```javascript
const MyComponent = (props) => {
  console.log("Rendering MyComponent");
  return <div>{props.value}</div>;
};

// 使用 React.memo 包装组件
const MemoizedComponent = React.memo(MyComponent);

// 使用 MemoizedComponent 时，只有当 props.value 发生变化时，才会重新渲染
```

在上面的示例中，如果 `MemoizedComponent` 的 `props.value` 没有变化，它将不会重新渲染，即使父组件重新渲染了。

### 总结

`React.memo` 通过对 props 进行浅比较和缓存渲染结果，实现了函数组件的性能优化。它适用于减少不必要的渲染开销，特别是纯展示组件或不频繁更新的组件。在使用时，开发者需要注意浅比较的局限性，并根据具体情况合理地使用自定义比较函数。
