## js基础
  1. 闭包 当一个函数返回另一个匿名函数时，且这个匿名函数依赖与其同级的变量，那么这个变量就不会因为函数执行完毕而销毁，被匿名函数保存了起来，匿名函数还可以取到
      1. 优缺点
        1. 优点：
          1. 用闭包解决递归调用问题
          1. 用闭包模仿块级作用域
        1. 缺点：
          1. 引用的变量可能发生变化
          1. this指向问题，浏览器环境下指向window，node环境下指向当前模块对象module.exports
          1. 内存泄露问题，闭包函数依赖变量没办法释放，除非主动释放
  1. 匿名函数
      1. 没有实际名字的函数。
      1. => 函数
          1. this指向问题，通常会指向定义时它所处的对象（宿主对象），可能是某个类，可能是window，可能是当前模块对象module.exports
          1. 用bind，或者传入this
  1. 原型链
      1. prototype 每个函数都有一个prototype属性，这个属性指向函数的原型对象。
      1. __proto__  这是每个对象(除null外)都会有的属性, 这个属性会指向该对象的原型。
      1. 每个原型 constructor 指向关联的构造函数。
      1. 实例与原型
          1. 当读取实例的属性时，如果找不到，就会查找与对象关联的原型中的属性，如果还查不到，就去找原型的原型，一直找到最顶层为止。也就是作用域链
      1. 原型的原型
          1. 其实原型对象就是通过 Object 构造函数生成的
      1. Object.prototype.__proto__ === null
      [原型关系图](./prototype.png)
  1. 作用域链 __proto__
      1. 当在函数中使用一个变量的时候,首先在本函数内部查找该变量,如果找不到则找其父级函数的变量，直到最顶级，window或者当前模块对象module.exports。

## js类（代码抽象，面向对象）

1. es5 及以前

    ```
      // 定义一个User类
      (function(){
            //定义类、构造函数
            function User(name,age){
                this.name=name;
                this.age=age;
            }
 
            //定义原型方法
            User.prototype.show=function(){
                console.log('...');
            }
 
            //定义静态方法
            User.run=function(){
                console.log('...');
            }
 
            window.User=user;
        })();

        // 继承
        function Father(){
        }
        Father.prototype.show=function(){
        }
 
        function Son(){
            //继承第一句：让子类实例化的对象具备父类的所有属性
            Father.apply(this,arguments);
        }
        //继承第二句：让子类实例化对象具备父类的所有原型方法
        Son.prototype=Object.create(Father.prototype);
 
        //继承第三句：找回丢失的构造函数
        Son.prototype.constructor=Son;
 
        Son.prototype.run=function(){}
    ```
1. es6 

    ```
    // 制作一个对象
    class User{
        //实例化类时默认会执行构造函数
        constructor(name,age){
            //初始化对象的语句
            this.name=name;
            this.age=age;
        }

        //原型方法
        show(){
            console.log('...');
        }

        //静态方法
        static run(){
            console.log('...');
        }
    }

    // es6 继承
    class Son extends Father{
          constructor(){
              super();
          }
      }
    ```

## es6 到 es10

1. let const
1. 运算符扩展
    1. ...
    1. ** 求幂
1. Array
    1. .includes
    1. .from
    1. .sort
    1. .map
    1. .filter
1. Object
    1. .values | keys | entries
1. String
    1. .padStart | padEnd
    1. .trim
1. async await
1. set
    1. new set([])
    1. set.size
    1. set.add
    1. set.delete 删除一个，返回1-0表示是否成功
    1. set.has
    1. set.clear 清除所有
    1. [...new Set([])]
    1. 遍历 values，keys，entries，插入顺序就是遍历顺序
    1. 其实就是与key与value相同的对象


## 算法（去重，排序；数组去重，冒泡算法，归并算法，set去重，原型方法去重，优缺点 ）
    1. 去重
        1. new set
        1. 循环与整体比较
        1. 用对象，空间换时间
    1. 排序
        1. sort原地排序
        1. 冒泡，左右比较，大小不一致换边，结果在左边
        1. 选择排序，循环取最小的，全查完放在最小的后面，要记录最小存到哪的
        1. 插排，取一个元素，和排列过的数组比较，比某个元素大，比下一个小，就把它插入在这个位置
        1. 二分归并，
            1. 制作一个递归函数，将数组从中间切开，分别放入这个递归函数，这个递归函数会判断传入的数组长度，若小于2，则返回数组本身。
            `return merge(mergeSort(left), mergeSort(right));`
            1. 制作一个左右比较函数，内部有一个结果数组，
                1. left[0]和right[0]比大小，将小得放入结果数组
                1. 然后先将left push进res，再将right push进res
                1. 最后返回res

## css

1. 水平居中
    1. 行内元素水平居中 text-align: center
    1. 块级元素水平居中 margin-left；margin-right auto
    1. flex justify-content center 
1. 垂直居中
    1. 单行行内元素垂直居中 height 和行高(line-height) 相等
    1. 块级元素垂直居中 通过绝对定位元素距离顶部50%，并设置margin-top向上偏移元素高度的一半
    1. flex align-items center
1. 水平垂直居中
    1. 通过margin平移元素整体宽度的一半，使元素水平垂直居中。
    1. 用2D变换，在水平和垂直方向都反向平移宽高的一半，从而使元素水平垂直居中。
    1. flex justify-content center align-items center
1. 伪类
    1. link, visited, hover, active
    1. first-child, last-child
    1. not()
    1. nth-child(n) 从0开始，first正顺序，last逆顺序

## 遇到问题怎么解决问题，事件绑定，打桩，debug，网络错误代码分析

1. 打印
1. 断点
1. 性能分析
1. chrom Performance
1. 青花瓷抓包
1. 单元测试
1. 测试框架 jest

## HTTP状态码

1. 1** 信息，服务器收到请求，需要请求者继续执行操作
    1. 100 continue 继续。客户端应继续其请求。
    1. 101 switching Protocols 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议
1. 2** 成功，操作被成功接收并处理
    1. 200 OK 请求成功。一般用于GET与POST请求
    1. 201 Create 已创建。成功请求并创建了新的资源
    1. 202 Accepted 已接收。已经接收请求，但未处理完成
    1. 203 Non-auth 非授权信息。请求成功，单反毁的meta信息不在原始的服务器，而是一个副本。
    1. 204 No content 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档。
    1. 