# React.Component

react类组件和函数组件的构造方法，==React.Component==主要做了下面几件事情

- 将 props, context, updater 挂载到 this 上
- 在 Component 原型链上添加 `isReactComponent `对象，用于标记类组件
- 在 Component 原型链上添加 `setState` 方法
- 在 Component 原型链上添加 `forceUpdate ` 方法



```javascript
// ReactBaseClasses.js

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

// 给类组件添加 `this.setState` 方法
Component.prototype.setState = function(partialState, callback) {
  // 验证参数是否合法
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );
  // 添加至 enqueueSetState 队列
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};

// 给类组件添加 `this.forceUpdate` 方法
Component.prototype.forceUpdate = function(callback) {
  // 添加至 enqueueForceUpdate 队列
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate');
};
```

