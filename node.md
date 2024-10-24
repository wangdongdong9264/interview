# node

## 介绍一下 require 的模块加载机制

  1. 计算模块绝对路径；
  2. 如果缓存中有该模块，则从缓存中取出该模块
  3. 按优先级依次寻找并编译执行模块，将模块推入缓存（require.cache）中；
  4. 输出模块的 exports 属性；

## 请介绍一下 Node 中的内存泄露问题和解决方案

内存泄露原因

  1. 全局变量：全局变量挂在 root 对象上，不会被清除掉；
  2. 全局变量挂在 root 对象上，不会被清除掉；
  3. 事件监听：对同一个事件重复监听，忘记移除（removeListener），将造成内存泄露。

解决方案

  1. 最容易出现也是最难排查的就是事件监听造成的内存泄露，所以事件监听这块需要格外注意小心使用。
  2. 如果出现了内存泄露问题，需要检测内存使用情况，对内存泄露的位置进行定位，然后对对应的内存泄露代码进行修复。

## Node 更适合处理 I/O 密集型任务还是 CPU 密集型任务？为什么

  1. Node 更适合处理 I/O 密集型的任务。因为 Node 的 I/O 密集型任务可以异步调用，利用事件循环的处理能力，资源占用极少，并且事件循环能力避开了多线程的调用，在调用方面是单线程，内部处理其实是多线程的。
  2. 并且由于 Javascript 是单线程的原因，Node 不适合处理 CPU 密集型的任务，CPU 密集型的任务会导致 CPU 时间片不能释放，使得后续 I/O 无法发起，从而造成阻塞。但是可以利用到多进程的特点完成对一些 CPU 密集型任务的处理，不过由于 Javascript 并不支持多线程，所以在这方面的处理能力会弱于其他多线程语言（例如 Java、Go）

## 进程的当前工作目录是什么？有什么作用

  1. 进程的当前工作目录默认值是当前进程启动的目录，通过 `process.cwd()` 可以获取当前工作目录（current working directory），文件操作等使用相对路径时会相对当前工作目录来获取文件。
  2. 一些获取配置的第三方模块（例如 webpack）就是通过你的当前工作目录来获取对应的配置文件的，在程序中可以通过 `process.chdir()` 来改变当前的工作目录。

## 父进程或子进程的死亡是否会影响对方? 什么是孤儿进程

  1. 子进程死亡不会影响父进程，不过子进程死亡时，会向它的父进程发送死亡信号。反之父进程死亡，一般情况下子进程也会随之死亡，但如果此时子进程处于可运行状态、僵死状态等等的话，子进程将被 init 进程收养，从而成为孤儿进程。
  2. 另外，子进程死亡的时候（处于“终止状态”），父进程没有及时调用 `wait()` 或 `waitpid()` 来返回死亡进程的相关信息，此时子进程还有一个 PCB 残留在进程表中，被成为僵尸进程。

## koa 洋葱模型

要实现洋葱模型必要条件

1. 要把上下文ctx对象和下一个中间件next传给当前的中间件
2. 必须要等待下一个中间件执行完，再执行当前中间件的后续逻辑

```js

const middleware = []
let mw1 = async function (ctx, next) {
  console.log("next前，第一个中间件")
  await next()
  console.log("next后，第一个中间件")
}
let mw2 = async function (ctx, next) {
  console.log("next前，第二个中间件")
  await next()
  console.log("next后，第二个中间件")
}
let mw3 = async function (ctx, next) {
  console.log("第三个中间件，没有next了")
}

function use(mw) {
  middleware.push(mw)
}

use(mw1)
use(mw2)
use(mw3)

let fn = function (ctx) {
  return dispatch(0)

  function dispatch(i) {
    let currentMW = middleware[i]
    if(!currentMW) {
        return
    }
    return currentMW(ctx, dispatch.bind(null, i + 1))
  }
}
fn()

```

## pm2原理

众所周知，Node.js中的JavaScript代码执行在单线程中，非常脆弱，一旦出现了未捕获的异常，那么整个应用就会崩溃。

通常的解决方案，便是使用Node.js中自带的`cluster`（集群）模块，以master-worker模式启动多个应用实例。

问： 为什么我的应用代码中明明有`app.listen(port)`，但cluter模块在多次fork这份代码时，却没有报端口已被占用？

答：`listen`函数会根据是不是主进程做不同的操作, 答案就在这个`cluster._getServer`函数的代码中。它主要干了两件事
  
  1. 向`master`进程组册该worker，若master进程是第一次接收到监听此端口/描述符下的worker，则起一个内部tcp服务，来承担监听该端口/描述符的职责，随后在master中记录该worker
  2. `hack`掉worker进程中的`net.Server`实例的listen方法里监听端口/描述符的部分，使其不再承担该职责

问：Master是如何将接收的请求传递至worker中进行处理然后响应的？

答：负载均衡大概流程

  1. 所有请求先统一经过内部tcp服务，真正监听端口的只有主进程
  2. 在内部tcp服务器的请求处理逻辑中，由负载均衡选出一个worker进程，将其发送一个`newconn`的内部消息，消息发送客户端句柄
  3. worker进程接收到`newconn`内部消息后，根据客户端句柄创建`net,Socket`实例，执行具体业务逻辑

## 什么是守护进程？Node如何实现守护进程？

守护进程是不依赖终端（tty）的进程，不会因为用户退出终端而停止运行的进程。

node实现守护进程的思路：

  1. 创建一个进程a
  2. 在进程a中创建进程b，可以使用`child_process.fork`或其它方法
  3. 启动子进程时，设置`detached`属性为true，保证子进程在父进程退出后继续运行
  4. 进程a退出，进程由`init进程`接管。此时进程b为守护进程

## npm 隐式依赖

npm v1&v2 初版 npm 使用了很简单嵌套结构来进行版本管理

会带来的问题：

1. 项目里会反复安装相同的依赖
2. 会带来依赖地狱
3. 不同的项目之间会重复安装相同的包

npm v3

为了解决上述的问题，npm v3 完全重写了 npm 的安装程序，采用`扁平化`的方式，将主依赖项和子依赖都装到 node_modules 一级目录下

1. 不完全解决的重复依赖
2. 不确定性 （取决于用户的install顺序）
3. 隐式依赖（又名幽灵依赖）

虽然 App1 没有直接声明引用 packageB，但项目里依旧可以正常的使用。原因是 npm3 将所有的依赖都平铺到node_modules 下，因此 require 函数可以查找到它

解决 pnpm 原理软链
