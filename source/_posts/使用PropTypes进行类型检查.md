---
title: 使用PropTypes进行类型检查
date: 2017-10-12 19:50:19
categories: React
tags: PropTypes
---

## 使用 PropTypes 进行类型检查

注意： 从 React v15.5 开始 ，React.PropTypes 助手函数已被弃用，我们建议使用 [prop-types库](https://www.npmjs.com/package/prop-types) 来定义contextTypes。

我们提供[一个 codemod 脚本](http://www.css88.com/blog/2017/04/07/react-v15.5.0.html#migrating-from-react.proptypes)来自动转换。

随着应用规模的提升，你可以通过类型检测捕获更多的bug。对于部分应用，你可能需要使用类似于 [Flow](https://flowtype.org/) 或者 [TypeScript](https://www.typescriptlang.org/) 等 JavaScript 扩展来对你整个应用类型进行类型检测。但即使你不使用这些，React 内置了类型检测的功能。要在组件中进行类型检测，你可以赋值 `propTypes` 属性。
```
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
```

<!--more-->

`PropTypes` 输出了一系列的验证器，可以用来确保接收到的参数是有效的。例如，我们可以使用`PropTypes.string` 语句。当给 prop 传递了一个不正确的值时，JavaScript控制台将会显示一条警告。出于性能的原因，`propTypes` 仅在开发模式中检测。

#### PropTypes
这里是一个示例记录提供的不同的验证器：
```
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 你可以声明一个 prop 是一个特定的 JS 原始类型。
  // 默认情况下，这些都是可选的。
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 任何东西都可以被渲染:numbers, strings, elements,或者是包含这些类型的数组(或者是片段)。
  optionalNode: PropTypes.node,

  // 一个 React 元素。
  optionalElement: PropTypes.element,

  // 你也可以声明一个 prop 是类的一个实例。
  // 使用 JS 的 instanceof 运算符。
  optionalMessage: PropTypes.instanceOf(Message),

  // 你可以声明 prop 是特定的值，类似于枚举
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 一个对象可以是多种类型其中之一
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // 一个某种类型的数组
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 属性值为某种类型的对象
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 一个特定形式的对象
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // 你可以使用 `isRequired' 链接上述任何一个，以确保在没有提供 prop 的情况下显示警告。
  requiredFunc: PropTypes.func.isRequired,

  // 任何数据类型的值
  requiredAny: PropTypes.any.isRequired,

  // 你也可以声明自定义的验证器。如果验证失败返回 Error 对象。不要使用 `console.warn` 或者 throw ，
  // 因为这不会在 `oneOfType` 类型的验证器中起作用。
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 也可以声明`arrayOf`和`objectOf`类型的验证器，如果验证失败需要返回Error对象。
  // 会在数组或者对象的每一个元素上调用验证器。验证器的前两个参数分别是数组或者对象本身，
  // 以及当前元素的键值。
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

#### 要求单独的 Child
使用 `PropTypes.element` ，你可以指定仅将单一子元素传递给组件，作为子节点。
```
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // 这里必须是一个元素，否则会发出警告。
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};
```

#### 默认的 prop 值
你可以通过赋值特定的 `defaultProps` 属性为 `props` 定义默认值：

```
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// 指定 props 的默认值：
Greeting.defaultProps = {
  name: 'Stranger'
};

// 渲染为 "Hello, Stranger":
ReactDOM.render(
  <Greeting />,
  document.getElementById('example')
);
```

如果你在使用像 [transform-class-properties](https://babeljs.io/docs/plugins/transform-class-properties/) 的 Babel 转换器，你也可以在 React 组件类中声明 `defaultProps` 作为静态属性。这个语法还没有最终通过，在浏览器中需要一步编译工作。更多信息，查看[类字段提议](https://github.com/tc39/proposal-class-fields)。

```
class Greeting extends React.Component {
  static defaultProps = {
    name: 'stranger'
  }

  render() {
    return (
      <div>Hello, {this.props.name}</div>
    )
  }
}
```

如果父组件没有为 `this.props.name` 传值，`defaultProps` 会给其一个默认值。`propTypes` 的类型检测是在`defaultProps` 解析之后发生的，因此也会对默认属性 `defaultProps` 进行类型检测。

[参考文章](http://www.css88.com/react/docs/typechecking-with-proptypes.html)