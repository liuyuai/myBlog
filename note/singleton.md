# 单例模式：全局唯一的"那个人"

> 「设计模式」系列第 1 篇 · 创建型。
> 相关：[AOP 面向切面编程](./aop.md) / [观察者模式](./observer.md)

## 什么是单例

单例模式保证**一个类只有一个实例**，并且提供一个全局访问点。

什么时候需要它：配置对象、日志器、数据库连接池、登录态管理……这类东西全应用只有一份，多了反而乱。

## 第一版：class + static 实例

最简单的想法：实例存在类上，构造时检查，已存在就直接返回它。

```js
class Config {
  static instance = null;

  constructor() {
    if (Config.instance) {
      return Config.instance;
    }
    this.theme = 'light';
    this.locale = 'zh-CN';
    Config.instance = this;
  }
}

const a = new Config();
const b = new Config();
console.log(a === b); // true
```

思路没错，但有两个问题：

1. `static instance = null` 是 ES2022 的类静态字段语法，老环境不支持；
2. 把实例挂在类上，全局裸奔，测试时不好替换（mock）。

## 第二版：闭包隐藏实例

把实例藏进闭包，只暴露一个构造入口，外部拿不到 `instance` 本身：

```js
const Config = (function () {
  let instance = null;

  return function () {
    if (instance) {
      return instance;
    }
    this.theme = 'light';
    this.locale = 'zh-CN';
    instance = this;
  };
})();

const a = new Config();
const b = new Config();
console.log(a === b); // true
```

`instance` 变成了模块私有，外面只能通过 `new Config()` 拿，没法直接改。

## 第三版：显式 getInstance + 惰性初始化

很多人更喜欢显式的获取方式，实例**第一次用到才创建**（惰性初始化）：

```js
class Config {
  constructor() {
    this.theme = 'light';
    this.locale = 'zh-CN';
  }

  static getInstance() {
    if (!Config._instance) {
      Config._instance = new Config();
    }
    return Config._instance;
  }
}

const a = Config.getInstance();
const b = Config.getInstance();
console.log(a === b); // true
```

## 三种写法对比

| 写法 | 实例是否私有 | 惰性加载 | 老环境兼容 | 测试替换 |
| --- | --- | --- | --- | --- |
| class + static 字段 | 否（类上公开） | 否（构造即建） | 否（ES2022） | 难 |
| 闭包 IIFE | 是 | 否 | 是 | 较难 |
| getInstance | 是（类私有字段） | 是 | 是 | 相对容易 |

## 注意（坑）

1. **单例本质是隐式全局**：任何模块都能拿到同一个对象，互相改来改去，状态混乱时很难排查是谁改的；
2. **测试困难**：单例跨用例残留状态，跑测试前要记得重置；上面三版都可以加个 `reset()` 方法用于测试；
3. **JS 单线程，不用考虑加锁**：Java 里的双检锁、`synchronized` 在 JS 里是伪需求，别照搬；
4. **能用模块就优先用模块**：ES Module 本身就是天然单例（模块只加载一次），很多场景直接 `export const config = {...}` 就够了，不需要写类；
5. **与依赖注入对比**：单例是"自己管自己"，依赖注入是"外部给我传"，后者在大型应用里更好测试、更灵活。

## 总结

单例 = 一个实例 + 全局访问。JS 里最常见的实现是**闭包私有变量 + 惰性初始化**；能用模块导出就尽量别写类。

下一篇：[观察者模式](./observer.md)——一对多通知，前端事件系统的核心。
