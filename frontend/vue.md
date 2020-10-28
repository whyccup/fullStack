# vue原理
1. 虚拟DOM树。通过document.createDocumentFragment(),遍历指定根节点内部节点，根据prop，v-model等规则进行更新。
1. 通过Object.defineProperty()进行数据变化拦截。
1. 截取到数据变化，动过发布者-订阅者模式，触发watcher，从而改变虚拟DOM中的具体数据
1. 通过改变虚拟DOM元素值，从而改变最后渲染DOM树的值，完成双向绑定。
 [看](https://segmentfault.com/a/1190000018659584)
 [看](https://cn.vuejs.org/v2/guide/reactivity.html)