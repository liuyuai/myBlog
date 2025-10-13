# 观察者模式：让别人替我盯着

> 「设计模式」系列第 2 篇 · 行为型。
> 相关：[单例模式](./singleton.md) / [AOP 面向切面编程](./aop.md)

## 什么是观察者

观察者模式定义**一对多**的依赖：一个对象（被观察者）状态变化时，自动通知所有"关注它的人"（观察者）。

场景到处都是：按钮点击、数据变化刷新视图、消息推送、自定义事件……前端几乎天天在用，只是很多人没意识到它的名字。

## 第一版：最简 EventEmitter

先写个最朴素的发布订阅（这里先不严格区分观察者/发布订阅，后面对比）：

```js
class EventBus {
  constructor() {
    this.handlers = new Map(); // 事件名 -> 回调数组
  }

  on(type, fn) {
    if (!this.handlers.has(type)) {
      this.handlers.set(type, []);
    }
    this.handlers.get(type).push(fn);
    return this; // 链式调用
  }

  emit(type, ...args) {
    const list = this.handlers.get(type) || [];
    list.forEach((fn) => fn.apply(null, args));
  }
}

const bus = new EventBus();
bus.on('login', (user) => console.log('欢迎回来', user.name));
bus.emit('login', { name: 'liuyuai' }); // 欢迎回来 liuyuai
```

能用，但缺两个高频能力：**取消订阅**和**只监听一次**。

## 第二版：加 off / once

```js
class EventBus {
  constructor() {
    this.handlers = new Map();
  }

  on(type, fn) {
    if (!this.handlers.has(type)) {
      this.handlers.set(type, []);
    }
    this.handlers.get(type).push(fn);
    return this;
  }

  once(type, fn) {
    const wrap = (...args) => {
      this.off(type, wrap); // 先取消再执行
      fn(...args);
    };
    return this.on(type, wrap);
  }

  off(type, fn) {
    const list = this.handlers.get(type);
    if (!list) return this;
    const i = list.indexOf(fn);
    if (i > -1) list.splice(i, 1);
    return this;
  }

  emit(type, ...args) {
    // slice() 复制一份再遍历：防止回调里 on/off 改数组导致遍历错乱
    const list = (this.handlers.get(type) || []).slice();
    list.forEach((fn) => fn(...args));
  }
}
```

关键细节：`emit` 里用 `slice()` 复制列表。否则回调里触发 `off` 或 `on`，正在遍历的数组被改，会出现漏通知或重复通知。

## 观察者 vs 发布订阅

严格来说两者有区别：

| 概念 | 谁通知谁 | 是否解耦 | 例子 |
| --- | --- | --- | --- |
| 观察者模式 | 被观察者**直接**通知观察者 | 观察者要注册到被观察者身上 | Vue2 响应式：Dep 通知 Watcher |
| 发布订阅 | 发布者 → 事件中心 → 订阅者 | 双方完全不见面，靠事件名中转 | DOM 事件、EventBus |

实现上经常混着写，但理解这个区别，看源码时能更快定位设计意图。

## 应用：DOM 事件

其实 `addEventListener` 就是观察者/发布订阅思想的官方实现：

```js
const btn = document.getElementById('btn');
const handler = () => console.log('clicked');

btn.addEventListener('click', handler); // 订阅
btn.removeEventListener('click', handler); // 取消订阅，防止内存泄漏
```

## 注意（坑）

1. **忘记 off = 内存泄漏**：组件销毁后回调还挂在事件中心上，对象永远不被回收。组件卸载时一定要清理订阅；
2. **回调异常会中断通知链**：一个回调抛错，后面的订阅者全收不到。生产代码建议给每个回调包 try/catch；
3. **循环通知**：A 通知 B，B 又通知 A，可能无限循环，要设计好触发条件或加深度限制；
4. **通知顺序**：JS 里一般按注册顺序执行，但**不要依赖顺序**写业务逻辑；
5. **事件名冲突**：字符串事件名容易撞车，规范命名（如 `user:login`）或改用 Symbol。

## 总结

观察者/发布订阅是前端最实用的模式：**一对多、解耦、可动态增删**。记住两件事：`once` 和 `off` 是标配；忘了解除订阅会漏内存。

「设计模式」两篇到此：单例管"唯一"，观察者管"通知"。后续可继续扩展工厂、策略等，索引见 [README](../README.md)。
