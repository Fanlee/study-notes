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

     