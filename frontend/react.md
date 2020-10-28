# react原理

## 做了什么
1. Virtual Dom, 虚拟DOM树。
    1. 所有操作都是在操作虚拟DOM树，经过diff算法，算出其差异，然后再进行实际的dom操作。
1. 生命周期管理。
    1. 三大类
        1. 挂载mount。
        1. update。
        1. unmount卸载。
    1. 前两类都有will和did节点
    1. 最后一类只有will，然后组件就被卸载了
1. setState机制
    1. 异步set进state修改队列，不直接修改this.state.
    1. 满足一定条件时，react会合并所有修改，触发一次update流程，更新 this.state.
    1. 因此减少了update触发次数。
    1. 因为setState会触发update钩子，所以在update钩子中调用setState会存在循环调用的风险。
    1. *特殊情况*
        1. 在mount钩子中调用setState，不会进入update流程，而是在mount时合并修改并render。
        1. 在setTimeOut/promise回调中调用，将不会进行队列的批更新，而是直接触发一次update流程。
1. diff算法
    1. 用于计算新旧虚拟DOM的差异，是消耗最大的地方。
    1. 传统diff算法直接遍历DOM树进行比较，消耗过大，超过原生消耗。
    1. react diff算法制定了3条策略，将传统diff算法的时间复杂度从O(n3)提升到了O(n)。
        1. WebUI中DOM节点跨节点的操作特别少，可以忽略不计。
        1. 拥有相同累的组件会拥有相似点DOM结构。反之。
        1. 同一层级的子节点，可以根据唯一的ID来区分。
    1. react diff实施的具体策略是
        1. diff对树进行分层比较，只对比两棵树同级别的节点。
            1. 跨层级移动节点，将会导致节点删除，重新插入，无法复用。
            1. 会影响React性能，所以不要进行DOM节点的跨层级操作（将子级移到祖父级）
            1. 在开发组件中，保持稳定的 DOM 结构有助于性能的提升。例如，可以通过CSS隐藏或显示节点，而不是真正的移除或添加 DOM 节点。
        1. diff对组件进行类比较，类相同的递归diff子组件，不同的直接销毁重建。
        1. diff对同一层级的子节点进行处理是会根据key进行简要复用。例：两颗树存在相同key的节点时，只会移动节点。
1. React Patch、事件系统（类似setState，内涵diff算法）
    1. React Patch实现了最后这关键的一步，将tree diff算法计算出来的差异队列更新到真实的DOM节点上，最终让浏览器能够渲染出更新的数据。
    1. Patch主要是通过遍历差异队列实现的，遍历差异队列时，通过更新类型进行相应的插入、移动和移除等操作。
    1. React并不是计算出一个差异就执行一次patch，而是计算出全部的差异并放入差异队列后，再一次性的去执行Patch方法完成真实的DOM更新。


## 根据原理怎么进行优化
1. 减少diff算法触发次数（=减少update的次数）
    1. 在基础使用setState已经做过批更新策略，已经降低了update的触发次数了。
        1. 在timeOut和promise中，减少setState的触发次数，例，在处理接口回调中，无论数据处理多么复杂，只在最后调用一次setState
    1. 父组件render
        1. 父组件render，子组件必然进update，无论props是否更新。
        1. 此时常用shouldComponentUpdate方法，在其中进行this.props和this.state的浅比较来判断组件是否需要更新。或者用 PureComponent
    1. forceUpdate
        1. 调用后会直接进入update阶段，应该少用或者弃用。
    1. 其他
        1. 在shouldComponentUpdat中处理下不用渲染的props改变导致的update
        1. 避免在shouldComponentUpdat中做一些比较复杂的操作，比如超大数据比较等。
        1. 合理设计state, 不需要渲染的state，尽量变为实例变量。
        1. 不需要渲染的props，合理使用context机制，或公共模块的变量来替换。
            *context提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。 类似hooks，这个针对的props，hooks针对的是state*
            *vue类似的叫注入provide 和 inject*
1. 正确使用diff算法
    1. 不适用跨级移动节点的操作。
    1. 对于条件渲染多个节点时，尽量才用隐藏等方式切换节点，而不是替换节点。
    1. 尽量避免将后面的子节点移动到祖父级等，一定会产生性能问题。
    