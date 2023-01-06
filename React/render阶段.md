# render阶段

render阶段的主要工作是构建Fiber树和生成effectList，react入口的两种模式会进入`performSyncWorkOnRoot`或者`performConcurrentWorkOnRoot`，而这两个方法分别会调用workLoopSync或者workLoopConcurrent

```javascript
// performSyncWorkOnRoot会调用该方法
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}

// performConcurrentWorkOnRoot会调用该方法
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

他们唯一的区别是是否调用`shouldYield`。如果当前浏览器帧没有剩余时间，`shouldYield`会中止循环，直到浏览器有空闲时间后再继续遍历。

```javascript
function performUnitOfWork(unitOfWork) {
  var current = unitOfWork.alternate;
  var next;
	// ...
  if ( (unitOfWork.mode & ProfileMode) !== NoMode) {
    next = beginWork$1(current, unitOfWork, subtreeRenderLanes);
  } else {
    next = beginWork$1(current, unitOfWork, subtreeRenderLanes);
  }

  unitOfWork.memoizedProps = unitOfWork.pendingProps;

  if (next === null) {
    completeUnitOfWork 0.(unitOfWork);
  } else {
    workInProgress = next;
  }
}
```

`performUnitOfWork`的工作可以分为两部分:

- 从rootFiber开始向下深度优先遍历。为遍历到的每个Fiber节点调用`beginWork`，该方法会根据传入的Fiber节点创建子Fiber节点，并将这两个Fiber节点连接起来

- 遍历到子节点之后，会执行`completeWork`方法，执行完成之后会判断此节点的兄弟节点存不存在，如果存在就会为兄弟节点执行`completeWork`，当全部兄弟节点执行完之后，会向上‘冒泡’到父节点执行`completeWork`，直到rootFiber

- 当遍历到只有一个子文本节点的Fiber时，该Fiber节点的子节点不会执行beginWork和completeWork

  

## beginWork

```javascript
function beginWork(current, workInProgress, renderLanes) {
  var updateLanes = workInProgress.lanes;
	// update时：如果current存在可能存在优化路径，可以复用current（即上一次更新的Fiber节点）
  if (current !== null) {
	   var oldProps = current.memoizedProps;
     var newProps = workInProgress.pendingProps;
     
     if (oldProps !== newProps  || workInProgress.type !== current.type ) {
       didReceiveUpdate = true;
     } else if (!includesSomeLane(renderLanes, updateLanes)) {
       didReceiveUpdate = false;
       switch (workInProgress.tag) {
        // ...
       }
       // 复用current
       return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
     } else {
       didReceiveUpdate = false;
     }
  } else {
    didReceiveUpdate = false;
  }
	
  // mount时，根据tag不同，创建不同的子fiber节点
  switch (workInProgress.tag) {
    case IndeterminateComponent:
		  // ...
    case LazyComponent:
  		// ...
    case FunctionComponent:
      // ...
    case ClassComponent:
      // ...
    case HostRoot:
      // ...
    case HostComponent:
      // ...
    case HostText:
      // ...
    case SuspenseComponent:
      // ...
    case HostPortal:
      // ...
    case ForwardRef:
      // ...	
    case Fragment:
      // ...
    case Mode:
      // ...
    case Profiler:
      // ...
    case ContextProvider:
      // ...
    case ContextConsumer:
      // ...
    case MemoComponent:
      // ...
    case SimpleMemoComponent:
      // ...
    case SuspenseListComponent:
      // ...
    case FundamentalComponent:
      // ...
    case ScopeComponent:
      // ...
    case Block:
      // ...
    case OffscreenComponent:
      // ...
    case LegacyHiddenComponent:
      // ...
  }
}
```

对于我们常见的组件类型， 最终会进入reconcileChildren方法。

## reconcileChildren

主要作用有以下几点：

- 对于mount的组件，会创建新的Fiber节点
- 对于update的组件，他会将当前组件与该组件在上次更新时对应的`Fiber节点`比较，将比较的结果生成新`Fiber节点`

```javascript
function reconcileChildren(current, workInProgress, nextChildren, renderLanes) {
  if (current === null) {
    workInProgress.child = mountChildFibers(workInProgress, null, nextChildren, renderLanes);
  } else {
    workInProgress.child = reconcileChildFibers(workInProgress, current.child, nextChildren, renderLanes);
  }
}
```

从代码可以看出，和`beginWork`一样，他也是通过`current === null ?`区分`mount`与`update`。

不论走哪个逻辑，最终他会生成新的子`Fiber节点`并赋值给`workInProgress.child`，作为本次`beginWork`的返回值，并作为下次`performUnitOfWork`执行时`workInProgress`的传参。

`mountChildFibers`与`reconcileChildFibers`这两个方法的逻辑基本一致。唯一的区别是：`reconcileChildFibers`会为生成的`Fiber节点`带上`effectTag`属性，而`mountChildFibers`不会。

```javascript
function bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes) {
  //...
	if (!includesSomeLane(renderLanes, workInProgress.childLanes)) {
    return null;
  } else {
    cloneChildFibers(current, workInProgress);
    return workInProgress.child;
  }
}
```

如果进入了`bailoutOnAlreadyFinishedWork`复用的逻辑，会判断优先级，优先级足够则进入`cloneChildFibers`否则返回`null`

## completeWork

```javascript
function completeWork(current, workInProgress, renderLanes) {
  var newProps = workInProgress.pendingProps;

  switch (workInProgress.tag) {
    case IndeterminateComponent:
    case LazyComponent:
    case SimpleMemoComponent:
    case FunctionComponent:
    case ForwardRef:
    case Fragment:
    case Mode:
    case Profiler:
    case ContextConsumer:
    case MemoComponent:
      // ...
    case ClassComponent:
      // ...
    case HostRoot:
      // ...
    case HostComponent:
      {
        popHostContext(workInProgress);
        var rootContainerInstance = getRootHostContainer();
        var type = workInProgress.type;
				
        if (current !== null && workInProgress.stateNode != null) {
      		// update的情况
           updateHostComponent$1(current, workInProgress, type, newProps, rootContainerInstance);
        } else {
          // mount的情况
          var currentHostContext = getHostContext();
          // 为fiber创建对应DOM节点
          var instance = createInstance(type, newProps, rootContainerInstance, currentHostContext, workInProgress);
          // 将子孙DOM节点插入刚生成的DOM节点中
          appendAllChildren(instance, workInProgress, false, false);
          // DOM节点赋值给fiber.stateNode
          workInProgress.stateNode = instance;
          if (finalizeInitialChildren(instance, type, newProps, rootContainerInstance)) {
            markUpdate(workInProgress);
          }
        }

        return null;
      }
    case HostText:
      // ...
    case SuspenseComponent:
      // ...
    case HostPortal:
      // ...
    case ContextProvider:
      // ...
    case IncompleteClassComponent:
      // ...
    case SuspenseListComponent:
      // ...
    case FundamentalComponent:
      // ...
    case ScopeComponent:
      // ...
    case Block:
    case OffscreenComponent:
    case LegacyHiddenComponent:
      // ...
  }
}
```

当`update`时，`Fiber节点`已经存在对应`DOM节点`，所以不需要生成`DOM节点`。需要做的主要是处理`props`，比如：

- `onClick`、`onChange`等回调函数的注册

- 处理`style prop`

- 处理`DANGEROUSLY_SET_INNER_HTML prop`

- 处理`children prop`

  

`mount`时的主要逻辑包括三个：

- 为`Fiber节点`生成对应的`DOM节点`
- 将子孙`DOM节点`插入刚生成的`DOM节点`中
- 与`update`逻辑中的`updateHostComponent`类似的处理`props`的过程

## effectList

在`completeWork`的上层函数`completeUnitOfWork`中，每个执行完`completeWork`且存在`effectTag`的`Fiber节点`会被保存在一条被称为`effectList`的单向链表中。

`effectList`中第一个Fiber节点保存在`fiber.firstEffect`，最后一个元素保存在`fiber.lastEffect`

```javascript
function completeUnitOfWork(unitOfWork) {
  // ...
    if (returnFiber !== null && (returnFiber.flags & Incomplete) === NoFlags) {
        if (returnFiber.firstEffect === null) {
          returnFiber.firstEffect = completedWork.firstEffect;
        }

        if (completedWork.lastEffect !== null) {
          if (returnFiber.lastEffect !== null) {
            returnFiber.lastEffect.nextEffect = completedWork.firstEffect;
          }
          returnFiber.lastEffect = completedWork.lastEffect;
        }


        var flags = completedWork.flags;.

        if (flags > PerformedWork) {
          if (returnFiber.lastEffect !== null) {
            returnFiber.lastEffect.nextEffect = completedWork;
          } else {
            returnFiber.firstEffect = completedWork;
          }
          returnFiber.lastEffect = completedWork;
        }
      }
    } 

    var siblingFiber = completedWork.sibling;

    if (siblingFiber !== null) {
      workInProgress = siblingFiber;
      return;
    }

    completedWork = returnFiber;
    workInProgress = completedWork
		// ...
}
```

最后commitRoot(root)，进入commit阶段。
