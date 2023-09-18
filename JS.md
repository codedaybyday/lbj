## 事件循环

原因：渲染主进程是个单进程，处理一些长时任务为了不阻塞线程，会选择将他们放入到任务队列
，任务队列是一个单独的进程，在没有任务时处理休眠状态，一旦有任务被放入，就会被唤醒，进行一个无限循环的状态，后面有任务进入则要开始排队

宏任务： setTimeout,setInterval,setImmediate, I/O, UI Rendering

微任务: MutationObserver,ResizeObserver,Promise.then, process.nextTick

事件循环的执行顺序：

1. 首先，执行主线程上的同步代码。
2. 当同步代码执行完毕后，检查微任务队列。如果微任务队列中有任务，执行所有的微任务，直到微任务队列为空。
3. 接着，从宏任务队列中取出一个任务并执行。注意，这里只执行一个宏任务。
4. 在执行完当前宏任务后，再次检查微任务队列。如果微任务队列中有任务，执行所有的微任务，直到微任务队列为空。
5. 重复步骤 3 和 4，直到宏任务队列为空。

promise.then回调被推入队列的时机：resolved或者reject时

.then/.catch可以执行多次
```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('timer')
    resolve('success')
  }, 1000)
})
const start = Date.now();
promise.then(res => {
  console.log(res, Date.now() - start)
})
promise.then(res => {
  console.log(res, Date.now() - start)
})
// timer
// success 1000
// success 1001
```
上题误区：分别执行两次promise.then，前一个会覆盖后一次，可结合内部实现理解，没调用一次then会在该对象内部的```[[PromiseFulfillReactions]]```存储回调

then里面返回任何非promise的值内部都会包装成：Promise.resolve(val);
```js
Promise.resolve().then(() => {
  return new Error('error!!!')
}).then(res => {
  console.log("then: ", res)
}).catch(err => {
  console.log("catch: ", err)
})

```
注意不是throw new Error(); 别想错了

结合源码理解下：then参数里面如果是个函数，则会发生透传

## 网络
