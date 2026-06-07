# Interview

## JavaScript

### 事件循环

#### 说说你对同步和异步的理解？

同步是按顺序一件一件做，前一个不结束，后一个不开始；异步是先登记任务，主线程不停，等时机到了再回调执行。

**1. 先看同步**
特点：
1. 代码直观，控制流好理解。
2. 遇到耗时操作会卡住后续逻辑。
3. 在浏览器里如果做了重 CPU 任务，会卡 UI；在 Node.js 里会卡住整个事件循环，影响并发请求。

**2. 再看异步**
异步不是“同时执行”，而是“任务调度”。JavaScript 主线程先把耗时工作交给宿主环境（浏览器或 Node），自己继续往下跑，等结果回来再处理。

常见异步来源：
1. 定时器（setTimeout / setInterval）
2. 网络请求（fetch / XHR）
3. I/O（文件、数据库、socket）
4. 用户事件（click、input）

#### 说说你对事件循环的理解

事件循环（Event Loop）的本质，是让 JavaScript 在“单线程”下也能协调很多异步任务，不阻塞主线程。

同步代码先执行，异步回调先登记；主线程空了就从队列取任务执行，并在合适时机更新页面。

执行一个宏任务（比如整段 <script>、一个 click 回调、一个 timeout 回调）。
这个宏任务执行期间，可能产生新的微任务（如 Promise.then）。
当前宏任务结束后，立刻清空微任务队列（全部执行完，期间新增的微任务也要继续执行，直到队列为空）。
微任务清空后，浏览器有机会做一次页面渲染（layout/paint）。
进入下一轮，从宏任务队列再取一个任务执行，重复以上步骤。

#### Promise

Promise 是 JS 里处理异步的标准方案，本质是一个“未来结果”的容器，有 pending、fulfilled、rejected 三种状态，而且状态一旦改变就不可逆。
它主要解决了回调地狱、错误处理分散和异步流程难组合的问题，让异步代码可以链式调用、统一 catch、并发控制更清晰。
常用实例方法有 then、catch、finally。
常用静态方法有 resolve、reject、all、race、allSettled、any。
all 是“全部成功才成功，任一失败就失败”；race 是“谁先结束就用谁的结果，不管成功还是失败”。

加分一句（可选）：
async/await 只是 Promise 的语法糖，底层还是基于 Promise。

#### 常见相关面试题

1. 什么是事件循环？它解决了什么问题？  
标准回答：  
JavaScript 是单线程，事件循环是运行时的调度机制，用来协调同步代码、异步回调和渲染。核心流程是：执行一个宏任务，清空微任务，再进入下一轮宏任务。它解决的是“单线程下如何不阻塞地处理异步任务”。

2. 宏任务和微任务分别有哪些？执行顺序是什么？  
标准回答：  
宏任务常见有整体 script、setTimeout、setInterval、I/O、UI 事件回调。微任务常见有 Promise.then/catch/finally、queueMicrotask、MutationObserver。顺序是：当前宏任务执行完后，先清空所有微任务，再执行下一个宏任务。

3. 为什么 Promise.then 通常比 setTimeout(fn, 0) 先执行？  
标准回答：  
Promise.then 属于微任务，setTimeout 属于宏任务。事件循环在当前宏任务结束后会先执行微任务，再取下一轮宏任务，所以 Promise.then 先于 setTimeout(fn, 0)。

4. setTimeout(fn, 0) 是“立即执行”吗？  
标准回答：  
不是。0ms 只是“最短延迟”，表示回调最早在下一轮宏任务执行。它仍要等待当前调用栈清空、当前轮微任务清空，以及调度时机到达。

5. async/await 在事件循环里是怎么执行的？  
标准回答：  
await 之前是同步执行。遇到 await 后，函数会暂停并让出线程，await 后续代码会被放入微任务队列（本质是 Promise continuation）。因此 await 后代码通常在当前宏任务结束后的微任务阶段执行。

6. 请判断下面代码输出顺序并解释：  
代码：  
console.log('1');  
setTimeout(() => console.log('2'), 0);  
Promise.resolve().then(() => console.log('3'));  
console.log('4');  
标准回答：  
输出是 1 4 3 2。  
解释：同步先输出 1、4；Promise.then 进微任务，setTimeout 进宏任务；同步结束后先跑微任务输出 3，再下一轮宏任务输出 2。

7. 微任务会不会导致页面卡顿？为什么？  
标准回答：  
会。如果微任务不断自我追加（比如 then 里继续 then），事件循环会长期停在“清空微任务”阶段，渲染和下一轮宏任务都被推迟，表现为页面卡顿或假死。这个现象常叫“微任务饥饿”。

8. 浏览器中的 requestAnimationFrame、setTimeout、微任务三者关系是什么？  
标准回答：  
微任务在当前宏任务结束后立即执行。requestAnimationFrame 在下一次重绘前执行，适合动画。setTimeout 是宏任务，会在后续事件循环中调度。一般来说，微任务优先于下一轮宏任务；rAF 与渲染时机绑定，不等同于微任务。

9. Node.js 事件循环有哪些阶段？  
标准回答：  
常见阶段是 timers、pending callbacks、idle/prepare（内部）、poll、check、close callbacks。setTimeout 在 timers 阶段，setImmediate 在 check 阶段，I/O 主要在 poll 阶段处理。Node 的事件循环是分阶段推进，不是简单“一个宏任务队列”。

10. Node.js 里 setTimeout(fn, 0) 和 setImmediate(fn) 谁先执行？  
标准回答：  
没有绝对固定答案，取决于上下文。顶层代码中顺序可能不稳定；在 I/O 回调内部，setImmediate 往往先于 setTimeout(fn, 0)。“取决于事件循环阶段和调用位置”。
