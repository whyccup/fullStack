# vue

## mvvm 原理 及 vue 原理

**react不是mvvm框架，类似其中的view层，但单独的react仅仅是一个库**

### mvc 是什么

1. view，用户界面
1. controller，业务逻辑
1. model，数据模型
1. 各部分为单向通信，用户操作触发view，view告知controller，controller告知model，model又改变view

### mvvm是什么 

1. view，用户界面
1. viewModel，类似controller，但在vue中相当于虚拟dom树
1. model, 数据模型
1. view 和 viewModel才用双向绑定，view的改变，自动反映在viewModel，反之亦然。
1. model 和 viewModel可以双向通信

### mvvm的实现方式

1. 发布者-订阅者模式，通过sub，pub的方式实现数据和视图的绑定监听
1. 脏值检查，通过其对比数据是否有变更，来决定是否更新视图，在指定的事件触发时进入脏值检查。
1. 数据劫持，通过Object.defineProperty()来劫持各个属性的setter和getter
1. 思路
    1. 实现一个数据监听器 Observe，对所有数据的所有属性进行监听，如有变动，拿到最新值，通知订阅者。
    1. 实现一个指令解析器 Compile, 对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数
    1. 实现一个watcher，作为连接其上两位的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图。
    1. 实现一个异步更新队列
        1. 当侦听到数据变化时，vue将开启一个队列，当多个watcher被触发时，只会被推到队列中一次。
        1. 然后再下一个事件循环中“tick”中，vue刷新队列并执行（已去重的）的工作，用diff更新虚拟dom树，再渲染到真实dom上。
          1. 为了在数据变化之后等待vue完成更新DOM，可以在数据变化之后立即使用 this.nextTick(cb) 
          1. 或者用 async，await this.$nextTick()
    1. mvvm入口函数，整合以上三者
    1. [思路图](./mvvmMind.png)
1. 实现
    1. Observer 
        1. 制作一个递归函数 observer，内部进行如下操作
        1. 遍历所有数据的可枚举属性，并调用 observeProperty(data, key, data[key])
        1. 制作一个劫持函数

            ```
            function observeProperty(data, key, val) {
                const dep = new Dep() // 消息订阅器
                observer(deepData) // 递归监听子属性
                Object.defineProperty(data, key, {
                  get () { // 劫持getter
                    Dep.target && dep.addSub(Dep.target) // 当target有值时，添加当前watcher
                    return val
                  }
                  set (newVal) { // 劫持setter
                    val = newVal
                    dep.notify() // 通知所有订阅者
                  }
                })
              }
            ```
        1. 实现一个消息订阅器，维护一个数组，用来收集订阅者，数据变动触发notify，再调用订阅者的update方法

            ```
            class Dep{
              constructor() {
                this.subs = [] // 所有订阅者，也就是watcher
              }
              addSub(sub) { // 新增订阅者
                this.subs.push(sub)
              }
              notify() { // 通知所有订阅者，更新数据
                this.subs.forEach(sub => {
                  sub.update()
                })
              }
            }
            ```
        1. 订阅者指的是watcher

            ```
            class watcher = {
              get(key) {
                  Dep.target = this;
                  this.value = data[key];    // 这里会触发属性的getter，从而添加订阅者
                  Dep.target = null;
              }
            }
            ```

    1. compile 
        1. 解析模板指令
            1. 解析时用 document.createDocumentFragment 转成虚拟DOM树，
        1. 将模板中的变量替换成数据 
            1. 解析完再将 虚拟DOM树添加回原来真实DOM节点中。
        1. 初始化渲染页面
        1. 对每个指令对应的节点绑定更新函数
        1. 添加监听数据的订阅者
        1. 一旦数据有变动，收到通知，更新视图
        1. [compile思路图](./compile.png)
        1. [详解](https://blog.csdn.net/dwfrost/article/details/85777900)
        1. 至此，视图和数据已经绑定成功，页面的初始化渲染由vm实例的属性初始化完成，并对节点上的事件进行了注册。
        1. 接下来就是watcher的事了，它将接受数据的变化，并重新更新视图。
    
    1. watcher
        1. Watcher订阅者作为Observer和Compile之间通信的桥梁，主要做的事情是:
            1. 在自身实例化时往属性订阅器(dep)里面添加自己
            1. 自身必须有一个update()方法
            1. 待属性变动dep.notify()通知时，能调用自身的update()方法，并触发Compile中绑定的回调。

              ```
              class Watcher {
                constructor(vm, exp, cb) {
                    this.vm = vm
                    this.exp = exp
                    this.cb = cb
                    this.value = this.get() // 缓存当前值（旧值）
                }
                get() {
                    Dep.target = this // 将当前订阅者指向自己
                    const value = this.vm[this.exp] // 触发getter，添加自己到属性订阅器中
                    Dep.target = null // 添加完毕，重置
                    return value
                }
                update() {
                    const value = this.get() // 获取新值
                    const oldValue = this.value
                    if (value !== oldValue) {
                        this.value = value
                        this.cb.call(this.vm, value, oldValue) // 执行compile中绑定的回调，更新视图
                    }
                }
              }
              ```

    1. MVVM 入口
        1. MVVM作为数据绑定的入口
        1. 整合Observer、Compile和Watcher三者，
        1. 通过Observer来监听自己的model数据变化
        1. 通过Compile来解析编译模板指令
        1. 最终利用Watcher搭起Observer和Compile之间的通信桥梁
            1. watcher获取数据时，被observe察觉到存入了订阅者数组中
            1. watch又存着Compile接卸出来的虚拟DOM绑定的回调
            1. 数据变化时，调用回调，更新虚拟DOM，根据特殊Diff算法，更新到真实DOM中
                1. 特殊Diff算法
                    1. 从DOM树从上往下，依层检查，并对同层虚拟DOM树进行依次对比，若有key值则直接更新指定key值的DOM，不动其他的。
            1. 达到数据变化（model） -> 视图更新（viewModel -> view）; 视图交互变化(view -> viewModel) -> 数据变更(model)的双向绑定效果。
      
                ```
                class MVVM {
                  constructor(options) {
                      this.$options = options
                      this.$el = options.el
                      this.$data = options.data
                      // 属性代理，实现 vm.xxx -> vm._data.xxx
                      for (let key in this.$data) {
                          this.$proxy(key)
                      }
                      observer(this.$data)
                      this.$compile = new Compile(this.$el || document.body, this)
                  }
                  $proxy(key) {
                      Object.defineProperty(this, key, {
                          configurable: false,
                          enumerable: true,
                          get() {
                              return this.$data[key]
                          },
                          set(newVal) {
                              this.$data[key] = newVal
                          }
                      })
                  }
                }
                ```

### vue原理官方说明

1. 虚拟DOM树。通过document.createDocumentFragment(),遍历指定根节点内部节点，根据prop，v-model等规则进行更新。
1. 通过Object.defineProperty()进行数据变化拦截。
1. 截取到数据变化，动过发布者-订阅者模式，触发watcher，从而改变虚拟DOM中的具体数据
1. 通过改变虚拟DOM元素值，从而改变最后渲染DOM树的值，完成双向绑定。


### vue不能检测数组和对象的变化

1. 对于对象，Vue 无法检测 property 的添加或移除。
    1. 原因
        1. Vue 会在初始化实例时对 property 执行 getter/setter 转化（劫持，制作observe）
        1. 所以对象的属性必须写在data内
        1. new Vue({data: {a: 1}}) `vm.a`响应的
        1. 继承上面的代码， vm.b = 1; `vm.b`非响应的
    1. 虽然不允许，但是可以用 vue.set(object, propertyName, value)方法向嵌套对象添加响应式 property。
    1. this.$set(object, propertyName, value)

1. 对于数组
    1. vue不能检测以下数组的变动：
        1. 当你利用索引直接设置一个数组项是，例如
            1. vm.items[indexOfItem] = newValue
            1. 应当用 vm.$set(vm.items, indexOfItem, newValue)
        1. 当你修改数组的长度时，例如
            1. vm.items.length = newLength
            1. 应当用 vm.items.splice(newLength)

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


## mixin

## watch 与 计算属性

## vue插槽 及其 优缺点

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


## vue性能优化




## webpack 配置

## vue-cli 配置（一个多入口开发，区别单入口）

## vue3.0概况 hook

## js原型链
  1. 闭包
  1. 匿名函数
  1. 原型链
  1. 怎么从函数式编程发展到面向对象

## js类（代码抽象，面向对象）

## this问题 （箭头函数的详解，this函数，对象内部的箭头函数的this问题）

## es6 到 es10

### map 和 set

## 算法（去重，排序；数组去重，冒泡算法，归并算法，set去重，原型方法去重，优缺点 ）

## css居中、transeform

## less 和 sass 区别，优缺点

## 遇到问题怎么解决问题，事件绑定，打桩，debug，网络错误代码分析

## 后端技术

