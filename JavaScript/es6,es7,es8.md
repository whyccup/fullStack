# es6
1. let const
1. 解构，默认数据
1. UNICODE表示法
1. 模板字符串
1. 字符串新增方法
    1. fromCodePoint() 用于从 Unicode 码点返回对应字符，
    1. raw() 该方法返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，往往用于模板字符串的处理方法。
    1. codePointAt() 能够正确处理 4 个字节储存的字符，返回一个字符的码点。
    [看](https://es6.ruanyifeng.com/#docs/decorator)
# es7
1. Array.prototype.includes() 查找一个值在不在数组里.
1. 求幂运算符 **=

# es8
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


