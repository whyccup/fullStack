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

### es6

1. let const
1. 解构，默认数据
1. UNICODE表示法
1. 模板字符串
1. 字符串新增方法
    1. fromCodePoint() 用于从 Unicode 码点返回对应字符，
    1. raw() 该方法返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，往往用于模板字符串的处理方法。
    1. codePointAt() 能够正确处理 4 个字节储存的字符，返回一个字符的码点。
    [看](https://es6.ruanyifeng.com/#docs/decorator)

### es7

1. Array.prototype.includes() 查找一个值在不在数组里.
1. 求幂运算符 **=

### es8

1. async await
    1. 并行 Promise.all
1. Object.entries() 将一个对象中可枚举属性的键名和键值按照二维数组的方式返回。*若对象是数组，则会将数组的下标作为键值返回。*
1. Object.values() 只返回自己的键值对中属性的值.
1. padStart()和padEnd() String.padStart(targetLength // 目标长度, padding // 填充元素 )， 到目标长度就会停！
1. Object.getOwnPropertyDescriptors() 该方法会返回目标对象中所有属性的属性描述符，该属性必须是对象自己定义的，不能是从原型链继承来的。
1. 修饰器Decorator 类，方法（用在类中的函数，属性时）
     @addSkill
    class Person { }
    function addSkill(target) {
        target.say = "hello world";
    }
    console.log(Person['say']) //'hello world' Person就有addSkill的属性了
    1. 顺序是洋葱型，从外到内，然后从内向外

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
    1. 二分归并
        1. 把长度为n的输入序列分成两个长度为n/2的子序列；
        1. 对这两个子序列分别采用归并排序；
        1. 将两个排序好的子序列合并成一个最终的排序序列。
            ```
            function mergeSort(arr) {
                var len = arr.length;
                if(len < 2) {
                    return arr;
                }
                va rmiddle = Math.floor(len / 2),
                    left = arr.slice(0, middle),
                    right = arr.slice(middle);
                return merge(mergeSort(left), mergeSort(right));
            }
            
            function merge(left, right) {
                var result = [];
            
                while(left.length>0 && right.length>0) {
                    if(left[0] <= right[0]) {
                        result.push(left.shift());
                    } else{
                        result.push(right.shift());
                    }
                }
            
                while(left.length)
                    result.push(left.shift());
            
                while(right.length)
                    result.push(right.shift());
            
                return result;
            }
            ```
    1. 快排
        1. 在数据集之中，选择一个元素作为"基准"（pivot）。
        2. 所有小于"基准"的元素，都移到"基准"的左边；所有大于"基准"的元素，都移到"基准"的右边。
        3. 对"基准"左边和右边的两个子集，不断重复第一步和第二步，直到所有子集只剩下一个元素为止。
            ```
            // 类似于归并的版本 空间O(n3)

            function quickSort(arr) {
                // 长度不够直接返回
                if (arr.length < 2) return arr
                // 取一个标杆
                const pivotIndex = Math.floor(arr.length / 2)
                const pivot = arr.splice(pivotIndex, 1)[0]
                const left = []
                const right = []
                // 从左往右依次比较，小的放左边，大的放右边
                for (let i = 0; i < arr.length; i++) {
                    if (arr[i] < pivot) {
                        left.push(arr[i])
                    } else {
                        right.push(arr[i])
                    }
                }
                // 递归处理
                return quickSort(left).concat([pivot], quickSort(right))
            }

            // 原地排序的版本
            function quickSort(arr, left, right) {
                var len = arr.length,
                    partitionIndex,
                    left = typeofleft != 'number'? 0 : left,
                    right = typeofright != 'number'? len - 1 : right;
            
                if(left < right) {
                    partitionIndex = partition(arr, left, right);
                    quickSort(arr, left, partitionIndex-1);
                    quickSort(arr, partitionIndex+1, right);
                }
                return arr;
            }
            
            function partition(arr, left ,right) {     // 分区操作
                var pivot = left,                      // 设定基准值（pivot）
                    index = pivot + 1;
                for(var i = index; i <= right; i++) {
                    if(arr[i] < arr[pivot]) {
                        swap(arr, i, index);
                        index++;
                    }       
                }
                swap(arr, pivot, index - 1);
                return index-1;
            }
            
            function swap(arr, i, j) {
                var temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }

            ```
## 宏任务 和 微任务

1. JS的本质是单线：
    1. 非阻塞性的任务采取同步的方式，直接在主线程的执行栈完成。
    1. 阻塞性的任务都会采用异步来执行，异步的工作一般会交给其他线程完成，然后回调函数会放到事件队列中。
1. 当主线程的任务执行完了(执行栈空了)，JS会去询问事件队列，其中：
    1. 执行一个宏任务(先执行同步代码)-->执行所有微任务-->UI render-->执行下一个宏任务-->执行所有微任务-->UI render-->......
1. 哪些是宏任务、微任务呢?
    1. 宏任务
        #|浏览器|Node
        ---|---|---
        同步代码|√|√
        UI rendering|√|√
        I/O|√|√
        setTimeout|√|√
        setInterval|√|√
        requestAnimationFrame|√|x
        setImmediate|x|√
    1. 微任务
        #|浏览器|Node
        ---|---|---
        process.nextTick|x|√
        Promise|√|√
        MutationObserver|√|x
1. 任务的优先级
    1. 宏任务macrotask：主代码块 > setImmediate > MessageChannel > setTimeout / setInterval >...
    1. 微任务microtask：process.nextTick > Promise = MutationObserver
1. 结论
    1. 任务执行的顺序是建立与优先级之上的
    1. 如果队列已经有一个setTImeout的宏任务，后来又加入了主代码的宏任务，会让主代码的的任务插队。