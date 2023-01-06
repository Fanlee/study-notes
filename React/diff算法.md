# diff算法

在render阶段更新Fiber节点时，我们会调用`reconcileChildFibers`对比`current Fiber`和`jsx`对象构建`workInProgress Fiber`，这里current Fiber是指当前dom对应的fiber树，jsx是class组件render方法或者函数组件的返回值。

在`reconcileChildFibers`中会根据`newChild`的类型来进入单节点的diff或者多节点diff

```javascript
function reconcileChildFibers(returnFiber, currentFirstChild, newChild, lanes) {
  var isObject = typeof newChild === 'object' && newChild !== null;

  if (isObject) {
    switch (newChild.$$typeof) {
      case REACT_ELEMENT_TYPE:
        // 单一节点diff
        return placeSingleChild(reconcileSingleElement(returnFiber, currentFirstChild, newChild, lanes));
        // ...
    }
  }

  // ...
  // 多节点diff
  if (isArray$1(newChild)) {
    return reconcileChildrenArray(returnFiber, currentFirstChild, newChild, lanes);
  }
  
  // ...
  
  // 删除节点
  return deleteRemainingChildren(returnFiber, currentFirstChild);
}
```



由于`Diff`操作本身也会带来性能损耗，将前后两棵树完全比对的算法的复杂程度为 O(n 3 )，react为了降低复杂度，提出了三个前提：

- 只对同级比较，跨层级的dom不会进行复用

- 不同类型节点生成的dom树不同，此时会直接销毁老节点及子孙节点，并新建节点

- 可以通过key来对元素diff的过程提供复用的线索，例如：

  ```javascript
  const a = (
      <>
        <p key="0">0</p>
        <p key="1">1</p>
      </>
    );
  
  const b = (
    <>
      <p key="1">1</p>
      <p key="0">0</p>
    </>
  );
  ```

  

## 单节点diff

单节点diff有如下几种情况：

- key和type都相同表示可以复用

- key不同直接标记删除节点，然后新建节点

- key相同type不同，标记删除该节点和兄弟节点，然后新建节点

  ```javascript
  function reconcileSingleElement(returnFiber, currentFirstChild, element, lanes) {
    var key = element.key;
    var child = currentFirstChild;
  
    while (child !== null) {
      // 比较key是否相同
      if (child.key === key) {
        switch (child.tag) {
          case Fragment:
            // ...
          case Block:
  					// ...
          default:
            {
              if (child.elementType === element.type) {
               // type相同则表示可以复用
               // 返回复用的fiber
                return existing;
              }
              break;
            }
        }
        //key相同，type不同则把fiber及和兄弟fiber标记删除
        deleteRemainingChildren(returnFiber, child);
        break;
      } else {
        // key不同直接标记删除该节点
        deleteChild(returnFiber, child);
      }
      child = child.sibling;
    }
    // ...
  }
  ```

  当`child !== null`且`key相同`且`type不同`时执行`deleteRemainingChildren`将`child`及其兄弟`fiber`都标记删除。

  当`child !== null`且`key不同`时仅将`child`标记删除，考虑如下例子：

  当前页面有3个`li`，我们要全部删除，再插入一个`p`。

  ```javascript
  // 当前页面显示的
  ul > li * 3
  
  // 这次需要更新的
  ul > p
  ```

  由于本次更新时只有一个`p`，属于单一节点的`Diff`，会走上面介绍的代码逻辑。

  在`reconcileSingleElement`中遍历之前的3个`fiber`（对应的`DOM`为3个`li`），寻找本次更新的`p`是否可以复用之前的3个`fiber`中某个的`DOM`。

  当`key相同`且`type不同`时，代表我们已经找到本次更新的`p`对应的上次的`fiber`，但是`p`与`li` `type`不同，不能复用。既然唯一的可能性已经不能复用，则剩下的`fiber`都没有机会了，所以都需要标记删除。

  当`key不同`时只代表遍历到的该`fiber`不能被`p`复用，后面还有兄弟`fiber`还没有遍历到。所以仅仅标记该`fiber`删除。

  

## 多节点diff

多节点diff比较复杂，我们分三种情况进行讨论，其中a表示更新前的节点，b表示更新后的节点

- 属性变化

  ```javascript
  // 更新前
  const a = (
    <>
      <p key="0" name='0'>0</p>
      <p key="1">1</p>
    </>
  );
  
  // 更新后
  const b = (
    <>
      <p key="0" name='00'>0</p>
      <p key="1">1</p>
    </>
  );
  ```

- type变化

  ```javascript
  // 更新前
  const a = (
    <>
      <p key="0">0</p>
      <p key="1">1</p>
    </>
  );
  
  // 更新后
  const b = (
    <>
      <div key="0">0</div>
      <p key="1">1</p>
    </>
  );
  ```

- 新增节点

  ```javascript
  // 更新前
  const a = (
    <>
      <p key="0">0</p>
      <p key="1">1</p>
    </>
  );
  // 更新后
  const b = (
    <>
      <p key="0">0</p>
      <p key="1">1</p>
      <p key="2">2</p>
    </>
  );
  ```

- 删除节点

  ```javascript
  // 更新前
  const a = (
    <>
      <p key="0">0</p>
      <p key="1">1</p>
      <p key="2">2</p>
    </>
  );
  // 更新后
  const b = (
    <>
      <p key="0">0</p>
      <p key="1">1</p>
    </>
  );
  ```

- 节点位置变化

  ```javascript
  // 更新前
  const a = (
    <>
      <p key="0">0</p>
      <p key="1">1</p>
    </>
  );
  // 更新后
  const b = (
    <>
      <p key="1">1</p>
      <p key="0">0</p>
    </>
  );
  ```

  