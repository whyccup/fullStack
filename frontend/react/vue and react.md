# vue 与 react 的相似点与区别

## 相似点
1. Virtual DOM （虚拟DOM，DOM树的虚拟表现）
    1. 改变真实的DOM状态远比改变一个JavaScript对象的花销要大的多。
    1. 当需要改变任何元素时，先在虚拟DOM上进行改变，并计算新旧DOM直接的差别，再把这些差别渲染到真实的DOM上。
1. 组件化
    1. 都建议将你的应用拆分成一个个功能模块。
1. Props
    1. 都有props的概念，properties（属性）的简写。用来进行父到子传参。
    1. 同样在组件中被定义。
1. 都有自己的构建工具，且可以快速搭建开发环境。
1. 都有Chrome开发扩展工具。
1. 配套框架，都是官方+社区维护



## 相似点中的区别点
1. 计算新旧虚拟DOM树的算法不同。
    1. Vue - 在渲染过程中，跟踪每一个组件的依赖关系，不重新渲染整个组件树。
    1. React - 每当应用的状态改变是，全部子组件都会重新渲染，可以通过shouldComponentUpdate这个生命周期来进行控制，但Vue将此视为默认的优化。
1. 组件写法不同
    1. vue - 模板语法，格式类似常规HTML，可以更容易的从老应用，迁移到新应用。
    1. react - JSX语法，是将HTML标签写入js对象或者方法中。
1. Props
    1. vue - 因为模板语法，可以更高效地传入的数据。比如v-for循环的数据，可以直接写在标签中传输。
    1. react - 必须传递，因为它依赖一个“单一数据源”作为它的“状态”。
1. 构建工具
    1. vue - vue-cli
        1. 按需创建不同模板，更灵活。 
    1. react Create React App (CRA)
        1. 选项多，但工具会迫使你使用webPack和babel。
1. 配套框架，常用部分框架维护团队不同。
    1. vue-router和vuex由vue官方维护。
    1. react-router和react-redux则由社区成因维护。

# 区别点
1. 模板编写。
    1. vue
        1. 鼓励按照常规HTML规范编写，只是元素多了一些属性。
        1. 鼓励使用HTML模板进行渲染，动态输出内容。
        1. 很容易使旧的应用升级。
    1. react
        1. Js扩展语法——JSX书写。(其实vue也支持render和JSX，但是非默认。)
        1. 赋予可开发者许多编程能力。
        1. 书写起来符合编程直觉。
1. 对象属性 vs 状态管理 
    1. vue
        1. 数据由data属性在Vue对象中进行管理。
        1. 直接改变data参数中的属性就可以更新状态。
    1. react
        1. 状态是React关键的概念，state对象在react中式不可变得，除了初始化时，其他任何时候不推荐、不能改变。
        1. 使用固定方法 setState() 方法去更新状态。
    1. 中大型应用一般必须要使用 Vuex 或 Redux 等状态管理方案。框架内置状态是不足以支撑大型应用的。
1. React Hooks
    1. 不再需要使用ES2015类语法来创建组件，可以直接写成函数就可以使用react特性。
    1. 减少了编写的代码
        1. 以前要先写继承，super，然后初始化state，然后写几个修改state的setState方法，最后再写个render，然后挂载在某个路由或者作为组件被引用使用。
        1. 现在直接用 const [someStateValue, setSomeState] = React.useState(), 就可以获取到指定的数据状态和修改状态的方法，再直接写render或return，一个组件就完成了。
    1. 增加了新的钩子
        1. useState()
            1. 解构出来的someStateValue是状态属性, setSomeState是该变量的“setter”函数。
        1. useEffect() 副作用
            1. 写在组件内可以直接访问state变量，或者其他Porps
            1. 在第一次渲染之后和每次更新之后都会执行。
            1. 不必再考虑是挂载 componentDidMount 还是更新 componentDidUpdate
            1. 清除副作用相当简单，只需要让副作用返回一个函数，即可在组件被清除时，清除这个副作用。
            1. 可以声明多个副作用，React会依次调用，用来分割不同功能的代码。
            1. 可以处理解决一个常见bug，忘记处理componentDidUpdate 钩子。
            1. effect每次渲染都会调用会出现性能问题，记得要做一些额外的跳过处理。
    
1. vue有类似React的Hooks吗？
    1. 在vue3的新功能中有一个类似的，叫做 Composition API，合成API。当然vue3还没发布。
    1. 可以将vue的多个钩子组合成一种方法，允许更紧密的耦合，并简化可重复性。

