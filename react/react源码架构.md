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

调用栈主要分为三部分，入口 ，render阶段，commit阶段

 ```mermaid
 graph TB;
     入口阶段-->render-->|创建根的fiber节点|createRootImpl-->|初始化合成事件|listenToNativeEvent
     render阶段-->|首次渲染非批量更新|unbatchedUpdates-->|排列任务的优先级|scheduleUpdateOnFiber-->performUnitWork;
     performUnitWork-->beginWork-->|创建或者对比fiber节点|reconcileChildren;
     performUnitWork-->completeWork-->|创建真实的元素|createInstance;
     commit阶段-->commitRoot-->runWithPriority;
     runWithPriority-->|操作真实节点前|commitBeforeMutationEffects
     commitBeforeMutationEffects-->|操作真实节点并把副作用应用到节点上|commitMutationEffects-->|操作真实节点后|commitLayoutEffects;
 ```



