# vue 原理

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