# react设计理念

## 异步可中断

因为js的执行是单线程的，在执行比较耗时的任务会阻塞浏览器的渲染

- 解决方案

  任务分割，异步执行，并且能够让出执行权

  1. Fiber: react15更新是同步的， 因为它不能将*任务分割*，所以需要一套数据结构让它既能对应真实的dom又能作为分割的单元

     ```javascript
     let firstFiber
     let nextFiber
     let shouldYield = false
     
     function performUnitOfWork(nextFiber){
         return nextFiber.next
     }
     
     function workLoop(deadline) {
         while(nextFiber !== null && !shouldYield) {
             nextFiber = performUnitOfWork(nextFiber)
             shouldYield = deadline.timeReaming() < 1
         }
         
         requestIdleCallback(workLoop)
     }
     
     requestIdleCallback(workLoop)
     ```

     

2. Scheduler：有了Fiber，我们就需要用浏览器的时间片==异步执行Fiber的工作单元==，react通过模拟requestIdleCallback的实现机制，在浏览器空闲的时候执行一部分任务，让高优先级的任务优先响应，因为requestIdleCallback存在浏览器兼容问题和触发不稳定性，所以自己实现了一套时间片运行的机制，在react中这部分就叫做Scheduler
3. Lane：更加细粒度的管理各个任务的优先级，让高优先级的任务优先执行，各个Fiber工作单元还能比较优先级，相同优先级的任务可以一起执行

- 上层实现

  有了这一套异步可中断更新，就可以实现batchUpdates批量更新和Suspense，为什么不通过generator来实现程序的暂停和恢复，因为generator暂停之后恢复执行，还是得把执行权交给直接 调用者，调用者会沿着调用栈继续上交，所以也是有传染性，并且generator不能计算优先级，排序优先级

```jsx
const ProductResource = createResource(fetchProduct)
const Product = props => {
    // 用同步的方式来编写异步的代码
    const p = ProductResource.read(props.id)
    return <h1>{p.price}</h1>
}

function App() {
    return (
     <div>
        <Suspense fallback={<div>loading...</div>}>
        <Product id={1}/>
     </div>
    )
}
```

可以看到ProductResource.read完全是同步的写法，把获取数据的部分完全分离出了Product组件之外，在源码中，ProductResource.read会在获取数据之前throw一个特殊的Promise，scheduler可以捕获这个promise，暂停更新，等数据获取之后交还执行权