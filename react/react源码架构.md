# react源码架构

react核心可以用UI = fn(state)来表示，函数式编程风格， 可以拆成以下两部分来表示

```javascript
const state = reconcile(update)
const UI = commit(state)
```

fn可以分为如下一个部分

- Scheduler（调度器）：排序优先级，让优先级高的任务先进行reconcile
- Reconciler（协调器）: 找出哪些节点发生变化，并打上不同的Flags
- Renderer（渲染器）: 将Reconciler中打好标签的节点渲染到视图上

## Scheduler
