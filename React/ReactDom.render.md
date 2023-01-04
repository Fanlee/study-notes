# ReactDOM**.**render

react有3种模式进入主体函数的入口：

- **legacy 模式：** `ReactDOM.render(<App />, rootNode)`。这是当前 React app 使用的方式。当前没有计划删除本模式，但是这个模式可能不支持这些新功能
- **blocking 模式：** `ReactDOM.createBlockingRoot(rootNode).render(<App />)`。目前正在实验中。作为迁移到 concurrent 模式的第一个步骤。
- **concurrent 模式：** `ReactDOM.createRoot(rootNode).render(<App />)`。目前在实验中，未来稳定之后，打算作为 React 的默认开发模式。这个模式开启了*所有的*新功能。

legacy 模式在合成事件中有自动批处理的功能，但仅限于一个浏览器任务。非 React 事件想使用这个功能必须使用 `unstable_batchedUpdates`。在 blocking 模式和 concurrent 模式下，所有的 `setState` 在默认情况下都是批处理的。会在开发中发出警告。

#### 不同模式在react运行时的含义

legacy模式是我们常用的，它构建dom的过程是同步的，所以在render的reconciler中，如果diff的过程特别耗时，那么导致的结果就是js一直阻塞高优先级的任务(例如用户的点击事件)，表现为页面的卡顿，无法响应。

concurrent Mode是react未来的模式，它用时间片调度实现了异步可中断的任务，根据设备性能的不同，时间片的长度也不一样，在每个时间片中，如果任务到了过期时间，就会主动让出线程给高优先级的任务



入口阶段主要做了下面几件事：

- render调用legacyRenderSubtreeIntoContainer，最后createRootImpl会调用到createFiberRoot创建fiberRootNode,然后调用createHostRootFiber创建rootFiber，其中fiberRootNode是整个项目的的根节点，rootFiber是当前应用挂在的节点，也就是ReactDOM.render调用后的根节点

  ``` mermaid
  graph TB;
  FiberRootNode-->|current|rootFiber;
  rootFiber --> |stateNode|FiberRootNode;
  ```

  

- 创建完Fiber节点后，legacyRenderSubtreeIntoContainer调用updateContainer创建创建Update对象挂载到updateQueue的环形链表上，然后执行scheduleUpdateOnFiber调用performSyncWorkOnRoot进入render阶段和commit阶段

```javascript
// const rootEl = document.getElementById("root");
// ReactDOM.render(<App />, rootEl);

export function render(element, container, callback) {
  // 验证node节点的合法性
  invariant(
    isValidContainer(container),
    'Target container is not a DOM element.',
  );
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback,
  );
}

function legacyRenderSubtreeIntoContainer(parentComponent,children,container,forceHydrate,callback) {
	var root = container._reactRootContainer;
  var fiberRoot;

  if (!root) {
    // Initial mount
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(container, forceHydrate);
    fiberRoot = root._internalRoot;

    if (typeof callback === 'function') {
      var originalCallback = callback;

      callback = function () {
        var instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    } // Initial mount should not be batched.


    unbatchedUpdates(function () {
      //创建update入口
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
    fiberRoot = root._internalRoot;

    if (typeof callback === 'function') {
      var _originalCallback = callback;

      callback = function () {
        var instance = getPublicRootInstance(fiberRoot);

        _originalCallback.call(instance);
      };
    } // Update


    updateContainer(children, fiberRoot, parentComponent, callback);
  }

  return getPublicRootInstance(fiberRoot);
}

function createRootImpl(container, tag, options) {
	//...
  var root = createContainer(container, tag, hydrate);
  //...
  return root;
}

function createFiberRoot(containerInfo, tag, hydrate, hydrationCallbacks) {
  // 创建fiberRootNode
  var root = new FiberRootNode(containerInfo, tag, hydrate);
  // 创建rootFiber
  var uninitializedFiber = createHostRootFiber(tag);
  // rootFiber和fiberRootNode连接
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;
  initializeUpdateQueue(uninitializedFiber);
  return root;
}

// 对于HostRoot或者ClassComponent会使用initializeUpdateQueue创建updateQueue，然后将updateQueue挂载到fiber节点上
function initializeUpdateQueue(fiber) {
  var queue = {
    baseState: fiber.memoizedState, // 初始state，后面会基于这个state，根据Update计算新的state
    firstBaseUpdate: null,// Update形成的链表的头
    lastBaseUpdate: null,// Update形成的链表的尾
    shared: {
      pending: null // 新产生的update会以单向环状链表保存在shared.pending上，计算state的时候会剪开这个环状链表，并且连接在lastBaseUpdate后
    },
    effects: null
  };
  fiber.updateQueue = queue;
}

function updateContainer(element, container, parentComponent, callback) {
  //...
  
  // 取当前可用lane
  var lane = requestUpdateLane(current$1);
  // 建update
  var update = createUpdate(eventTime, lane); 
  update.payload = {
    element: element //JSX
  };
	// update入队
  enqueueUpdate(current$1, update);
  // 调度update
  scheduleUpdateOnFiber(current$1, lane, eventTime);
  return lane;
}

function scheduleUpdateOnFiber(fiber, lane, eventTime) {
	// ...
  // 同步lane，对应legacy模式
  if (lane === SyncLane) {
      // render阶段的起点
      performSyncWorkOnRoot(root);
  } else {
    // 确保root被调度
    ensureRootIsScheduled(root, eventTime);
    schedulePendingInteractions(root, lane);
  }
  mostRecentlyUpdatedRoot = root;
}

```