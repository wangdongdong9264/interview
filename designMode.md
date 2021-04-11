# 设计模式

## 发布/订阅模式

也叫做观察者模式

定义：

  对象间的一种一对多的关系，让多个观察者对象同时监听某一个主题对象，当一个对象发生改变时，所有依赖于它的对象都将得到通知

优点：

  1. 支持简单的广播通信，当对象状态发生改变时，会自动通知已经订阅过的对象
  2. 发布者与订阅者耦合性降低，发布者只管发布一条消息出去，它不关心这条消息如何被订阅者使用，同时，订阅者只监听发布者的事件名，只要发布者的事件名不变，它不管发布者如何改变

缺点：

  1. 创建订阅者需要消耗一定的时间和内存。
  2. 虽然可以弱化对象之间的联系，如果过度使用的话，反而使代码不好理解及代码不好维护

```js

class EventEmitter {
  constructor() {
    // 事件对象，存放订阅的名称和事件
    this.events = {}
  }
  // 订阅事件的方法
  on(eventName,callback) {
    // 注意，一个名字可以订阅多个事件函数
    if(!this.events[eventName]) {
      this.events[eventName] = [ callback ]
    } else {
      this.events[eventName].push(callback)
    }
  }
  // 触发事件的方法
  emit(eventName) {
    // 遍历执行所有订阅的事件
    this.events[eventName] && this.events[eventName].forEach(cb => {
      cb()
    });
  }
  // 移除订阅模式
  removeListener(eventName,callback) {
    if (this.events[eventName]) {
      this.events[eventName] = this.events[eventName].filter(cb =>  cb != callback)
    }
  }
  // 只执行一次的订阅事件，然后移除
  once(eventName,callback) {
    // 绑定fn， 执行的时候触发fn
    let fn = () => {
      callback()
      this.removeListener(eventName, fn)
    }
    this.on(eventName, fn)
  }
}

// 使用
const em = new EventEmitter()
let workday = 0
// 添加订阅
em.on('work', function() {
  workday++
  console.log('work everyday');
})
em.once('love', function() {
  console.log('just love you')
})
function needMoney() {
  console.log('need more money');
}
em.on('money', needMoney)

// 发布
em.emit('work')
em.removeListener('money', needMoney)
em.emit('money')
em.emit('love')

// work everyday
// just love you

```
