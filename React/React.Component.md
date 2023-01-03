# React.Component

react类组件和函数组件的构造方法

```javascript
// 接收 props，context，updater 进行初始化，挂载到 this 上
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  // updater 上挂载了 isMounted、enqueueForceUpdate、enqueueSetState 等触发器方法
  this.updater = updater || ReactNoopUpdateQueue;
}

// 用于区分类组件和函数组件
Component.prototype.isReactComponent = {}
```

