---
title: React中的高阶组件
---

## 什么是高阶组件

- 高阶组件的英文是 Higher-Order Components,简称为 HOC
- 官方的定义:高阶组件是参数为组件,返回值为新组件的函数

个人理解：

1. 高阶组件 本身不是一个组件,而是一个函数
1. 这个函数的参数是一个组件，返回值也是一个组件

```js
function hoc(wrapperComponent) {
  return class extends PureComponent {
    render()  {
      return <WrapperComponent/>
    }
  }
}

const EnhancedComponent = hoc(WrappedComponent)
```

- 高阶组件并不是React API的一部分,它是基于React的组合特性而形成的设计模式
- 高阶组件在一些React第三方库中非常常见:如 redux、react-router

## 高阶组件的应用

### 应用一：props的增强

- 不修改原有的代码下，添加新的props

  ```js
  function enhanceProps(WrapperCpn, otherProps) {
    return props => <WrapperCpn {...props} {...otherProps} />
  }
  ```

- 利用高阶组件来共享Context

  ```js
  function withUser(WrapperCpn) {
    return props => {
      return (
        <UserContext.Consumer>
          {
            value => {
              return <WrapperCpn {...props} {...value} />
            }
          }
        </UserContext.Consumer>
      )
    }
  }
  ```

### 应用二：渲染判断鉴权

- 在开发中,我们可能遇到这样的场景:
  - 某些页面是必须用户登录成功才能进行进入;
  - 如果用户没有登录成功,那么直接跳转到登录页面;
- 这个时候，我们就可以使用高阶组件完成鉴权操作：

  ```js
  function loginAuth(page) {
    return props => {
      if(props.isLogin) {
        return <Page />
      } else {
        return <LoginPage />
      }
    }
  }
  ```

### 应用三：生命周期劫持

我们可以利用高阶组件来劫持生命周期，在生命周期中完成自己的逻辑：

```js
function logRenderTime(WrapperCpn) {
  return class extends PureComponent {
    componentWillMount() {
      this.begin = Date.now()
    }

    componentDidMount() {
      this.end = Date.now() 
      const interval = this.end - this.begin
      console.log(interval)
    }

    render() {[
      return <WrapperCpn {...this.props} />
    ]}
  }
}
```

## 高阶函数的意义

- 我们会发现利用高阶组件可以针对某些React代码进行更加优雅的处理
- HOC也有自己的一些缺陷:
  - HOC需要在原组件上进行包裹或者嵌套,如果大量使用HOC,将会产生非常多的嵌套,这让调试变得非常困难;
  - HOC可以劫持props,在不遵守约定的情况下也可能造成冲突;

- Hooks的出现,是开创性的,它解决了很多React之前的存在的问题
