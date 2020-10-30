# vue

## vue指令，及渲染优化问题

### v-show 与 v-if

1. 本质区别
    1. v-if是动态的向DOM树内添加或者删除DOM元素
    1. v-show本质是利用标签的display属性，通过visibility和none控制显隐
    1. v-if=“false”在DOM不能获取到该标签，v-show=false可以在DOM中获取到
1. 编译区别
    1. v-show其实是在控制css
    1. v-if切换有一个局部编译\卸载的过程，切换过程中合适地重建或销毁内部的事件监听和子组件
1. 编译的条件
    1. v-show都会编译，初始值为false，只是将display设为none，但它也编译了（在dom创建了）
    1. v-if初始值为false，就不会编译（未在dom创建）
1. 性能
    1. v-show只编译一次，后面其实就是控制css，
    1. v-if是完整的销毁和重新创建
    1. 所以在切换功能上v-show性能更好点
    1. 但，若需求是进入快，切换少；或者进入时缓慢，要进行优化，切换的开销可以承受；此时就应该将v-show替换为v-if来优化进入速度。！！extra 
1. 与 css display:none 和 visibility:hidden 的区别
    1. 相同点：都是隐藏标签，但对应的标签仍存在DOM中，类似v-show
    1. 不同点：
        1. 设置 display:none 后，不会占据标签原来所在的位置，会触发重排 repaint (其实就是 v-show 的实现方式)
            1. 重排消耗极大，会触发重排的有
                1. 添加或删除可见的DOM元素
                1. 元素位置改变
                1. 元素本身的尺寸发生改变
                1. 内容改变
                1. 页面渲染器初始化
                1. 浏览器窗口大小发生改变
                1. 获取offset
            1. 如何优化
                1. 最小化，给DOM赋内联样式时，不要一条命令一条命令的赋值，应该将其合并成一条
                  1. el.style.borderLeft = '1px';el.style.borderRight = '2px';el.style.padding = '5px'; => el.style.cssText = 'border-left: 1px; border-right: 2px; padding: 5px';
                      1. 同理，改类名
                1. 批量修改DOM时，让元素脱离文档流，对其进行多重改变，再将其放回文档中
                  ```
                  let fragment = document.createDocumentFragment();
                  appendNode(fragment, data);
                  ul.appendChild(fragment);
                  ```
                1. 避免每次操作获取offset，应在文档渲染完毕后，获取保存，再进行操作
        1. 设置 visibility:hidden 后，会占据位置，但是隐藏掉元素，只会触发重绘 reflow
            1. 重绘消耗相对较小，会触发重绘的有
                1. 改变背景
                1. visibility:hidden

### v-model，其实就是语法糖

  1. 表单控件上创建双向数据绑定，它会根据控件类型自动选取正确的方法来更新元素
      1. input 中 type text 的 v-model === :value=val + @input=>val=$event.target.value
      1. checkbox 绑定的 checked 的值；多个绑定的是value组成的数组。
      1. radio 绑定的是选中的哪一项的
      1. select & option 绑定的是选中的那项的value或内容，multiple 会让绑定的数据变为数组
      1. textarea 就是其 value

###  v-for，列表渲染

  1. 写法类似es6的
      1. for...of(循环可迭代对象，包括 Array，Map，Set，String，TypedArray，arguments)，
      1. for...in（任意顺序遍历一个对象的除Symbol以外的可枚举属性，对象的属性基本都是可枚举的，但除过Object.defineProperty 等定义的属性）
      1. v-for="item in/of items"
  1. 为什么v-for必须加key呢？
      1. 因为vue的虚拟DOM的Diff算法，只会对同层dom树进行对比，然后依次进行修改。
      1. **若没有加key，且插入一个DOM，则Diff算法会从左边头部往右依次比较，并依次更新，相当没有效率**。
      1. 所以需要一个key作为Diff算法的标识，它就可以正确识别此节点，找到正确的位置进行插入、更新。

### v-on:，绑定事件（缩写@）

1. 绑定事件都是在dom上完成的，但所有的事件处理方法和表达式，都严格绑定在当前视图的viewModel上
1. 常用有 click，dbclick, keydown, keyup, mouseover，mouseout, mousedown, contextmenu
    1. 事件修饰符（可以串联）
        1. native，原生修饰
        1. stop，阻止事件继续传播，阻止冒泡，也用作禁止传入的方法执行，此时可以传下参数
            1. 事件冒泡，当元素的事件被触发以后，会将事件转发给父级，层层传递一直到最顶层。
        1. prevent，阻止原生事件 
        1. capture，事件捕获，在事件穿透到内部元素时进行捕获，然后再发往内部（内部执行完又冒泡上来）
        1. self 只处理自身触发的事件，对于冒泡之类的不管
        1. once 事件只会触发一次
        1. passive 不阻止默认事件触发，并使默认事件不再等待自定义事件执行完毕，直接立即执行
    1. 按键修饰符
        1. enter 回车，常用于`<input v-on:keyup.enter="submit">`，可以使用户按下回车触发提交事件
        1. ...KeyboardEvent.key 下任意有效按键名转为 kebab-case 来作为修饰符。
          1. tab，delete，esc，space，up，down，left，right，ctrl，alt, shift, meta
          1. 鼠标 .left .right .middle

### v-bind:, 绑定属性（缩写：）

1. :src="imgUrl" // imgUrl 为 data 中定义的数据名
1. 在绑定class 或 style时，支持其他类型的值，如数组或对象，处理某些动态样式改变非常好用
    1. :class="{red: isRed}" // red就是类名，isRed是data中定义的数据名，值为true或false
    1. :class="[classA, classB]" // 都是数据名，他们的值是类名
    1. :class="[classA, {classB: isB}]" // 混合写法
1. 没有属性名时，可以绑定到一个包含键值对的对象，可以一次性赋予多种属性及其值
    1. v-bind="{id: id, style: {width: 10px}}"
    1. 类似 react 的
      
        ```
        <input {...attrs}> // attrs={id: id, style: {width: 10px}}
        ```
1. 通过prop修饰符绑定DOM属性
    1. v-bind:text-content.prop="text"
1. 在绑定prop时，prop 必须在子组件中声明。可以用修饰符指定不同的绑定类型。用于父传子
    1. :prop="someProp"
1. **::**
    1. :: 与 : 没有关系，前者会按js语法解析“”中的值，后者是v-bind的缩写，会将其进行数据绑定
    1. :: 会按照js语法来解析""中的值，"1"就会变成1, "[1,2,3]"就会变成[1,2,3]，普通属性只会解析成字符串

### v-once 数据只绑定一次，只渲染元素和组件一次，随后的重新渲染，元素/组件及其所有的子节点，都被视为静态内容并跳过。

1. v-once 可以使用在单个元素，有子元素，组件中
1. 还可以搭配v-for使用，比如此列表只会修改一次，就可以使用v-once

### v-html 更新元素的innerHTML, 相当于内容都会放在元素内部，且按普通HTML插入， - 不会作为 Vue模板进行编译。

1. **若想修改插入HTML的样式，一定不能在有scope属性的style标签内**
  
### v-text 更新元素的textContent

1. ```<span v-text="msg"></span>```
1. ```<span> {{msg}} </span>```
1. 两者相同

## 怎么写一个子组件，怎么引用
  
1. template 标签，内部写html
1. export default {name, components, data(){return {}}, computed // 计算属性, methods，watch, 生命钩子}
1. import as from '', components: {as} // 父组件引入后解构定义标签名，用标签名在模板中使用

## vue组件传值

1. 父组件向子组件传值
    1. 父组件在调用的子组件标签上加属性，子组件通过props来获取
        1. 子组件可以在prop中定义传入属性的类型
            1. 普通类型，String，Number，Boolean，Null
            1. 引用类型，Array，Object
    1. 基于vue单向数据流，子组件是不能修改prop中的值，即便传来的是一个对象
        1. 除非把prop的值附给data中的一个变量，然后再改变那个变量
            1. **这种做法会有问题，当父组件修改属性之后，虽然子组件中的prop改变了，但关注的这个变量没有变化，因为变量只在创建子组件时进行的赋值。**
            1. **若要处理这种问题，则需要用watch 判断prop，在prop改变后，再次给变量赋值**
1. 子组件向父组件传值
    1. 自定义事件及emit
        1. 父组件给子组件上绑定一个自定义事件，子组件用$emit触发这个事件
        1. 通过回调函数，在父组件内部定义一个函数，然后通过prop把方法传输过去，子组件再在合适的地方进行调用
    1. 通过$parent/$children或$refs访问组件实例，然后用实例上的内容及方法
    1. $attrs
        1. 父组件中对子组件标签进行数据绑定
        1. 子组件可以从$attrs中获取到上层传递的所有数据，当声明了prop之后，$attrs会包含剩余的所有属性
        1. 孙组件获取值
            1. 在子、孙组件层设置inheritAttrs为false，它会使属性不会当做html的属性渲染，且可以使自组件在声明周期内通过 this.$attrs 获取里面的值
            1. 此时对孙组件进行数据绑定 v-bind:$attrs （这里面也会有子组件给孙组件标签上绑定的属性）
            1. 这样孙组件就可以直接通过  this.$attrs 获取里面的值
    1. $listeners
        1. 同上，只不过$listeners获取的是上层组件的事件（方法）
1. 兄弟之间传值
    1. $emit 和 props结合，逐层传递
    1. 制作一个空vue（bus），每次传值或接收值时，引入这个vue文件
        1. 获取值的通过$on监听bus回调
        1. 传值的通过$emit传值
    1. vuex
1. 更多层组件传值
    1. provide 和 inject
        1. provide 祖先组件的实例 （祖先组件，provide() {retrun {parentObj: this}}）
        1. 在子孙组件中注入依赖 （子组件，inject:{parentObj})
        1. 缺点： 会在实例上挂载很多没必要的东西，比如prop，methods
            1. Vue.observable( object ) 与 react hook比较像，可以用来优化响应式 provide
    1. vuex

## mixins

1. 类似与组件引用，不过打破了组件之间的隔和，像合体！！！
    1. 单纯组件引用：父组件 + 子组件 => 父组件 + 子组件
    1. mixins：父组件 + 子组件 => new父组件
1. 将data、method、computed、生命钩子、引用的组件混入进组件中
    1. **当命名冲突时，都是以组件为标准的，混入的就被忽略掉了**
    1. **钩子例外，因为它是方法，所以会被混成一个数组方法，并按照混入钩子，自身钩子的顺序执行！！**
    1. 这里都才用的是简单覆盖已有值的策略，如果想自定义如何合并，则有方法 Vue.config.optionMergeStrategies.[方法名] = function(toVal=组件, fromVal=被混入组件, Vue实例上下文) {// 返回合并后的值}
1. 全局混入（慎用，在大型项目中资源消耗太大）
    1. 混入后每一个实例都有
        1. 一般把请求接口封装方法、简单工具函数混入等等


## 计算属性 computed  与 侦听属性 watch

1. 相同点
    1. 自动监听依赖变量，从而动态的返回内容。在监听的值变化时，可以触发一个回调，并做一些事。
1. 两者区别
    1. 计算属性 只返回动态值
    1. watch 知道值以后，可以进行业务逻辑执行
1. 原理 （劫持早在生成变量之时就已经做了）
    1. 在创建后，获取一遍内部变量，并在其内部的 Observe 进行 作为watcher 进行订阅者注册。
    1. 当数据变化以后，observe 对 watcher 进行通知notify，计算属性、侦听属性再执行一遍自己，缓存数据或执行业务。
1. computed 内部的方法可以写成 对象，有get和set方法，这样计算属性就可以不仅被获取，还可以被set值了
1. computed 不可以接受参数， methhode可以；前者可以缓存，后者不行。
1. computed 可以依赖其他 computed、或者其他组件的data （这是因为劫持早已经发生在其内部）
1. watch 内部的方法可以写成 对象，{handler=通常的处理方法, deep深度，immediate 是否立即执行}

## vue插槽 slot 及其 优缺点

1. 当组件的 template 包含一个 slot 元素，在被引用的组件标签中的任何内容，都会被放到 slot 标签中去。若无，则中间内容会被抛弃。
  1. 插槽的作用域只存在于组件中，不能访问到父组件中的值。**父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。**
      1. 但是我们可以给插槽绑定一个值:data="data",在再父级上使用v-slot:default="slotProps", 这样{{slotProps.user}} 就不会是undefined了。
          1. 还可以把slotProps解构成{user} || {slotUser: user = {}}
  1. default内容，可以直接写在子组件的 slot 中间
  1. 当有多个插槽时，slot 的属性 name， 可以进行区分父组件的标签名、或 元素用 v-slot:[指定的名字] 这也是动态写法
      1. 都没有的会被扔进 default slot，为了更准确使用插槽，可以加上 v-slot: default
  1. 缩写 # = v-slot:

## vuex

1. 一个专门为vuejs开发的**状态管理模式**，才用**集中式存储**管理应用的**所有组件**的状态，并以相应的规则保证状态以一种**可预测**的方式发生变化。
1. 这既是vue单向数据流的思路，也是vuex的设计思路
    1. state，数据源
    1. view，声明视图
    1. actions，响应视图修改数据源的方法
1. stroe（仓库），包含着大部分的state。
    1. 存储为响应式的，意为当组件依赖了vuex中某个stroe里的state，当state改变了后，组件也会更新。
    1. 无法直接修改stroe里面的状态，唯一的方法就是提交（commit）mutation，告知做了什么，并修改state。
    1. 例子

        ```
        import Vue from 'vue'
        import Vuex from 'vuex'

        Vue.use(Vuex)
      
        const store = new Vuex.Store({
          state: {
            count: 0
          },
          mutation: {
            addCount(state) {
              state.count++
            }
          }
        })

        // 在main中
        new Vue({
          el: '#app',
          store
        })

        // 在组件中
        computed: {
          addCount() {
            this.$store.commit('addCount') // count=0 => count=1
            return this.$store.state.count // 一定要放在计算属性中获取，若放在data中会变成给数据名赋值，导致不能响应式更新组件
          }
        }
        ```
1. state
    1. 为了将vuex里面的state映射到组件中，并保持响应式，一定要写成计算属性
    1. 为了解决不写大量的计算属性，使用mapState函数

        ```
        computed: mapState({
          count: state => state.count
          asCount: count // 传字符串参数 'count' 等同于 `state => state.count`
          countSumLocalState (state) { // 为了能够使用 `this` 获取局部状态，必须使用常规函数
            return state.count + this.localCount
          }
        })

        // 还可以更简单，直接解构
        computed: mapState([
          'count'
        ])

        // 与其他计算属性使用
        computed: {
          localComputed() {}
          ...mapState({
            //..
          })
        }    
        ```
1. getter
    1. 当在state中派生出一些状态，例如过滤，且多个组件都想共用这一个方法，这时候就需要定义getter了,而且如计算属性特性一样，会缓存。
    1. 例子

        ```
        const store = new Vuex.Store({
          state: {
            todos: [
              { id: 1, text: '...', done: true },
              { id: 2, text: '...', done: false }
            ]
          },
          getters: {
            doneTodos: (state, getters|其他getter) => {
              return state.todos.filter(todo => todo.done)
            }
          }
        })

        // Getter 会暴露为 store.getters 对象
        // 在组件内直接用 store.getters.doneTodos

        // 通过方法访问
        getters: {
          // ...
          getTodoById: (state) => (id) => {
            return state.todos.find(todo => todo.id === id)
          }
        }
        // 在组件内直接用 store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }

        ```
    1. mapGetters 辅助函数
      ```
      computed: {
        ...mapGetters(['doneTodos', 'getTodoById'])
        // ...mapGetters({asname: 'doneTodos', 'getTodoById'})
      }
      ```
1. Mutation 
    1. 更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件。
    1. 每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。
    1. 例子
        
        ```
        const store = new Vuex.Store({
          state: {
            count: 1
          },
          mutations: {
            increment (state, payload={}||基础类型) {
              // 变更状态
              state.count++
            }
          }
        })

        // 但是不能直接调用 increment 方法
        // 在组件内用
        this.$store.commit('increment', payload={}||基础类型) 
        //  this.$store.commit({type: 'increment', payloaditem: 1})

        ```
    1. 遵守响应规则
        1. 提前在store中初始化属性
        1. 当需要给{}对象添加新属性时，要么用vue.set(obj, 'keyname', val) 或者 直接替换老对象。
            1. state.obj = {...state.obj, key: 123}
    1. 使用常量来命名类型
        1. 这样定义一张常量表，表示常量都有哪些作用，这样就不用去 store.js中找方法了
        1. ```[SOME_NAME](state){ //... }```
    1. 必须是同步函数！！不要再mutation里面写回调函数，比如请求服务器，比如定时器
        1. 会让状态改变都变成不可追踪的
    1. 辅助方法 (还是提交commit，不是调用了方法)

        ```
        methods: {
          ...mapMutations([
            'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

            // `mapMutations` 也支持载荷：
            'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
          ]),
          ...mapMutations({
            add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
          })
        }
        ```
1. Action
    1. 类似与Mutation，但不是直接变更状态，且可以包含任意异步操作。
    1. 例子

        ```
        const store = new Vuex.Store({
          state: {
            count: 0
          },
          mutations: {
            increment (state) {
              state.count++
            }
          },
          actions: {
            increment (context || {commit} // 解构, payload) { // context 上下文，指向store实例
              const {count} = context.state
              const {otherGetCount} = context.getters
              context.commit('increment', payload)
            }
          }
        })

        // 在组件中
        this.$store.dispatch('increment')
        ```
    1. 辅助方法

      ```
      methods: {
        ...mapActions([
          'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

          // `mapActions` 也支持载荷：
          'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
        ]),
        ...mapActions({
          add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
        })
      }
      ```
    1. dispatch 返回promise，所以可以.then串联起来
        1. .then是异步微任务，排在异步宏任务之前，如setTimeout。
        1. js的执行顺序总是
            1. 对列中从上往下顺序执行
            1. 队列之间顺序为 同步任务对列=>微任务（promise，nextTick，mutationObserver）=>宏任务（定时器，事件绑定，接口请求，回调函数，nodejs-fs中的I/O操作）。
    1. async/await
        1. 例子

        ```
        // 假设 getData() 和 getOtherData() 返回的是 Promise
        actions: {
          async actionA ({ commit }) {
            commit('gotData', await getData())
          },
          async actionB ({ dispatch, commit }) {
            await dispatch('actionA') // 等待 actionA 完成
            commit('gotOtherData', await getOtherData())
          }
        }
        ```
1. Module
    1. 由于使用单一状态树，应用所有状态会集中到一个比较大的对象，会使store臃肿。
    1. 将store分割成模块（module）
    1. 例子

        ```
        const moduleA = {
          state: () => ({ ... }),
          mutations: { ... },
          actions: { ... },
          getters: { ... }
        }

        const moduleB = {
          state: () => ({ ... }),
          mutations: { ... },
          actions: { ... }
        }

        const store = new Vuex.Store({
          modules: {
            a: moduleA,
            b: moduleB
          }
        })

        store.state.a // -> moduleA 的状态
        store.state.b // -> moduleB 的状态
        ```
    1. 嵌套模块
        1. mutation 和 getter，改为接收的第一个参数是模块的**局部状态对象**。
        1. action 的 context 中局部的为 state，根节为 rootState (最底层，非全局) 
        1. getter 增加 第三个 参数为 rootState，第四个 参数为 rootGetters (最地层，非全局)
        1. 默认情况下部的 action、mutation 和 getter 是注册在**全局命名空间**中的。
            1. 意思就是说可以在多个modul store中调用同一个方法
            1. 可以在配置modul store时，加上namespaced：true，使其成为带命名空间的模块
                1. 意思就是，在调用时，只要store选的对，同名的方法执行的代码不同。
        1. （namespaced：true）局部命名空间的action中访问全局空间, 在paylod后面加 { root: true }

            ```
            dispatch('someOtherAction', null, { root: true })
            commit('someMutation', null, { root: true })
            ```
        1. 在（namespaced：true）局部模块 注册 全局action, 加 root: true

            ```
              modules: {
              foo: {
                namespaced: true,

                actions: {
                  someAction: {
                    root: true, 
                    handler (namespacedContext, payload) { ... } // -> 'someAction'
                  }
                }
              }
            }
            ```
        1. mapState, mapGetters, mapActions 和 mapMutations
            1. 这些函数来绑定带命名空间的模块时，写起来可能比较繁琐
            1. 可以将模块的空间名称字符串作为第一个参数传递给上述函数
            1. 例子

                ```
                computed: {
                  ...mapState('some/nested/module', {
                    a: state => state.a,
                    b: state => state.b
                  })
                },
                methods: {
                  ...mapActions('some/nested/module', [
                    'foo', // -> this.foo()
                    'bar' // -> this.bar()
                  ])
                }

                // 或者使用 createNamespacedHelpers 
                import { createNamespacedHelpers } from 'vuex'
                const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')
                computed: {
                  // 在 `some/nested/module` 中查找
                  ...mapState({
                    a: state => state.a,
                    b: state => state.b
                  })
                },
                methods: {
                  // 在 `some/nested/module` 中查找
                  ...mapActions([
                    'foo',
                    'bar'
                  ])
                }
                ```
    1. 动态注册模块
        1. 使用 store.registerModule 方法注册模块

        ```
        import Vuex from 'vuex'
        const store = new Vuex.Store({ /* 选项 */ })
        // 注册模块 `myModule`
        store.registerModule('myModule', {
          // ...
        })
        // 注册嵌套模块 `nested/myModule`
        store.registerModule(['nested', 'myModule'], {
          // ...
        })

        // 若已经定义好state，只想注册新的action、getter、mutation, 则用 { preserveState: true }
        store.registerModule('a', module, { preserveState: true })
        ```
        1. store.unregisterModule(moduleName) 来动态卸载模块。不能卸载初始模块。
        1. store.hasModule(moduleName) 否已经被注册
    1. 模块重用 
        1. 与 vue 的 data有同样问题，共用一个模块或者多次注册同一个模块时，数据间会相互污染（浅拷贝对象指向同一内存地址，多次浅拷贝会有污染的风险，这个问题有利有弊。利是可以在多处维护一套数据，占用还小，不需要同步。）
        1. 所以（vuex 2.3+）支持state通过函数来声明
            1. state：() => {foo: 'bar'}
1. 项目结构
    1. 应用层级的状态应该集中到单个 store 对象中。
    1. 提交 mutation 是更改状态的唯一方法，并且这个过程是同步的。
    1. 异步逻辑都应该封装到 action 里面。
    1. 将action、utation 和 getter 分割到单独的文件。
    1. modules 写个index文件，用于组合各个模块
    1. 单独的模块单写一个文件（写个文件夹，内部拆分action、utation 和 getter ）
1. 严格模式
    1. 仅需在创建 store 的时候传入 strict: true：
    1. 在严格模式下，无论何时发生了状态变更且不是由 mutation 函数引起的，将会抛出错误。
    1. 不要在发布环境下启用严格模式！！！
        1. 请确保在发布环境下关闭严格模式，以避免性能损失。
1. 表单处理
    1. `<input v-model="obj.message">` obj是state时，会报错，因为v-model语法糖绑定的 input事件往state里面set值了，因为这不是mutation执行的。
        1. 所以不要用 v-model，自己实现双向绑定(react 叫可控组件）
          `:value="value" @input=“(e) => {this.$store.commit('update', e.target.value)}”`
        1. 另一个方法是修改计算属性为 setter
        
            ```
            computed: {
              message: {
                get () {
                  return this.$store.state.obj.message
                },
                set (value) {
                  this.$store.commit('updateMessage', value)
                }
              }
            }
            ```

## vue-router

1. router-link标签，渲染为 a标签。
    1. target属性，_blank新开一个页面，_self默认当前窗口，_parent父级窗口打开，_top打开新的浏览器
1. 编程导航
    1. router.push('/home') 会向history添加记录 === window.history.pushState
    1. router.replace() 它不会向 history 添加新记录，而是跟它的方法名一样 —— 替换掉当前的 history 记录 === window.history.replaceState
    1. router.go(n) 在 history 记录中向前或者后退多少步 === window.history.go
1. $router 和 $route
    1. $router 为 vueRouter 的实例，相当于一个全局路由器对象，history，命令跳转用的也是它 this.$router.push
    1. $route 相当于当前正在跳转的路由对象，可以从里面获取name, path, params, query等
1. 传参方式
    1. 拼接path
        1. ```this.$router.push({path:`/user/${userId}`})```
        1. 这样传递需要的配置时 加上 path: user/:userId
        1. 接受方式为 this.$route.params.userId (!!! route,当前页)
    2. params 传递, 隐式传递
        1. router.push({name: 'user', params: {userId: 123}})
        1. this.$route.params.
    3. query 类似拼接path
        1. router.push({path: 'register', query: {plan: 'private'}})
        1. this.$route.query.
1. 配置中的 '/' 当以其为开头，则会被当成根路径，不会进行嵌套处理
1. 匹配所有路由 path: *
1. 嵌套路由 

    ```
      const router = new VueRouter({
        routes: [
          { 
            name: 'index', // 非必须，用的时候要传成对象{name: index}
            path: '/user/:id', 
            component: User,
            children: [
              {
                // 当 /user/:id/profile 匹配成功，
                // UserProfile 会被渲染在 User 的 <router-view> 中
                path: 'profile',
                component: UserProfile
              },
              {
                // 当 /user/:id/posts 匹配成功
                // UserPosts 会被渲染在 User 的 <router-view> 中
                path: 'posts',
                component: UserPosts
              }
            ]
          }
        ]
      })
    ```
1. 命名视图
    1. 在布局页面，可以写上多个 router-view + name 属性
    1. 这样配置时 components: {default: Foo,a: Bar,b: Baz} 用于展示不同组件了
1. 重定向

    ```
    { path: '/a', redirect: '/b' ||  { name: 'foo' } || redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}}
    ```
1. 别名， { path: '/a', component: A, alias: '/b' }， 当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a，就像用户访问 /a 一样。
1. HTML5 History 模式

    ```
    const router = new VueRouter({
      mode: 'history',
      routes: [
        { path: '*', component: NotFoundComponent }
      ]
    })
    ```
    1. 需要后台支持，当用户访问域名下的其他url, 让后台返回index页，避免出现404
        
        ```
        nignx 配置
        location / {
          try_files $uri $uri/ /index.html;
        }
        ```
    1. 需要兜底处理404页面


1. 路由守卫
    1. 全局前置 router.beforeEach（(to, from, next) => {
        // ...
        next() 进行下一个守卫
        next(false) 中断
        next('/') 跳转 path，name，replace等等
        next(error) 错误终止导航，进router.onError()回调
      }）
    1. 全局解析 router.beforeResolve
        1. **同时在所有组件内守卫和异步路由组件被解析（进入等）之后**
    1. 全局后置钩子 router.afterEach(to, from) 不接受next
    1. 独享守卫，配置router时，添加
    1. 组件内守卫
        1. beforeRouteEnter 进入组件路由 // 不能访问 this，此时this还没创建
        1. beforeRouteUpdate 改变组件路由
        1. beforeRouteLeave 离开组件路由
    1. 完整的导航解析流程
        1. 导航被触发。
        1. 在失活的组件里调用 beforeRouteLeave 守卫。
        1. 调用全局的 beforeEach 守卫。
        1. 在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
        1. 在路由配置里调用 beforeEnter。
        1. 解析异步路由组件。
        1. 在被激活的组件里调用 beforeRouteEnter。
        1. 调用全局的 beforeResolve 守卫 (2.5+)。
        1. 导航被确认。
        1. 调用全局的 afterEach 钩子。
        1. 触发 DOM 更新。
        1. 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。
    1. 若做redirect，一定要跳过自己，否则进死循环

1. 路由原信息 meta，配置中自定义属性
    1. 如何取？在导航守卫中遍历$route.matched 数组，el就是当前路由的meta信息
1. 过度动效
    1. transition
1. 滚动行为 scrollBehavior (to, from, savedPosition=切换之前的位置) {// return 期望滚动到哪个的位置}
1. 路由懒加载

    ```
    const router = new VueRouter({
      routes: [
        { path: '/foo', component: () => import('./Foo.vue') }
      ]
    })

    // 分包，Webpack > 2.4，chunk name
    const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
    ```

## vue性能优化

1. v-if,v-shou区分场景
1. 文件路由懒加载
1. 长列表分页
1. Webpack 优化
    1. 图片压缩  image-webpack-loade 配置为 test:/png|jp?g$/,use:[{loader:'image-webpack-loade', options: {bypassOnDebug:true}}]
    1. 提取公共代码 CommonsChunkPlugin
1. gzip 压缩
1. cdn
1. 使用 chrome Performance 查找性能瓶颈

## webpack

1. entry // 检查入口，vue的main文件
1. output // 出口文件 path地址，filename文件名
1. loader 处理非JavaScript文件

    ```
    // webpack.config.js
    const path = require('path');

    const config = {
      output: {
        filename: 'my-first-webpack.bundle.js'
      },
      module: {
        rules: [
          { test: /\.txt$/, use: 'raw-loader', options: {} } // test如何识别，use用什么loader,options选项
        ]
      }

    };

    module.exports = config;
    ```
    1. image-webpack-loade 压缩图片
1. 插件（plugins）
    ```
    plugins: [
      new HtmlWebpackPlugin({template: './src/index.html'})
    ]
    ```
    1. CommonsChunkPlugin 提取 chunks 之间共享的通用模块
1. 模式 （mode）development 开发 production 生产，文件名不一样，webpack.[mode].config.js
1. 构建目标 targets 以什么环境进行编译

## vue-cli 配置（一个多入口开发，区别单入口）