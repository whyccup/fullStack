# vue指令，及渲染优化问题

## v-show 与 v-if
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




后端技术

mvvm原理

组建传旨

插槽详细优坏

vue底层源码

代码抽象

js 闭包，匿名函数，原型链，怎么从函数式编程发展到面向对象

数组去重，冒泡算法，归并算法，set去重，原型方法去重，优缺点

箭头函数的详解，this函数，对象内部的箭头函数的this问题

css居中，transeform相关问题

sass less 区别，优缺点

vue性能优化，从底层到顶层

es6到es10

vue-cli 配置一个多入口开发，区别单入口

遇到问题怎么解决问题，事件绑定，打桩，debug，网络错误代码分析

进项问题，特定ip才能访问，没办法在网上搜索

vue3.0概况 hook