Vue 响应系统，其核心有三点：`observe`、`watcher`、`dep`：

1. `observe`：遍历 `data` 中的属性，使用 [Object.defineProperty](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FObject%2FdefineProperty) 的 `get/set` 方法对其进行数据劫持；
2. `dep`：每个属性拥有自己的消息订阅器 `dep`，用于存放所有订阅了该属性的观察者对象；
3. `watcher`：观察者（对象），通过 `dep` 实现对响应属性的监听，监听到结果后，主动触发自己的回调进行响应。

