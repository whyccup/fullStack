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
 [看](https://segmentfault.com/a/1190000018659584)
 [看](https://cn.vuejs.org/v2/guide/reactivity.html)

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

  1. <span v-text="msg"></span>
  1. <span> {{msg}} </span>
  1. 两者相同

## vue组件传值

  1. 父组件向子组件传值
    1. 父组件在调用的子组件标签上加属性，子组件通过props来获取
    1. 子组件可以在prop中定义传入属性的类型
      1. 普通类型，String，Number，Boolean，Null
      1. 引用类型，Array，Object
    1. 基于vue单向数据流，子组件是不能修改prop中的值，即便传来的是一个对象
      1. 除非把prop的值附给data中的一个变量，然后再改变那个变量

## mixin

## watch 与 计算属性

## vue插槽 及其 优缺点

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

## 算法（去重，排序；数组去重，冒泡算法，归并算法，set去重，原型方法去重，优缺点 ）

## css居中、transeform

## less 和 sass 区别，优缺点

## 遇到问题怎么解决问题，事件绑定，打桩，debug，网络错误代码分析

## 后端技术

