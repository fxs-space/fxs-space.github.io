---
title: React Native的组件通信方式
date: 2017-07-02 21:21:18
categories: React Native
tags: React Native
---

## React Native的组件通信方式

题外话，简单总结下 React、ReactJS 以及 React Native 之间的关系：

- React 是非常热门的一个前端开发框架，其本身作为 MVC 中的 View 层可以用来构建 UI；同时，React 通过对虚拟 DOM 中的微操作来实对现实际 DOM 的局部更新，提高性能。其组件的模块化开发提高了代码的可维护性。单向数据流的特点，让每个模块根据数据量自动更新，让开发者可以只专注于数据部分，改善程序的可预测性。

- React Native是一个框架，而ReactJS是用来构建站点的JavaScript库。当你用ReactJS开始一个新的项目，你或许需要选择一个类似Webpack的打包器，然后去指定你工程中所需要的打包模块。React-Native包含了你需要的所有东西，你几乎不再需要其他东西了。当你开始一个新项目，你会发现一切都很简单——你可以只需要在命令行敲一行命令就行了——然后你就可使用ES6, 某些ES7特性，甚至一些比较新的polyfills开始你的编码。

- React Native不使用HTML来渲染App，但是提供了可代替它的类似组件。这些React Native组件映射到渲染到App中的真正的原生iOS和Android UI组件。

**言归正传，正文开始**

<!--more-->

##### React 最基础的 props 和 state

- **组件内部用 state**
```
constructor(props) {
    super(props);
    this.state = {
        isOnline: true	//组件 state
    };
}
render() {
    if(this.state.isOnline){
        //...剩余代码
    }
    //...剩余代码
}
```

- **父子组件通信用 props**
```
//父组件设置属性参数
<MyComponet isOnline={true} />

//子组件
class MyComponent extends Component {
    constructor(props) {
        super(props);
        //子组件获取属性
        let isOnline = this.props.isOnline;
    }
    //...剩余代码
}
```

- **子父组件通信也可用 props**
```
//子组件
class MyComponent extends Component {
    constructor(props) {
        super(props);
    }
    componentDidMount() {
        //子组件给父组件的方法传参
        this.props.onChange('newVal');
    }
    render() {
        return (
            <View />
        );
    }
}
```

```
//父组件
class parentCpt extends Component {
    constructor(props) {
        super(props);
        this.state = {
            key: 'defVal'
        };
    }
    //父组件接受子组件的参数，并改变 state
    handleChange(val) {
        this.setState({
            key: val
        });
    }
    render() {
        //...剩余代码
        return (
            <MyComponent onChange={(val) => {this.handleChange(val)}} />
        );
    }
}
```

- **使用 Refs**
```
//子组件
class MyComponent extends Component {
    constructor(props) {
        super(props);
    }
    //开放的实例方法
    doIt() {
        //...做点什么
    }
    render() {
        return (
            <View />
        );
    }
}
```

```
//父组件
class parentCpt extends Component {
    constructor(props) {
        super(props);
    }
    componentDidMount() {
        //调用组件的实例方法
        this.myCpt.doIt();
    }
    render() {
        //this.myCpt 保存组件的实例
        return (
            <MyComponent ref={(c) => {this.myCpt = c;}} />
        );
    }
}
```

- **使用 global**

**global**类似浏览器里的 **window** 对象，它是全局的，一处定义，所有组件都可以访问，一般用于存储一些全局的配置参数或方法。

使用场景：全局参数不想通过 props 层层组件传递，有些组件对此参数并不关心，只有嵌套的某个组件使用

```
global.isOnline = true;
```

- **使用 RCTDeviceEventEmitter**

**RCTDeviceEventEmitter** 是一种事件机制，**React Native** 的文档只是草草带过，也可以使用 **DeviceEventEmitter** ，它是把 **RCTDeviceEventEmitter** 封装了一层，用法略不同。

按文档所言，**RCTDeviceEventEmitter** 主要用于 **Native** 发送事件给 **JavaScript**，实际上也可以用来发送自定义事件。

*使用场景：多个组件都使用了异步模块，且异步模块之间有顺序依赖时，可以使用。*

```
//引入模块
import RCTDeviceEventEmitter from 'RCTDeviceEventEmitter';
//监听自定义事件
RCTDeviceEventEmitter.addListener('customEvt', (o) => {
    console.log(o.data);    //'some data'
    //做点其它的
});
//发送自定义事件，可传数据
RCTDeviceEventEmitter.emit('customEvt', {
    data: 'some data'
});
```

- **使用 AsyncStorage**

这是官方提供的持久缓存的模块，类似浏览器端的 **localStorage**，用法也很类似，不过比** localStorage** 多了不少 **API**。

*使用场景：当然也类似，退出应用需要保存的少量数据，可以存在这里，至于大小限制，Android 貌似是 6M 。*

```
import {
  AsyncStorage
} from 'react-native';
//设置
AsyncStorage.setItem('@MySuperStore:key', 'I like to save it.');
//获取
AsyncStorage.getItem('@MySuperStore:key')
```

[参考文章](https://jinlong.github.io/2016/12/16/react-native-component-communication/)